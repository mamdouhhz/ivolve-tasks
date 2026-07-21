# Lab 15: Node.js Application Deployment with ClusterIP Service

## Overview
This lab deploys the Node.js app (from Lab 9's Docker Hub image) as a Kubernetes `Deployment`, wired up to the ConfigMap/Secret from Lab 12 and the persistent log volume from Lab 13, and exposes it internally via a `ClusterIP` Service.

## Prerequisites
- minikube running, 2 nodes, worker node tainted `node=worker:NoSchedule` (Lab 10)
- `ivolve` namespace exists, with `ivolve-pod-quota` limiting it to 2 pods (Lab 11)
- `mysql-config` ConfigMap and `mysql-secret` Secret applied (Lab 12)
- `app-logs-pvc` PVC bound (Lab 13)
- Custom app image pushed to Docker Hub (Lab 9)

## 1. Deployment

`nodejs-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
  namespace: ivolve
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      tolerations:
        - key: "node"
          operator: "Equal"
          value: "worker"
          effect: "NoSchedule"
      containers:
        - name: nodejs-app
          image: <your-dockerhub-username>/ivolve-app:latest
          ports:
            - containerPort: 3000
          envFrom:
            - configMapRef:
                name: mysql-config
            - secretRef:
                name: mysql-secret
          volumeMounts:
            - name: app-logs
              mountPath: /app/logs
      volumes:
        - name: app-logs
          persistentVolumeClaim:
            claimName: app-logs-pvc
```
```bash
kubectl apply -f nodejs-deployment.yaml
```

**Design notes:**
- `envFrom` with `configMapRef`/`secretRef` pulls every key from `mysql-config` and `mysql-secret` in as environment variables, rather than listing each one individually.
- The toleration lets these pods schedule onto the tainted worker node; without it the scheduler would avoid that node entirely.
- The image is pulled from Docker Hub — replace `<your-dockerhub-username>/ivolve-app:latest` with the actual image pushed in Lab 9.

### Why only 1 of 2 replicas ends up running
The `ivolve-pod-quota` ResourceQuota (Lab 11) hard-caps the namespace at **2 pods total**. `mysql-0` already occupies one slot, leaving room for exactly 1 `nodejs-app` pod. The 2nd replica stays unscheduled with a quota-exceeded event — this is expected behavior given the quota, not a misconfiguration:
```bash
kubectl describe replicaset -n ivolve -l app=nodejs-app | grep -A5 Events
```

### Why `app-logs-pvc` specifically
It's the PVC provisioned in Lab 13 with `ReadWriteMany` access — the mode that would let multiple replicas mount it concurrently if the quota ever allowed both to run.

## 2. ClusterIP Service

`nodejs-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodejs-service
  namespace: ivolve
spec:
  type: ClusterIP
  selector:
    app: nodejs-app
  ports:
    - port: 80
      targetPort: 3000
```
```bash
kubectl apply -f nodejs-service.yaml
```
The `selector` matches the Deployment's pod label (`app: nodejs-app`), so the Service automatically balances traffic across however many replicas are actually `Running`.

## 3. Verify

```bash
kubectl get deployment nodejs-app -n ivolve
kubectl get pods -n ivolve -l app=nodejs-app
kubectl get svc nodejs-service -n ivolve
```

Check the running pod is reading its config/secret and has the log volume mounted:
```bash
kubectl exec -it <nodejs-app-pod-name> -n ivolve -- env | grep DB_
kubectl exec -it <nodejs-app-pod-name> -n ivolve -- ls /app/logs
```

## Cleanup
```bash
kubectl delete -f nodejs-service.yaml
kubectl delete -f nodejs-deployment.yaml
```
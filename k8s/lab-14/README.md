# Lab 14: StatefulSet with Headless Service

## Overview
This lab deploys MySQL as a Kubernetes `StatefulSet` — the appropriate workload type for stateful, identity-sensitive apps like databases — fronted by a headless Service for stable, predictable pod DNS. It builds directly on the `ivolve` namespace, `mysql-secret`, and taint set up in earlier labs.

## Prerequisites
- minikube running, 2 nodes, worker node tainted `node=worker:NoSchedule` (Lab 10)
- `ivolve` namespace exists (Lab 11)
- `mysql-config` ConfigMap and `mysql-secret` Secret applied (Lab 12)

## 1. Persistent storage for MySQL data

`mysql-pv.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/mysql-data
```

`mysql-pvc.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: ivolve
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 1Gi
```

```bash
kubectl apply -f mysql-pv.yaml
kubectl apply -f mysql-pvc.yaml
```

## 2. Headless Service

`mysql-headless-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: ivolve
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
```
```bash
kubectl apply -f mysql-headless-service.yaml
```
`clusterIP: None` is what makes this headless — instead of load-balancing, cluster DNS resolves the Service name directly to pod IP(s), and each pod also gets its own stable DNS entry (`mysql-0.mysql.ivolve.svc.cluster.local`). This is the standard pairing for StatefulSets. It also matches `DB_HOST: mysql` from the `mysql-config` ConfigMap (Lab 12).

## 3. StatefulSet

`mysql-statefulset.yaml`:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: ivolve
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      tolerations:
        - key: "node"
          operator: "Equal"
          value: "worker"
          effect: "NoSchedule"
      containers:
        - name: mysql
          image: mysql:8.0
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pvc
```
```bash
kubectl apply -f mysql-statefulset.yaml
```

**Design notes:**
- The root password is sourced from `mysql-secret` (`MYSQL_ROOT_PASSWORD` key) via `secretKeyRef` — never hardcoded.
- The toleration lets this pod schedule onto the tainted worker node (`node=worker:NoSchedule` from Lab 10); without it, the scheduler would avoid that node entirely.
- `/var/lib/mysql` is backed by the `mysql-pvc` PVC, so data survives pod restarts/rescheduling.

## 4. Verify the StatefulSet and pod

```bash
kubectl get statefulset -n ivolve
kubectl get pods -n ivolve -l app=mysql
kubectl get svc mysql -n ivolve
kubectl get pvc mysql-pvc -n ivolve
```
Expect the pod named `mysql-0` (StatefulSet pods are named `<name>-<ordinal>`), `Bound` PVC, and `ClusterIP: None` on the Service.

## 5. Confirm the database is operational

Shell into the pod:
```bash
kubectl exec -it mysql-0 -n ivolve -- bash
```
Then connect with the MySQL client, using the root password already injected as an env var in the container:
```bash
mysql -uroot -p"$MYSQL_ROOT_PASSWORD"
```
Or connect in one line from outside the pod:
```bash
kubectl exec -it mysql-0 -n ivolve -- \
  mysql -uroot -p"$(kubectl get secret mysql-secret -n ivolve -o jsonpath='{.data.MYSQL_ROOT_PASSWORD}' | base64 -d)" \
  -e "SHOW DATABASES;"
```
A successful connection and database listing confirms MySQL is operational. (`ivolve` won't exist yet at this stage — it's created by the init container in Lab 16.)

## Troubleshooting
- `kubectl exec -it mysql -n ivolve -- bash` fails with `you must specify at least one command` / pod not found — two separate issues: (1) `exec` needs a command after `--`, and (2) the pod name is `mysql-0`, not `mysql` (that's the Service name).

## Cleanup
```bash
kubectl delete -f mysql-statefulset.yaml
kubectl delete -f mysql-headless-service.yaml
kubectl delete -f mysql-pvc.yaml
kubectl delete -f mysql-pv.yaml
```
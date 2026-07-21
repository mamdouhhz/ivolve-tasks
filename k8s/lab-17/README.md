# Lab 16: Kubernetes Init Container for Pre-Deployment Database Setup

## Overview
This lab modifies the existing `nodejs-app` Deployment (Lab 15) to add an `initContainers` step that provisions the `ivolve` database and its app-specific MySQL user *before* the main app container starts. Kubernetes guarantees init containers run to completion before any regular container in the pod begins — that ordering is exactly what makes this pattern reliable for pre-deployment setup.

## Prerequisites
- `mysql-0` StatefulSet running and reachable at Service `mysql` (Lab 14)
- `mysql-config` ConfigMap and `mysql-secret` Secret applied (Lab 12)
- `nodejs-app` Deployment + `nodejs-service` running (Lab 15)

## 1. Add the init container

Edit the existing `nodejs-deployment.yaml`, adding `initContainers` under `spec.template.spec`, above `containers`:

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
      initContainers:
        - name: init-db
          image: mysql:8.0
          env:
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: mysql-config
                  key: DB_HOST
            - name: DB_USER
              valueFrom:
                configMapKeyRef:
                  name: mysql-config
                  key: DB_USER
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: DB_PASSWORD
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
          command:
            - sh
            - -c
            - |
              mysql -h "$DB_HOST" -uroot -p"$MYSQL_ROOT_PASSWORD" -e "
                CREATE DATABASE IF NOT EXISTS ivolve;
                CREATE USER IF NOT EXISTS '$DB_USER'@'%' IDENTIFIED BY '$DB_PASSWORD';
                GRANT ALL PRIVILEGES ON ivolve.* TO '$DB_USER'@'%';
                FLUSH PRIVILEGES;
              "
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
- DB connection parameters (`DB_HOST`, `DB_USER`) come from the `mysql-config` ConfigMap; credentials (`DB_PASSWORD`, `MYSQL_ROOT_PASSWORD`) come from the `mysql-secret` Secret — same separation of concerns as the main app container.
- `CREATE DATABASE IF NOT EXISTS` / `CREATE USER IF NOT EXISTS` make the init container idempotent — safe to re-run on every pod restart without erroring on already-existing objects.
- `mysql-config`'s `DB_USER` was set to `app_user` (not `root`) so the created user is distinct from the MySQL root account — the lab explicitly calls for granting privileges to "the app user."

### Image architecture gotcha: use `mysql:8.0`, not `mysql:5.7`
The lab's `mysql:5.7` was only an example tag. On Apple Silicon (arm64), `mysql:5.7`'s image manifest has **no `linux/arm64/v8` build**, so the init container fails with:
```
Failed to pull image "mysql:5.7": no matching manifest for linux/arm64/v8 in the manifest list entries
```
`mysql:8.0` publishes proper arm64 images and also matches the server version already running in the Lab 14 StatefulSet, so client/server versions line up. Use `mysql:8.0` for the init container image.

## 2. Verify

Check the init container completed without errors:
```bash
kubectl logs <nodejs-app-pod-name> -n ivolve -c init-db
```
A clean run only shows MySQL's harmless CLI password warning — no SQL errors.

Confirm the database and user exist with the expected privileges:
```bash
kubectl exec -it mysql-0 -n ivolve -- \
  mysql -uroot -p"$(kubectl get secret mysql-secret -n ivolve -o jsonpath='{.data.MYSQL_ROOT_PASSWORD}' | base64 -d)" \
  -e "SHOW DATABASES; SHOW GRANTS FOR 'app_user'@'%';"
```
Expect:
- `ivolve` present in `SHOW DATABASES`
- `GRANT ALL PRIVILEGES ON \`ivolve\`.* TO \`app_user\`@\`%\`` in the grants output

## Troubleshooting

### Rollout gets stuck fighting over the pod quota
The namespace's `ivolve-pod-quota` (Lab 11) caps it at **2 pods total**, and `mysql-0` already occupies one slot — leaving room for exactly 1 `nodejs-app` pod. Kubernetes' default `RollingUpdate` strategy needs `desired + 1` pods momentarily to replace an old pod with a new one, which can never succeed under this quota. Symptoms:
- `kubectl get events` shows repeated `FailedCreate ... exceeded quota: ivolve-pod-quota, requested: pods=1, used: pods=2, limited: pods=2`
- The old pod (from before your edit) keeps running instead of being replaced

**Fix for right now:** manually delete the old pod to free the slot so the new ReplicaSet can schedule:
```bash
kubectl get pods -n ivolve -l app=nodejs-app
kubectl delete pod <old-pod-name> -n ivolve
```

**Permanent fix:** switch the Deployment to `Recreate` strategy, which tears down old pods before creating new ones instead of needing extra headroom:
```yaml
spec:
  strategy:
    type: Recreate
```
Add this under `spec` in `nodejs-deployment.yaml` and re-apply — future edits won't hit this race.

### Never `kubectl scale replicaset` by hand during a rollout
Manually scaling a ReplicaSet (as a workaround for the quota race) takes it out of sync with the Deployment controller — it can end up with a stale `desired` count that keeps recreating outdated pods (e.g. an old `mysql:5.7` init image) even after you've fixed and re-applied the Deployment. If this happens:
```bash
# find every RS's init image and desired replica count
kubectl get replicaset -n ivolve -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.template.spec.initContainers[0].image}{"\t"}{.spec.replicas}{"\n"}{end}'

# zero out every RS except the one matching the current correct image
kubectl scale replicaset <stale-rs-name> -n ivolve --replicas=0
```
Old ReplicaSets at `DESIRED: 0` are harmless revision history and don't need cleanup.

## Cleanup
```bash
kubectl delete -f nodejs-deployment.yaml
```
# Lab 12 & Lab 13: Configuration, Secrets, and Persistent Storage in Kubernetes

## Overview
This lab covers two foundational Kubernetes patterns used to prep an app for a real deployment:

- **Lab 12** — separating non-sensitive configuration (`ConfigMap`) from sensitive credentials (`Secret`) for a MySQL-backed app.
- **Lab 13** — provisioning persistent storage (`PersistentVolume` + `PersistentVolumeClaim`) so application logs survive pod restarts.

Both run on the same **minikube** cluster, in the `ivolve` namespace created in Lab 11.

## Prerequisites
- minikube running (`minikube start`)
- `kubectl` configured against the cluster
- `ivolve` namespace already created (Lab 11)

---

## Lab 12: ConfigMaps and Secrets

### 1. ConfigMap — non-sensitive MySQL config

`configmap.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
  namespace: ivolve
data:
  DB_HOST: mysql
  DB_USER: root
```

**Declarative:**
```bash
kubectl apply -f configmap.yaml
```

**Imperative:**
```bash
kubectl create configmap mysql-config \
  --from-literal=DB_HOST=mysql \
  --from-literal=DB_USER=root \
  -n ivolve
```

> `DB_HOST` is set to `mysql` — the name of the Service that will front the MySQL StatefulSet in a later lab. Kubernetes DNS resolves a Service name to the right pod(s) within the same namespace automatically, so this only needs to match whatever name that Service is given.

**Verify:**
```bash
kubectl get configmap mysql-config -n ivolve -o yaml
```

### 2. Secret — sensitive MySQL credentials

Base64-encode the values manually:
```bash
echo -n 'dbpass123' | base64
echo -n 'rootpass123' | base64
```

`secret.yaml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: ivolve
type: Opaque
data:
  DB_PASSWORD: <base64-encoded-db-password>
  MYSQL_ROOT_PASSWORD: <base64-encoded-root-password>
```

**Declarative:**
```bash
kubectl apply -f secret.yaml
```

**Imperative** (kubectl base64-encodes the values for you):
```bash
kubectl create secret generic mysql-secret \
  --from-literal=DB_PASSWORD='YourDbPassword123' \
  --from-literal=MYSQL_ROOT_PASSWORD='YourRootPassword123' \
  -n ivolve
```

**Verify (and confirm decoding works):**
```bash
kubectl get secret mysql-secret -n ivolve -o yaml
kubectl get secret mysql-secret -n ivolve -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
```

---

## Lab 13: Persistent Volume + Claim for App Logging

### 1. PersistentVolume

`pv.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-logs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/app-logs
```
```bash
kubectl apply -f pv.yaml
```
PVs are cluster-scoped and created declaratively — there's no direct imperative `kubectl create pv` command.

> **Note:** `hostPath` ties the volume to whichever node's filesystem it lives on. Pairing it with `ReadWriteMany` only behaves correctly if all pods land on that same node (true by default on minikube's single-VM setup). In a real multi-node cluster you'd use NFS or a cloud RWX-capable storage class instead.

### 2. PersistentVolumeClaim

`pvc.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-logs-pvc
  namespace: ivolve
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  resources:
    requests:
      storage: 1Gi
```
```bash
kubectl apply -f pvc.yaml
```

### 3. Verify binding
```bash
kubectl get pv app-logs-pv
kubectl get pvc app-logs-pvc -n ivolve
```
Expected: both show `STATUS: Bound`, and the PVC's capacity/access mode match the PV (`1Gi`, `RWX`).

```bash
kubectl describe pv app-logs-pv
kubectl describe pvc app-logs-pvc -n ivolve
```

To inspect the actual log directory on minikube (it lives inside the minikube VM/container, not the host Mac filesystem):
```bash
minikube ssh
ls /mnt/app-logs
```

---

## Troubleshooting Notes
- If `minikube start` reports errors enabling `storage-provisioner` or `default-storageclass` (API server not reachable yet during a fresh container recreation), first confirm the cluster itself is healthy:
  ```bash
  kubectl get nodes
  kubectl get pods -A
  ```
  If healthy, just re-enable the addons manually:
  ```bash
  minikube addons enable storage-provisioner
  minikube addons enable default-storageclass
  ```
  If the cluster is genuinely unreachable, a clean reset resolves it:
  ```bash
  minikube delete
  minikube start
  ```

## Cleanup
```bash
kubectl delete -f pvc.yaml -n ivolve
kubectl delete -f pv.yaml
kubectl delete -f secret.yaml -n ivolve
kubectl delete -f configmap.yaml -n ivolve
```
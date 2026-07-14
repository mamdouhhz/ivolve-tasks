# Lab 10 & Lab 11: Node Taints, Namespaces, and Resource Quotas in Kubernetes

## Overview
This lab covers two related Kubernetes scheduling/governance concepts on a local multi-node cluster:

- **Lab 10** — isolating a node from the default scheduler using a taint.
- **Lab 11** — creating a namespace and enforcing a hard limit on the number of pods that can run inside it via a `ResourceQuota`.

Both labs run on the same 2-node **minikube** cluster.

## Prerequisites
- [minikube](https://minikube.sigs.k8s.io/) installed
- `kubectl` installed
- Docker (or another supported driver) running, since minikube nodes run as containers/VMs

## Cluster Setup
```bash
minikube start --nodes 2 -p lab-10
kubectl get nodes
```
Expected output:
```
NAME              STATUS   ROLES           AGE   VERSION
lab-10        Ready    control-plane   1m    v1.x.x
lab-10-m02     Ready    <none>          1m    v1.x.x
```

---

## Lab 10: Node Isolation Using Taints

**Goal:** taint one node with key `node`, value `worker`, effect `NoSchedule`, so new pods without a matching toleration won't be scheduled there.

### Apply the taint

**Imperative:**
```bash
kubectl taint nodes lab-10-m02 node=worker:NoSchedule
```

### Verify
```bash
kubectl describe nodes | grep -A1 "Taints:"
```
Expected:
```
Taints:             node=worker:NoSchedule    # lab-10-m02
Taints:             <none>                    # lab-10 (control-plane)
```

---

## Lab 11: Namespace Management and Resource Quota Enforcement

**Goal:** create a namespace called `ivolve` and cap it at 2 pods.

### 1. Create the namespace

**Imperative:**
```bash
kubectl create namespace ivolve
```

**Declarative** (`namespace.yaml`):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ivolve
```
```bash
kubectl apply -f namespace.yaml
```

### 2. Apply the resource quota

**Imperative:**
```bash
kubectl create quota ivolve-pod-quota --hard=pods=2 -n ivolve
```

**Declarative** (`resource-quota.yaml`):
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ivolve-pod-quota
  namespace: ivolve
spec:
  hard:
    pods: "2"
```
```bash
kubectl apply -f resource-quota.yaml
```

### 3. Verify
```bash
kubectl get resourcequota -n ivolve
kubectl describe resourcequota ivolve-pod-quota -n ivolve
```

---

## Cleanup
```bash
minikube delete -p lab-10
```
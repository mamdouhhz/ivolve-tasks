# Lab 20: Securing Kubernetes with RBAC and Service Accounts

## Overview
This lab creates a dedicated `jenkins-sa` ServiceAccount scoped to read-only Pod access in the `ivolve` namespace — the standard pattern for giving a CI tool (Jenkins) least-privilege access to a cluster instead of using a broad admin identity.

## Prerequisites
- `ivolve` namespace exists (Lab 11)

## 1. Create the `jenkins-sa` ServiceAccount

**Imperative:**
```bash
kubectl create serviceaccount jenkins-sa -n ivolve
```

**Declarative** (`jenkins-sa.yaml`):
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-sa
  namespace: ivolve
```
```bash
kubectl apply -f jenkins-sa.yaml
```

## 2. Create a token for the ServiceAccount

Since Kubernetes 1.24+, ServiceAccounts no longer auto-generate long-lived Secret tokens — request one explicitly:
```bash
kubectl create token jenkins-sa -n ivolve
```
Prints a short-lived JWT (default TTL 1 hour).

For a longer-lived token (e.g. for a CI tool that needs to keep using the same credential), create it as a Secret instead:
```yaml
# jenkins-sa-token.yaml
apiVersion: v1
kind: Secret
metadata:
  name: jenkins-sa-token
  namespace: ivolve
  annotations:
    kubernetes.io/service-account.name: jenkins-sa
type: kubernetes.io/service-account-token
```
```bash
kubectl apply -f jenkins-sa-token.yaml
kubectl get secret jenkins-sa-token -n ivolve -o jsonpath='{.data.token}' | base64 -d
```

## 3. Define the `pod-reader` Role

`pod-reader-role.yaml`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: ivolve
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```
```bash
kubectl apply -f pod-reader-role.yaml
```
`apiGroups: [""]` is the core API group, which `Pod` belongs to. `verbs` is deliberately limited to `get`/`list` — no `create`, `delete`, `watch`, or `update`.

## 4. Bind the Role to the ServiceAccount

`pod-reader-binding.yaml`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins-sa-pod-reader-binding
  namespace: ivolve
subjects:
  - kind: ServiceAccount
    name: jenkins-sa
    namespace: ivolve
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
```bash
kubectl apply -f pod-reader-binding.yaml
```

## 5. Validate — read access is granted, everything else is denied

```bash
kubectl auth can-i list pods -n ivolve --as=system:serviceaccount:ivolve:jenkins-sa
kubectl auth can-i get pods -n ivolve --as=system:serviceaccount:ivolve:jenkins-sa
```
Both return `yes`.

Confirm nothing beyond read access is granted:
```bash
kubectl auth can-i create pods -n ivolve --as=system:serviceaccount:ivolve:jenkins-sa
kubectl auth can-i delete pods -n ivolve --as=system:serviceaccount:ivolve:jenkins-sa
kubectl auth can-i watch pods -n ivolve --as=system:serviceaccount:ivolve:jenkins-sa
kubectl auth can-i list pods -n default --as=system:serviceaccount:ivolve:jenkins-sa
```
All four return `no` — the first three because those verbs were never granted, and the last because a `Role`/`RoleBinding` (unlike `ClusterRole`/`ClusterRoleBinding`) is scoped strictly to its own namespace.

Actual verified output:
```
$ kubectl auth can-i list pods -n ivolve --as=system:serviceaccount:ivolve:jenkins-sa
yes
$ kubectl auth can-i get pods -n ivolve --as=system:serviceaccount:ivolve:jenkins-sa
yes
$ kubectl auth can-i create pods -n ivolve --as=system:serviceaccount:ivolve:jenkins-sa
no
$ kubectl auth can-i delete pods -n ivolve --as=system:serviceaccount:ivolve:jenkins-sa
no
$ kubectl auth can-i watch pods -n ivolve --as=system:serviceaccount:ivolve:jenkins-sa
no
$ kubectl auth can-i list pods -n default --as=system:serviceaccount:ivolve:jenkins-sa
no
```
This confirms the Role grants exactly `get`+`list` on Pods, scoped only to the `ivolve` namespace — `kubectl auth can-i` queries Kubernetes' own authorization API directly, so this is a real evaluation of the binding, not just a dry-run guess.

## Cleanup
```bash
kubectl delete -f pod-reader-binding.yaml
kubectl delete -f pod-reader-role.yaml
kubectl delete -f jenkins-sa-token.yaml   # if created
kubectl delete -f jenkins-sa.yaml
```
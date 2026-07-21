# Lab 18: Control Pod-to-Pod Traffic via Network Policy

## Overview
This lab defines a `NetworkPolicy` that restricts ingress to the MySQL pod (`mysql-0`, Lab 14) so that only pods labeled `app=nodejs-app` can reach it, and only on port 3306.

## Prerequisites
- `mysql-0` StatefulSet running with label `app: mysql` (Lab 14)
- `nodejs-app` Deployment running with label `app: nodejs-app` (Lab 15)

## 1. NetworkPolicy manifest

`allow-app-to-mysql.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-to-mysql
  namespace: ivolve
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: nodejs-app
      ports:
        - protocol: TCP
          port: 3306
```
```bash
kubectl apply -f allow-app-to-mysql.yaml
```

**How this satisfies each requirement:**
- `podSelector: {app: mysql}` — the policy applies to pods matching this label (`mysql-0`).
- `policyTypes: [Ingress]` — only incoming traffic is restricted; egress from the MySQL pod is left unaffected.
- `ingress.from.podSelector: {app: nodejs-app}` — only pods with that label can initiate connections to MySQL; everything else in the namespace is denied.
- `ports: [{protocol: TCP, port: 3306}]` — even allowed pods are limited to MySQL's default port.

## 2. Verify

```bash
kubectl get networkpolicy -n ivolve
kubectl describe networkpolicy allow-app-to-mysql -n ivolve
```
Expected `describe` output:
```
Spec:
  PodSelector:     app=mysql
  Allowing ingress traffic:
    To Port: 3306/TCP
    From:
      PodSelector: app=nodejs-app
  Not affecting egress traffic
  Policy Types: Ingress
```
This confirms the object matches every part of the spec: pod selector, ingress-only policy type, source pod selector, and port restriction.

## Note on enforcement

`NetworkPolicy` objects are only enforced if the cluster's CNI plugin supports them (e.g. Calico, Cilium, Weave). **Minikube's default networking setup does not include a policy-enforcing CNI** — checked via:
```bash
kubectl get pods -n kube-system
```
which shows only `coredns`, `kube-proxy`, `etcd`, `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, `metrics-server`, and `storage-provisioner` — no `calico`/`cilium`/`weave` pod present. `kube-proxy` alone handles Service routing, not NetworkPolicy rules.

This means the policy above is **correctly defined and applied per the lab's specification**, but traffic to `mysql:3306` is not actually being blocked at the network layer on this cluster — any pod could still reach it regardless of label, since nothing is enforcing the rule.

To get live enforcement, the cluster would need to be recreated with a policy-capable CNI from the start:
```bash
minikube start --nodes 2 --cni=calico -p <profile-name>
```
This requires a fresh cluster (CNI can't be swapped on a running one) and re-applying every prior lab's resources, so it was intentionally left out of scope here — this lab's requirement is a correctly specified `NetworkPolicy` resource, not a live traffic-blocking demo.

## Cleanup
```bash
kubectl delete -f allow-app-to-mysql.yaml
```
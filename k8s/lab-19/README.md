# Lab 19: Node-Wide Pod Management with DaemonSet

## Overview
This lab deploys Prometheus `node-exporter` as a `DaemonSet` so every node in the cluster exposes host-level metrics on port 9100, regardless of any taints. The bonus section builds a full monitoring stack on top (Prometheus + Grafana + Alertmanager) and wires alerts through to Gmail.

## Prerequisites
- minikube running

---

## Core Lab

### 1. Create the `monitoring` namespace

**Imperative:**
```bash
kubectl create namespace monitoring
```

**Declarative** (`namespace.yaml`):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```
```bash
kubectl apply -f namespace.yaml
```

### 2. DaemonSet for node-exporter

`node-exporter-daemonset.yaml`:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      tolerations:
        - operator: "Exists"
      hostNetwork: true
      hostPID: true
      containers:
        - name: node-exporter
          image: prom/node-exporter:latest
          ports:
            - containerPort: 9100
              protocol: TCP
          args:
            - --path.rootfs=/host
          volumeMounts:
            - name: root
              mountPath: /host
              readOnly: true
      volumes:
        - name: root
          hostPath:
            path: /
```
```bash
kubectl apply -f node-exporter-daemonset.yaml
```

**Design notes:**
- `tolerations: [{operator: "Exists"}]` — wildcard toleration matching **any** taint with any key/value/effect, so the DaemonSet runs on every node regardless of what's tainted.
- `hostNetwork: true` — binds directly to the node's own network namespace so `:9100/metrics` is reachable at the node's IP, not a pod IP.
- `hostPID: true` + `/host` rootfs mount — gives node-exporter visibility into host-level processes and filesystem, not just its own container.

### 3. Validate a pod runs on every node
```bash
kubectl get daemonset node-exporter -n monitoring
kubectl get pods -n monitoring -o wide
```
Expect `DESIRED`/`CURRENT`/`READY` to equal the node count, with one pod per node.

### 4. Confirm metrics exposure
```bash
kubectl get nodes -o wide
minikube ssh
curl localhost:9100/metrics
```
A large plaintext dump of Prometheus-format metrics (`node_cpu_seconds_total`, `node_os_info`, `node_uname_info`, etc.) confirms it's working.

### Troubleshooting
- `minikube ssh -p <profile>` fails with "Profile not found" — confirm the actual active profile first with `minikube profile list`; `minikube ssh` with no `-p` uses the default `minikube` profile.

---

## Bonus: Prometheus + Grafana + Alertmanager → Gmail

### 1. Resource check
`kube-prometheus-stack` bundles Prometheus, Grafana, Alertmanager, kube-state-metrics, and its own node-exporter — noticeably heavier than any prior lab. If pods get stuck `Pending`, give minikube more resources:
```bash
minikube stop
minikube start --cpus=4 --memory=6144
```

### 2. Install Helm and add the chart repo
```bash
brew install helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 3. Avoid a duplicate node-exporter
The chart deploys its own node-exporter, which collides with the one from the core lab (same port, same node). Remove the manual one before installing (the manifest still exists in this repo as the graded deliverable):
```bash
kubectl delete daemonset node-exporter -n monitoring
```

### 4. Create credentials as Secrets (not plaintext in values)
```bash
kubectl create secret generic grafana-admin-secret \
  --from-literal=admin-user=admin \
  --from-literal=admin-password='<strong-password>' \
  -n monitoring
```

Gmail SMTP requires an **App Password**, not the normal account password:
1. Enable 2-Step Verification: https://myaccount.google.com/security
2. Generate an app password: https://myaccount.google.com/apppasswords → "Mail"
3. Copy the 16 characters **with spaces removed**

```bash
kubectl create secret generic alertmanager-gmail-secret \
  --from-literal=password='<16-char-app-password-no-spaces>' \
  -n monitoring
```

To recreate either secret later (e.g. after fixing a typo):
```bash
kubectl delete secret alertmanager-gmail-secret grafana-admin-secret -n monitoring
# then re-run the create commands above
```
Note: recreating the Secret object doesn't refresh an already-running pod's mounted copy — restart the relevant workload afterward:
```bash
kubectl rollout restart statefulset alertmanager-kube-prom-stack-kube-prome-alertmanager -n monitoring
kubectl rollout restart deployment kube-prom-stack-grafana -n monitoring
```

### 5. `values.yaml`
```yaml
grafana:
  admin:
    existingSecret: grafana-admin-secret
    userKey: admin-user
    passwordKey: admin-password
  defaultDashboardsEnabled: true
  service:
    type: ClusterIP

prometheus-node-exporter:
  enabled: false   # chart's own node-exporter, since the manual one was removed

alertmanager:
  config:
    global:
      smtp_smarthost: 'smtp.gmail.com:587'
      smtp_from: '<your-email>@gmail.com'
      smtp_auth_username: '<your-email>@gmail.com'
      smtp_auth_password_file: /etc/alertmanager/secrets/alertmanager-gmail-secret/password
      smtp_require_tls: true
    route:
      receiver: gmail-alerts
      group_by: ['alertname']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 3h
      routes: []   # explicitly clear inherited default sub-routes (see Troubleshooting)
    receivers:
      - name: gmail-alerts
        email_configs:
          - to: '<your-email>@gmail.com'
  alertmanagerSpec:
    secrets:
      - alertmanager-gmail-secret
```

### 6. Install
```bash
helm install kube-prom-stack prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f values.yaml
```

### 7. Verify pods
```bash
kubectl get pods -n monitoring
```
Expect pods for `prometheus`, `alertmanager`, `grafana`, `kube-state-metrics`, plus the operator.

Confirm Alertmanager specifically reconciled (it's a CRD reconciled by the Prometheus Operator, so a healthy pod alone doesn't guarantee success):
```bash
kubectl get alertmanager -n monitoring
```
Expect `READY: 1`, `RECONCILED: True`, `AVAILABLE: True`.

### 8. Access Grafana professionally, via Ingress

**Enable the Ingress controller:**
```bash
minikube addons enable ingress
kubectl get pods -n ingress-nginx
```

**Ingress manifest** (`grafana-ingress.yaml`):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: grafana.ivolve.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kube-prom-stack-grafana
                port:
                  number: 80
```
```bash
kubectl apply -f grafana-ingress.yaml
```

**On macOS with the Docker driver, `minikube tunnel` is required** to bridge Ingress traffic out to the host:
```bash
minikube tunnel
```
Leave this running in its own terminal (needs `sudo` once, for binding privileged ports 80/443).

**Point the hostname at `127.0.0.1`, not `minikube ip`** — with the Docker driver + tunnel on macOS, tunneled traffic surfaces on localhost, not the Docker-internal node IP:
```bash
sudo sed -i '' '/grafana.ivolve.local/d' /etc/hosts
echo "127.0.0.1 grafana.ivolve.local" | sudo tee -a /etc/hosts
```

Access at `http://grafana.ivolve.local`, log in with the `grafana-admin-secret` credentials.

### 9. Confirm Prometheus is scraping node-exporter
In Grafana: **Dashboards → Node Exporter Full** (auto-provisioned by the chart). Or check Prometheus directly:
```bash
kubectl port-forward -n monitoring svc/kube-prom-stack-kube-prome-prometheus 9090:9090
```
`http://localhost:9090/targets` → `node-exporter` should show `State: UP`.

### 10. Test Gmail alert delivery
```bash
kubectl port-forward -n monitoring svc/kube-prom-stack-kube-prome-alertmanager 9093:9093
```
```bash
curl -H "Content-Type: application/json" -d '[{
  "labels": {"alertname": "TestAlert", "severity": "critical"},
  "annotations": {"summary": "Test alert to confirm Gmail delivery"}
}]' http://localhost:9093/api/v2/alerts
```
Check the configured Gmail inbox — confirmed working end-to-end.

---

## Troubleshooting (Bonus)

### Alertmanager CRD stuck at `RECONCILED: False`
```
provision alertmanager configuration: failed to initialize from secret: undefined receiver "null" used in route
```
The chart's default `values.yaml` ships a built-in `route.routes` sub-array that references a `'null'` receiver (used internally to silence the default `Watchdog` alert). Overriding only `route.receiver` and `receivers` — without also clearing `route.routes` — leaves that dangling reference in place after Helm merges configs, since the default receivers array gets replaced but the default sub-routes don't. Fix: explicitly set `route.routes: []` in `values.yaml` (see step 5), then:
```bash
helm upgrade kube-prom-stack prometheus-community/kube-prometheus-stack -n monitoring -f values.yaml
```

### Grafana Ingress hangs / browser keeps loading
On macOS with the Docker driver, `minikube ip` returns a Docker-internal IP that the host's network stack often can't route to directly, even with `minikube tunnel` running. Diagnose layer by layer:
```bash
kubectl get ingress -n monitoring          # confirm ADDRESS is populated
curl -H "Host: grafana.ivolve.local" http://127.0.0.1   # test with tunnel active
```
Fix: point `/etc/hosts` at `127.0.0.1`, not the minikube node IP (step 8).

### Gmail SMTP `535 Username and Password not accepted`
Gmail rejects the normal account password for SMTP — an **App Password** is required, generated only after enabling 2-Step Verification. Also strip the spaces Google displays it with when copying into the Secret.

### Gmail `454 Too many login attempts, please try again later`
A side effect of repeated failed-auth retries (from unrelated default alerts like `Watchdog`/`TargetDown` also firing and retrying against the same bad credentials) tripping Gmail's rate limiter. Not a separate bug — fix the credentials, then wait ~15–30 minutes before retesting even after the fix is applied, since Gmail's rate limit needs to clear.

## Cleanup
```bash
helm uninstall kube-prom-stack -n monitoring
kubectl delete -f grafana-ingress.yaml
kubectl delete -f node-exporter-daemonset.yaml
kubectl delete namespace monitoring
minikube addons disable ingress
```
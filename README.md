# ivolve-tasks
A collection of hands-on DevOps and Java build/containerization labs, completed as part of the NTI DevOps program (Hire Ready scholarship) internship at ivolve company.
Each folder is a standalone lab with its own README, prerequisites, and run instructions.

## Repo Structure
- `build_tools/` — Java build tooling labs (lab-1, lab-2)
- `containerization/` — Docker labs (lab-3&4 to lab-9)
- `k8s/` — Kubernetes labs (lab-10&11 to lab-20)

## Labs

### `build_tools/`
| Lab | Description | Tech |
|---|---|---|
| [`lab-1`](./build_tools/lab-1) | Building and packaging a Java calculator app with Gradle | Java 17, Gradle, JUnit 5 |
| [`lab-2`](./build_tools/lab-2) | Building and packaging a Java calculator app with Maven | Java 17, Maven, JUnit 5 |

### `containerization/`
| Lab | Description | Tech |
|---|---|---|
| [`lab-3&4`](./containerization/lab-3&4) | Dockerizing a Spring Boot REST app with a secure, non-root runtime image | Java 17, Spring Boot, Docker |
| [`lab-5`](./containerization/lab-5) | Multi-stage Docker build for a Spring Boot app | Java 17, Spring Boot, Docker |
| [`lab-6`](./containerization/lab-6) | Managing Docker environment variables across build and runtime | Python, Flask, Docker |
| [`lab-7`](./containerization/lab-7) | Docker volumes and bind mounts with Nginx | Docker, Nginx |
| [`lab-8`](./containerization/lab-8) | Custom Docker network for microservice communication | Python, Flask, Docker |
| [`lab-9`](./containerization/lab-9) | Containerized Node.js + MySQL stack with Docker Compose, health/readiness checks, and Docker Hub image push | Node.js, Express, MySQL, Docker Compose |

### `k8s/`
| Lab | Description | Tech |
|---|---|---|
| [`lab-10&11`](./k8s/lab-10&11) | Node isolation via taints, and namespace-scoped resource quota enforcement on a multi-node cluster | Kubernetes, minikube |
| [`lab-12&13`](./k8s/lab-12&13) | ConfigMap/Secret-based configuration management and persistent storage provisioning (PV/PVC) | Kubernetes, minikube |
| [`lab-14`](./k8s/lab-14) | MySQL StatefulSet with a headless Service, PVC-backed storage, and secret-based root password | Kubernetes, minikube, MySQL |
| [`lab-15`](./k8s/lab-15) | Node.js Deployment (2 replicas) with ConfigMap/Secret env vars, node toleration, PVC-backed logging, and a ClusterIP Service | Kubernetes, minikube, Node.js |
| [`lab-16`](./k8s/lab-16) | Init container for pre-deployment MySQL database and app-user provisioning | Kubernetes, minikube, MySQL |
| [`lab-17`](./k8s/lab-17) | CPU/memory resource requests and limits, verified via `describe pod` and `kubectl top pod` | Kubernetes, minikube, metrics-server |
| [`lab-18`](./k8s/lab-18) | NetworkPolicy restricting MySQL ingress to app pods only, on port 3306 | Kubernetes, minikube |
| [`lab-19`](./k8s/lab-19) | Node-wide DaemonSet for Prometheus node-exporter, plus a bonus Prometheus/Grafana/Alertmanager stack with Gmail alerting | Kubernetes, minikube, Prometheus, Grafana, Alertmanager |
| [`lab-20`](./k8s/lab-20)   | RBAC: ServiceAccount, Role, and RoleBinding granting read-only Pod access | Kubernetes, minikube |


## Prerequisites
Varies per lab — see each lab's own README for exact versions and setup. In general:
- Java JDK 17
- Gradle and/or Maven
- Docker
- Docker Compose
- Python 3 (for containerization/lab-6, lab-8)
- Node.js 18 (for containerization/lab-9)
- minikube and kubectl (for k8s/lab-10&11 onward)
- Helm (for k8s/lab-19 bonus)

## Learning Outcomes
Across these labs:
- Configuring and running Java builds with both Gradle and Maven
- Writing and executing automated unit tests (JUnit 5)
- Generating runnable jar artifacts
- Writing a production-conscious Dockerfile (non-root user, slim runtime base)
- Building, running, and verifying a containerized service
- Writing multi-stage Dockerfiles to reduce image size
- Managing environment-specific configuration via CLI flags, env files, and Dockerfile defaults
- Persisting data with Docker volumes vs. live host content with bind mounts
- Creating custom Docker networks and verifying container-to-container communication
- Orchestrating multi-service apps with Docker Compose, including service dependencies, environment variables, and named volumes
- Verifying application health via `/health` and `/ready` endpoints and inspecting log output
- Publishing built images to Docker Hub
- Running and inspecting a multi-node Kubernetes cluster
- Controlling pod scheduling with node taints
- Managing namespaces and enforcing resource quotas, both imperatively and declaratively
- Separating non-sensitive configuration from sensitive credentials with ConfigMaps and Secrets
- Provisioning and binding persistent storage with PersistentVolumes and PersistentVolumeClaims
- Deploying stateful workloads with StatefulSets and headless Services for stable network identity
- Deploying application workloads with Deployments and exposing them internally via ClusterIP Services
- Using init containers to sequence pre-deployment setup tasks
- Setting and verifying pod resource requests/limits, and monitoring live usage with metrics-server
- Restricting pod-to-pod network traffic with NetworkPolicies
- Running node-level monitoring with DaemonSets, and building a full metrics/alerting stack with Prometheus, Grafana, and Alertmanager
- Securing cluster access with RBAC: ServiceAccounts, Roles, and RoleBindings scoped to least privilege

## Author
Mamdouh ([@mamdouhhz](https://github.com/mamdouhhz))
# ivolve-tasks
A collection of hands-on DevOps and Java build/containerization labs, completed as part of the NTI DevOps program (Hire Ready scholarship) internship at ivolve company.
Each folder is a standalone lab with its own README, prerequisites, and run instructions.

## Repo Structure
- `build_tools/` — Java build tooling labs (lab-1, lab-2)
- `containerization/` — Docker labs (lab-3&4 through lab-9)
- `k8s/` — Kubernetes labs (lab-10&11 onward)

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

## Overview
These labs progress from core Java build tooling to containerization and orchestration:

1. **Gradle & Maven labs** — understanding the Java build lifecycle: compiling, testing, dependency management, and generating runnable jar artifacts using the two most common JVM build tools.
2. **Docker lab** — taking a Spring Boot application and packaging it into a minimal, secure container image, including non-root user configuration and image size optimization.
3. **Multi-stage build lab** — separating build and runtime environments in a single Dockerfile to produce a smaller, more secure final image.
4. **Environment variables lab** — managing configuration across environments (development, staging, production) using Docker's different mechanisms for passing environment variables.
5. **Volumes and bind mounts lab** — persisting container data with named volumes and serving live-editable content from the host with bind mounts.
6. **Custom network lab** — connecting containers over a user-defined Docker network and verifying DNS-based service discovery and network isolation.
7. **Docker Compose lab** — orchestrating a multi-container Node.js + MySQL stack with a single `docker-compose.yml`, verifying application health/readiness endpoints and log output, and publishing the resulting image to Docker Hub.
8. **Kubernetes scheduling & governance lab** — running a multi-node cluster, isolating a node from the scheduler with taints, and enforcing per-namespace resource limits with a `ResourceQuota`.
9. **Kubernetes configuration & storage lab** — separating non-sensitive config (`ConfigMap`) from sensitive credentials (`Secret`), and provisioning persistent storage (`PersistentVolume`/`PersistentVolumeClaim`) for application logging.
10. **Kubernetes stateful workload lab** — running MySQL as a `StatefulSet` with a headless Service for stable pod DNS, PVC-backed data storage, and root credentials sourced from a `Secret`.
11. **Kubernetes app deployment lab** — deploying the Node.js app as a `Deployment` wired to the ConfigMap/Secret and PVC, exposed via a `ClusterIP` Service.
12. **Kubernetes init container lab** — using an init container to provision the app database and user before the main app container starts.
13. **Kubernetes resource management lab** — setting CPU/memory requests and limits on the app container, and monitoring live usage with `metrics-server`.

## Prerequisites
Varies per lab — see each lab's own README for exact versions and setup. In general:
- Java JDK 17
- Gradle and/or Maven
- Docker
- Docker Compose
- Python 3 (for containerization/lab-6, lab-8)
- Node.js 18 (for containerization/lab-9)
- minikube and kubectl (for k8s/lab-10&11 onward)

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

## Author
Mamdouh ([@mamdouhhz](https://github.com/mamdouhhz))
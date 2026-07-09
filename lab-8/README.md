# Lab 8 – Custom Docker Network for Microservices

## Overview

A frontend and backend Flask service communicating over a custom user-defined Docker network. Demonstrates how Docker's built-in DNS resolves container names within a shared network, and how containers on different networks are isolated from each other.

## Application

- **backend** — returns `"Hello from Backend!"` on port 5000
- **frontend** — calls `http://backend:5000` and returns the backend's response

## Dockerfiles

**backend/Dockerfile:**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
RUN pip install flask
EXPOSE 5000
CMD ["python", "app.py"]
```

**frontend/Dockerfile:**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

## Build

```bash
docker build -t backend-app ./backend
docker build -t frontend-app ./frontend
```

## Create the Custom Network

```bash
docker network create --subnet=192.168.10.0/24 ivolve-network
```

## Run

```bash
# Backend on the custom network
docker run -d --name backend --network ivolve-network backend-app

# Frontend1 on the same custom network
docker run -d --name frontend1 --network ivolve-network -p 5001:5000 frontend-app

# Frontend2 on the default bridge network
docker run -d --name frontend2 -p 5002:5000 frontend-app
```

## Verify

```bash
curl localhost:5001
# Frontend received: Hello from Backend!

curl localhost:5002
# Could not connect to backend.
```

`frontend1` can resolve and reach `backend` by container name because they share `ivolve-network`. `frontend2` is on the default bridge network, has no DNS entry for `backend`, and cannot connect.

## Clean Up

```bash
docker stop backend frontend1 frontend2
docker rm backend frontend1 frontend2
docker network rm ivolve-network
```

## Learning Outcomes

- Creating a custom Docker network with a defined subnet
- Docker's built-in DNS resolution between containers on the same user-defined network
- Network isolation between containers on different networks
- Debugging inter-container connectivity issues
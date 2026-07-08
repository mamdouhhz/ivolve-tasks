# Lab 6 – Managing Docker Environment Variables Across Build and Runtime

## Overview

A simple Flask app that reads two environment variables (`APP_MODE` and `APP_REGION`) and returns them in the response. This lab demonstrates three different ways to set environment variables in Docker: at runtime via the CLI, via an env file, and baked into the image at build time.

## Application

```python
@app.route('/')
def show_env():
    mode = os.getenv("APP_MODE", "default")
    region = os.getenv("APP_REGION", "unknown")
    return f"App mode: {mode}, Region: {region}"
```

## Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
RUN pip install flask
ENV APP_MODE=production
ENV APP_REGION=canada-west
EXPOSE 8080
CMD ["python", "app.py"]
```

The `ENV` values here act as defaults (production/canada-west) that are used when no override is provided at runtime.

## Build

```bash
docker build -t docker3-app .
```

## Run – Three Ways to Set Environment Variables

### 1. Development / us-east — passed directly in the run command

```bash
docker run -d --name dev-container -p 8081:8080 \
  -e APP_MODE=development -e APP_REGION=us-east \
  docker3-app
```

```bash
curl localhost:8081/
# App mode: development, Region: us-east
```

### 2. Staging / us-west — passed via an env file

```bash
cat > staging.env << 'EOF'
APP_MODE=staging
APP_REGION=us-west
EOF

docker run -d --name staging-container -p 8082:8080 \
  --env-file staging.env \
  docker3-app
```

```bash
curl localhost:8082/
# App mode: staging, Region: us-west
```

### 3. Production / canada-west — set in the Dockerfile, no override

```bash
docker run -d --name prod-container -p 8083:8080 docker3-app
```

```bash
curl localhost:8083/
# App mode: production, Region: canada-west
```

## Clean Up

```bash
docker stop dev-container staging-container prod-container
docker rm dev-container staging-container prod-container
docker rmi docker3-app
```

## Learning Outcomes

- Setting environment variables at container runtime with `-e`
- Passing multiple environment variables via an `--env-file`
- Setting default environment variables in the Dockerfile with `ENV`
- Understanding override precedence: CLI/env-file values override Dockerfile `ENV` defaults
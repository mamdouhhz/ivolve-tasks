# Lab-7  Docker Volume and Bind Mount with Nginx

## Overview

Runs an Nginx container using two different storage mechanisms at once: a named **volume** for persisting logs, and a **bind mount** for serving live-editable HTML content from the host.

## Setup

**Create the named volume:**
```bash
docker volume create nginx_logs
docker volume inspect nginx_logs
```

**Create the bind-mount source and content:**
```bash
mkdir -p html
echo "Hello from Bind Mount" > html/index.html
```

## Run

```bash
docker run -d --name nginx-lab \
  -p 8080:80 \
  -v nginx_logs:/var/log/nginx \
  -v ./html:/usr/share/nginx/html \
  nginx
```

- `nginx_logs:/var/log/nginx` — named volume, Docker-managed, persists independently of the container
- `./html:/usr/share/nginx/html` — bind mount, directly linked to a host folder

## Verify

```bash
curl localhost:8080
# Hello from Bind Mount
```

Edit the file on the host and confirm the change is reflected immediately (no restart needed, since it's a live bind mount):
```bash
echo "new html content" > html/index.html
curl localhost:8080
# Updated content via Bind Mount
```

Verify logs are persisted in the volume:
```bash
docker exec nginx-lab bash
 > ls /var/log/nginx
 > exit
```

## Clean Up

```bash
docker stop nginx-lab
docker rm nginx-lab
docker volume rm nginx_logs
```

## Learning Outcomes

- Difference between named volumes and bind mounts
- Persisting container logs independently of container lifecycle with a volume
- Live-editing served content from the host using a bind mount
- Inspecting and verifying data inside a Docker-managed volume
# Lab-9: Containerized Node.js and MySQL Stack Using Docker Compose

## Overview
This lab containerizes a Node.js/Express e-commerce app and a MySQL database using Docker Compose. The app connects to MySQL, requires a database named `ivolve` to be present, exposes `/health` and `/ready` endpoints, and writes access logs to `/app/logs/`.

**App source:** [kubernets-app](https://github.com/Ibrahim-Adel15/kubernets-app.git) (Node.js/Express, `mysql2`, `morgan`)

## Architecture

| Service | Image / Build | Port | Purpose |
|---------|---------------|------|---------|
| `app`   | Built from local `Dockerfile` (`node:18-alpine`) | `3000:3000` | Express app serving the frontend + API |
| `db`    | `mysql:8.0` | internal only | MySQL database, auto-creates the `ivolve` schema on first init |

## Prerequisites
- Docker & Docker Compose installed
- Docker Hub account (for the image push step)

## Setup

### 1. Clone the app source
```bash
git clone https://github.com/Ibrahim-Adel15/kubernets-app.git
cd kubernets-app
```

### 2. `docker-compose.yml`
```yaml
version: "3.9"

services:
  app:
    build: .
    container_name: ivolve_app
    ports:
      - "3000:3000"
    environment:
      DB_HOST: db
      DB_USER: root
      DB_PASSWORD: rootpassword
    volumes:
      - ./logs:/app/logs
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: mysql:8.0
    container_name: ivolve_db
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: ivolve
    volumes:
      - db_data:/var/lib/mysql
    restart: unless-stopped

volumes:
  db_data:
```

**Design notes:**
- `MYSQL_DATABASE: ivolve` makes MySQL auto-create the `ivolve` schema on first init, satisfying the app's internal `SHOW DATABASES LIKE 'ivolve'` startup check.
- `DB_USER: root` — the app authenticates directly with root credentials; it never creates a dedicated MySQL user.
- `depends_on` only controls container start order, not DB readiness. That's fine here because the app's `connectToDatabaseWithRetry()` function retries the connection every 5 seconds until MySQL (and the `ivolve` database) is available.
- `./logs:/app/logs` bind-mounts the app's log directory to the host for easy inspection.

### 3. Build and run
```bash
docker compose up -d --build
docker compose ps
docker compose logs -f app
```
Wait for:
```
✅ Connected to MySQL and 'ivolve' DB found.
🚀 Server started on http://0.0.0.0:3000
```

## Verification

### App is working
```bash
curl -i http://localhost:3000/
```
Returns the frontend (200 OK).

### Health & readiness checks
```bash
curl -i http://localhost:3000/health
curl -i http://localhost:3000/ready
```
Both return `200` once the DB connection is live:
- `/health` → `🚀 iVolve web app is working! Keep calm and code on! 🎉`
- `/ready` → `👍 iVolve web app is ready to rock and roll! 🤘`

### Access logs
```bash
cat ./logs/access.log
# or
docker compose exec app cat /app/logs/access.log
```
Populated with `morgan` `combined`-format entries for every request made above.

## Push image to Docker Hub
```bash
docker login

docker images | grep lab-9   # confirm local build tag, e.g. lab-9-app

docker tag lab-9-app:latest <your-dockerhub-username>/ivolve-app:latest
docker push <your-dockerhub-username>/ivolve-app:latest
```

## Notes / Gotchas
- The repo contains a `db.js` file with alternate DB-init logic, but it's dead code — `server.js` (the actual entrypoint per `package.json`) has its own inline connection/retry logic and never imports `db.js`.
- If `docker compose up --build` fails to pull `node:18-alpine` with a registry `EOF` error, this is a Docker Desktop ↔ Docker Hub connectivity issue, not a compose/Dockerfile problem — restart Docker Desktop or retry the build.
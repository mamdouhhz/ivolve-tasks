# Lab – Dockerizing a Spring Boot Application

## Overview

This lab containerizes a simple Spring Boot REST application using Docker. The goal is to build a minimal, secure runtime image: small base image, non-root user, and a single exposed port.

## Application

A minimal Spring Boot app with one endpoint:

```java
@GetMapping("/")
public String hello() {
    return "Hello from Dockerized Spring Boot!";
}
```

## Dockerfile

The final image is built from a slim JRE base rather than a full JDK/Maven image, since the jar is pre-built:

```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
RUN addgroup --system ivolve && adduser --system --ingroup ivolve mamdouh
COPY --chown=mamdouh:ivolve target/*.jar app.jar
USER mamdouh
EXPOSE 8080
CMD ["sh", "-c", "java -jar app.jar"]
```

Key decisions:
- **Non-root user** (`ivolve` group / `mamdouh` user) — the app never runs as root inside the container.
- **`eclipse-temurin:17-jre`** instead of a full JDK or Maven image — smaller final image since Maven/build tools aren't needed at runtime.
- **`--chown` on COPY** — sets file ownership in a single layer instead of an extra `RUN chown` step.

The Dockerfile also keeps two earlier iterations commented out at the top, showing the build evolution:
1. Single-stage build compiling inside the container with Maven.
2. Copying a pre-built jar but still using the full Maven image as the runtime base.
3. **(final)** Copying the pre-built jar into a slim JRE-only image.

## Prerequisites

- Docker
- A pre-built jar (`mvn package`) in `target/`

## Build the Image

```bash
mvn package
docker build -t docker-1-demo .
```

## Run the Container

```bash
docker run -p 8080:8080 docker-1-demo
```

## Verify

```bash
curl localhost:8080/
```

Expected output:
Hello from Dockerized Spring Boot!

## Screenshots

See `screenshots/` for build and run verification.

## Learning Outcomes

- Writing a Dockerfile for a Java/Spring Boot application
- Running containers as a non-root user
- Reducing image size by separating build and runtime bases
- Exposing and testing a containerized web service
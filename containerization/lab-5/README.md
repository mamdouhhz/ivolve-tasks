# Lab-5 Multi-Stage Build for a Spring Boot App

## Overview

Multi-stage Dockerfile that builds a Spring Boot app with Maven in one stage, then runs it from a minimal JRE image in a second stage — keeping the final image small and running as a non-root user.

## Dockerfile

```dockerfile
# Stage 1: build
FROM maven:3.9.16-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn package

# Stage 2: run
FROM eclipse-temurin:17-jre
WORKDIR /app
RUN addgroup --system ivolve && adduser --system --ingroup ivolve mamdouh
COPY --from=build --chown=mamdouh:ivolve /app/target/*.jar app.jar
USER mamdouh
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

## Build

```bash
docker build -t app3 .
docker images app3
```

## Run

```bash
docker run -d --name container3 -p 8080:8080 app3
```

## Test

```bash
curl localhost:8080/
```

## Clean Up

```bash
docker stop container3
docker rm container3
docker rmi app3
```

## Learning Outcomes

- Writing a multi-stage Dockerfile to separate build and runtime environments
- Reducing final image size by discarding build tools (Maven) from the runtime image
- Running the container as a non-root user
# ivolve-tasks
A collection of hands-on DevOps and Java build/containerization labs, completed as part of the NTI DevOps program (Hire Ready scholarship) internship at ivolve company.
Each folder is a standalone lab with its own README, prerequisites, and run instructions.

## Labs
| Lab | Description | Tech |
|---|---|---|
| [`calculator-gradle-main`](./lab-1) | Building and packaging a Java calculator app with Gradle | Java 17, Gradle, JUnit 5 |
| [`calculator-maven-main`](./lab-2) | Building and packaging a Java calculator app with Maven | Java 17, Maven, JUnit 5 |
| [`Docker-1-main`](./lab-3&4) | Dockerizing a Spring Boot REST app with a secure, non-root runtime image | Java 17, Spring Boot, Docker |
| [`lab-5`](./lab-5) | Multi-stage Docker build for a Spring Boot app | Java 17, Spring Boot, Docker |
| [`lab-6`](./lab-6) | Managing Docker environment variables across build and runtime | Python, Flask, Docker |

## Overview
These labs progress from core Java build tooling to containerization:

1. **Gradle & Maven labs** — understanding the Java build lifecycle: compiling, testing, dependency management, and generating runnable jar artifacts using the two most common JVM build tools.
2. **Docker lab** — taking a Spring Boot application and packaging it into a minimal, secure container image, including non-root user configuration and image size optimization.
3. **Multi-stage build lab** — separating build and runtime environments in a single Dockerfile to produce a smaller, more secure final image.
4. **Environment variables lab** — managing configuration across environments (development, staging, production) using Docker's different mechanisms for passing environment variables.

## Prerequisites
Varies per lab — see each lab's own README for exact versions and setup. In general:
- Java JDK 17
- Gradle and/or Maven
- Docker
- Python 3 (for lab-6)
  
## Learning Outcomes
Across these labs:
- Configuring and running Java builds with both Gradle and Maven
- Writing and executing automated unit tests (JUnit 5)
- Generating runnable jar artifacts
- Writing a production-conscious Dockerfile (non-root user, slim runtime base)
- Building, running, and verifying a containerized service
- Writing multi-stage Dockerfiles to reduce image size
- Managing environment-specific configuration via CLI flags, env files, and Dockerfile defaults

## Author
Mamdouh ([@mamdouhhz](https://github.com/mamdouhhz))

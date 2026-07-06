# ivolve-tasks

# Java Build Automation Labs – Gradle & Maven

## Overview

This repository contains two hands-on labs demonstrating how to build, test, package, and run Java applications using the two most popular Java build automation tools:

* **Gradle**
* **Maven**

The objective is to understand the Java build lifecycle, dependency management, unit testing, and artifact generation.

---

# Lab 1 – Building and Packaging with Gradle

## Prerequisites

* Java JDK 17
* Gradle

Verify the installation:

```bash
java -version
javac -version
gradle -version
```

---

## Clone the Repository

```bash
git clone https://github.com/Ibrahim-Adel15/calculator-gradle.git
cd calculator-gradle
```

---

## Run Unit Tests

```bash
gradle clean test
```

Expected output:

```
BUILD SUCCESSFUL
```

---

## Build the Application

```bash
gradle build
```

Generated artifact:

```
build/libs/calculator.jar
```

---

## Run the Application

```bash
java -jar build/libs/calculator.jar
```

---

# Lab 2 – Building and Packaging with Maven

## Prerequisites

* Java JDK 17
* Apache Maven

Verify the installation:

```bash
java -version
mvn -version
```

---

## Clone the Repository

```bash
git clone https://github.com/Ibrahim-Adel15/calculator-maven.git
cd calculator-maven
```

---

## Run Unit Tests

```bash
mvn test
```

Expected output:

```
BUILD SUCCESS
```

---

## Build the Application

```bash
mvn package
```

Generated artifact:

```
target/calculator.jar
```

---

## Run the Application

```bash
java -jar target/calculator.jar
```

---

# Project Structure

## Gradle

```
calculator-gradle
├── build.gradle
├── settings.gradle
├── src
│   ├── main
│   └── test
└── build
```

## Maven

```
calculator-maven
├── pom.xml
├── src
│   ├── main
│   └── test
└── target
```

---

# Technologies Used

* Java 17
* Gradle
* Maven
* JUnit

---

# Learning Outcomes

After completing these labs, you will be able to:

* Install and configure Java build tools.
* Execute automated unit tests.
* Build Java applications.
* Generate executable JAR artifacts.
* Run Java applications from the command line.
* Understand the differences between Gradle and Maven build systems.

# Lab-2 Building and Packaging with Maven

## Overview

A simple Java calculator application used to demonstrate the Maven build lifecycle: compiling, testing, and packaging into a runnable jar.

## Prerequisites

- Java JDK 17
- Apache Maven

Verify installation:

```bash
java -version
mvn -version
```

## Project Structure
calculator-maven-main
├── pom.xml
└── src
├── main
│   └── java/com/example/calculator/
│       ├── Main.java
│       └── Calculator.java
└── test
└── java/com/example/calculator/
└── CalculatorTest.java

## Run Unit Tests

```bash
mvn test
```

Expected output:
BUILD SUCCESS

## Build the Application

```bash
mvn package
```

Generated artifact:
target/calculator.jar

The `maven-jar-plugin` is configured to set the manifest's main class and produce a fixed jar name (`calculator.jar`) instead of the default `artifactId-version.jar`.

## Run the Application

```bash
java -jar target/calculator.jar
```

## Technologies Used

- Java 17
- Maven
- JUnit 5 (Jupiter)
- `maven-surefire-plugin` (test execution)
- `maven-jar-plugin` (runnable jar packaging)

## Learning Outcomes

- Configuring a Maven `pom.xml` for a Java application
- Running automated unit tests with the Surefire plugin
- Producing a runnable jar with a custom manifest via the Jar plugin
- Understanding Maven's build lifecycle vs. Gradle's
# Lab-1 Building and Packaging with Gradle

## Overview

A simple Java calculator application used to demonstrate the Gradle build lifecycle: compiling, testing, and packaging into a runnable jar.

## Prerequisites

- Java JDK 17
- Gradle

Verify installation:

```bash
java -version
javac -version
gradle -version
```

## Project Structure

calculator-gradle-main
├── build.gradle
├── settings.gradle
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
gradle clean test
```

Expected output:
BUILD SUCCESSFUL

## Build the Application

```bash
gradle build
```

Generated artifact:
build/libs/calculator.jar

## Run the Application

```bash
java -jar build/libs/calculator.jar
```

## Screenshots

See `screenshots/` for build and run verification.

## Technologies Used

- Java 17
- Gradle
- JUnit 5 (Jupiter)

## Learning Outcomes

- Configuring a Gradle build (`build.gradle`, `settings.gradle`)
- Running automated unit tests with JUnit via Gradle
- Producing a runnable, named jar artifact via the `jar` task
- Understanding Gradle's build lifecycle vs. Maven's
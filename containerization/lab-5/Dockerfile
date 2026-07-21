# stage 1
FROM maven:3.9.16-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn package

# stage 2
FROM eclipse-temurin:17-jre
WORKDIR /app
RUN addgroup --system ivolve && adduser --system --ingroup ivolve mamdouh
COPY --from=build --chown=mamdouh:ivolve /app/target/*.jar app.jar
USER mamdouh
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
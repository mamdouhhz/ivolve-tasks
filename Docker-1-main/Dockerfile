# image 1
# FROM maven:3.9.16-eclipse-temurin-17
# WORKDIR /app
# COPY pom.xml .
# RUN mvn dependency:go-offline
# COPY src ./src
# RUN mvn package
# RUN addgroup --system ivolve && adduser --system --ingroup ivolve mamdouh
# USER mamdouh
# EXPOSE 8080
# CMD ["sh", "-c", "java -jar app.jar"]

# image 2
# FROM maven:3.9.16-eclipse-temurin-17
# WORKDIR /app
# COPY target/*.jar app.jar
# RUN addgroup --system ivolve && adduser --system --ingroup ivolve mamdouh
# USER mamdouh
# EXPOSE 8080
# CMD ["sh", "-c", "java -jar app.jar"]

# image 3
FROM eclipse-temurin:17-jre
WORKDIR /app
RUN addgroup --system ivolve && adduser --system --ingroup ivolve mamdouh
COPY --chown=mamdouh:ivolve target/*.jar app.jar
USER mamdouh
EXPOSE 8080
CMD ["sh", "-c", "java -jar app.jar"]
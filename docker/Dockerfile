# ----------------------------------------------------------------------
# STAGE 1: Build (Maven and JDK)
# ----------------------------------------------------------------------
# FIX: Use a verified Maven tag with the JDK 21
FROM maven:3-jdk-21 AS build 

WORKDIR /app
COPY pom.xml .
RUN mvn dependency:resolve
COPY src ./src
RUN mvn package -DskipTests

# ----------------------------------------------------------------------
# STAGE 2: Run (JRE only)
# ----------------------------------------------------------------------
# FIX: Use a reliable, verified JRE 21 tag (e.g., Eclipse Temurin)
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app
EXPOSE 8080

# Copy the built JAR from the 'build' stage
COPY --from=build /app/target/*.jar app.jar

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
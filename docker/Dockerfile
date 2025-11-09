# ----------------------------------------------------------------------
# STAGE 1: Build (Maven and JDK)
# ----------------------------------------------------------------------
# Using the slim-based tag to improve pull reliability and reduce size,
# while explicitly keeping Java 21 for the enforcer plugin.
FROM maven:3.9.6-openjdk-21-slim AS build 

WORKDIR /app
COPY pom.xml .
RUN mvn dependency:resolve
COPY src ./src
RUN mvn package -DskipTests

# ----------------------------------------------------------------------
# STAGE 2: Run (JRE only)
# ----------------------------------------------------------------------
# Use the official, current tag for OpenJDK 21 JRE runtime
FROM openjdk:21-jre-slim

WORKDIR /app
EXPOSE 8080

# The JAR file name is 'my-app-1.0-SNAPSHOT.jar' based on your pom.xml
COPY --from=build /app/target/my-app-1.0-SNAPSHOT.jar /app/app.jar

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
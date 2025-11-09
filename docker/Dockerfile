# ----------------------------------------------------------------------
# STAGE 1: Build - Compiles the application using a full JDK/Maven image
# ----------------------------------------------------------------------
# Use a Maven image with Java 21 JDK for building
FROM openjdk:21-jre-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the Maven project file first to leverage Docker's layer caching 
# (The dependencies layer won't rebuild if pom.xml hasn't changed)
COPY pom.xml .

# Download project dependencies
RUN mvn dependency:resolve

# Copy the remaining source code
COPY src ./src

# Build the application, creating the final JAR in the target directory
# We skip tests here since they should have passed in the Jenkins 'Unit Tests' stage
RUN mvn package -DskipTests

# ----------------------------------------------------------------------
# STAGE 2: Run - Creates the final, minimal runtime image
# ----------------------------------------------------------------------
# Use a JRE-only image for a smaller and more secure final container
FROM openjdk:21-jre-slim

# Set the working directory for the runtime environment
WORKDIR /app

# Expose the port your application listens on (e.g., 8080 for Spring Boot)
EXPOSE 8080

# Copy the built JAR from the 'build' stage into the final image
# The JAR file is copied to the root of the /app directory and renamed to app.jar
COPY --from=build /app/target/*.jar app.jar

# Define the command to run the application when the container starts
ENTRYPOINT ["java", "-jar", "/app/app.jar"]

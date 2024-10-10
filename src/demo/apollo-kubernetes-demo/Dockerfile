# Use a Maven image to build the project
FROM maven:3.8.5-openjdk-11 AS build

# Set the working directory inside the container
WORKDIR /app

# Copy the pom.xml and the source code
COPY pom.xml .
COPY src ./src

# Build the project (this will create the JAR file in the target/ directory)
RUN mvn clean package

# Use a slim OpenJDK image for the final running image
FROM openjdk:11-jre-slim

# Set the working directory
WORKDIR /app

# Copy the JAR file from the Maven build stage to the current stage
COPY --from=build /app/target/apollo-kubernetes-demo-1.0-SNAPSHOT.jar apollo-kubernetes-demo.jar

# Command to run the application, allowing for additional arguments like "myKey"
ENTRYPOINT ["java", "-jar", "apollo-kubernetes-demo.jar"]

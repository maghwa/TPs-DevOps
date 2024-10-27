# ✨ Docker Project: 3-Tier Web Application ✨

This repository demonstrates the setup of a 3-tier web application using Docker, including an HTTP server, a backend API, and a PostgreSQL database. Each component is containerized and managed using Docker and Docker Compose.

## ❓Question : Why should we run the container with the -e flag to give environment variables?

⭐️ Running the container with the -e flag to provide environment variables is essential because it allows us to securely and dynamically pass necessary configuration data, such as database credentials (e.g., POSTGRES_DB, POSTGRES_USER, and POSTGRES_PASSWORD), directly to the container at runtime. This avoids the need to hardcode sensitive information in the Dockerfile or application code, which enhances security and flexibility. By using environment variables, we can easily change configurations for different environments (e.g., development, testing, production) without modifying the container image itself, making the setup more portable and manageable.

## ❓Question : Why do we need a volume to be attached to our PostgreSQL container?

⭐️ Attaching a volume to the PostgreSQL container is essential for data persistence. When you run a container, any data stored inside the container's file system will be lost if the container stops, crashes, or is removed. By attaching a volume, you ensure that your database data is stored on the host machine rather than within the ephemeral container storage. This way, even if the container is recreated or stopped, the data remains intact and can be accessed when the container restarts, which is critical for maintaining consistent and reliable data storage in a database environment.

## 1-1 Documenting Database Container Essentials: Commands and Dockerfile
Here's the Dockerfile for setting up a PostgreSQL container :

<img width="730" alt="image" src="https://github.com/user-attachments/assets/2bad855d-0674-47a6-8ab4-e53c662c281f">

In this Dockerfile:


- Base Image: We're using postgres:14.1-alpine, a lightweight version of PostgreSQL.
- Environment Variables: We set environment variables to configure the database name, user, and password.
- Initialization Scripts: SQL scripts in /docker-entrypoint-initdb.d/ are automatically executed when the container is first started, allowing us to set up our schema and seed data.

### Commands to Run the PostgreSQL Container

Build the Docker Image:
```bash

docker build -t database .
```
Create a Network:
```bash
docker network create app-network
```
Run the PostgreSQL Container:
```bash
docker run --name mydatabase -v /my/folder/datadir:/var/lib/postgresql/data --net=app-network -d database
```
## ❓Question : 1-2 Why do we need a multistage build? And explain each step of this dockerfile.

⭐️ A multistage build is essential in Docker to optimize the final image size and streamline the build process. By using multistage builds, we can include all the dependencies needed for building the application in one stage and then copy only the necessary artifacts into a smaller, final image. This reduces the image size and improves security by excluding unnecessary build tools and dependencies.

```bash
# Build Stage
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# Final Stage
FROM amazoncorretto:17
WORKDIR /app
COPY --from=myapp-build /app/target/*.jar myapp.jar
ENTRYPOINT ["java", "-jar", "myapp.jar"]
```
⭐️ Explanation of Each Step
#### First Stage (Build Stage):
- FROM maven:3.8.6-amazoncorretto-17 AS myapp-build: This stage uses a Maven image with Amazon Corretto JDK 17, which is suitable for building Java applications.
- WORKDIR /app: Sets the working directory inside the container.
- COPY . .: Copies all files from the current directory into the /app directory in the container.
- RUN mvn clean package -DskipTests: Builds the application using Maven and skips the tests. The resulting JAR file is created in the target directory.
#### Second Stage (Final Stage):
- FROM amazoncorretto:17: Uses a smaller base image with only the Amazon Corretto JDK, without any build tools like Maven. This reduces the final image size.
- WORKDIR /app: Sets the working directory for the final image.
- COPY --from=myapp-build /app/target/*.jar myapp.jar: Copies only the built JAR file from the previous build stage (myapp-build) to the final image.
- ENTRYPOINT ["java", "-jar", "myapp.jar"]: Specifies the command to run the application.

## ❓Question : Why do we need a reverse proxy?
















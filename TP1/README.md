# ✨ Docker Project: 3-Tier Web Application ✨

This repository demonstrates the setup of a 3-tier web application using Docker, including an HTTP server, a backend API, and a PostgreSQL database. Each component is containerized and managed using Docker and Docker Compose.

## ❓Question : Why should we run the container with the -e flag to give environment variables?

⭐️ Running the container with the -e flag to provide environment variables is essential because it allows us to securely and dynamically pass necessary configuration data, such as database credentials (e.g., POSTGRES_DB, POSTGRES_USER, and POSTGRES_PASSWORD), directly to the container at runtime. This avoids the need to hardcode sensitive information in the Dockerfile or application code, which enhances security and flexibility. By using environment variables, we can easily change configurations for different environments (e.g., development, testing, production) without modifying the container image itself, making the setup more portable and manageable.

## ❓Question : Why do we need a volume to be attached to our postgres container?

⭐️ Attaching a volume to the PostgreSQL container is essential for data persistence. When you run a container, any data stored inside the container's file system will be lost if the container stops, crashes, or is removed. By attaching a volume, you ensure that your database data is stored on the host machine rather than within the ephemeral container storage. This way, even if the container is recreated or stopped, the data remains intact and can be accessed when the container restarts, which is critical for maintaining consistent and reliable data storage in a database environment.

## 1-1 Documenting Database Container Essentials: Commands and Dockerfile
<img width="382" alt="image" src="https://github.com/user-attachments/assets/577bd676-e956-4902-9085-893cc653bf94">

In this Dockerfile:
Base Image: We're using postgres:14.1-alpine, a lightweight version of PostgreSQL.
Environment Variables: We set environment variables to configure the database name, user, and password.
Initialization Scripts: SQL scripts in /docker-entrypoint-initdb.d/ are automatically executed when the container is first started, allowing you to set up your schema and seed data.

We can find the connection information needed to access the database.

### 1. Build the Docker Image

```bash
docker build -t database .
```
### 2. Create a Network

```bash
docker network create app-network
```

### 3. Run the PostgreSQL Container

```bash
docker run --name mydatabase -v /my/folder/datadir:/var/lib/postgresql/data --net=app-network -d database
```

The -v option is used to attach a volume, ensuring that data persists even if the container is removed. This stores the database files outside the container, preventing data loss on container deletion.

## ❓Question : Why do we need a multistage build? And explain each step of this dockerfile.
A multistage build is essential in Docker to optimize the final image size and streamline the build process. By using multistage builds, we can include all the dependencies needed for building the application in one stage and then copy only the necessary artifacts into a smaller, final image. This reduces the image size and improves security by excluding unnecessary build tools and dependencies.
Here’s an example explanation for each step of a Dockerfile using a multistage build:

```bash
# Build Stage
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests
```

First Stage (Build Stage):
- FROM maven:3.8.6-amazoncorretto-17 AS myapp-build: This stage uses a Maven image with Amazon Corretto JDK 17, which is suitable for building Java applications.
- WORKDIR /app: Sets the working directory inside the container.
- COPY . .: Copies all files from the current directory into the /app directory in the container.
- RUN mvn clean package -DskipTests: Builds the application using Maven and skips the tests. The resulting JAR file is created in the target directory.

```bash
# Final Stage
FROM amazoncorretto:17
WORKDIR /app
COPY --from=myapp-build /app/target/*.jar myapp.jar
ENTRYPOINT ["java", "-jar", "myapp.jar"]

```
Second Stage (Final Stage):
- FROM amazoncorretto:17: Uses a smaller base image with only the Amazon Corretto JDK, without any build tools like Maven. This reduces the final image size.

- WORKDIR /app: Sets the working directory for the final image.
- COPY --from=myapp-build /app/target/*.jar myapp.jar: Copies only the built JAR file from the previous build stage (myapp-build) to the final image.
- ENTRYPOINT ["java", "-jar", "myapp.jar"]: Specifies the command to run the application.

## ❓Question : Why do we need a reverse proxy?

⭐️ A reverse proxy, like Apache or Nginx, sits in front of our application servers, forwarding client requests to the appropriate backend service. It can enhance security, enable load balancing, and centralize SSL termination for better performance and scalability.


## ❓Question : Why is docker-compose so important?

⭐️ Docker Compose is essential because it simplifies managing multi-container applications. It allows you to define all services (e.g., database, backend, HTTP server) in a single file, enabling seamless setup, networking, and environment configuration. You can start everything with one command, manage persistent data easily, and have a centralized configuration for streamlined development, testing, and deployment. This is particularly useful in your TP for efficient management of interconnected services.

## 1-3 Document docker-compose most important commands. 
### 1.Start Services
```bash

docker-compose up
```
### 2. Stop Services
```bash

docker-compose down
```
### 3. Rebuild Services
```bash

docker-compose up --build
```

### 4. View Running Containers
```bash

docker-compose ps
```

## 1-4 Document the docker-compose file.
```bash
version: '3.7'

services:
  # Database service configuration
  database:
    image: postgres:14
    environment:
      POSTGRES_USER: usr
      POSTGRES_PASSWORD: pwd
      POSTGRES_DB: db
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

  # Backend API service configuration
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    depends_on:
      - database
    networks:
      - app-network

  # HTTP server (reverse proxy) configuration
  httpd:
    image: httpd:latest
    volumes:
      - ./config/httpd.conf:/usr/local/apache2/conf/httpd.conf
    depends_on:
      - backend
    networks:
      - app-network

networks:
  app-network:

volumes:
  db-data:
```

## ❓Question : Why do we put our images into an online repo?

⭐️ Putting images in an online repo allows easy access, consistent sharing, version control, automated deployments, and serves as a reliable backup for your Docker images. It’s essential for collaboration, consistency across environments, and scaling.









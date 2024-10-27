# ✨ Docker Project: 3-Tier Web Application ✨

This repository demonstrates the setup of a 3-tier web application using Docker, including an HTTP server, a backend API, and a PostgreSQL database. Each component is containerized and managed using Docker and Docker Compose.

## ❓Question : Why should we run the container with the -e flag to give environment variables?

⭐️ Running the container with the -e flag to provide environment variables is essential because it allows us to securely and dynamically pass necessary configuration data, such as database credentials (e.g., POSTGRES_DB, POSTGRES_USER, and POSTGRES_PASSWORD), directly to the container at runtime. This avoids the need to hardcode sensitive information in the Dockerfile or application code, which enhances security and flexibility. By using environment variables, we can easily change configurations for different environments (e.g., development, testing, production) without modifying the container image itself, making the setup more portable and manageable.

## ❓Question : Why do we need a volume to be attached to our PostgreSQL container?

⭐️ Attaching a volume to the PostgreSQL container is essential for data persistence. When you run a container, any data stored inside the container's file system will be lost if the container stops, crashes, or is removed. By attaching a volume, you ensure that your database data is stored on the host machine rather than within the ephemeral container storage. This way, even if the container is recreated or stopped, the data remains intact and can be accessed when the container restarts, which is critical for maintaining consistent and reliable data storage in a database environment.

## ❓ 1-1 Documenting Database Container Essentials: Commands and Dockerfile

⭐️ Here's the Dockerfile for setting up a PostgreSQL container :

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

⭐️ A reverse proxy is essential for load balancing, enhancing security, simplifying client access, and improving performance with caching and compression. In Docker, it helps manage multi-container applications by streamlining communication and centralizing access.

## ❓Question : Why is docker-compose so important? 

⭐️ Docker Compose is essential because it simplifies managing multi-container applications. It allows you to define all services (e.g., database, backend, HTTP server) in a single file, enabling seamless setup, networking, and environment configuration. You can start everything with one command, manage persistent data easily, and have a centralized configuration for streamlined development, testing, and deployment. This is particularly useful in your TP for efficient management of interconnected services.

## ❓ 1-3 Most Important Docker Compose Commands

⭐️ Here are some essential `docker-compose` commands that are widely used:

1. **Start Services**  
   ```bash
   docker-compose up
   ```

2. **Stop Services**  
   ```bash
   docker-compose down
   ```
   Stops and removes all the containers created by `docker-compose up`.

3. **Rebuild Services**  
   ```bash
   docker-compose up --build
   ```
   Builds or rebuilds services before starting them.

4. **View Running Containers**  
   ```bash
   docker-compose ps
   ```
   Lists the containers managed by Docker Compose.

5. **Check Logs**  
   ```bash
   docker-compose logs
   ```
   Displays logs for all services. Use `docker-compose logs -f` for continuous log output.

6. **Execute a Command in a Service Container**  
   ```bash
   docker-compose exec <service-name> <command>
   ```
   Runs a command in a running service container, e.g., `docker-compose exec db psql` to access the database container.

### ❓ 1-4 Document the `docker-compose.yml` File

Here’s an example explanation of a `docker-compose.yml` file configuration based on a typical setup:

```yaml
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

#### Explanation:

1. **Services**  
   - **database**: Defines the PostgreSQL database service with environment variables for user, password, and database name. A volume (`db-data`) is used to persist data.
   - **backend**: Defines the backend API, built from a Dockerfile located in the `./backend` directory. The `depends_on` key ensures the database service starts before this service.
   - **httpd**: Configures the HTTP server (reverse proxy) using the Apache HTTP image. It mounts a custom configuration file (`httpd.conf`) for reverse proxy setup.

2. **Networks**  
   Creates a custom network (`app-network`) to allow the services to communicate.

3. **Volumes**  
   Defines a named volume (`db-data`) to persist the database data even if the container stops or is removed.

This structure supports an organized multi-service application with inter-service communication, simplified deployment, and persistent storage.


## ❓ Question : Why do we put our images into an online repo?

⭐️ Putting images in an online repo allows easy access, consistent sharing, version control, automated deployments, and serves as a reliable backup for your Docker images. It’s essential for collaboration, consistency across environments, and scaling.












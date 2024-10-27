# Docker Project: 3-Tier Web Application

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
Build the Docker Image:
### 1. Build the Docker Image

Use the following command to build the Docker image for your PostgreSQL database:

```bash
docker build -t database .





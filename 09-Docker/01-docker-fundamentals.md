# Docker — File 01: Docker Fundamentals

This is the first Docker file for backend interview preparation.

The goal is to understand Docker from the ground up before moving into Dockerfiles, Compose and production deployment.

---

# 1. What Is Docker?

Docker is a platform for packaging and running applications in isolated environments called containers.

A simplified model:

```text
Application
    +
Dependencies
    +
Runtime configuration
        ↓
      Image
        ↓
    Container
```

Docker helps make application environments more consistent across:

```text
Developer machine
Test environment
CI/CD
Production
```

---

# 2. Why Do We Need Docker?

Without containers, an application may depend on:

```text
Java version
Maven version
OS libraries
Environment variables
Database version
Other services
```

A developer may say:

> "It works on my machine."

Docker helps package the application environment more consistently.

---

# 3. Traditional Deployment

Without Docker:

```text
Server
 ├── Java
 ├── Maven
 ├── Application
 ├── OS dependencies
 └── Configuration
```

Different applications may require different versions.

This can create:

```text
Dependency conflicts
Environment differences
Deployment issues
```

---

# 4. Docker Deployment

With Docker:

```text
Docker Host
 ├── Container A
 │    └── Application
 │
 ├── Container B
 │    └── Application
 │
 └── Container C
      └── Database
```

Each container has its own isolated process environment.

---

# 5. Container

A container is a running instance of an image.

Think:

```text
Image
  ↓
Container
```

Similar to:

```text
Class
  ↓
Object
```

The analogy isn't perfect, but it is useful for interviews.

---

# 6. Docker Image

An image is a packaged, immutable template containing the application and required filesystem content.

Example:

```text
Spring Boot JAR
+
Java runtime
+
Linux filesystem layers
+
Configuration
```

becomes an image.

---

# 7. Image vs Container

### Image

```text
Template
Read-only layers
Can be stored/shared
```

### Container

```text
Running instance
Uses image
Has writable container layer
Has isolated processes/network/filesystem
```

Simple mental model:

```text
Image
 ↓ run
Container
```

---

# 8. Dockerfile

A Dockerfile describes how to build an image.

Example:

```dockerfile
FROM eclipse-temurin:21-jre

COPY target/app.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Conceptually:

```text
Dockerfile
    ↓
docker build
    ↓
Image
    ↓
docker run
    ↓
Container
```

---

# 9. Docker Engine

Docker Engine is the core technology that builds and runs containers.

Conceptually:

```text
Docker CLI
    ↓
Docker Engine
    ↓
Containers / Images / Networks / Volumes
```

---

# 10. Docker CLI

The command-line interface is used to interact with Docker.

Examples:

```bash
docker build
docker run
docker ps
docker images
docker stop
docker rm
```

---

# 11. Docker Desktop

On macOS and Windows, Docker Desktop provides the Docker environment and tooling needed to run Linux containers.

On Linux, Docker can run more directly using the Linux container runtime stack.

For development, the important mental model is:

```text
Your machine
    ↓
Docker environment
    ↓
Linux containers
```

---

# 12. Container Isolation

Containers isolate processes using operating-system features.

Important Linux concepts include:

```text
Namespaces
cgroups
Union/overlay filesystems
```

You don't need kernel-level implementation details for most Java backend interviews, but understand the purpose.

---

# 13. Namespaces

Namespaces help isolate things such as:

```text
Processes
Network
Mounts
Hostname
Users
```

A process inside a container sees an isolated environment rather than the entire host environment.

---

# 14. cgroups

Control groups help limit and measure resource usage such as:

```text
CPU
Memory
```

This allows container workloads to be controlled.

---

# 15. Containers vs Virtual Machines

### Virtual Machine

```text
Hardware
 ↓
Host OS
 ↓
Hypervisor
 ↓
Guest OS
 ↓
Application
```

### Container

```text
Hardware
 ↓
Host OS
 ↓
Container runtime
 ↓
Container
 ↓
Application
```

Containers generally share the host kernel rather than requiring a complete guest OS per application.

---

# 16. Why Containers Are Lightweight

A VM generally includes:

```text
Guest OS
Libraries
Application
```

A container typically shares the host kernel and packages:

```text
Application
Libraries
Required filesystem
```

Therefore containers can start faster and use fewer resources in many workloads.

---

# 17. Container vs VM Interview Answer

Say:

> "A VM virtualizes hardware and normally includes a full guest operating system. Containers isolate application processes while sharing the host kernel, so they are generally lighter and faster to start."

---

# 18. Docker Image Layers

Docker images are built from layers.

Example:

```text
Base image layer
      ↓
Java runtime layer
      ↓
Application dependency layer
      ↓
Application layer
```

Layers can be reused between images.

---

# 19. Why Layers Matter

Suppose:

```text
Image A
Image B
```

both use:

```text
Java 21 base image
```

Docker can potentially reuse the common layers.

This improves:

```text
Build speed
Storage efficiency
Image distribution
```

---

# 20. Image Immutability

Images are treated as immutable artifacts.

If you want a changed application:

```text
Modify source
 ↓
Build new image
```

rather than manually modifying the image as a running server.

---

# 21. Container Writable Layer

When a container runs from an image, Docker adds a writable layer for container-specific changes.

Conceptually:

```text
Image layers
   ↓
Writable container layer
```

Important:

> Data written only to the container's writable layer is not a good place for persistent application data.

---

# 22. Why Containers Are Ephemeral

A container can be:

```text
Stopped
Removed
Recreated
```

If important data exists only inside the container:

```text
docker rm
 ↓
data may disappear
```

For persistent data, use:

```text
Volumes
Bind mounts
External storage
Database
```

---

# 23. Container Lifecycle

Basic lifecycle:

```text
Image
 ↓
create
 ↓
Created
 ↓
start
 ↓
Running
 ↓
stop
 ↓
Stopped
 ↓
remove
 ↓
Deleted
```

---

# 24. docker run

The simplest command:

```bash
docker run nginx
```

This generally:

```text
Pull image if needed
Create container
Start container
```

---

# 25. Run in Detached Mode

```bash
docker run -d nginx
```

`-d` means:

```text
Detached
```

The container runs in the background.

---

# 26. List Running Containers

```bash
docker ps
```

Shows running containers.

Typical information includes:

```text
Container ID
Image
Command
Created
Status
Ports
Name
```

---

# 27. List All Containers

```bash
docker ps -a
```

This includes:

```text
Running
Stopped
Exited
Created
```

containers.

---

# 28. List Images

```bash
docker images
```

or:

```bash
docker image ls
```

You can see:

```text
Repository
Tag
Image ID
Created
Size
```

---

# 29. Container Names

Docker can assign random names.

You can choose one:

```bash
docker run --name my-app nginx
```

Then:

```bash
docker stop my-app
```

is easier than remembering the container ID.

---

# 30. Container ID

Every container has an ID.

Example:

```text
8c1f2a...
```

You can use the ID in commands:

```bash
docker stop 8c1f2a
```

Usually the short ID is sufficient.

---

# 31. Docker Logs

View container logs:

```bash
docker logs my-app
```

Follow logs:

```bash
docker logs -f my-app
```

`-f` means:

```text
Follow
```

---

# 32. Spring Boot Logs

If a Spring Boot application runs in a container:

```text
Spring Boot
    ↓
stdout/stderr
    ↓
Docker logs
```

This is why applications should generally log to stdout/stderr in containerized environments.

---

# 33. Execute a Command Inside Container

```bash
docker exec -it my-app sh
```

This opens an interactive shell if the image contains `sh`.

Some images may have:

```bash
bash
```

instead.

---

# 34. exec vs run

Important:

```text
docker run
→ Creates and starts a new container

docker exec
→ Runs a command inside an existing running container
```

---

# 35. Stop a Container

```bash
docker stop my-app
```

Docker asks the process to stop gracefully and then forcefully terminates it if it doesn't exit within the configured grace period.

---

# 36. Start a Stopped Container

```bash
docker start my-app
```

This starts an existing stopped container.

It does not create a new container.

---

# 37. Restart a Container

```bash
docker restart my-app
```

This restarts the existing container.

---

# 38. Remove a Container

```bash
docker rm my-app
```

Normally the container should be stopped first.

Force removal:

```bash
docker rm -f my-app
```

Use force carefully.

---

# 39. Remove an Image

```bash
docker rmi my-image
```

or:

```bash
docker image rm my-image
```

The image may need to be unused before removal.

---

# 40. Pull an Image

```bash
docker pull redis
```

Docker retrieves the image from a configured registry, commonly Docker Hub.

---

# 41. Image Tags

Example:

```bash
docker pull redis:7
```

Here:

```text
redis
→ repository

7
→ tag
```

Avoid assuming:

```text
latest
```

is always a stable production version.

---

# 42. Why Pin Versions?

Instead of:

```dockerfile
FROM redis:latest
```

production builds may prefer:

```dockerfile
FROM redis:7.x
```

or an appropriately pinned digest/version strategy.

This improves:

```text
Reproducibility
Predictability
Security management
```

---

# 43. Docker Registry

A registry stores Docker images.

Examples:

```text
Docker Hub
Amazon ECR
Google Artifact Registry
Azure Container Registry
GitHub Container Registry
Private registries
```

Conceptually:

```text
Developer
 ↓
docker push
 ↓
Registry
 ↓
docker pull
 ↓
Server
```

---

# 44. Docker Hub

Docker Hub is a public/private container image registry.

You can:

```text
Pull images
Push your images
Store repositories
```

Example:

```bash
docker pull mysql
```

---

# 45. Image Naming

Typical image reference:

```text
registry/repository:tag
```

Example:

```text
docker.io/library/redis:7
```

For your own organization:

```text
registry.example.com/ecommerce-api:1.0
```

---

# 46. Port Mapping

Containers have their own network namespace.

Suppose Spring Boot listens inside the container on:

```text
8080
```

To expose it on your host:

```bash
docker run -p 8080:8080 my-app
```

Format:

```text
hostPort:containerPort
```

---

# 47. Port Example

```bash
docker run -p 9090:8080 my-app
```

Means:

```text
Host:
9090

Container:
8080
```

Request:

```text
localhost:9090
```

is forwarded to:

```text
container:8080
```

---

# 48. EXPOSE

Dockerfile can contain:

```dockerfile
EXPOSE 8080
```

This documents the intended container port.

Important:

> `EXPOSE` does not by itself publish the port to the host.

You still use:

```bash
docker run -p 8080:8080 image
```

to publish it.

---

# 49. Environment Variables

Containers commonly receive configuration through environment variables.

Example:

```bash
docker run \
  -e SPRING_PROFILES_ACTIVE=docker \
  my-app
```

Application reads:

```text
SPRING_PROFILES_ACTIVE
```

---

# 50. Why Environment Variables?

They allow the same image to run with different configuration:

```text
Development
Test
Production
```

without rebuilding the application image.

---

# 51. Don't Bake Secrets Into Images

Bad:

```dockerfile
ENV DB_PASSWORD=secret123
```

Better:

```text
Secret manager
Environment
Docker secrets/orchestrator secrets
```

depending on deployment environment.

---

# 52. Docker Networking

Containers can communicate through Docker networks.

Example:

```text
Spring Boot
    |
Docker network
    |
MySQL
```

The application can use the container/service name as the hostname in many Docker networking setups.

---

# 53. Why Not localhost?

Inside a container:

```text
localhost
```

means:

```text
That same container
```

It does NOT automatically mean:

```text
Your host machine
```

This is one of the most common Docker mistakes.

---

# 54. Example: Spring Boot + MySQL

Suppose:

```text
spring-app
mysql
```

are on the same Docker network.

Spring Boot should generally connect using:

```text
mysql:3306
```

rather than:

```text
localhost:3306
```

because:

```text
mysql
```

is the other container's network hostname.

---

# 55. Docker Volume

A volume provides persistent storage managed by Docker.

Example:

```bash
docker volume create mysql-data
```

Then mount it:

```bash
docker run \
  -v mysql-data:/var/lib/mysql \
  mysql
```

---

# 56. Why Volumes?

Without a volume:

```text
MySQL container
 ↓
Remove container
 ↓
Data may disappear
```

With a volume:

```text
MySQL container
 ↓
Docker volume
 ↓
Container removed
 ↓
Volume remains
```

---

# 57. Container vs Volume

Container:

```text
Ephemeral runtime
```

Volume:

```text
Persistent data storage
```

---

# 58. Bind Mount

A bind mount maps a host filesystem path into a container.

Conceptually:

```text
Host directory
      ↓
Container directory
```

Example:

```bash
docker run -v $(pwd)/data:/app/data my-app
```

Volumes are generally preferred for Docker-managed application data, while bind mounts are useful for development and specific host-file scenarios.

---

# 59. Docker Image Build

Suppose project contains:

```text
Dockerfile
target/app.jar
```

Build:

```bash
docker build -t ecommerce-api:1.0 .
```

Meaning:

```text
docker build
 ↓
-t ecommerce-api:1.0
 ↓
context = current directory
```

---

# 60. Build Context

The final:

```text
.
```

means:

```text
Current directory
```

Docker sends the build context to the builder.

Avoid putting unnecessary large files into the context.

---

# 61. .dockerignore

Create:

```text
.dockerignore
```

Example:

```text
.git
.idea
node_modules
target
*.log
.env
```

But be careful:

If your Dockerfile needs:

```text
target/app.jar
```

don't ignore `target` unless you're using a multi-stage build that creates the artifact inside Docker.

---

# 62. Dockerfile Example

Simple Spring Boot image:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Flow:

```text
Base image
 ↓
Working directory
 ↓
Copy JAR
 ↓
Document port
 ↓
Start application
```

---

# 63. WORKDIR

```dockerfile
WORKDIR /app
```

sets the working directory for subsequent Dockerfile instructions and the container process.

Instead of:

```text
/
```

application commands operate from:

```text
/app
```

---

# 64. COPY

```dockerfile
COPY target/app.jar app.jar
```

copies a file from the build context into the image.

Conceptually:

```text
Host/build context
      ↓
Image
```

---

# 65. ENTRYPOINT

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

defines the primary process that starts when the container runs.

---

# 66. CMD

Example:

```dockerfile
CMD ["java", "-jar", "app.jar"]
```

`CMD` provides the default command/arguments.

A common interview distinction:

```text
ENTRYPOINT
→ Defines the main executable

CMD
→ Provides default command/arguments
```

Their exact interaction depends on exec vs shell forms and runtime arguments.

---

# 67. ENTRYPOINT vs CMD

A simple interview answer:

> "I usually use ENTRYPOINT for the application's main executable and CMD for default arguments that can be overridden."

---

# 68. Exec Form

Preferred for many application containers:

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

This starts the process directly.

Benefits include better signal handling compared with wrapping the application unnecessarily in a shell.

---

# 69. Shell Form

Example:

```dockerfile
ENTRYPOINT java -jar app.jar
```

This uses shell processing.

For production application containers, exec form is generally preferable when you don't need shell behavior.

---

# 70. Container Main Process

A container normally lives as long as its main process runs.

Example:

```text
java -jar app.jar
```

If that process exits:

```text
Container exits
```

This is a key Docker concept.

---

# 71. Container Status

A container may show:

```text
Up
Exited (0)
Exited (1)
Restarting
Created
```

`Exited (0)` generally indicates normal process termination.

A non-zero code usually indicates an error.

---

# 72. Debugging a Container

Useful sequence:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
docker exec -it <container> sh
```

Check:

```text
Status
Logs
Environment
Network
Mounts
Command
```

---

# 73. docker inspect

```bash
docker inspect my-app
```

Provides detailed metadata.

Useful for investigating:

```text
Network
Mounts
Environment
IP information
Container configuration
```

---

# 74. Container Health

Docker supports health checks.

Conceptually:

```dockerfile
HEALTHCHECK CMD ...
```

A health check can indicate whether the application is responding as expected.

Important:

```text
Running
≠
Healthy
```

A process can be alive while the application is broken.

---

# 75. Spring Boot Actuator + Docker

A common backend setup is:

```text
Spring Boot Actuator
       ↓
/actuator/health
       ↓
Docker health check
```

This can provide application-level health information.

---

# 76. Restart Policies

Containers can be configured to restart under certain conditions.

Examples include:

```text
no
on-failure
always
unless-stopped
```

Use restart policies thoughtfully.

They should not hide application bugs.

---

# 77. Restart Loop

If application crashes immediately:

```text
Container starts
 ↓
Application crashes
 ↓
Restart
 ↓
Application crashes
 ↓
...
```

This can create a restart loop.

Always inspect:

```bash
docker logs
```

and root cause the application failure.

---

# 78. Docker Resource Limits

Containers can be given resource constraints.

Conceptually:

```text
CPU limit
Memory limit
```

This prevents one workload from consuming unlimited resources.

Example concepts:

```bash
--memory
--cpus
```

Exact settings depend on the deployment environment.

---

# 79. Why Memory Limits Matter

Suppose:

```text
Container A
```

has a memory leak.

Without proper resource control:

```text
Memory usage ↑
 ↓
Host pressure
 ↓
Other workloads affected
```

Limits help isolate workloads.

---

# 80. Docker Security Basics

Important practices:

```text
Use trusted base images
Pin versions where appropriate
Run as non-root where practical
Keep images updated
Minimize image contents
Don't bake secrets into images
Scan dependencies/images
Use least privilege
```

---

# 81. Running as Root

Many containers historically run as root by default.

For production applications, consider:

```dockerfile
USER appuser
```

where practical.

Running as a non-root user reduces the impact of certain container compromises.

---

# 82. Minimal Base Images

Smaller images can provide:

```text
Lower storage
Faster transfer
Smaller attack surface
```

But don't optimize size blindly.

You also need:

```text
Compatibility
Debuggability
Security updates
Operational support
```

---

# 83. Docker Image Tags

Example:

```text
ecommerce-api:1.0
```

Avoid relying entirely on:

```text
latest
```

because it can change over time.

For reproducible production deployments, versioning and/or immutable digests are preferable.

---

# 84. Docker Layer Caching

If Dockerfile:

```dockerfile
FROM ...
COPY dependencies ...
RUN install ...
COPY source ...
```

changes only source files:

```text
Earlier layers
```

may be reused.

This can make builds much faster.

---

# 85. Why Dockerfile Order Matters

Bad:

```dockerfile
COPY . .
RUN mvn dependency:go-offline
```

Every source change can invalidate the layer.

A better multi-stage/Maven build can separate:

```text
Dependency files
```

from:

```text
Source code
```

to improve caching.

---

# 86. Multi-Stage Builds

Multi-stage builds use multiple `FROM` stages.

Example concept:

```text
Build stage
   ↓
Compile application
   ↓
Runtime stage
   ↓
Copy JAR
```

This allows the final image to contain only what is required to run the application.

---

# 87. Spring Boot Multi-Stage Example

Conceptually:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /app
COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre

WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

The final image doesn't need Maven.

---

# 88. Why Multi-Stage Builds?

Benefits:

```text
Smaller runtime image
Less unnecessary tooling
Smaller attack surface
Cleaner deployment artifact
```

---

# 89. Dockerfile vs Image

```text
Dockerfile
→ Instructions

Image
→ Built artifact

Container
→ Running instance
```

This distinction is essential.

---

# 90. Docker Compose Preview

Docker Compose is used to define and run multi-container applications.

Example:

```text
Spring Boot
     |
     +--- MySQL
     |
     +--- Redis
```

Instead of manually running three containers, Compose can define the entire application stack.

We'll cover it in a later file.

---

# 91. Backend Example

Your e-commerce backend could eventually look like:

```text
docker-compose
      |
      +-- spring-app
      |
      +-- mysql
      |
      +-- redis
```

Spring Boot:

```text
8080
```

MySQL:

```text
3306
```

Redis:

```text
6379
```

These services can communicate over a Docker network.

---

# 92. Docker and CI/CD

A typical pipeline:

```text
Developer
 ↓
Git push
 ↓
CI
 ↓
Build
 ↓
Test
 ↓
Docker image
 ↓
Image scan
 ↓
Push registry
 ↓
Deploy
```

Docker gives CI/CD a consistent application artifact.

---

# 93. Docker Image Lifecycle

Think:

```text
Source code
 ↓
Build
 ↓
Image
 ↓
Tag
 ↓
Push
 ↓
Registry
 ↓
Pull
 ↓
Run
 ↓
Monitor
 ↓
Replace/update
```

---

# 94. Immutable Deployment

Instead of changing a production container manually:

```text
SSH
 ↓
Edit files
 ↓
Restart
```

prefer:

```text
Build new image
 ↓
Tag version
 ↓
Deploy new container
 ↓
Replace old version
```

This makes deployments more reproducible.

---

# 95. Container vs Process

A container is not a tiny VM.

It is essentially:

```text
Application process
+
isolated environment
+
resource controls
+
filesystem/network configuration
```

This is an important conceptual distinction.

---

# 96. Docker Interview Question

### What problem does Docker solve?

Answer:

> "Docker packages an application and its required environment into a portable image so it can run consistently across different environments. It also provides process, filesystem and network isolation through containers."

---

# 97. Docker Interview Question

### Image vs Container?

Answer:

> "An image is an immutable template used to create containers. A container is a running or stopped instance of that image with its own runtime state."

---

# 98. Docker Interview Question

### Container vs VM?

Answer:

> "A VM generally includes a complete guest operating system, while containers share the host kernel and isolate application processes. Containers are therefore typically lighter and faster to start."

---

# 99. Docker Interview Question

### What happens when a container stops?

Answer:

> "The container's process stops, but the container object can still exist in an exited state. It can be started again unless it has been removed."

---

# 100. Docker Interview Question

### What happens to data when a container is removed?

Answer:

> "Data stored only in the container's writable layer can be lost. Persistent application data should be stored in volumes, bind mounts or external durable storage."

---

# 101. Docker Interview Question

### Why use Docker volumes?

Answer:

> "Volumes provide persistent storage that is independent of a container's lifecycle, which is especially important for stateful services such as MySQL."

---

# 102. Docker Interview Question

### What does `-p 8080:8080` mean?

Answer:

> "It maps port 8080 on the host to port 8080 inside the container."

---

# 103. Docker Interview Question

### Why doesn't localhost work between containers?

Answer:

> "Inside a container, localhost refers to that same container. Containers should normally communicate through a Docker network using the appropriate service or container hostname."

---

# 104. Docker Interview Question

### What is Dockerfile?

Answer:

> "A Dockerfile is a set of instructions used to build a Docker image."

---

# 105. Docker Interview Question

### ENTRYPOINT vs CMD?

Answer:

> "ENTRYPOINT defines the main executable for the container, while CMD provides default command or arguments that can be overridden depending on how the container is started."

---

# 106. Docker Interview Question

### Why use multi-stage builds?

Answer:

> "They separate build-time dependencies from runtime dependencies, allowing the final image to contain only what is needed to run the application."

---

# 107. Docker Interview Question

### What is Docker Compose?

Answer:

> "Docker Compose lets us define and run a multi-container application using a declarative configuration file. It's useful for local development and testing stacks such as Spring Boot, MySQL and Redis."

---

# 108. Docker Interview Question

### How do you debug a crashing container?

Answer:

> "I'd first check `docker ps -a` and `docker logs`. Then I'd inspect the container configuration and environment using `docker inspect`, and if it stays running I'd use `docker exec` to inspect the filesystem and processes."

---

# 109. Docker Interview Question

### Why shouldn't secrets be inside an image?

Answer:

> "Images can be stored, shared and inspected. Baking secrets into them risks exposing credentials and makes rotation difficult. Secrets should be injected through an appropriate secret-management mechanism."

---

# 110. Docker Interview Question

### Why use a non-root user?

Answer:

> "Running the application as a non-root user follows least privilege and reduces the potential impact if the application or container is compromised."

---

# 111. Docker Interview Question

### What is a Docker registry?

Answer:

> "A registry stores and distributes Docker images. Docker Hub is one example, while organizations can also use private registries such as Amazon ECR, Azure Container Registry or GitHub Container Registry."

---

# 112. Practical Exercise

Start with Redis:

```bash
docker pull redis
```

Run it:

```bash
docker run -d --name my-redis -p 6379:6379 redis
```

Check:

```bash
docker ps
```

View logs:

```bash
docker logs my-redis
```

---

# 113. Practical Exercise: MySQL

Later, you can run:

```bash
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=ecommerce \
  -p 3306:3306 \
  mysql
```

For real projects, use a proper secret-management approach rather than committing or casually sharing passwords.

---

# 114. Practical Exercise: Spring Boot

Build your JAR:

```bash
mvn clean package
```

Then create:

```text
Dockerfile
```

with:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build:

```bash
docker build -t ecommerce-api:1.0 .
```

Run:

```bash
docker run -d \
  --name ecommerce-api \
  -p 8080:8080 \
  ecommerce-api:1.0
```

---

# 115. Important Spring Boot Docker Detail

If Spring Boot runs inside Docker and needs MySQL in another container:

Don't configure:

```text
localhost:3306
```

Instead, once both services share a Docker network:

```text
mysql:3306
```

Similarly for Redis:

```text
redis:6379
```

---

# 116. Docker Network Example

Create:

```bash
docker network create backend-net
```

Run MySQL:

```bash
docker run -d \
  --name mysql-db \
  --network backend-net \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=ecommerce \
  mysql
```

Run Redis:

```bash
docker run -d \
  --name redis \
  --network backend-net \
  redis
```

Run Spring Boot:

```bash
docker run -d \
  --name ecommerce-api \
  --network backend-net \
  -p 8080:8080 \
  ecommerce-api:1.0
```

Now the application can use:

```text
mysql:3306
redis:6379
```

---

# 117. Docker Networking Mental Model

```text
             backend-net
                  |
       +----------+----------+
       |          |          |
   Spring Boot  MySQL      Redis
       |          |          |
      8080       3306       6379
```

Container names can act as service hostnames on a user-defined Docker network.

---

# 118. First Docker Checklist

You should now understand:

```text
□ Docker
□ Container
□ Image
□ Dockerfile
□ Docker Engine
□ Docker CLI
□ Image layers
□ Container lifecycle
□ docker run
□ docker ps
□ docker logs
□ docker exec
□ docker stop
□ docker rm
□ docker pull
□ Docker registry
□ Port mapping
□ Environment variables
□ Volumes
□ Bind mounts
□ Docker networks
□ .dockerignore
□ ENTRYPOINT
□ CMD
□ Health checks
□ Restart policies
□ Resource limits
□ Multi-stage builds
□ Docker Compose concept
□ Docker + Spring Boot
```

---

# 119. What Comes Next

```text
File 02 → Docker Images, Dockerfile & Build Optimization
```

Next we'll go deeper into:

```text
Dockerfile instructions
FROM
RUN
COPY
ADD
WORKDIR
ENV
ARG
EXPOSE
USER
ENTRYPOINT
CMD
Layer caching
Build context
.dockerignore
Multi-stage builds
Spring Boot Dockerfile
JAR layering
Build optimization
Image size optimization
Security
Docker interview questions
```

Key takeaway:

> **Docker's core mental model is simple: a Dockerfile builds an image, and an image runs as a container. Containers are ephemeral, so persistent data belongs in volumes or external storage. For Spring Boot, the next important step is learning how to build small, secure, reproducible images and connect the application to MySQL and Redis through Docker networks.**

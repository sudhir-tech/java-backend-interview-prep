# Docker — File 05: Dockerizing a Spring Boot Application

This file takes the Docker concepts and applies them to a real Spring Boot backend.

Goal:

```text
Spring Boot project
      ↓
Maven build
      ↓
Docker image
      ↓
Container
      ↓
MySQL + Redis
```

The examples are written with a typical Java 21 + Spring Boot + Maven backend in mind.

---

# 1. What We Are Trying to Build

Our local backend environment:

```text
                    Browser / Postman
                           |
                           ↓
                    Spring Boot API
                       :8080
                           |
                     Docker network
                    /              \
                   ↓                ↓
                MySQL             Redis
                :3306             :6379
```

MySQL stores durable business data.

Redis handles things such as:

```text
Caching
Sessions
Rate limiting
Temporary state
```

depending on the application's design.

---

# 2. Starting Spring Boot Project

Typical structure:

```text
ecommercebackend/
│
├── src/
│   ├── main/
│   └── test/
│
├── pom.xml
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
└── .gitignore
```

The Docker files live alongside the application project.

---

# 3. Build the Application First

Before Docker:

```bash
mvn clean package
```

This should produce something like:

```text
target/
└── ecommercebackend-0.0.1-SNAPSHOT.jar
```

Make sure the application builds successfully before debugging Docker.

---

# 4. Simple Dockerfile

The easiest approach is:

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
docker run -p 8080:8080 ecommerce-api:1.0
```

---

# 5. What Happens During the Build?

```text
Dockerfile
    ↓
FROM Java runtime
    ↓
WORKDIR /app
    ↓
COPY JAR
    ↓
Create image
```

The result:

```text
ecommerce-api:1.0
```

---

# 6. What Happens During Runtime?

```text
docker run
    ↓
Create container
    ↓
Start:
java -jar app.jar
    ↓
Spring Boot
    ↓
Tomcat
    ↓
Port 8080
```

Then:

```text
localhost:8080
```

can reach the application if the port is published.

---

# 7. Check the Container

```bash
docker ps
```

You should see:

```text
ecommerce-api
```

If it exits:

```bash
docker ps -a
```

Then:

```bash
docker logs ecommerce-api
```

---

# 8. Spring Boot Port

Inside the container:

```text
8080
```

Host mapping:

```bash
-p 8080:8080
```

means:

```text
Host 8080
   ↓
Container 8080
```

---

# 9. Change Host Port

You don't have to use 8080 on the host.

Example:

```bash
docker run -p 9090:8080 ecommerce-api:1.0
```

Now:

```text
localhost:9090
```

maps to:

```text
container:8080
```

---

# 10. Container Configuration

Don't hardcode environment-specific settings into the image.

Instead:

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

spring.data.redis.host=${REDIS_HOST}
spring.data.redis.port=${REDIS_PORT}
```

This lets Docker supply configuration at runtime.

---

# 11. Docker Environment Variables

Example:

```bash
docker run \
  -e DB_URL="jdbc:mysql://mysql:3306/ecommerce" \
  -e DB_USERNAME="root" \
  -e DB_PASSWORD="root" \
  -e REDIS_HOST="redis" \
  -e REDIS_PORT="6379" \
  ecommerce-api:1.0
```

For local learning this is fine.

For production credentials:

```text
Use a secret-management solution.
```

---

# 12. `application.properties`

A Docker-friendly configuration can look like:

```properties
spring.application.name=ecommercebackend

spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

spring.data.redis.host=${REDIS_HOST:localhost}
spring.data.redis.port=${REDIS_PORT:6379}
```

The defaults can be useful for local non-Docker execution.

---

# 13. Docker Profile

Another approach:

```text
application.properties
application-docker.properties
application-prod.properties
```

Activate Docker:

```bash
-e SPRING_PROFILES_ACTIVE=docker
```

Then Docker-specific configuration can contain:

```properties
spring.datasource.url=jdbc:mysql://mysql:3306/ecommerce
spring.data.redis.host=redis
```

---

# 14. Which Approach Should You Use?

Both are valid.

### Environment variables

Good for:

```text
Deployment configuration
Secrets
Containerized environments
CI/CD
```

### Spring profiles

Good for:

```text
Environment-specific configuration
Developer convenience
Structured configuration
```

In production, sensitive values should still come from a secure secret mechanism.

---

# 15. Dockerfile with Non-Root User

A stronger runtime image:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/*.jar app.jar

RUN useradd --system appuser
USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

The exact user-creation command depends on the base image.

---

# 16. Why Non-Root?

If the application process is compromised:

```text
root
```

has much more authority than:

```text
non-root user
```

Least privilege reduces the potential impact.

---

# 17. Multi-Stage Spring Boot Build

Instead of building the JAR outside Docker:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=build /app/target/*.jar app.jar

RUN useradd --system appuser
USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

This is a useful production-style starting point.

---

# 18. Multi-Stage Flow

```text
             Docker build
                  |
          +-------+-------+
          |               |
       Build stage     Runtime stage
          |               |
      Maven + JDK      Java runtime
          |               |
       compile            JAR
          |               |
          +------->-------+
                    |
                 Image
```

The final runtime image doesn't need:

```text
Maven
Source code
```

---

# 19. Build Cache Improvement

The simple example:

```dockerfile
COPY pom.xml .
COPY src ./src
RUN mvn clean package
```

can be improved depending on the Maven project.

A common pattern is:

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn clean package -DskipTests
```

The dependency layer can then be reused when only source code changes.

---

# 20. Important Maven Caveat

Some projects have:

```text
Parent POM
Maven wrapper
Multiple modules
Private repositories
Generated sources
Additional build files
```

In those cases, the Dockerfile needs to copy the relevant files before dependency resolution.

Don't blindly assume:

```text
pom.xml + src
```

is sufficient for every project.

---

# 21. `.dockerignore`

Example:

```text
.git
.idea
.vscode
*.log
.env
```

If building the JAR outside Docker:

```text
Don't ignore target/
```

because the Dockerfile needs:

```text
target/*.jar
```

---

# 22. If Using Multi-Stage Build

If Maven builds the JAR inside Docker:

```text
target/
```

doesn't need to come from the host build context.

Therefore the Dockerfile strategy can be:

```text
COPY pom.xml
COPY source
RUN Maven build
```

and the final image gets the JAR from:

```text
build stage
```

---

# 23. Image Naming

Use a meaningful name:

```bash
docker build -t ecommerce-api:1.0 .
```

Think:

```text
repository:tag
```

Here:

```text
ecommerce-api
→ repository

1.0
→ tag
```

---

# 24. Versioning

Better than relying only on:

```text
latest
```

use:

```text
1.0
1.1
1.2
```

or:

```text
git-a1b2c3d
```

or a release version.

This makes rollback easier.

---

# 25. Run the Image

```bash
docker run -d \
  --name ecommerce-api \
  -p 8080:8080 \
  ecommerce-api:1.0
```

Check:

```bash
docker ps
```

Logs:

```bash
docker logs -f ecommerce-api
```

---

# 26. Add Docker Network

Create:

```bash
docker network create backend-net
```

Run MySQL:

```bash
docker run -d \
  --name mysql \
  --network backend-net \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=ecommerce \
  -v mysql-data:/var/lib/mysql \
  mysql:8.4
```

Run Redis:

```bash
docker run -d \
  --name redis \
  --network backend-net \
  redis:7
```

Run Spring Boot:

```bash
docker run -d \
  --name ecommerce-api \
  --network backend-net \
  -p 8080:8080 \
  -e DB_URL="jdbc:mysql://mysql:3306/ecommerce" \
  -e DB_USERNAME="root" \
  -e DB_PASSWORD="root" \
  -e REDIS_HOST="redis" \
  -e REDIS_PORT="6379" \
  ecommerce-api:1.0
```

---

# 27. The Result

```text
backend-net
     |
 +---+-----------+
 |       |       |
App    MySQL   Redis
 |       |       |
8080   3306     6379
```

The host only needs:

```text
localhost:8080
```

The application internally uses:

```text
mysql:3306
redis:6379
```

---

# 28. Why MySQL Uses `mysql`

Because:

```text
mysql
```

is the Compose/container network hostname.

Do NOT use:

```text
localhost
```

from the Spring Boot container.

---

# 29. Why Redis Uses `redis`

Same idea:

```text
redis
```

resolves to the Redis container on the Docker network.

So:

```properties
spring.data.redis.host=redis
```

---

# 30. Spring Boot + MySQL Connection Flow

```text
HTTP Request
     ↓
Spring Controller
     ↓
Service
     ↓
Repository
     ↓
HikariCP
     ↓
mysql:3306
     ↓
MySQL container
```

Docker is only providing the runtime/network environment.

Your Spring Data/JPA code remains the same.

---

# 31. Spring Boot + Redis Connection Flow

```text
HTTP Request
     ↓
Service
     ↓
RedisTemplate / Spring Cache
     ↓
redis:6379
     ↓
Redis container
```

Again, the application code doesn't need to know the container IP.

---

# 32. HikariCP in Docker

Spring Boot commonly uses:

```text
HikariCP
```

for JDBC connection pooling.

Containerized applications still need sensible:

```text
maximum pool size
connection timeout
idle timeout
```

configuration.

Don't automatically make the pool huge.

---

# 33. Connection Pool + Docker

Suppose:

```text
10 application containers
```

and each has:

```text
50 DB connections
```

Potentially:

```text
500 DB connections
```

to MySQL.

This can overload the database.

System design must consider:

```text
Container count
×
Pool size
```

---

# 34. Redis Connections

Similarly, don't create excessive Redis connections.

Use:

```text
Appropriate connection pooling
Timeouts
Connection reuse
```

according to the client/library.

---

# 35. Spring Boot Logging

Container-friendly approach:

```text
Spring Boot
    ↓
stdout/stderr
    ↓
Docker logs
```

Run:

```bash
docker logs -f ecommerce-api
```

You should see:

```text
Spring Boot startup
Tomcat
Hibernate
HikariCP
Application ready
```

---

# 36. Don't Depend on Log Files in Container

Avoid relying on:

```text
/app/application.log
```

as your only production logging mechanism.

Containers can be replaced.

Prefer:

```text
stdout/stderr
+
centralized logging
```

---

# 37. Actuator Health

If Spring Boot Actuator is installed:

```text
/actuator/health
```

can provide application health information.

Example:

```text
http://localhost:8080/actuator/health
```

This can be used by:

```text
Docker
Load balancer
Orchestrator
Monitoring system
```

depending on the deployment environment.

---

# 38. Healthcheck Consideration

A Dockerfile could define a healthcheck if the image contains a suitable tool.

But a minimal image may not contain:

```text
curl
```

So don't blindly copy:

```dockerfile
HEALTHCHECK CMD curl ...
```

without checking the image.

---

# 39. Graceful Shutdown

When Docker stops the container:

```text
SIGTERM
   ↓
Spring Boot
   ↓
Graceful shutdown
   ↓
Connections closed
   ↓
Process exits
```

This is preferable to abrupt termination.

---

# 40. Spring Boot Container Memory

If container memory is:

```text
512 MB
```

don't assume:

```text
Java heap = 512 MB
```

The JVM also needs memory for:

```text
Metaspace
Threads
Native allocations
Code cache
GC
```

Leave headroom.

---

# 41. Resource Limits

Example:

```bash
docker run \
  --memory=512m \
  --cpus=1 \
  ecommerce-api:1.0
```

This protects the host from an uncontrolled application.

---

# 42. What Happens on OOM?

If the container exceeds its memory limit:

```text
OOM
 ↓
Container/process may be killed
```

Investigate:

```text
Heap
Traffic
Memory leak
GC
Container limit
JVM configuration
```

---

# 43. Docker Compose Version

Modern Docker uses:

```bash
docker compose
```

rather than requiring the old standalone:

```bash
docker-compose
```

For current setups, prefer:

```bash
docker compose up
```

---

# 44. Using Compose for Our Project

Our project structure:

```text
ecommercebackend/
│
├── src/
├── pom.xml
├── Dockerfile
├── docker-compose.yml
└── .dockerignore
```

Then:

```bash
docker compose up --build
```

starts:

```text
Spring Boot
MySQL
Redis
```

---

# 45. Compose Environment

Example:

```yaml
services:

  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      DB_URL: jdbc:mysql://mysql:3306/ecommerce
      DB_USERNAME: root
      DB_PASSWORD: root
      REDIS_HOST: redis
      REDIS_PORT: 6379

  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: ecommerce
    volumes:
      - mysql-data:/var/lib/mysql

  redis:
    image: redis:7

volumes:
  mysql-data:
```

---

# 46. Application Startup

Potential flow:

```text
docker compose up
        ↓
Network created
        ↓
MySQL starts
        ↓
Redis starts
        ↓
App starts
        ↓
Spring Boot connects
        ↓
API ready
```

Remember:

```text
starts
≠
ready
```

Use healthchecks and application retry behavior when needed.

---

# 47. Database Migrations

If your project uses:

```text
Flyway
Liquibase
```

migrations can run during application startup.

This is usually better than manually editing the production database.

Typical flow:

```text
App starts
 ↓
Migration check
 ↓
Schema update
 ↓
Application ready
```

Migration strategy should still be designed carefully for production deployments.

---

# 48. Docker and JPA

Hibernate/JPA does not fundamentally change because the app runs in Docker.

The important changes are generally:

```text
Database hostname
Configuration
Network
Environment
```

For example:

```text
localhost:3306
```

becomes:

```text
mysql:3306
```

inside the Docker network.

---

# 49. Docker and Redis

Same principle:

```text
localhost:6379
```

becomes:

```text
redis:6379
```

when Redis is another container.

---

# 50. Debugging Spring Boot Container

Use:

```bash
docker compose ps
```

Then:

```bash
docker compose logs app
```

Then:

```bash
docker compose logs mysql
```

Then:

```bash
docker compose logs redis
```

Look for:

```text
Connection refused
Communications link failure
Authentication failure
Unknown host
Port errors
Application startup exceptions
```

---

# 51. `UnknownHostException`

Example:

```text
UnknownHostException: mysql
```

Likely investigate:

```text
Network
Service name
Compose configuration
Container membership
```

---

# 52. `Connection refused`

Example:

```text
Connection refused: mysql:3306
```

Possible causes:

```text
MySQL not running
MySQL not ready
Wrong port
Network issue
```

---

# 53. `Communications link failure`

For MySQL:

```text
Check:
DB URL
mysql hostname
port
container network
MySQL readiness
credentials
```

A Docker networking issue is one possibility, but don't assume it is always Docker.

---

# 54. Redis `Connection refused`

Check:

```text
redis container
network
port 6379
Redis logs
Spring configuration
```

---

# 55. Container Is Running but API Is Inaccessible

Check:

```text
1. docker ps
2. docker logs app
3. Application port
4. Docker port mapping
5. Spring server.port
6. Host port conflict
```

Example:

```yaml
ports:
  - "8080:8080"
```

---

# 56. Spring `server.port`

If configured:

```properties
server.port=9090
```

then the container listens on:

```text
9090
```

Your mapping must match:

```yaml
ports:
  - "8080:9090"
```

if you want the host to use:

```text
localhost:8080
```

---

# 57. The Port Rule

Always think:

```text
HOST : CONTAINER
```

Example:

```text
8080:9090
```

means:

```text
localhost:8080
       ↓
container:9090
```

---

# 58. Build Failure vs Runtime Failure

Important distinction.

### Build failure

```text
docker build
```

fails.

Investigate:

```text
Dockerfile
Maven
Dependencies
Build context
```

### Runtime failure

```text
docker run
```

starts but application fails.

Investigate:

```text
Environment
Network
Database
Redis
Application configuration
```

---

# 59. Image Build Failure

Example:

```text
COPY target/*.jar app.jar
```

fails.

Likely:

```text
JAR wasn't built
Wrong filename
target ignored/missing
Wrong build context
```

Run:

```bash
mvn clean package
```

and inspect:

```text
target/
```

---

# 60. Dockerfile Path Problems

Suppose:

```text
Dockerfile
target/app.jar
```

Then:

```dockerfile
COPY target/app.jar app.jar
```

works if:

```text
target/
```

is inside the build context.

Don't use paths outside the build context.

---

# 61. Build Context Example

Command:

```bash
docker build -t app .
```

means:

```text
.
```

is the build context.

If the JAR is outside that directory:

```text
COPY
```

cannot simply reach into arbitrary parent directories.

---

# 62. Production Dockerfile Goal

A good runtime image should ideally contain:

```text
Java runtime
Application JAR
Required runtime configuration
```

Not:

```text
Git
Maven
Source code
IDE files
Temporary logs
Development credentials
```

---

# 63. Security Checklist

For the Spring Boot image:

```text
□ Trusted base image
□ Maintained Java version
□ Image scanning
□ Dependency scanning
□ Non-root user
□ No secrets in image
□ Minimal packages
□ Versioned image
□ Rebuild regularly
□ Proper runtime configuration
```

---

# 64. CI/CD Flow

A realistic pipeline:

```text
Git push
   ↓
Unit tests
   ↓
Maven build
   ↓
Docker build
   ↓
Image scan
   ↓
Integration tests
   ↓
Push image
   ↓
Deploy
```

---

# 65. Why Build the Image in CI?

Because CI provides:

```text
Consistent build environment
Automated tests
Security scanning
Versioned artifacts
```

The image can then be deployed without rebuilding it on the server.

---

# 66. Build Once, Deploy Many

Strong deployment principle:

```text
Build image once
        ↓
Test image
        ↓
Same image
        ↓
Dev
        ↓
Test
        ↓
Production
```

Only configuration should normally change between environments.

---

# 67. Image Registry

After CI builds:

```text
ecommerce-api:1.2
```

it can push to:

```text
Docker Hub
ECR
ACR
GCR/Artifact Registry
GHCR
Private registry
```

Then deployment systems pull the exact image.

---

# 68. Rollback

Suppose:

```text
1.1 → stable
1.2 → broken
```

Rollback:

```text
Deploy 1.1
```

This is much cleaner than rebuilding old source manually.

---

# 69. Docker and Kubernetes

Later, if moving to Kubernetes:

```text
Dockerfile
   ↓
Image
   ↓
Registry
   ↓
Kubernetes
```

The Docker image remains the application artifact.

Kubernetes handles:

```text
Scheduling
Scaling
Service discovery
Health
Rolling deployment
```

---

# 70. Interview Question

### How would you Dockerize a Spring Boot application?

Answer:

> "I'd build the application with Maven, create a multi-stage Dockerfile with a JDK/Maven builder and a smaller Java runtime image, copy only the JAR into the runtime stage, run as a non-root user and expose the application port. Configuration such as database and Redis endpoints would be injected at runtime."

---

# 71. Interview Question

### How does your Spring Boot container connect to MySQL?

Answer:

> "If both containers are on the same Docker network, Spring Boot uses the MySQL service name as the hostname, for example `mysql:3306`. I wouldn't use localhost because localhost refers to the application container itself."

---

# 72. Interview Question

### How do you connect Spring Boot to Redis?

Answer:

> "I configure the Redis host as the Docker service name, such as `redis`, and use port 6379. The containers communicate over the Docker network."

---

# 73. Interview Question

### How would you handle database credentials?

Answer:

> "I wouldn't bake them into the image. I'd inject them at runtime using environment variables or, preferably in production, a proper secret-management solution."

---

# 74. Interview Question

### Why use multi-stage Docker builds?

Answer:

> "The builder stage contains Maven and the JDK required to compile the application. The runtime stage only contains the JAR and runtime dependencies, making the final image smaller and reducing unnecessary attack surface."

---

# 75. Interview Question

### How do you debug a Spring Boot container?

Answer:

> "I'd first check the container status and logs. Then I'd verify environment variables, port mappings, Docker network connectivity and dependencies such as MySQL and Redis. For a database error I'd specifically check the hostname, port, readiness and credentials."

---

# 76. Interview Scenario

### It works locally but fails in Docker.

Think:

```text
Local:
localhost
```

Docker:

```text
mysql
redis
```

Check:

```text
Environment variables
Profiles
Network
Ports
Filesystem paths
Case sensitivity
Java version
Configuration
```

---

# 77. Interview Scenario

### Spring Boot starts before MySQL.

Don't solve it with:

```text
sleep 60
```

Prefer:

```text
Healthcheck
+
depends_on condition
+
Application retry
```

The application should tolerate temporary dependency failures.

---

# 78. Interview Scenario

### Container has no `curl`.

Don't immediately install it just for debugging.

Use:

```text
Application logs
Docker inspect
Actuator
Temporary diagnostic container
```

or another tool appropriate to your environment.

Minimal images intentionally contain fewer utilities.

---

# 79. Interview Scenario

### MySQL volume is huge.

Investigate:

```text
Database growth
Logs
Binary logs
Temporary files
Unused data
Volume mount
Backup strategy
```

Don't simply delete the volume.

---

# 80. Interview Scenario

### Application container keeps restarting.

Check:

```bash
docker ps -a
docker logs ecommerce-api
```

Then investigate:

```text
Application exception
Environment
Database
Redis
Memory
Healthcheck
Entrypoint
```

---

# 81. Interview Scenario

### Application can reach MySQL but not Redis.

Compare:

```text
mysql hostname
redis hostname
```

Check:

```text
Network membership
Redis status
Redis port
Configuration
```

The fact that MySQL works doesn't prove Redis is correctly configured.

---

# 82. Interview Scenario

### Host can access API, but another container cannot.

Check:

```text
Container network
Service hostname
Container port
Application binding
```

Don't confuse:

```text
host published port
```

with:

```text
container service port
```

---

# 83. Interview Scenario

### Need to run two versions of the application locally.

Use different:

```text
Container names
Host ports
Compose projects
```

Example:

```text
app-v1 → localhost:8080
app-v2 → localhost:8081
```

Both can still listen on:

```text
8080
```

inside their own containers.

---

# 84. Practical Exercise

For your project:

### Step 1

Build:

```bash
mvn clean package
```

### Step 2

Create:

```text
Dockerfile
```

### Step 3

Build:

```bash
docker build -t ecommerce-api:1.0 .
```

### Step 4

Create network:

```bash
docker network create backend-net
```

### Step 5

Start MySQL and Redis.

### Step 6

Start Spring Boot.

### Step 7

Check:

```bash
docker ps
```

### Step 8

Check logs:

```bash
docker logs ecommerce-api
```

### Step 9

Call an API:

```text
http://localhost:8080
```

---

# 85. What You Should Be Able to Explain

At this point, you should be able to explain:

```text
Why Docker?
Why container?
Why image?
How Dockerfile works?
How Spring Boot becomes an image?
How container starts?
How ports work?
How environment variables work?
How Spring Boot reaches MySQL?
How Spring Boot reaches Redis?
Why localhost doesn't work?
Why volumes are needed?
How Compose connects everything?
How to debug failures?
```

---

# 86. One-Minute Dockerized Backend Explanation

If an interviewer asks:

> "How would you containerize your Spring Boot backend?"

A natural answer:

> "I'd package the Spring Boot application into a Docker image using a multi-stage Dockerfile. The build stage would use Maven and the JDK, and the final stage would use a smaller Java runtime image. I'd run the application as a non-root user and inject environment-specific configuration at runtime. For local development, I'd use Docker Compose to run the API along with MySQL and Redis on a shared network. The application would connect to them using the service names `mysql` and `redis`, and MySQL data would be stored in a persistent volume."

---

# 87. Quick Revision

```text
Dockerfile
    ↓
Build
    ↓
Image
    ↓
Container
```

For your backend:

```text
Spring Boot
    ↓
Docker image
    ↓
Compose
    ↓
+---------+---------+
|         |         |
MySQL    Redis     API
```

Inside Docker:

```text
API → mysql:3306
API → redis:6379
```

From your Mac:

```text
Mac → localhost:8080 → API
```

Persistent database:

```text
MySQL → mysql-data
```

---

# 88. File 05 Checklist

```text
□ Build Spring Boot JAR
□ Basic Dockerfile
□ Multi-stage Dockerfile
□ Docker build
□ Docker run
□ Container logs
□ Port mapping
□ Environment variables
□ Spring profiles
□ Non-root user
□ Maven build caching
□ .dockerignore
□ Docker network
□ MySQL connection
□ Redis connection
□ HikariCP considerations
□ Healthcheck
□ Graceful shutdown
□ Container memory
□ OOMKilled
□ Docker Compose
□ Database persistence
□ CI/CD
□ Registry
□ Versioning
□ Rollback
□ Debugging
□ Production security
□ Interview answers
```

---

# 89. What Comes Next

```text
File 06 → Docker Security, Production Practices & CI/CD
```

We'll cover:

```text
Image security
Container security
Non-root containers
Secrets
Capabilities
Read-only filesystem
Resource limits
Image scanning
SBOM
Supply-chain security
Docker Content Trust concepts
CI/CD image pipeline
Registries
Tagging
Immutable images
Vulnerability handling
Production checklist
Docker interview scenarios
```

**Key takeaway:**

> **Dockerizing Spring Boot is not just putting the JAR into a container. A good setup separates build-time dependencies from runtime, injects configuration at runtime, uses Docker networking correctly, persists state outside the container and produces a versioned image that can move through CI/CD without being rebuilt for every environment.**

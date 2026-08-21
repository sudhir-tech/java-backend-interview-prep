# Docker — File 02: Docker Images, Dockerfile & Build Optimization

This file goes deeper into how Docker images are built and how to write better Dockerfiles for Java/Spring Boot applications.

---

# 1. Docker Build Flow

The basic flow is:

```text
Dockerfile
    ↓
docker build
    ↓
Build context
    ↓
Image layers
    ↓
Docker image
```

Then:

```text
Docker image
    ↓
docker run
    ↓
Container
```

---

# 2. Dockerfile

A Dockerfile is a text file containing instructions for building an image.

Example:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Each instruction can create or influence an image layer.

---

# 3. FROM

Example:

```dockerfile
FROM eclipse-temurin:21-jre
```

`FROM` defines the base image.

For a Spring Boot application, the base image could provide:

```text
Linux filesystem
Java runtime
Required runtime libraries
```

---

# 4. Choosing a Base Image

Common Java choices include:

```text
JDK image
JRE-oriented image
Distroless image
Minimal Linux image
```

For a runtime-only Spring Boot application, you generally don't need the full JDK in the final image.

---

# 5. JDK vs JRE Runtime Image

Build stage may need:

```text
JDK
+
Maven
```

Runtime stage may only need:

```text
Java runtime
```

Therefore multi-stage builds can reduce the final image.

---

# 6. RUN

`RUN` executes a command during image construction.

Example:

```dockerfile
RUN apt-get update
```

or:

```dockerfile
RUN mvn clean package
```

The command executes while building the image, not when the container starts.

---

# 7. RUN vs CMD

Important:

```text
RUN
→ Build time

CMD
→ Default runtime command
```

Example:

```dockerfile
RUN mvn clean package
```

happens during:

```bash
docker build
```

while:

```dockerfile
CMD ["java", "-jar", "app.jar"]
```

is used when the container starts.

---

# 8. COPY

Example:

```dockerfile
COPY target/app.jar app.jar
```

copies files from the build context into the image.

Prefer `COPY` when you simply need to copy local files.

---

# 9. ADD

`ADD` can do more than `COPY`, including some archive handling and URL-related behavior.

For normal application files:

```text
Prefer COPY
```

because it makes the Dockerfile's intent clearer.

---

# 10. COPY vs ADD

Interview answer:

> "COPY is usually preferred for straightforward file copying. ADD has additional behavior such as archive extraction, so I use it only when that behavior is intentionally needed."

---

# 11. WORKDIR

Example:

```dockerfile
WORKDIR /app
```

This sets the working directory for subsequent instructions and the container process.

Instead of:

```text
/
```

the application operates from:

```text
/app
```

---

# 12. ENV

`ENV` defines environment variables in the image/container environment.

Example:

```dockerfile
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75"
```

Be careful:

> Don't put secrets into `ENV` in the Dockerfile.

Image metadata and layers can expose them.

---

# 13. ARG

`ARG` defines build-time variables.

Example:

```dockerfile
ARG JAR_FILE=target/app.jar
COPY ${JAR_FILE} app.jar
```

The value is available during image construction.

---

# 14. ARG vs ENV

Important:

```text
ARG
→ Build-time

ENV
→ Available in image/container environment
```

Example:

```dockerfile
ARG VERSION=1.0
ENV APP_ENV=prod
```

---

# 15. ARG Is Not a Secret Store

Bad:

```dockerfile
ARG DB_PASSWORD
```

and passing a real production password during build.

Build arguments can appear in build metadata/history depending on how they are used.

Use proper secret mechanisms for sensitive build-time information.

---

# 16. EXPOSE

Example:

```dockerfile
EXPOSE 8080
```

This documents the port the application expects to use.

It does not publish the port to the host.

Host publishing requires:

```bash
docker run -p 8080:8080 image
```

---

# 17. USER

Example:

```dockerfile
USER appuser
```

This tells Docker to run subsequent commands/processes as that user.

For production applications, running as a non-root user is generally preferable when practical.

---

# 18. Creating a Non-Root User

The exact command depends on the base image.

Conceptually:

```dockerfile
RUN useradd --system appuser
USER appuser
```

Minimal images may use different commands or user-management tools.

Always verify the base image.

---

# 19. ENTRYPOINT

Example:

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

This defines the primary executable.

For a Spring Boot container, this is commonly the Java process.

---

# 20. CMD

Example:

```dockerfile
CMD ["--server.port=8080"]
```

can provide default arguments.

With:

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
CMD ["--server.port=8080"]
```

the resulting command is conceptually:

```text
java -jar app.jar --server.port=8080
```

---

# 21. ENTRYPOINT + CMD

Useful mental model:

```text
ENTRYPOINT
→ Main executable

CMD
→ Default arguments
```

Runtime arguments can override CMD in common exec-form configurations.

---

# 22. Exec Form

Preferred:

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

This launches the Java process directly.

It generally provides cleaner signal handling.

---

# 23. Shell Form

Example:

```dockerfile
ENTRYPOINT java -jar app.jar
```

This uses shell processing.

Shell form can be useful when shell features are required, but for a simple Java application the exec form is usually cleaner.

---

# 24. Signal Handling

Docker sends signals to the container's main process.

For example:

```text
SIGTERM
```

can request graceful shutdown.

A Java application running as PID 1 can handle lifecycle signals appropriately.

This matters for:

```text
Graceful shutdown
Rolling deployments
Kubernetes
Container orchestration
```

---

# 25. Docker Image Layers

Suppose:

```dockerfile
FROM java
COPY dependencies
RUN setup
COPY app.jar
```

Each instruction can contribute to image layers.

Docker can reuse unchanged layers.

---

# 26. Build Cache

If a Dockerfile instruction and its inputs have not changed, Docker may reuse the cached result.

Example:

```text
Dependency layer
```

can remain cached even if:

```text
Application source changes
```

This makes builds faster.

---

# 27. Bad Dockerfile Ordering

Example:

```dockerfile
COPY . .

RUN mvn dependency:go-offline

RUN mvn package
```

Changing any source file can invalidate the earlier layer.

Dependency downloads may happen again.

---

# 28. Better Maven Build Strategy

A common optimization is to copy dependency metadata separately:

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package
```

Then:

```text
pom.xml unchanged
```

means dependency-related cache can potentially be reused.

---

# 29. Important Caveat

The exact optimal Maven caching strategy depends on:

```text
Maven version
Plugins
Project structure
Parent POM
Private repositories
Build files
```

Don't blindly copy a caching pattern without testing it.

---

# 30. Build Context

Command:

```bash
docker build -t ecommerce-api:1.0 .
```

The final:

```text
.
```

means the current directory is the build context.

Docker can access files within the context according to the Dockerfile/build rules.

---

# 31. Why Build Context Matters

If your project contains:

```text
.git
node_modules
target
logs
large files
```

and everything is sent as build context:

```text
Build becomes slower
```

Use:

```text
.dockerignore
```

to exclude unnecessary files.

---

# 32. .dockerignore

Example:

```text
.git
.gitignore
.idea
.vscode
*.log
.env
node_modules
```

The exact contents depend on your build strategy.

---

# 33. .dockerignore and target/

If you build the JAR outside Docker:

```bash
mvn clean package
```

then:

```text
target/app.jar
```

is required by:

```dockerfile
COPY target/app.jar app.jar
```

Therefore:

```text
Do NOT ignore target/
```

in that particular Dockerfile strategy.

---

# 34. Multi-Stage Build

Multi-stage Dockerfiles have multiple `FROM` instructions.

Example:

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

---

# 35. Multi-Stage Flow

```text
Build stage
 ├── Maven
 ├── JDK
 ├── Source
 └── Dependencies
        ↓
      JAR
        ↓
Runtime stage
 ├── Java runtime
 └── JAR
```

The final image doesn't need:

```text
Maven
Source code
Build cache
```

---

# 36. Why Multi-Stage Builds Matter

Benefits:

```text
Smaller final image
Reduced attack surface
Less unnecessary tooling
Cleaner runtime environment
Better deployment efficiency
```

---

# 37. Multi-Stage Interview Answer

> "I use a builder stage containing Maven and the JDK to compile the application, then copy only the generated JAR into a smaller runtime image. This keeps build dependencies out of production."

---

# 38. Spring Boot Layered JARs

Spring Boot can create layered JARs.

Conceptually:

```text
Dependencies
Spring Boot loader
Application dependencies
Application code
```

This can improve Docker layer reuse.

---

# 39. Why Layered JARs Help

Application source changes frequently.

Dependencies change less frequently.

If dependencies are in separate image layers:

```text
Source changes
 ↓
Only application layer changes
```

instead of rebuilding everything.

---

# 40. Spring Boot Layered Image

Modern Spring Boot tooling can support efficient container image creation through:

```text
Layered JAR
Buildpacks
Container image tooling
```

The exact approach depends on your Spring Boot version and build setup.

---

# 41. Buildpacks

Spring Boot can create container images using buildpacks without requiring you to manually write a Dockerfile.

For Maven, the concept is:

```bash
mvn spring-boot:build-image
```

This can produce an OCI-compatible container image.

---

# 42. Dockerfile vs Buildpacks

### Dockerfile

```text
Maximum control
Explicit instructions
Custom runtime setup
```

### Buildpacks

```text
Less manual Docker configuration
Automatic runtime detection
Good Spring Boot integration
```

Both are useful.

---

# 43. Image Size

Smaller images generally provide:

```text
Faster pulls
Faster deployment
Less storage
Smaller attack surface
```

But don't make image size the only objective.

---

# 44. How to Reduce Image Size

Use:

```text
Multi-stage builds
Minimal runtime image
.dockerignore
Layer caching
Only required files
No build tools in runtime image
```

---

# 45. Don't Add Unnecessary Packages

Avoid installing tools just because they are convenient:

```text
curl
vim
git
Maven
Node
```

unless the runtime genuinely needs them.

Production runtime images should contain only what is required.

---

# 46. Debugging Minimal Images

A very small image may not contain:

```text
bash
curl
ps
netstat
```

This can make debugging harder.

Use:

```text
Logs
Metrics
Health endpoints
Ephemeral debug containers/tools
```

where supported.

Don't automatically make production images huge just to make interactive debugging convenient.

---

# 47. Image Scanning

Container images can contain:

```text
OS vulnerabilities
Java library vulnerabilities
Other dependency vulnerabilities
```

Use image/dependency scanning in CI/CD.

Typical flow:

```text
Build
 ↓
Scan
 ↓
Test
 ↓
Push
```

---

# 48. Dependency Security

Docker doesn't remove application dependency risk.

For Spring Boot:

```text
Spring dependencies
Jackson
Tomcat
Logging libraries
Database drivers
```

can all have vulnerabilities.

Keep dependencies updated and scan them.

---

# 49. Base Image Updates

Your application can be unchanged while the base image becomes outdated.

Example:

```text
Java 21
```

may still require:

```text
OS security updates
JVM updates
```

Therefore rebuild images regularly.

---

# 50. Pinning Images

Using:

```dockerfile
FROM eclipse-temurin:21-jre
```

provides version-level predictability.

For stronger reproducibility, organizations may pin an image digest:

```text
image@sha256:...
```

This ensures the exact image content is selected.

---

# 51. Trade-Off of Digest Pinning

Digest pinning gives:

```text
Maximum reproducibility
```

but creates a maintenance responsibility:

```text
New secure base image
 ↓
Update digest
 ↓
Rebuild
```

Automated dependency/base-image update processes can help.

---

# 52. Dockerfile ENV Best Practice

Avoid:

```dockerfile
ENV DB_PASSWORD=production-secret
```

Use runtime configuration:

```text
Environment
Secret manager
Orchestrator secret
```

instead.

---

# 53. Docker ARG Best Practice

Use `ARG` for non-sensitive build configuration:

```dockerfile
ARG APP_VERSION=1.0
```

Avoid treating it as a secure secret mechanism.

---

# 54. Build-Time vs Runtime Configuration

Good separation:

```text
Build time
→ Which application version?

Runtime
→ Which database?
→ Which Redis?
→ Which environment?
→ Which credentials?
```

The same image should ideally be deployable to:

```text
Dev
Test
Production
```

with different runtime configuration.

---

# 55. Twelve-Factor Principle

Containerized applications benefit from separating:

```text
Code
Config
Dependencies
```

Configuration should not require rebuilding the application image.

---

# 56. Spring Boot Configuration

Example:

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.data.redis.host=${REDIS_HOST}
spring.data.redis.port=${REDIS_PORT}
```

Then inject values at runtime.

---

# 57. Example Docker Run

```bash
docker run \
  -e DB_URL="jdbc:mysql://mysql:3306/ecommerce" \
  -e DB_USERNAME="app" \
  -e DB_PASSWORD="secret" \
  -e REDIS_HOST="redis" \
  -e REDIS_PORT="6379" \
  ecommerce-api:1.0
```

For production, credentials should come from a proper secret mechanism.

---

# 58. Graceful Shutdown

Spring Boot can gracefully shut down when it receives termination signals.

This matters when:

```text
Container stops
Deployment replaces container
Host drains workloads
```

The application should:

```text
Stop accepting new work
Finish appropriate in-flight work
Close resources
Exit
```

---

# 59. Container Logging

Prefer:

```text
Application
 ↓
stdout/stderr
 ↓
Docker/orchestrator logging
```

instead of relying only on files inside the container.

Why?

Because container runtimes and orchestrators can collect stdout/stderr centrally.

---

# 60. Don't Store Logs Permanently in Container Filesystem

A container's filesystem is ephemeral.

For production:

```text
stdout/stderr
+
centralized logging
```

is generally preferable.

Examples of centralized logging platforms include:

```text
ELK
Cloud logging
OpenTelemetry-compatible systems
```

---

# 61. Health Check Types

Think about:

```text
Process health
Application health
Dependency health
```

Example:

```text
Java process alive
```

does not necessarily mean:

```text
Database connectivity healthy
```

Design health endpoints according to what the deployment platform needs.

---

# 62. Liveness vs Readiness

Conceptually:

```text
Liveness
→ Should this process be restarted?

Readiness
→ Should this instance receive traffic?
```

This distinction becomes especially important in Kubernetes.

---

# 63. Docker HEALTHCHECK

Conceptually:

```dockerfile
HEALTHCHECK --interval=30s \
  CMD curl --fail http://localhost:8080/actuator/health || exit 1
```

But the image must actually contain the required tool such as `curl`.

For minimal images, use an appropriate health-check strategy.

---

# 64. Don't Overuse Health Checks

A health check should be:

```text
Fast
Reliable
Meaningful
Cheap
```

Avoid an expensive health check that performs multiple downstream operations every few seconds.

---

# 65. Resource Limits

For example:

```bash
docker run \
  --memory=512m \
  --cpus=1.0 \
  ecommerce-api:1.0
```

This constrains the container's resources.

---

# 66. Java Memory in Containers

Modern JVMs can understand container resource limits, but JVM configuration still matters.

For example:

```text
Heap sizing
GC
CPU availability
Memory percentage
```

Avoid assuming the JVM should use all container memory.

Leave room for:

```text
Non-heap memory
Threads
Metaspace
Native allocations
```

---

# 67. OOMKilled

If a container exceeds its memory limit, it can be killed.

You may see:

```text
OOMKilled
```

Investigate:

```text
Heap
Native memory
Container limit
Traffic
Memory leak
GC
```

---

# 68. Build Reproducibility

A reproducible image should control:

```text
Base image
Dependency versions
Build configuration
Application version
```

Avoid:

```text
Random downloads
Floating versions
Uncontrolled external dependencies
```

when reproducibility is important.

---

# 69. Docker Build Cache vs Runtime Cache

Don't confuse:

```text
Docker build cache
```

with:

```text
Redis/application cache
```

Build cache:

```text
Speeds image construction
```

Runtime cache:

```text
Speeds application requests
```

Completely different concepts.

---

# 70. Docker Image vs JAR

A JAR contains:

```text
Java application
+
Java dependencies
```

A Docker image contains:

```text
Application
+
Runtime filesystem
+
Java runtime or required runtime environment
+
Container metadata
```

This makes the image a more complete deployment artifact.

---

# 71. Docker and Maven

Typical external-build flow:

```text
mvn clean package
 ↓
target/app.jar
 ↓
docker build
 ↓
Docker image
```

Typical multi-stage flow:

```text
docker build
 ↓
Maven build inside builder stage
 ↓
JAR
 ↓
Runtime image
```

---

# 72. External Build vs Multi-Stage

### External build

Pros:

```text
Simple Dockerfile
Fast local iteration
```

Cons:

```text
Requires Maven/JDK outside Docker
Environment consistency depends on build setup
```

### Multi-stage

Pros:

```text
Reproducible build environment
No Maven required on host
Smaller final runtime image
```

Cons:

```text
Docker build can be slower
Caching needs careful setup
```

---

# 73. Docker Build Command

```bash
docker build -t ecommerce-api:1.0 .
```

Useful variations:

```bash
docker build --no-cache -t ecommerce-api:1.0 .
```

`--no-cache` forces rebuild rather than using cached layers.

Use it for debugging cache-related issues, not every build.

---

# 74. Inspect Image

Useful command:

```bash
docker image inspect ecommerce-api:1.0
```

This can show:

```text
Configuration
Architecture
Environment
Entrypoint
Working directory
Layers
```

---

# 75. Image History

Useful command:

```bash
docker history ecommerce-api:1.0
```

This helps inspect image layers and the commands associated with them.

It can also help identify unexpectedly large layers.

---

# 76. Tagging

Example:

```bash
docker tag ecommerce-api:1.0 \
  username/ecommerce-api:1.0
```

Then push:

```bash
docker push username/ecommerce-api:1.0
```

The exact registry depends on where you publish the image.

---

# 77. CI/CD Image Versioning

Prefer meaningful immutable versions:

```text
ecommerce-api:1.4.2
ecommerce-api:git-abc123
```

rather than relying only on:

```text
latest
```

This makes rollback easier.

---

# 78. Rollback

Suppose production has:

```text
ecommerce-api:1.5
```

and it fails.

If:

```text
ecommerce-api:1.4
```

still exists:

```text
Deploy 1.4
```

This is much safer than trying to modify the broken container manually.

---

# 79. Immutable Infrastructure

Core idea:

```text
Don't repair production containers manually.

Build
→ Test
→ Version
→ Deploy
```

If something changes:

```text
Build a new image.
```

---

# 80. Common Dockerfile Mistakes

Avoid:

```text
Running everything as root
Baking secrets into image
Using latest blindly
Huge build context
No .dockerignore
Maven/JDK in final runtime unnecessarily
Too many RUN layers where avoidable
Unpinned dependencies
Storing important data in container filesystem
```

---

# 81. Interview Question

### What is the difference between `RUN` and `CMD`?

Answer:

> "`RUN` executes during image build and contributes to the image. `CMD` provides the default command or arguments used when a container starts."

---

# 82. Interview Question

### COPY vs ADD?

Answer:

> "`COPY` is the straightforward file-copy instruction and is preferred for most application files. `ADD` has additional behavior, so I use it only when that behavior is actually required."

---

# 83. Interview Question

### ARG vs ENV?

Answer:

> "`ARG` is primarily for build-time variables, while `ENV` defines environment variables available to the image/container. Neither should be treated as a secure secret store."

---

# 84. Interview Question

### Why use multi-stage Docker builds?

Answer:

> "They separate the build environment from the runtime environment, allowing Maven and the JDK to stay out of the final production image."

---

# 85. Interview Question

### How would you reduce a Spring Boot image size?

Answer:

> "I'd use a multi-stage build, a suitable runtime-only base image, exclude unnecessary files with `.dockerignore`, take advantage of layer caching and keep build tools out of the final image."

---

# 86. Interview Question

### How would you secure a Docker image?

Answer:

> "I'd use a trusted and maintained base image, scan the image and dependencies, avoid secrets in the image, run as a non-root user where practical, minimize unnecessary packages and rebuild regularly for security updates."

---

# 87. Interview Question

### Why should containers be stateless?

Answer:

> "Stateless containers are easier to replace, scale and recover. Persistent state should be externalized into databases, volumes or managed storage."

---

# 88. Interview Question

### How do containers communicate?

Answer:

> "Containers can communicate through Docker networks. On a user-defined network, services can generally reach each other using their container or service names rather than localhost."

---

# 89. Interview Scenario

### Spring Boot container cannot connect to MySQL.

Check:

```text
1. Is MySQL running?
2. Are both containers on the same network?
3. Is hostname correct?
4. Is the port correct?
5. Is MySQL ready to accept connections?
6. Are credentials correct?
7. Is JDBC URL using mysql rather than localhost?
```

Example:

```text
jdbc:mysql://mysql:3306/ecommerce
```

---

# 90. Interview Scenario

### Container starts and immediately exits.

Check:

```bash
docker ps -a
docker logs <container>
```

Likely causes:

```text
Application crashed
Wrong ENTRYPOINT
Missing configuration
Port/config issue
JVM failure
Missing dependency
```

---

# 91. Interview Scenario

### Docker image is 1.5 GB.

Investigate:

```text
Base image
Build tools
Dependencies
Large files
Layer history
Copied source
Build cache
```

Then:

```text
Multi-stage build
Smaller runtime image
.dockerignore
Layer optimization
```

---

# 92. Interview Scenario

### Every source change makes Docker download Maven dependencies again.

Likely issue:

```text
Dockerfile copies source before dependency resolution.
```

Reorder layers so stable files such as:

```text
pom.xml
```

are copied before frequently changing source files.

---

# 93. Interview Scenario

### Production container contains a database password.

Problem:

```text
Secret baked into image
```

Fix:

```text
Remove secret
Rotate credential
Inject at runtime
Use secret manager
Rebuild image
```

Treat the exposed credential as compromised.

---

# 94. Interview Scenario

### Container works locally but fails in CI.

Check:

```text
Base image
Architecture
Environment variables
Build context
Dockerfile paths
Network access
Private Maven repository credentials
Dependency versions
```

Docker improves consistency but doesn't eliminate configuration differences automatically.

---

# 95. Interview Scenario

### Container works but receives no traffic.

Check:

```text
Application listening port
Docker EXPOSE
-p host:container mapping
Network
Firewall
Application bind address
```

For a containerized Spring Boot service, verify the application is listening appropriately inside the container.

---

# 96. Practical Spring Boot Dockerfile

A clean runtime-oriented Dockerfile:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/*.jar app.jar

RUN useradd --system appuser
USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

The exact non-root user creation command depends on the base image.

---

# 97. Practical Multi-Stage Dockerfile

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

For production, further optimize dependency caching, user creation and base-image selection according to the actual project.

---

# 98. Recommended Build Flow

For your Spring Boot e-commerce backend:

```text
Code
 ↓
mvn test
 ↓
mvn package
 ↓
docker build
 ↓
Image scan
 ↓
docker run locally
 ↓
Integration tests
 ↓
Push registry
 ↓
Deploy
```

---

# 99. Docker + Redis + MySQL

Eventually your local environment should look like:

```text
             Docker network
                   |
      +------------+------------+
      |            |            |
 Spring Boot     MySQL        Redis
      |            |            |
    8080          3306         6379
```

Spring Boot configuration:

```text
DB_HOST=mysql
REDIS_HOST=redis
```

rather than:

```text
localhost
```

for container-to-container communication.

---

# 100. File 02 Checklist

You should now understand:

```text
□ FROM
□ RUN
□ COPY
□ ADD
□ WORKDIR
□ ENV
□ ARG
□ EXPOSE
□ USER
□ ENTRYPOINT
□ CMD
□ Exec form
□ Shell form
□ Signal handling
□ Image layers
□ Build cache
□ Build context
□ .dockerignore
□ Multi-stage builds
□ Layered JARs
□ Buildpacks
□ Image size
□ Image scanning
□ Base image security
□ Image pinning
□ Runtime configuration
□ Resource limits
□ OOMKilled
□ Health checks
□ Liveness/readiness concept
□ Image tagging
□ CI/CD
□ Rollback
□ Immutable deployments
□ Docker + Spring Boot
```

---

# 101. What Comes Next

```text
File 03 → Docker Networking, Volumes & Container Communication
```

Next we'll go deeper into:

```text
Bridge networks
User-defined networks
DNS between containers
Port publishing
Container-to-container communication
Host networking
Volumes
Bind mounts
Named volumes
MySQL persistence
Redis persistence
Spring Boot → MySQL
Spring Boot → Redis
Network troubleshooting
Docker commands
Interview scenarios
```

Key takeaway:

> **A good Dockerfile is not just a file that makes the application run. It should produce a small, reproducible, secure and maintainable image. For Spring Boot, the strongest pattern is usually a build stage with Maven/JDK followed by a minimal runtime stage containing only the application and its runtime requirements.**

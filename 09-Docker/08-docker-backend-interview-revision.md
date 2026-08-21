# Docker — File 08: Backend Interview Revision
## Questions, Scenarios & Short Answers

This is the final Docker revision file.

The goal is not to learn new Docker concepts here.

Use this file for:

```text
Quick revision
Interview practice
Short answers
Troubleshooting scenarios
System-design discussions
```

---

# 1. Docker Fundamentals

### Q1. What is Docker?

**Answer:**

> "Docker is a containerization platform that packages an application with its runtime dependencies so it can run consistently across environments."

---

### Q2. What problem does Docker solve?

**Answer:**

> "It reduces environment differences. Instead of depending on what's installed on a machine, we package the application and its required runtime into an image."

---

### Q3. What is a container?

**Answer:**

> "A container is a running instance of an image with isolated processes, networking and filesystem resources."

---

### Q4. What is a Docker image?

**Answer:**

> "An image is an immutable template used to create containers. It contains the application and the files and dependencies needed to run it."

---

### Q5. Image vs container?

```text
Image
→ Template / artifact

Container
→ Running instance
```

Interview answer:

> "An image is the packaged artifact, while a container is a runtime instance of that image."

---

# 2. Dockerfile

### Q6. What is a Dockerfile?

**Answer:**

> "A Dockerfile contains instructions for building a Docker image."

Example:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### Q7. What does `FROM` do?

It defines the base image.

```dockerfile
FROM eclipse-temurin:21-jre
```

---

### Q8. What does `WORKDIR` do?

It sets the working directory inside the image/container.

```dockerfile
WORKDIR /app
```

---

### Q9. `COPY` vs `ADD`?

Prefer:

```dockerfile
COPY
```

for normal file copying.

`ADD` has additional behavior such as archive extraction and URL-related behavior.

For most application Dockerfiles:

```text
COPY
→ simpler and explicit
```

---

### Q10. `RUN` vs `CMD`?

```text
RUN
→ Executes during image build

CMD
→ Default runtime command/arguments
```

Example:

```dockerfile
RUN mvn package
```

vs:

```dockerfile
CMD ["java", "-jar", "app.jar"]
```

---

### Q11. `ENTRYPOINT` vs `CMD`?

Simple interview explanation:

```text
ENTRYPOINT
→ Main executable

CMD
→ Default arguments / command
```

Example:

```dockerfile
ENTRYPOINT ["java", "-jar"]
CMD ["app.jar"]
```

---

### Q12. Does `EXPOSE` publish a port?

No.

```dockerfile
EXPOSE 8080
```

documents the intended container port.

To publish it:

```bash
docker run -p 8080:8080 app
```

---

# 3. Docker Build

### Q13. How do you build an image?

```bash
docker build -t ecommerce-api:1.0 .
```

The final `.` is the build context.

---

### Q14. What is build context?

The files/directories available to the Docker build.

Example:

```bash
docker build .
```

means the current directory is the context.

---

### Q15. Why use `.dockerignore`?

To prevent unnecessary files from entering the build context.

Example:

```text
.git
.idea
*.log
.env
```

This can improve:

```text
Build speed
Image cleanliness
Security
```

---

### Q16. Why use multi-stage builds?

**Answer:**

> "They separate build dependencies from runtime dependencies. For a Spring Boot application, Maven and the JDK can stay in the builder stage while the final image contains only the application JAR and Java runtime."

---

### Q17. Why is a smaller image useful?

Benefits:

```text
Less storage
Faster pulls
Faster deployment
Smaller attack surface
```

But:

```text
Small ≠ automatically secure
```

---

# 4. Docker Layers & Cache

### Q18. What are Docker image layers?

Each Dockerfile instruction can contribute to the image's layered filesystem.

Example:

```text
FROM
 ↓
WORKDIR
 ↓
COPY
 ↓
RUN
```

---

### Q19. Why does Docker cache layers?

To avoid rebuilding unchanged work.

If dependencies haven't changed, Docker can reuse the corresponding layer.

---

### Q20. How can Dockerfile order improve caching?

For Maven:

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package
```

The dependency layer can remain cached when only source code changes.

---

# 5. Ports

### Q21. What does `8080:8080` mean?

```text
HOST:CONTAINER
```

So:

```text
localhost:8080
      ↓
container:8080
```

---

### Q22. What does `8081:8080` mean?

```text
Host 8081
     ↓
Container 8080
```

So the API is accessed from the host through:

```text
localhost:8081
```

---

### Q23. Does every container need a published port?

No.

Containers can communicate through a Docker network without publishing their ports to the host.

Example:

```text
app → mysql:3306
```

No:

```text
3306:3306
```

is required if only the app needs MySQL.

---

# 6. Docker Networking

### Q24. How do containers communicate?

Usually through a Docker network.

Example:

```text
app
 |
backend-net
 |
mysql
```

The app can use:

```text
mysql:3306
```

---

### Q25. Why doesn't `localhost` work between containers?

Because:

```text
localhost
```

means the current container.

So from the Spring Boot container:

```text
localhost:3306
```

means:

```text
Spring Boot container
```

not:

```text
MySQL container
```

---

### Q26. What should Spring Boot use for MySQL?

```properties
spring.datasource.url=jdbc:mysql://mysql:3306/ecommerce
```

---

### Q27. What should Spring Boot use for Redis?

```properties
spring.data.redis.host=redis
spring.data.redis.port=6379
```

---

### Q28. Why use service names instead of container IPs?

Because Docker DNS resolves service/container names.

IP addresses can change when containers are recreated.

---

# 7. Volumes

### Q29. Why does MySQL need a volume?

Because container filesystems are not the right place for important persistent database data.

Use:

```yaml
volumes:
  - mysql-data:/var/lib/mysql
```

---

### Q30. What happens when the MySQL container is deleted?

If the named volume remains:

```text
Container deleted
      ↓
Volume remains
      ↓
Data remains
```

---

### Q31. What does this do?

```bash
docker compose down -v
```

It removes Compose-managed volumes in addition to containers and networks.

Be careful with databases.

---

### Q32. Volume vs bind mount?

```text
Named volume
→ Managed by Docker

Bind mount
→ Maps a specific host path
```

For database persistence, named volumes are often convenient.

---

# 8. Docker Compose

### Q33. What is Docker Compose?

**Answer:**

> "Compose lets me define and run a multi-container application using a declarative YAML configuration."

---

### Q34. Why use Compose for Spring Boot?

For local development:

```text
Spring Boot
MySQL
Redis
```

can be started together.

```bash
docker compose up
```

---

### Q35. What does `depends_on` do?

It expresses a service dependency/startup relationship.

Important:

```text
depends_on
≠
dependency is fully ready
```

Use healthchecks and application resilience where needed.

---

### Q36. What is a healthcheck?

It checks whether a service is actually responding as expected.

Example:

```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
```

---

### Q37. `docker compose stop` vs `down`?

```text
stop
→ Stop services

down
→ Stop and remove Compose-managed containers/network
```

Named volumes normally remain after `down`.

---

### Q38. What does `up --build` do?

```bash
docker compose up --build
```

rebuilds images before starting services when necessary.

---

# 9. Spring Boot + Docker

### Q39. How would you Dockerize Spring Boot?

Answer:

> "I'd build the application with Maven and use a multi-stage Dockerfile. The builder stage would compile the application, while the runtime stage would contain a smaller Java runtime and the JAR. I'd run it as a non-root user and inject environment-specific configuration at runtime."

---

### Q40. Does Docker change your Spring Boot architecture?

No.

The application still has:

```text
Controller
Service
Repository
Database
Cache
```

Docker provides the runtime environment.

---

### Q41. How does Spring Boot know the database hostname?

Through configuration.

Example:

```properties
spring.datasource.url=${DB_URL}
```

Compose provides:

```yaml
environment:
  DB_URL: jdbc:mysql://mysql:3306/ecommerce
```

---

### Q42. Why inject configuration at runtime?

Because the same image can be used across environments.

```text
Same image
   ↓
Different configuration
```

---

# 10. Docker Security

### Q43. Why run containers as non-root?

**Answer:**

> "It follows the least-privilege principle. If the application is compromised, the process has fewer permissions."

---

### Q44. Should secrets be stored in a Dockerfile?

No.

Avoid:

```dockerfile
ENV DB_PASSWORD=productionPassword
```

Use a proper runtime secret-management mechanism.

---

### Q45. Why avoid `--privileged`?

It grants significantly expanded privileges to the container.

A normal Spring Boot application shouldn't require it.

---

### Q46. Why avoid unnecessary Docker socket mounts?

```text
/var/run/docker.sock
```

can provide powerful control over Docker.

A compromised container with access to it may have a much larger impact.

---

### Q47. Why scan Docker images?

Images can contain vulnerabilities in:

```text
OS packages
Java runtime
Application dependencies
Transitive dependencies
```

---

### Q48. What is an SBOM?

**Answer:**

> "A Software Bill of Materials is an inventory of the components and dependencies contained in an application or artifact."

---

# 11. Production

### Q49. Should you use `latest` in production?

Prefer a version or immutable identifier:

```text
ecommerce-api:1.4.0
```

or a commit-based tag.

---

### Q50. What is an immutable image?

An image that is treated as a fixed deployment artifact.

If code changes:

```text
Build new image
```

rather than modifying the running container.

---

### Q51. What is build once, deploy many?

```text
Build
 ↓
Test
 ↓
Scan
 ↓
Publish
 ↓
Dev
 ↓
Test
 ↓
Production
```

The same tested artifact moves through environments.

---

### Q52. Why is this useful?

It prevents:

```text
"Dev used one build,
production used another build."
```

The artifact is consistent.

---

### Q53. How do you rollback?

Deploy the previous known-good image.

Example:

```text
1.4.0 → broken
1.3.0 → stable
```

Rollback:

```text
Deploy 1.3.0
```

---

# 12. CI/CD

### Q54. What would your Docker CI/CD pipeline look like?

```text
Git push
   ↓
Maven build
   ↓
Unit tests
   ↓
Docker build
   ↓
Image scan
   ↓
Integration tests
   ↓
Push registry
   ↓
Deploy
```

---

### Q55. Why scan before deployment?

To catch known vulnerabilities before the artifact reaches production.

---

### Q56. Why push images to a registry?

The registry provides a centralized location for deployment systems to pull versioned images.

Examples:

```text
ECR
ACR
GHCR
Docker Hub
Artifact Registry
```

---

# 13. Troubleshooting

### Q57. Spring Boot gets `Connection refused`.

Check:

```text
Container status
Application logs
MySQL/Redis status
Hostname
Port
Network
Readiness
Credentials
```

---

### Q58. `UnknownHostException: mysql`

Check:

```text
Network
Service name
Compose configuration
```

The app must be on the same network as MySQL.

---

### Q59. `Communications link failure`

For MySQL check:

```text
mysql:3306
MySQL readiness
Credentials
Database name
Network
```

---

### Q60. API is running but `localhost:8080` doesn't work.

Check:

```text
docker ps
Port mapping
Spring server.port
Application logs
Host port conflicts
```

---

### Q61. Container immediately exits.

Run:

```bash
docker ps -a
docker logs <container>
```

Then investigate the actual application/entrypoint error.

---

### Q62. Image cannot find the JAR.

Check:

```bash
ls target/
```

Then:

```bash
mvn clean package
```

Also check:

```text
Build context
COPY path
.dockerignore
```

---

### Q63. MySQL data disappeared.

First ask:

```text
Was the volume deleted?
```

Check:

```bash
docker volume ls
```

If you ran:

```bash
docker compose down -v
```

the Compose-managed volume may have been removed.

---

# 14. Scenario Questions

### Q64. Your app works locally but fails in Docker. What do you check?

```text
localhost references
Environment variables
Spring profile
Network
Ports
Filesystem paths
Java version
Database
Redis
```

---

### Q65. App starts before MySQL is ready. What do you do?

Use:

```text
Healthcheck
+
dependency/readiness handling
+
application retry/resilience
```

Don't rely on arbitrary:

```text
sleep 30
```

---

### Q66. You need to expose the API but not MySQL.

Use:

```yaml
app:
  ports:
    - "8080:8080"
```

Don't publish:

```text
3306:3306
```

unless required.

---

### Q67. You need two application versions locally.

Use different host ports or separate Compose projects.

Example:

```text
v1 → localhost:8080
v2 → localhost:8081
```

Both can use:

```text
8080
```

inside their containers.

---

### Q68. Production container uses too much memory.

Investigate:

```text
JVM heap
GC
Threads
Native memory
Traffic
Memory leaks
Container limits
```

Don't blindly increase the limit.

---

### Q69. New Docker image causes production errors.

If the image is versioned:

```text
Deploy previous stable version
```

Then investigate the new release.

---

### Q70. A developer accidentally committed a production password.

Treat it as compromised.

Do:

```text
Rotate credential
Remove exposure
Check CI/artifacts/history
Create new secret
Audit access
```

---

# 15. Docker + System Design

### Q71. Why shouldn't application containers store important business data?

Containers are replaceable.

Persistent state belongs in:

```text
Database
Object storage
Persistent volumes
Managed services
```

depending on the data.

---

### Q72. Why might you use managed MySQL in production?

Because the platform can provide:

```text
Backups
Patching
Failover
Monitoring
Storage management
Recovery
```

This reduces operational burden.

---

### Q73. Why might Redis be managed?

For:

```text
High availability
Failover
Monitoring
Scaling
Operational simplicity
```

The exact requirements depend on whether Redis is a cache or holds important state.

---

### Q74. What should an application container ideally be?

```text
Stateless
Replaceable
Versioned
Observable
Least-privileged
Configured at runtime
```

---

# 16. Docker vs Kubernetes

### Q75. Is Docker a replacement for Kubernetes?

No.

Docker focuses on:

```text
Containerization
Images
Runtime
Networking
Volumes
```

Kubernetes adds:

```text
Scheduling
Scaling
Self-healing
Service discovery
Rolling deployments
```

---

### Q76. Where does the Docker image fit in Kubernetes?

```text
Dockerfile
 ↓
Image
 ↓
Registry
 ↓
Kubernetes
 ↓
Pod
```

The image is the application artifact.

---

# 17. Short Scenario Drill

Try answering these aloud before reading the hints.

### Q77

```text
Spring Boot → localhost:3306
```

Why is it wrong?

**Hint:**

```text
localhost = current container
```

---

### Q78

```text
8081:8080
```

What does it mean?

**Hint:**

```text
Host:Container
```

---

### Q79

MySQL container is deleted. Data survives. Why?

**Hint:**

```text
Named volume
```

---

### Q80

`docker compose down -v` is dangerous. Why?

**Hint:**

```text
Volumes
```

---

### Q81

Why use:

```text
mysql
```

instead of a container IP?

**Hint:**

```text
Docker DNS
```

---

### Q82

Why use multi-stage builds?

**Hint:**

```text
Build dependencies
vs
Runtime dependencies
```

---

### Q83

Why not put production credentials in the image?

**Hint:**

```text
Image inspection
Layers
Registry
```

---

### Q84

Why version images?

**Hint:**

```text
Traceability
Rollback
Reproducibility
```

---

# 18. 30-Second Answers

These are useful when the interviewer wants a quick answer.

### Docker

> "Docker packages applications into portable containers."

### Image

> "An image is the immutable artifact used to create containers."

### Container

> "A container is a running instance of an image."

### Dockerfile

> "A Dockerfile defines how an image is built."

### Compose

> "Compose defines and runs multiple services together."

### Volume

> "A volume provides persistent storage outside the container lifecycle."

### Network

> "A Docker network allows containers to communicate, usually through service names."

### Multi-stage build

> "It keeps build dependencies out of the final runtime image."

### Healthcheck

> "It tells us whether a service is actually healthy, not merely running."

### Registry

> "A registry stores and distributes container images."

---

# 19. Your E-commerce Backend Answer

If asked:

### "Explain how Docker is used in your project."

Say:

> "For local development, I containerized my Spring Boot e-commerce backend and used Docker Compose to run the API with MySQL and Redis. The containers communicate over a Docker network, so the application connects to `mysql:3306` and `redis:6379` rather than localhost. MySQL uses a named volume for persistence. The application image is built from a multi-stage Dockerfile, configuration is injected at runtime, and the same versioned image can be used through CI/CD."

---

# 20. Final Docker Revision Map

```text
Docker
│
├── Fundamentals
│   ├── Image
│   ├── Container
│   └── Runtime
│
├── Dockerfile
│   ├── FROM
│   ├── COPY
│   ├── RUN
│   ├── CMD
│   ├── ENTRYPOINT
│   └── Multi-stage
│
├── Networking
│   ├── Bridge
│   ├── DNS
│   ├── Ports
│   └── Service names
│
├── Storage
│   ├── Volumes
│   └── Bind mounts
│
├── Compose
│   ├── Services
│   ├── Environment
│   ├── Healthchecks
│   └── Dependencies
│
├── Spring Boot
│   ├── Dockerfile
│   ├── MySQL
│   └── Redis
│
├── Security
│   ├── Non-root
│   ├── Secrets
│   ├── Scanning
│   └── Least privilege
│
├── CI/CD
│   ├── Build
│   ├── Test
│   ├── Scan
│   ├── Registry
│   └── Deploy
│
└── Practical
    ├── Debugging
    ├── Rollback
    └── Production
```

---

# 21. Final Checklist

Before moving on from Docker:

```text
□ I can explain image vs container.
□ I can write a basic Dockerfile.
□ I understand multi-stage builds.
□ I understand Docker layers and cache.
□ I understand host vs container ports.
□ I know why localhost causes problems.
□ I understand Docker DNS.
□ I can explain volumes.
□ I can write basic Compose.
□ I understand depends_on vs readiness.
□ I can connect Spring Boot to MySQL.
□ I can connect Spring Boot to Redis.
□ I can debug connection failures.
□ I know basic Docker security.
□ I understand non-root containers.
□ I know why secrets shouldn't be baked into images.
□ I understand image scanning.
□ I understand SBOM.
□ I understand immutable/versioned images.
□ I can explain a Docker CI/CD pipeline.
□ I can explain rollback.
□ I can discuss Docker in system design.
□ I can explain my e-commerce Docker setup.
```

---

# 22. Final Mental Model

Don't memorize 100 Docker commands.

Remember this:

```text
SOURCE CODE
    ↓
Maven
    ↓
DOCKERFILE
    ↓
IMAGE
    ↓
REGISTRY
    ↓
CONTAINER
    ↓
NETWORK
   / \
  /   \
DB   REDIS
 |
VOLUME
```

And around it:

```text
Security
Observability
CI/CD
Configuration
Health
Scaling
Rollback
```

That is the Docker knowledge you need as a Java backend developer.

**Docker section complete.**

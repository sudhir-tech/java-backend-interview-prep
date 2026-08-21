# Docker — File 07: Practical Lab
## Spring Boot + MySQL + Redis

This is the hands-on Docker file for the backend project.

The goal is not to memorize commands.

The goal is to be able to explain and reproduce this flow:

```text
Spring Boot
    ↓
Docker image
    ↓
Docker Compose
    ↓
+-----------+-----------+
|           |           |
MySQL      Redis      API
```

---

# 1. Lab Goal

By the end, you should be able to:

```text
□ Build a Spring Boot JAR
□ Create a Docker image
□ Run the image
□ Create a Docker network
□ Run MySQL
□ Run Redis
□ Connect Spring Boot to both
□ Persist MySQL data
□ Use Docker Compose
□ Debug common failures
□ Explain the setup in an interview
```

---

# 2. Project Structure

Your project should eventually look like:

```text
ecommercebackend/
│
├── src/
├── pom.xml
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
└── .gitignore
```

---

# 3. Step 1 — Make Sure Spring Boot Works

Before Docker:

```bash
mvn clean package
```

Then run normally:

```bash
java -jar target/*.jar
```

Confirm the application starts.

This is important:

> Don't introduce Docker while the application itself is already broken.

---

# 4. Step 2 — Create the Dockerfile

Start simple:

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

---

# 5. Step 3 — Verify the Image

```bash
docker images
```

Look for:

```text
ecommerce-api
```

Inspect:

```bash
docker image inspect ecommerce-api:1.0
```

Check history:

```bash
docker history ecommerce-api:1.0
```

---

# 6. Step 4 — Run the API Alone

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

# 7. What You Should Notice

The application may start successfully.

But if it requires MySQL:

```text
Database connection fails
```

And if it requires Redis:

```text
Redis connection fails
```

That's expected if those services aren't available.

---

# 8. Step 5 — Create the Docker Network

```bash
docker network create backend-net
```

Check:

```bash
docker network ls
```

Inspect:

```bash
docker network inspect backend-net
```

---

# 9. Step 6 — Run MySQL

For local development:

```bash
docker run -d \
  --name mysql \
  --network backend-net \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=ecommerce \
  -v mysql-data:/var/lib/mysql \
  mysql:8.4
```

Check:

```bash
docker ps
```

Logs:

```bash
docker logs mysql
```

---

# 10. Step 7 — Run Redis

```bash
docker run -d \
  --name redis \
  --network backend-net \
  redis:7
```

Check:

```bash
docker ps
```

Logs:

```bash
docker logs redis
```

---

# 11. Step 8 — Put the API on the Same Network

If the API container already exists:

```bash
docker network connect backend-net ecommerce-api
```

Check:

```bash
docker network inspect backend-net
```

You should see:

```text
mysql
redis
ecommerce-api
```

---

# 12. Step 9 — Configure Spring Boot

The important part:

```text
localhost
```

must not be used for MySQL or Redis when those services are separate containers.

Use:

```text
mysql
redis
```

Example:

```properties
spring.datasource.url=jdbc:mysql://mysql:3306/ecommerce
spring.datasource.username=root
spring.datasource.password=root

spring.data.redis.host=redis
spring.data.redis.port=6379
```

---

# 13. Better Configuration

Prefer environment variables:

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

spring.data.redis.host=${REDIS_HOST}
spring.data.redis.port=${REDIS_PORT}
```

Then Docker supplies:

```text
DB_URL
DB_USERNAME
DB_PASSWORD
REDIS_HOST
REDIS_PORT
```

---

# 14. Important Restart

If configuration is baked into the application JAR:

```text
Change configuration
        ↓
Rebuild JAR
        ↓
Rebuild image
        ↓
Recreate container
```

If configuration is injected through environment variables:

```text
Same image
        ↓
Different runtime configuration
```

This is usually preferable for deployment.

---

# 15. Step 10 — Use Environment Variables

Example:

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

If an old container already exists, remove/recreate it first.

---

# 16. Verify the Stack

```bash
docker ps
```

Expected:

```text
ecommerce-api
mysql
redis
```

Then:

```bash
docker logs ecommerce-api
```

Look for:

```text
HikariPool
Hibernate
Tomcat
Spring Boot
Application started
```

---

# 17. Test MySQL Connectivity

If your API successfully starts and Hibernate connects:

```text
Spring Boot
      ↓
mysql:3306
      ↓
MySQL
```

The important part is:

```text
mysql
```

is being resolved through Docker networking.

---

# 18. Test Redis Connectivity

Similarly:

```text
Spring Boot
      ↓
redis:6379
      ↓
Redis
```

If using Spring Data Redis, check startup/application logs and test an endpoint that uses Redis.

---

# 19. Step 11 — Test an API Endpoint

Use:

```text
http://localhost:8080
```

or one of your actual endpoints.

For example:

```bash
curl http://localhost:8080/products
```

The exact endpoint depends on your application.

---

# 20. Request Flow

A request might look like:

```text
Postman
   ↓
localhost:8080
   ↓
Docker port mapping
   ↓
Spring Boot
   ↓
Service
   ↓
Repository
   ↓
MySQL
```

If caching is involved:

```text
Service
   ↓
Redis
   ↓
Cache hit / miss
```

---

# 21. Step 12 — Check MySQL Persistence

Stop the MySQL container:

```bash
docker stop mysql
```

Remove it:

```bash
docker rm mysql
```

Create another MySQL container using the same volume:

```bash
docker run -d \
  --name mysql \
  --network backend-net \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=ecommerce \
  -v mysql-data:/var/lib/mysql \
  mysql:8.4
```

Your database data should still exist.

Why?

```text
Container
   ↓
mysql-data
```

The volume survived container removal.

---

# 22. Verify the Volume

```bash
docker volume ls
```

Then:

```bash
docker volume inspect mysql-data
```

---

# 23. Important Lesson

This is one of the most important Docker concepts:

```text
Container lifecycle
        ≠
Data lifecycle
```

A database container can be replaced without intentionally deleting its persistent volume.

---

# 24. Step 13 — Replace Manual Commands with Compose

Now create:

```text
docker-compose.yml
```

Use:

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
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy

  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: ecommerce
    volumes:
      - mysql-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mysql-data:
```

This is a local learning setup.

Don't use the example password as a production credential.

---

# 25. Step 14 — Start Everything

From the project directory:

```bash
docker compose up --build
```

Or in background:

```bash
docker compose up --build -d
```

---

# 26. Check Services

```bash
docker compose ps
```

You should see:

```text
app
mysql
redis
```

---

# 27. Check All Logs

```bash
docker compose logs
```

Follow:

```bash
docker compose logs -f
```

Only the API:

```bash
docker compose logs -f app
```

---

# 28. Stop the Environment

```bash
docker compose stop
```

This stops containers without removing the Compose resources.

---

# 29. Remove the Environment

```bash
docker compose down
```

This normally removes:

```text
Containers
Compose network
```

The named volume remains.

---

# 30. Delete Database Data

Only for disposable local development:

```bash
docker compose down -v
```

Remember:

```text
-v
→ remove volumes
```

For this project that can remove:

```text
mysql-data
```

and therefore your local database.

---

# 31. Step 15 — Add `.dockerignore`

Create:

```text
.dockerignore
```

Example:

```text
.git
.idea
.vscode
*.log
.env
```

If using the Dockerfile that copies:

```text
target/*.jar
```

do not ignore `target/`.

---

# 32. Step 16 — Improve the Dockerfile

After the basic version works, use:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/*.jar app.jar

RUN useradd --system appuser
USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

The exact user creation command depends on the chosen base image.

---

# 33. Step 17 — Test Rebuild

Make a small application change.

Then:

```bash
mvn clean package
```

Build:

```bash
docker build -t ecommerce-api:1.1 .
```

Run through Compose:

```bash
docker compose up --build -d
```

Now you have:

```text
1.0
1.1
```

as separate image versions.

---

# 34. Step 18 — Check Image History

```bash
docker history ecommerce-api:1.1
```

Ask yourself:

```text
What is each layer doing?
```

This helps you understand Docker rather than merely using it.

---

# 35. Step 19 — Inspect the Container

```bash
docker inspect <container-name>
```

Look at:

```text
Environment
Network
Mounts
Ports
Entrypoint
Working directory
```

This is one of the best commands for debugging Docker.

---

# 36. Step 20 — Test Network Resolution

From a suitable diagnostic container on the same network, verify:

```text
mysql
redis
```

resolve correctly.

Minimal production images may not contain:

```text
ping
nslookup
curl
```

so don't assume those commands exist.

---

# 37. Debugging Scenario 1

### API cannot connect to MySQL

Check:

```bash
docker compose ps
```

Then:

```bash
docker compose logs mysql
```

Then:

```bash
docker compose logs app
```

Verify:

```text
DB_URL
mysql hostname
3306
credentials
database name
network
health
```

---

# 38. Debugging Scenario 2

### `Communications link failure`

Don't immediately assume Docker is broken.

Check:

```text
Is MySQL ready?
Is hostname mysql?
Is port 3306?
Are credentials correct?
Is the database initialized?
```

---

# 39. Debugging Scenario 3

### `UnknownHostException: mysql`

Likely check:

```text
Is app connected to Compose network?
Is service actually named mysql?
Is the Compose project running correctly?
```

Don't hardcode the container IP.

---

# 40. Debugging Scenario 4

### Redis connection refused

Check:

```bash
docker compose logs redis
```

Then:

```text
REDIS_HOST=redis
REDIS_PORT=6379
```

and verify the Redis service is healthy.

---

# 41. Debugging Scenario 5

### Port 8080 already in use

Change the host side:

```yaml
ports:
  - "8081:8080"
```

Now:

```text
localhost:8081
```

reaches:

```text
container:8080
```

---

# 42. Debugging Scenario 6

### MySQL data disappeared

Ask:

```text
Did I run docker compose down -v?
```

If yes:

```text
Volume was removed.
```

Otherwise inspect:

```bash
docker volume ls
```

and:

```bash
docker inspect mysql
```

---

# 43. Debugging Scenario 7

### App keeps restarting

Start with:

```bash
docker compose ps
docker compose logs app
```

Check:

```text
Java exception
Database
Redis
Environment variables
Memory
Entrypoint
Healthcheck
```

---

# 44. Debugging Scenario 8

### Docker build cannot find the JAR

Check:

```bash
ls target/
```

Then:

```bash
mvn clean package
```

Make sure:

```text
target/*.jar
```

exists and is inside the Docker build context.

---

# 45. Debugging Scenario 9

### Dockerfile change isn't reflected

Try:

```bash
docker compose up --build
```

If debugging cache behavior:

```bash
docker build --no-cache ...
```

Don't use `--no-cache` as the default workflow.

---

# 46. Debugging Scenario 10

### App works outside Docker but not inside

Compare:

```text
Outside Docker:
localhost

Inside Docker:
service names
```

Also compare:

```text
Environment variables
Java version
Filesystem paths
Ports
Network
Profiles
```

---

# 47. Practical Architecture

Your final local environment should look like:

```text
                  Host
                   |
              localhost:8080
                   |
                   ↓
             Spring Boot
                   |
              Compose DNS
              /          \
             ↓            ↓
          mysql          redis
          :3306          :6379
             |              |
             ↓              ↓
        mysql-data      optional
```

---

# 48. What Docker Is Actually Doing

Docker is not changing your backend architecture.

Your application still has:

```text
Controller
Service
Repository
Database
Cache
```

Docker provides:

```text
Packaging
Isolation
Networking
Runtime
Persistence mounts
Environment configuration
```

---

# 49. Interview Exercise

Explain this diagram aloud:

```text
Browser
   ↓
localhost:8080
   ↓
Spring Boot container
   ↓
Docker network
  /       \
 ↓         ↓
MySQL    Redis
```

Try explaining it without reading.

---

# 50. Interview Answer

### "How does your Spring Boot application connect to MySQL in Docker?"

> "The Spring Boot and MySQL containers are connected to the same Docker network. The application uses `mysql` as the hostname and `3306` as the container port. I don't use localhost because localhost inside the Spring Boot container refers to that container itself."

---

# 51. Interview Answer

### "Why use Docker Compose?"

> "It lets me define the complete local environment declaratively. For my backend, I can start Spring Boot, MySQL and Redis together, including their networking, environment variables, healthchecks and persistent volumes."

---

# 52. Interview Answer

### "How do you persist MySQL data?"

> "I mount a named Docker volume to MySQL's data directory. That separates the database data lifecycle from the MySQL container lifecycle."

---

# 53. Interview Answer

### "How would you debug a containerized Spring Boot application?"

> "I'd first check the container status and application logs. Then I'd verify environment variables, port mappings, network connectivity and dependency health. For database issues I'd check the hostname, port, readiness and credentials."

---

# 54. Interview Answer

### "Why not publish MySQL's port?"

> "If only the application container needs MySQL, I don't need to expose MySQL to the host. The application can access it through the private Docker network using `mysql:3306`."

---

# 55. Interview Answer

### "What happens if you delete the Spring Boot container?"

> "The container disappears, but the application image remains. Any external persistent data such as MySQL data stored in a volume is unaffected."

---

# 56. Interview Answer

### "What happens if you delete the MySQL container?"

> "If MySQL uses a named volume, deleting the container doesn't delete the volume. I can create a new MySQL container using the same volume and recover the existing data, assuming the volume itself wasn't removed."

---

# 57. Interview Answer

### "What happens if you run `docker compose down -v`?"

> "It removes the Compose-managed volumes as well as the containers and network. If MySQL data is stored in one of those volumes, the local database data can be deleted."

---

# 58. Interview Answer

### "How would you make this production-ready?"

> "I'd use a minimal maintained runtime image, run as non-root, keep secrets outside the image, scan and version the image, add health and observability, limit resources, use proper CI/CD and preferably use managed MySQL and Redis services rather than managing stateful containers manually."

---

# 59. Final Practical Checklist

Before saying:

```text
"I know Docker"
```

make sure you can actually do:

```text
□ mvn clean package

□ docker build

□ docker run

□ docker ps

□ docker logs

□ docker inspect

□ docker network create

□ docker network inspect

□ docker volume create

□ docker volume inspect

□ docker compose up

□ docker compose ps

□ docker compose logs

□ docker compose down
```

And explain:

```text
□ localhost vs service name
□ host port vs container port
□ image vs container
□ volume vs container filesystem
□ Dockerfile vs Compose
□ depends_on vs readiness
□ build time vs runtime configuration
```

---

# 60. Mini Challenge

Try doing this without looking at the previous files:

### Challenge

Build a local environment containing:

```text
Spring Boot
MySQL
Redis
```

Requirements:

```text
1. API accessible at localhost:8080
2. API connects to mysql:3306
3. API connects to redis:6379
4. MySQL data survives container recreation
5. Everything starts with docker compose
6. No MySQL port needs to be public
7. No Redis port needs to be public
```

If you can do this from scratch, you understand the practical Docker basics.

---

# 61. Mini Interview Challenge

Answer these without looking:

### Q1
Why doesn't `localhost:3306` work from the Spring Boot container?

### Q2
What does:

```text
8080:8080
```

mean?

### Q3
Why does MySQL need a volume?

### Q4
What does `depends_on` do?

### Q5
Why use a multi-stage Dockerfile?

### Q6
Why shouldn't production secrets be in the Dockerfile?

### Q7
What would you check if Spring Boot gets `Connection refused`?

### Q8
What happens to a named volume after:

```bash
docker compose down
```

### Q9
What happens after:

```bash
docker compose down -v
```

### Q10
How would you make this setup production-ready?

---

# 62. Answers

### Q1

```text
localhost = current container
```

Use:

```text
mysql:3306
```

---

### Q2

```text
HOST_PORT:CONTAINER_PORT
```

So:

```text
localhost:8080
        ↓
container:8080
```

---

### Q3

Because the container filesystem is ephemeral. A named volume lets database data survive container replacement.

---

### Q4

It defines service startup dependencies. It does not automatically guarantee that a dependency is ready unless readiness/health conditions are configured appropriately.

---

### Q5

It keeps build dependencies such as Maven and the JDK out of the final runtime image.

---

### Q6

Because secrets can be exposed through source, image layers, metadata, history or registries. Inject them through secure runtime configuration instead.

---

### Q7

Check:

```text
Container status
Logs
Hostname
Port
Network
Readiness
Credentials
Configuration
```

---

### Q8

Normally the named volume remains.

---

### Q9

Compose-managed volumes are removed.

---

### Q10

Think:

```text
Non-root
Minimal image
Secrets
Scanning
Versioning
Health
Logging
Metrics
Resource limits
CI/CD
Managed stateful services
Rollback
```

---

# 63. Real-World Backend Mental Model

When you see:

```text
Docker
```

don't just think:

```text
docker run
```

Think:

```text
Application
 ↓
Image
 ↓
Registry
 ↓
Deployment
 ↓
Container
 ↓
Network
 ↓
Dependencies
 ↓
Persistent data
 ↓
Observability
 ↓
Security
```

That's the level expected in a backend interview.

---

# 64. Docker Practical Lab Complete

Your Docker preparation now has:

```text
01 → Fundamentals
02 → Images & Dockerfile
03 → Networking & Volumes
04 → Docker Compose
05 → Dockerizing Spring Boot
06 → Security & CI/CD
07 → Practical Lab
```

At this point, don't keep adding Docker theory just for the sake of having more files.

The best next step is to actually Dockerize your e-commerce backend and use the commands from this lab.

**Key takeaway:**

> **The real Docker skill is being able to take a Spring Boot application, package it into an image, run it with MySQL and Redis, understand how the containers communicate, keep database data persistent, and debug the setup when something fails.**

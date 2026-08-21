# Docker — File 04: Docker Compose
## Spring Boot + MySQL + Redis

This is where Docker becomes useful for a real backend project.

Instead of starting every container manually, we'll define the whole local stack in one file.

---

# 1. The Problem Compose Solves

Without Compose, you might run:

```bash
docker network create backend-net

docker run ... mysql
docker run ... redis
docker run ... ecommerce-api
```

That's annoying to repeat.

With Compose:

```text
docker-compose.yml
        ↓
Spring Boot
MySQL
Redis
        ↓
docker compose up
```

One command can start the whole environment.

---

# 2. Our Backend Stack

For the e-commerce backend:

```text
                Spring Boot
                    |
          +---------+---------+
          |                   |
        MySQL               Redis
```

Ports:

```text
Spring Boot → 8080
MySQL       → 3306
Redis       → 6379
```

But remember:

> The app uses the container ports for internal communication.

---

# 3. What Is Docker Compose?

Docker Compose lets you define multiple containers/services declaratively in a YAML file.

Example:

```yaml
services:
  app:
    ...
  mysql:
    ...
  redis:
    ...
```

Then:

```bash
docker compose up
```

Compose creates the required resources and starts the services.

---

# 4. docker-compose.yml

A basic file:

```yaml
services:

  app:
    build: .
    ports:
      - "8080:8080"

  mysql:
    image: mysql:8.4

  redis:
    image: redis:7
```

This already describes three services.

---

# 5. `services`

Everything under:

```yaml
services:
```

defines an application service.

Example:

```yaml
services:
  app:
  mysql:
  redis:
```

These names become important because they can also be used for service-to-service communication.

---

# 6. Service Names = Hostnames

If we have:

```yaml
services:
  mysql:
    image: mysql:8.4

  redis:
    image: redis:7
```

Spring Boot can use:

```text
mysql
```

for MySQL and:

```text
redis
```

for Redis.

So:

```properties
spring.datasource.url=jdbc:mysql://mysql:3306/ecommerce
spring.data.redis.host=redis
```

Not:

```text
localhost
```

---

# 7. Compose Network

Compose normally creates a network for the application.

Conceptually:

```text
ecommerce_default
       |
  +----+----+
  |    |    |
 app mysql redis
```

All services on that network can communicate using their service names.

You usually don't need to manually create the network.

---

# 8. `build`

For our Spring Boot application:

```yaml
app:
  build: .
```

means:

```text
Use the Dockerfile in the current directory
```

Compose builds the image and starts the resulting container.

---

# 9. `image`

For MySQL:

```yaml
mysql:
  image: mysql:8.4
```

Compose pulls the image if it isn't available locally.

Same for Redis:

```yaml
redis:
  image: redis:7
```

---

# 10. `build` vs `image`

Simple rule:

```text
build
→ Build an image from Dockerfile

image
→ Use an existing image
```

You can also use both when you want Compose to build an image and assign/use a specific image name.

---

# 11. Port Mapping

For the application:

```yaml
ports:
  - "8080:8080"
```

Means:

```text
Host:      8080
Container: 8080
```

Access from your Mac:

```text
localhost:8080
```

---

# 12. Do We Need MySQL Ports?

Not necessarily.

If only the Spring Boot container needs MySQL:

```yaml
mysql:
  image: mysql:8.4
```

is enough for internal communication.

Spring Boot uses:

```text
mysql:3306
```

You only publish:

```yaml
ports:
  - "3306:3306"
```

if you want to connect to MySQL directly from your host.

---

# 13. Same for Redis

The application can use:

```text
redis:6379
```

without publishing Redis to your host.

You might publish it during development if you want to use:

```text
redis-cli
RedisInsight
```

from the host.

---

# 14. Environment Variables

MySQL needs initial configuration.

Example:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: root
  MYSQL_DATABASE: ecommerce
```

For a local learning environment, this is okay.

For real production:

```text
Don't commit passwords into Git.
```

Use secrets or environment-specific secure configuration.

---

# 15. Spring Boot Environment

Instead of hardcoding:

```properties
spring.datasource.url=jdbc:mysql://mysql:3306/ecommerce
```

we can use:

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

spring.data.redis.host=${REDIS_HOST}
spring.data.redis.port=${REDIS_PORT}
```

Then Compose supplies the values.

---

# 16. Compose Environment

Example:

```yaml
app:
  build: .
  environment:
    DB_URL: jdbc:mysql://mysql:3306/ecommerce
    DB_USERNAME: root
    DB_PASSWORD: root
    REDIS_HOST: redis
    REDIS_PORT: 6379
```

Now the same image can be configured differently in different environments.

---

# 17. Volumes

MySQL should not depend on the container filesystem.

Add:

```yaml
volumes:
  mysql-data:
```

Then:

```yaml
mysql:
  image: mysql:8.4
  volumes:
    - mysql-data:/var/lib/mysql
```

---

# 18. Volume Structure

```text
MySQL container
      |
      ↓
mysql-data
      |
      ↓
Persistent database files
```

Remove the container:

```text
Container gone
      |
      ↓
Volume remains
```

---

# 19. Redis Volume

If Redis is configured to persist data:

```yaml
volumes:
  redis-data:
```

and:

```yaml
redis:
  image: redis:7
  volumes:
    - redis-data:/data
```

Whether Redis needs persistence depends on how you're using it.

If Redis is only a disposable cache:

```text
Persistence may not be necessary.
```

---

# 20. Named Volumes

At the bottom:

```yaml
volumes:
  mysql-data:
  redis-data:
```

These are named Docker volumes.

Compose manages them for the project.

---

# 21. Complete Basic Compose File

A good starting point:

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
      - mysql
      - redis

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

This is enough to understand the basic architecture.

---

# 22. `depends_on`

Example:

```yaml
app:
  depends_on:
    - mysql
    - redis
```

This expresses a startup dependency.

It does **not** automatically mean:

```text
MySQL is fully ready
```

That's an important interview point.

---

# 23. The Startup Problem

Imagine:

```text
docker compose up
       ↓
MySQL starts
       ↓
Spring Boot starts
       ↓
MySQL still initializing
       ↓
Connection refused
```

The containers are running, but the database isn't ready yet.

---

# 24. Healthchecks

A better approach is to define a health check.

Example concept:

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

This checks whether MySQL is responding.

---

# 25. `depends_on` + Healthcheck

You can express:

```yaml
depends_on:
  mysql:
    condition: service_healthy
```

and similarly for Redis if an appropriate healthcheck is configured.

This makes the Compose startup behavior more meaningful.

---

# 26. Important Limitation

Even with healthchecks:

```text
Startup ordering
```

is not the same thing as:

```text
Application resilience
```

Your Spring Boot application should still handle temporary dependency failures appropriately.

---

# 27. Redis Healthcheck

A simple Redis healthcheck can use:

```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 10s
  timeout: 5s
  retries: 5
```

The image must contain the required command.

---

# 28. Application Healthcheck

If the Spring Boot image contains a suitable health endpoint/tooling:

```text
/actuator/health
```

can be used for application health.

But don't assume `curl` exists in a minimal runtime image.

---

# 29. `restart`

Compose can define restart behavior.

Example:

```yaml
restart: unless-stopped
```

Useful for local services that should come back after an unexpected stop.

For production orchestration, restart behavior is usually handled by the orchestrator/platform.

---

# 30. Compose Commands

Start:

```bash
docker compose up
```

Start in background:

```bash
docker compose up -d
```

Stop/remove containers:

```bash
docker compose down
```

See status:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

---

# 31. Rebuild the Application

If your Dockerfile or application changes:

```bash
docker compose build
```

Then:

```bash
docker compose up
```

Or:

```bash
docker compose up --build
```

This is convenient during development.

---

# 32. Restart One Service

Example:

```bash
docker compose restart app
```

Useful when:

```text
Configuration/runtime state changed
```

but you don't want to manually restart every service.

---

# 33. Stop Without Removing

```bash
docker compose stop
```

This stops services while preserving their containers.

---

# 34. Down vs Stop

Important:

```text
docker compose stop
→ Stop containers

docker compose down
→ Stop and remove Compose-managed containers/networks
```

Named volumes are normally retained unless you explicitly request volume removal.

---

# 35. Dangerous Command

Be careful with:

```bash
docker compose down -v
```

The `-v` removes Compose-managed named volumes.

For our MySQL volume:

```text
mysql-data
```

that can mean:

```text
Database data deleted
```

Do not use it casually.

---

# 36. Compose Logs

All services:

```bash
docker compose logs
```

Only app:

```bash
docker compose logs app
```

Only MySQL:

```bash
docker compose logs mysql
```

Follow:

```bash
docker compose logs -f app
```

---

# 37. Execute Into a Service

```bash
docker compose exec app sh
```

For MySQL:

```bash
docker compose exec mysql sh
```

For Redis:

```bash
docker compose exec redis redis-cli
```

The exact shell/tool availability depends on the image.

---

# 38. Compose Project

Compose groups resources into a project.

You can have:

```text
Project A
 ├── app
 ├── mysql
 └── redis
```

and:

```text
Project B
 ├── app
 ├── mysql
 └── redis
```

without necessarily mixing their resources.

---

# 39. Environment File

Compose can use a `.env` file.

Example:

```text
DB_USERNAME=root
DB_PASSWORD=root
DB_NAME=ecommerce
```

Then:

```yaml
environment:
  DB_USERNAME: ${DB_USERNAME}
  DB_PASSWORD: ${DB_PASSWORD}
```

---

# 40. Don't Commit Secrets

For local learning:

```text
.env
```

can be convenient.

For real credentials:

```text
Secret manager
CI/CD secret store
Cloud secret service
Orchestrator secrets
```

are better choices.

Add sensitive local files to:

```text
.gitignore
```

where appropriate.

---

# 41. Environment Variable Precedence

Compose supports multiple ways to provide configuration.

The exact precedence can become complex.

For interviews, remember:

```text
Runtime/environment configuration
can override image defaults.
```

When debugging, inspect the actual environment rather than assuming which source won.

---

# 42. Compose and Dockerfile

Dockerfile:

```text
How to build the application image
```

Compose:

```text
How multiple services run together
```

Example:

```text
Dockerfile
    ↓
ecommerce-api image

Compose
    ↓
ecommerce-api
mysql
redis
```

---

# 43. Compose and Networking

Compose automatically creates a network for the project.

So:

```text
app
mysql
redis
```

can communicate through:

```text
mysql
redis
```

as DNS names.

You don't normally need:

```text
172.x.x.x
```

addresses.

---

# 44. Custom Network

You can explicitly define:

```yaml
networks:
  backend:
```

Then:

```yaml
services:
  app:
    networks:
      - backend

  mysql:
    networks:
      - backend

  redis:
    networks:
      - backend
```

This gives you more control.

---

# 45. Two-Network Architecture

Example:

```yaml
networks:
  frontend:
  backend:
```

Potential design:

```text
frontend
   |
frontend-net
   |
API
   |
backend-net
 /       \
MySQL   Redis
```

The API is connected to both networks.

MySQL only needs:

```text
backend
```

---

# 46. Why This Is Useful

It reduces unnecessary network access.

For example:

```text
Frontend
  X
MySQL
```

while:

```text
Frontend
  ↓
API
  ↓
MySQL
```

is allowed.

---

# 47. MySQL Initialization

Official MySQL images can use environment variables for initial setup:

```yaml
MYSQL_DATABASE
MYSQL_USER
MYSQL_PASSWORD
MYSQL_ROOT_PASSWORD
```

Initialization generally happens when the database directory is initialized.

---

# 48. Important MySQL Volume Detail

If the volume already contains an initialized database:

```text
Changing MYSQL_DATABASE
```

does not necessarily recreate the database.

Why?

Because the initialization logic is primarily for a fresh data directory.

This is a common Docker/MySQL surprise.

---

# 49. Resetting Local MySQL

If you intentionally want a fresh local database:

```bash
docker compose down -v
docker compose up
```

But remember:

```text
-v = delete Compose-managed volumes
```

Only do this when losing the local database is acceptable.

---

# 50. Redis Configuration

Basic:

```yaml
redis:
  image: redis:7
```

For a cache:

```text
No persistence may be required.
```

For critical Redis state:

```text
Configure persistence
+
volume
+
backup strategy
```

---

# 51. Redis Cache in Local Development

For your e-commerce project:

```text
MySQL
→ Source of truth

Redis
→ Cache
```

If Redis disappears:

```text
Application
 ↓
Cache miss
 ↓
MySQL
 ↓
Repopulate Redis
```

This makes Redis easier to treat as replaceable.

---

# 52. Application Dependencies

Your application depends on:

```text
MySQL
Redis
```

But your architecture should not assume these services are immortal.

The application should handle:

```text
Connection failure
Timeout
Restart
Temporary unavailability
```

appropriately.

---

# 53. Compose vs Kubernetes

Compose:

```text
Local development
Small test environments
Learning
Simple multi-container applications
```

Kubernetes:

```text
Large distributed deployments
Scheduling
Service discovery
Scaling
Self-healing
Rolling deployments
```

Don't treat Compose as a production Kubernetes replacement.

---

# 54. Compose vs Dockerfile

A common interview question.

Answer:

> "A Dockerfile defines how to build an image for an application. Docker Compose defines how multiple containers, their configuration, networks and volumes work together."

---

# 55. `docker compose up`

What happens conceptually?

```text
Read compose file
 ↓
Create network
 ↓
Create volumes if required
 ↓
Build/pull images
 ↓
Create containers
 ↓
Start services
```

Actual behavior depends on the configuration and whether resources already exist.

---

# 56. `docker compose down`

Conceptually:

```text
Stop services
 ↓
Remove containers
 ↓
Remove project network
```

Named volumes are retained unless explicitly removed.

---

# 57. `docker compose up --build`

Useful when:

```text
Dockerfile changed
Application code changed
Dependencies changed
```

It ensures the application image is rebuilt before starting the stack.

---

# 58. `docker compose pull`

Pull images defined by the Compose configuration.

Example:

```bash
docker compose pull
```

Useful when updating external service images.

---

# 59. `docker compose config`

Useful for inspecting the resolved Compose configuration:

```bash
docker compose config
```

This can help debug:

```text
Environment interpolation
Networks
Volumes
Service configuration
```

---

# 60. Compose Healthcheck Debugging

If:

```text
mysql
```

is unhealthy:

```bash
docker compose ps
```

Then:

```bash
docker compose logs mysql
```

Check:

```text
Credentials
Initialization
Volume
Port
Healthcheck command
```

---

# 61. Common Problem: App Starts Too Early

Symptoms:

```text
Communications link failure
Connection refused
Redis connection refused
```

Fix:

```text
Healthchecks
depends_on conditions
Application retries
Proper readiness handling
```

Don't simply add arbitrary:

```text
sleep 30
```

and call the problem solved.

---

# 62. Common Problem: App Uses localhost

Configuration:

```text
jdbc:mysql://localhost:3306/ecommerce
```

Inside the app container:

```text
localhost
```

means the app container.

Fix:

```text
jdbc:mysql://mysql:3306/ecommerce
```

---

# 63. Common Problem: Port Already in Use

Error:

```text
Bind for 0.0.0.0:8080 failed
```

Check:

```bash
docker ps
```

and other host processes.

Either stop the conflicting service or change the host port:

```yaml
ports:
  - "8081:8080"
```

---

# 64. Common Problem: Database Data Doesn't Reset

You changed:

```yaml
MYSQL_DATABASE: ecommerce2
```

but the old database remains.

Likely:

```text
Existing volume
```

If this is only local test data:

```bash
docker compose down -v
```

then recreate.

---

# 65. Common Problem: Permission Denied on Volume

Check:

```text
Container user
UID/GID
Volume ownership
Host filesystem
```

Don't immediately run everything as root.

---

# 66. Common Problem: Redis Data Missing

If Redis was being used only as a cache:

```text
Data loss may be acceptable.
```

If Redis holds important state:

```text
Persistence
Volume
Backups
HA
```

need to be designed intentionally.

---

# 67. A Better Local Compose File

Here is a more realistic starting point:

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

This is suitable as a local learning setup, not a production security configuration.

---

# 68. Where Docker Compose Fits in Your Project

Your workflow can become:

```text
Spring Boot code
      ↓
Dockerfile
      ↓
Compose
      ↓
+-----------+
| Spring App|
| MySQL     |
| Redis     |
+-----------+
```

Then:

```bash
docker compose up --build
```

starts the local backend environment.

---

# 69. Recommended Project Structure

For your e-commerce backend:

```text
ecommercebackend/
│
├── src/
├── pom.xml
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
└── README.md
```

Keep deployment-related configuration understandable and documented.

---

# 70. What Goes in README

Useful local setup instructions:

```text
Prerequisites
Docker installation
Environment variables
Start command
Stop command
Application URL
Database details
Redis details
Troubleshooting
```

Example:

```bash
docker compose up --build
```

Then:

```text
API → http://localhost:8080
```

---

# 71. Interview Question

### Why use Docker Compose?

Answer:

> "For local development, Compose lets me define the complete application stack declaratively. Instead of manually creating Spring Boot, MySQL and Redis containers, I can start them together with one command and keep their networking, volumes and configuration in version-controlled code."

---

# 72. Interview Question

### Does `depends_on` mean the database is ready?

Answer:

> "Not by itself. It mainly controls service dependency/startup behavior. For readiness, I'd use healthchecks and appropriate application retry or resilience logic."

---

# 73. Interview Question

### What happens to a named volume after `docker compose down`?

Answer:

> "Normally the named volume remains. If I run `docker compose down -v`, Compose removes the project's named volumes as well."

---

# 74. Interview Question

### How does Compose provide service discovery?

Answer:

> "Compose creates a project network and Docker DNS lets services reach each other using their service names, such as `mysql` or `redis`."

---

# 75. Interview Question

### Why use healthchecks?

Answer:

> "A healthcheck lets us distinguish a running container from a service that is actually ready to handle requests."

---

# 76. Interview Question

### Would you run MySQL in Docker in production?

Good answer:

> "It can be done, but for many production environments I'd prefer a managed database service because backups, failover, storage, patching and operational management are handled more reliably. For local development and testing, Dockerized MySQL is very useful."

---

# 77. Interview Question

### Would you persist Redis?

Answer:

> "It depends on the role Redis plays. If it's only a cache, losing the data may be acceptable. If Redis contains important state or durable streams, I'd design persistence, backups and high availability accordingly."

---

# 78. Interview Scenario

### `docker compose up` works, but Spring Boot cannot connect to MySQL.

Walk through:

```text
1. docker compose ps
2. mysql logs
3. MySQL health status
4. JDBC hostname
5. Network
6. Credentials
7. Database name
8. Startup timing
```

Most common mistake:

```text
localhost
```

instead of:

```text
mysql
```

---

# 79. Interview Scenario

### You changed the MySQL password in Compose, but the old password still works.

Likely:

```text
Existing MySQL volume
```

The initialization variables aren't reapplying to an already initialized database.

For local development, recreate the volume if data can be discarded.

---

# 80. Interview Scenario

### You run `docker compose down -v` and your database disappears.

That's expected.

```text
-v
→ remove volumes
```

This is why destructive commands must be used carefully.

---

# 81. Interview Scenario

### Redis keeps restarting.

Check:

```text
docker compose ps
docker compose logs redis
healthcheck
configuration
memory
permissions
```

Don't assume the healthcheck is the cause just because it reports unhealthy.

---

# 82. Interview Scenario

### API needs to access MySQL, but users should not access MySQL directly.

Design:

```text
Browser
   |
   ↓
Spring Boot
   |
backend network
   |
MySQL
```

Do not publish MySQL's port unless there is a specific operational requirement.

---

# 83. Interview Scenario

### Need frontend + API + database.

Use:

```text
frontend-net
backend-net
```

Architecture:

```text
Browser
  ↓
Frontend
  ↓
API
  ↓
MySQL
```

with the API connected to both networks and MySQL only connected to the backend network.

---

# 84. Interview Scenario

### You need a clean database for testing.

For disposable local data:

```bash
docker compose down -v
docker compose up
```

For real environments:

```text
Never blindly delete volumes.
```

Use proper database reset/migration procedures.

---

# 85. Compose Mental Model

Remember:

```text
Dockerfile
→ Build one application's image

Compose
→ Run the whole local application stack
```

For your project:

```text
Dockerfile
    ↓
Spring Boot image

docker-compose.yml
    ↓
Spring Boot
MySQL
Redis
Network
Volumes
Healthchecks
```

---

# 86. Your E-commerce Backend

The target local setup:

```text
                    localhost:8080
                          |
                          ↓
                    Spring Boot
                          |
                     Docker DNS
                    /           \
                   /             \
              mysql:3306       redis:6379
                  |                 |
             mysql-data        optional data
```

The important part:

```text
Browser → localhost:8080
App → mysql:3306
App → redis:6379
```

Different contexts, different addresses.

---

# 87. Quick Command Sheet

Start:

```bash
docker compose up -d
```

Build + start:

```bash
docker compose up -d --build
```

Status:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs -f
```

App logs:

```bash
docker compose logs -f app
```

Shell:

```bash
docker compose exec app sh
```

Stop:

```bash
docker compose stop
```

Remove containers/network:

```bash
docker compose down
```

Remove containers + volumes:

```bash
docker compose down -v
```

Use the last command carefully.

---

# 88. File 04 Checklist

```text
□ Docker Compose
□ docker-compose.yml
□ services
□ build
□ image
□ ports
□ environment
□ volumes
□ networks
□ service names
□ Docker DNS
□ depends_on
□ healthchecks
□ restart
□ Compose commands
□ .env
□ Compose networking
□ Multiple networks
□ MySQL initialization
□ Redis configuration
□ Persistent volumes
□ Compose vs Dockerfile
□ Compose vs Kubernetes
□ Troubleshooting
□ E-commerce stack
□ Interview questions
```

---

# 89. What Comes Next

```text
File 05 → Dockerizing Your Spring Boot Application
```

We'll build the actual flow:

```text
Spring Boot project
      ↓
Maven
      ↓
Dockerfile
      ↓
Image
      ↓
Compose
      ↓
MySQL
      ↓
Redis
      ↓
API
```

We'll also cover:

```text
application.properties
environment variables
Docker profiles
JAR packaging
container logs
healthchecks
database connection
Redis connection
debugging
production-style Dockerfile
```

**Key takeaway:**

> Docker Compose turns a collection of containers into one reproducible local environment. For your backend, the most important practical idea is that the application talks to `mysql:3306` and `redis:6379` over the Compose network, while users access the API through the published host port such as `localhost:8080`.

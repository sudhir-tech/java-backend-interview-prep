# Docker — File 03: Networking, Volumes & Container Communication

This file focuses on one of the most important practical Docker areas for backend developers:

```text
Container networking
Docker DNS
Port publishing
Container-to-container communication
Volumes
Bind mounts
Named volumes
MySQL persistence
Redis persistence
Spring Boot → MySQL
Spring Boot → Redis
Network troubleshooting
Interview scenarios
```

---

# 1. Docker Networking

Containers are isolated, but applications often need to communicate.

Typical backend stack:

```text
Spring Boot
    |
    +---- MySQL
    |
    +---- Redis
```

Docker networks provide the communication layer.

---

# 2. Why Docker Networks?

Without a suitable network:

```text
Container A
     X
Container B
```

With a shared Docker network:

```text
Container A
     |
Docker Network
     |
Container B
```

Containers can communicate using network addresses and Docker DNS.

---

# 3. Default Bridge Network

Docker provides a default bridge network.

However, for multi-container applications, a user-defined bridge network is generally preferred because it provides better built-in service discovery and isolation behavior.

---

# 4. Create a Network

```bash
docker network create backend-net
```

Check networks:

```bash
docker network ls
```

Inspect:

```bash
docker network inspect backend-net
```

---

# 5. Run Containers on a Network

MySQL:

```bash
docker run -d \
  --name mysql \
  --network backend-net \
  mysql
```

Redis:

```bash
docker run -d \
  --name redis \
  --network backend-net \
  redis
```

Spring Boot:

```bash
docker run -d \
  --name ecommerce-api \
  --network backend-net \
  ecommerce-api:1.0
```

Now all three are connected to:

```text
backend-net
```

---

# 6. Docker DNS

On a user-defined Docker network, containers can generally resolve each other using container/service names.

Example:

```text
Spring Boot
     |
     ↓
mysql
```

The hostname can be:

```text
mysql
```

instead of a hard-coded container IP.

---

# 7. Why Not Use Container IP?

Bad configuration:

```text
jdbc:mysql://172.18.0.5:3306/ecommerce
```

Container IPs can change when containers are recreated.

Better:

```text
jdbc:mysql://mysql:3306/ecommerce
```

Docker's internal DNS resolves:

```text
mysql
```

to the current container address.

---

# 8. Container Name as Hostname

Example:

```bash
docker run -d \
  --name redis \
  --network backend-net \
  redis
```

Another container on the same network can use:

```text
redis:6379
```

as the Redis endpoint.

---

# 9. Service Name in Compose

Docker Compose commonly provides service-name DNS.

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

The application can generally use:

```text
mysql
redis
```

as hostnames.

Compose networking will be covered in detail later.

---

# 10. localhost Inside a Container

This is critical.

Inside:

```text
Spring Boot container
```

the address:

```text
localhost
```

means:

```text
Spring Boot container itself
```

It does not automatically mean:

```text
MySQL container
```

or:

```text
Redis container
```

---

# 11. Correct Example

Wrong:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce
```

when MySQL runs in another container.

Correct:

```properties
spring.datasource.url=jdbc:mysql://mysql:3306/ecommerce
```

assuming the containers share a network and MySQL is reachable under the hostname `mysql`.

---

# 12. Spring Boot → Redis

Wrong:

```properties
spring.data.redis.host=localhost
```

if Redis runs in another container.

Correct:

```properties
spring.data.redis.host=redis
spring.data.redis.port=6379
```

assuming the Redis container/service is named `redis`.

---

# 13. Container Port vs Host Port

Suppose:

```bash
docker run -p 9090:8080 ecommerce-api
```

This means:

```text
Host:      9090
Container: 8080
```

A browser on the host can use:

```text
localhost:9090
```

But another container on the same Docker network generally uses:

```text
ecommerce-api:8080
```

not:

```text
ecommerce-api:9090
```

---

# 14. Important Rule

For container-to-container communication:

```text
Use container/service port.
```

For host-to-container communication:

```text
Use published host port.
```

Example:

```text
Host → Spring Boot
localhost:9090 → container:8080

Spring Boot → MySQL
mysql:3306
```

---

# 15. Do You Need to Publish MySQL Port?

Not necessarily.

If only Spring Boot needs MySQL:

```text
Spring Boot
   ↓
Docker network
   ↓
MySQL
```

you may not need:

```bash
-p 3306:3306
```

Publishing ports is primarily for access from outside the container network.

---

# 16. Do You Need to Publish Redis Port?

Similarly, if only application containers need Redis:

```text
Spring Boot
   ↓
Docker network
   ↓
Redis
```

you may not need:

```bash
-p 6379:6379
```

---

# 17. Exposing vs Publishing

Dockerfile:

```dockerfile
EXPOSE 8080
```

means:

```text
Documentation / metadata
```

It does not publish the port.

Publishing:

```bash
docker run -p 8080:8080 app
```

creates host-to-container port forwarding.

---

# 18. Network Isolation

You can create separate networks.

Example:

```text
frontend-net
backend-net
```

Potential architecture:

```text
Frontend
   |
frontend-net
   |
API
   |
backend-net
   |
MySQL
Redis
```

The API can be connected to both networks.

---

# 19. Why Multiple Networks?

Network separation can reduce unnecessary connectivity.

Example:

```text
Internet-facing frontend
        |
       API
        |
   private backend
        |
    DB / Redis
```

MySQL does not need to be directly accessible by the frontend.

---

# 20. Network Security Principle

Use:

```text
Least connectivity
```

Only allow services to communicate with what they actually need.

---

# 21. Docker Network Types

Common Docker network drivers include:

```text
bridge
host
none
overlay
```

The exact availability and behavior depend on the Docker environment.

---

# 22. Bridge Network

Common for local multi-container applications.

Conceptually:

```text
Host
 |
Docker bridge
 |
+---- Container A
|
+---- Container B
```

User-defined bridge networks provide useful container-to-container DNS behavior.

---

# 23. Host Network

With host networking, the container can share the host's network namespace in supported environments.

Conceptually:

```text
Host network
     |
Container
```

This removes some network isolation.

Use it only when there is a specific reason.

---

# 24. None Network

A container using:

```text
none
```

has networking disabled/minimized according to the Docker environment.

Useful for workloads that do not need network access.

---

# 25. Overlay Network

Overlay networks allow container communication across multiple Docker hosts in supported distributed Docker environments.

They are more relevant to:

```text
Docker Swarm
Distributed deployments
```

than basic local Docker development.

---

# 26. Network Inspection

Useful command:

```bash
docker network inspect backend-net
```

It can show:

```text
Connected containers
Network configuration
Subnet
Gateway
```

---

# 27. Container Inspection

```bash
docker inspect ecommerce-api
```

Useful information includes:

```text
Network settings
IP addresses
Mounted volumes
Environment
Ports
```

---

# 28. Troubleshooting Network Connectivity

If:

```text
Spring Boot → MySQL
```

fails, check:

```text
1. Is MySQL running?
2. Are both on the same network?
3. Is hostname correct?
4. Is container port correct?
5. Is MySQL ready?
6. Are credentials correct?
7. Is firewall/network policy involved?
```

---

# 29. Check Container Status

```bash
docker ps
```

If MySQL isn't running:

```bash
docker ps -a
```

Then:

```bash
docker logs mysql
```

---

# 30. Check Network

```bash
docker network inspect backend-net
```

Verify:

```text
mysql
redis
ecommerce-api
```

are connected to the expected network.

---

# 31. Check DNS

From an application/debug container on the same network, test whether:

```text
mysql
```

or:

```text
redis
```

resolves correctly.

Minimal images may not include:

```text
ping
nslookup
curl
```

so use suitable debugging tools or a temporary diagnostic container.

---

# 32. Check Application Configuration

Spring Boot:

```properties
spring.datasource.url=jdbc:mysql://mysql:3306/ecommerce
spring.data.redis.host=redis
spring.data.redis.port=6379
```

Check that environment variables are actually being injected.

---

# 33. MySQL Startup Timing

A common problem:

```text
Spring Boot starts
      ↓
MySQL container starts
      ↓
MySQL isn't ready yet
      ↓
Connection fails
```

Container running does not necessarily mean the application inside is ready.

---

# 34. MySQL Readiness

MySQL may need time to:

```text
Initialize database
Create users
Create schema
Start accepting connections
```

Therefore applications should use appropriate:

```text
Retry
Readiness checks
Startup orchestration
```

rather than assuming immediate availability.

---

# 35. Redis Startup Timing

Similarly:

```text
Redis container running
```

does not necessarily mean your application should blindly assume all dependencies are ready.

Use appropriate health/readiness mechanisms.

---

# 36. Why Docker DNS Is Better Than IPs

Container IP:

```text
172.18.0.10
```

can change.

Service/container name:

```text
mysql
```

remains the logical endpoint.

Therefore:

```text
Application
 ↓
mysql
 ↓
Docker DNS
 ↓
Current container IP
```

---

# 37. Volumes

Containers are ephemeral.

For persistent data:

```text
Volume
```

can outlive the container.

Conceptually:

```text
Container
    |
    ↓
Volume
    |
    ↓
Persistent data
```

---

# 38. Named Volume

Create:

```bash
docker volume create mysql-data
```

List:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect mysql-data
```

---

# 39. Mount Named Volume

```bash
docker run -d \
  --name mysql \
  -v mysql-data:/var/lib/mysql \
  mysql
```

The exact data directory depends on the image.

For official MySQL images, `/var/lib/mysql` is the standard database data location.

---

# 40. Why MySQL Needs a Volume

Without persistence:

```text
MySQL container
 ↓
Container removed
 ↓
Database files removed with container filesystem
```

With volume:

```text
MySQL container
 ↓
mysql-data
 ↓
Container removed
 ↓
Volume remains
```

---

# 41. Redis Volume

If Redis persistence is enabled and you want its persisted files to survive container replacement:

```bash
docker run -d \
  --name redis \
  -v redis-data:/data \
  redis
```

The image and Redis persistence configuration determine what is actually written to `/data`.

---

# 42. Volume Lifecycle

```text
docker volume create
        ↓
Volume exists
        ↓
Container mounts volume
        ↓
Container removed
        ↓
Volume still exists
```

Unless explicitly removed.

---

# 43. Remove Volume

```bash
docker volume rm mysql-data
```

This can destroy persistent data.

Be extremely careful in production.

---

# 44. Prune Volumes

Docker supports volume cleanup commands.

Be careful with:

```bash
docker volume prune
```

because unused volumes may still contain valuable data.

Never run destructive cleanup commands blindly.

---

# 45. Named Volume vs Container Filesystem

### Container filesystem

```text
Ephemeral
```

### Named volume

```text
Persistent
Docker-managed
```

Use a volume for state that must survive container replacement.

---

# 46. Bind Mount

Example:

```bash
docker run \
  -v $(pwd)/config:/app/config \
  my-app
```

This maps:

```text
Host:
./config

Container:
/app/config
```

---

# 47. Named Volume vs Bind Mount

### Named volume

```text
Managed by Docker
Good for persistent application data
```

### Bind mount

```text
Maps explicit host path
Useful for development/config/source sharing
```

---

# 48. Development Bind Mount

For development:

```text
Host source code
      ↓
Container
```

can enable rapid iteration in some workflows.

For production, application images are generally preferable to mounting the source code from the host.

---

# 49. Volume Permissions

A common issue:

```text
Container user
     ↓
Cannot write
     ↓
Permission denied
```

Investigate:

```text
UID/GID
Volume ownership
Filesystem permissions
Container user
```

This becomes especially important when running containers as non-root.

---

# 50. Database Persistence

A database container should generally use:

```text
Named volume
or
Managed database service
```

rather than relying on its writable container layer.

---

# 51. Production Database Recommendation

For many production systems:

```text
Spring Boot
     |
Managed MySQL
```

rather than:

```text
Spring Boot
     |
MySQL container on application host
```

Managed database services often provide:

```text
Backups
Failover
Monitoring
Storage management
Patching
```

depending on the provider.

---

# 52. Docker and State

Good architecture:

```text
Application container
→ Stateless

MySQL
→ Persistent storage

Redis
→ Depends on use case
```

If Redis is only a cache:

```text
Redis data can often be rebuilt.
```

If Redis stores critical state:

```text
Persistence + backups + HA
```

may be required.

---

# 53. Mounting Configuration

A container can receive configuration through:

```text
Environment variables
Volumes
Secrets
Config files
```

For Spring Boot, environment variables are often convenient for deployment-specific settings.

---

# 54. Environment-Based Configuration

Example:

```text
DEV
DB_HOST=localhost

DOCKER
DB_HOST=mysql

PROD
DB_HOST=managed-mysql.example
```

Same application image:

```text
Different runtime configuration
```

---

# 55. Spring Profiles

You can use:

```text
application.properties
application-docker.properties
application-prod.properties
```

and activate:

```text
SPRING_PROFILES_ACTIVE=docker
```

This can be useful, but avoid duplicating secrets in profile files.

---

# 56. Docker Network Architecture for E-commerce

A useful local architecture:

```text
                    Host
                     |
                  :8080
                     |
              Spring Boot API
                     |
               backend-net
              /             \
           MySQL            Redis
           3306             6379
```

Only Spring Boot needs to access:

```text
MySQL
Redis
```

---

# 57. Frontend + Backend Architecture

```text
Browser
   |
   ↓
Frontend container
   |
   ↓
API container
   |
backend-net
 /          \
MySQL      Redis
```

You can separate:

```text
frontend network
backend network
```

so the database isn't directly connected to the frontend.

---

# 58. Network Example

```text
frontend-net
    |
Frontend
    |
    +---- API
            |
        backend-net
          /     \
       MySQL   Redis
```

This is a useful system-design pattern.

---

# 59. Port Publishing in This Architecture

Potentially:

```text
Frontend
→ published port

API
→ published port if external access is required

MySQL
→ no host port required

Redis
→ no host port required
```

The API can communicate with MySQL and Redis through the private Docker network.

---

# 60. Why Avoid Publishing Internal Ports?

Publishing:

```text
3306
6379
```

to the host unnecessarily increases the access surface.

If only containers need the services:

```text
Keep them internal to the Docker network.
```

---

# 61. Network Aliases

Docker networks can support aliases.

Conceptually:

```text
mysql
database
```

could resolve to the same service/container depending on configuration.

This can be useful for compatibility during migrations.

---

# 62. Multiple Networks per Container

A container can be connected to multiple networks.

Example:

```text
API
 ├── frontend-net
 └── backend-net
```

while:

```text
MySQL
 └── backend-net
```

This creates controlled connectivity.

---

# 63. Network Troubleshooting Checklist

When container A cannot reach B:

```text
□ Both containers running?
□ Same Docker network?
□ Correct hostname?
□ Correct container port?
□ Application listening on correct interface?
□ Service ready?
□ Credentials correct?
□ Network policy/firewall?
□ DNS resolution?
□ Logs?
```

---

# 64. Common Mistake: localhost

Scenario:

```text
Spring Boot container
MySQL container
```

Configuration:

```text
localhost:3306
```

Result:

```text
Spring Boot tries its own container
```

Fix:

```text
mysql:3306
```

---

# 65. Common Mistake: Host Port

Suppose:

```bash
mysql -p 13306:3306
```

Another container should normally use:

```text
mysql:3306
```

not:

```text
mysql:13306
```

because:

```text
13306
```

is the host-published port.

---

# 66. Common Mistake: Container IP

Avoid hardcoding:

```text
172.18.0.5
```

because recreation can change the IP.

Use:

```text
mysql
redis
```

or Compose service names.

---

# 67. Common Mistake: Container Running ≠ Ready

```text
docker ps
```

shows:

```text
Up
```

but the application/database may still be initializing.

Use:

```text
Health checks
Readiness checks
Retries
```

where appropriate.

---

# 68. Common Mistake: Data in Container

Bad:

```text
MySQL
 ↓
Container filesystem only
```

Better:

```text
MySQL
 ↓
Named volume
```

---

# 69. Common Mistake: Delete Everything

Commands like:

```bash
docker system prune
docker volume prune
```

can remove resources you did not intend to delete.

Always understand exactly what will be removed.

---

# 70. Useful Network Commands

```bash
docker network ls
```

```bash
docker network inspect backend-net
```

```bash
docker network create backend-net
```

Connect:

```bash
docker network connect backend-net my-app
```

Disconnect:

```bash
docker network disconnect backend-net my-app
```

---

# 71. Useful Volume Commands

```bash
docker volume ls
```

```bash
docker volume create mysql-data
```

```bash
docker volume inspect mysql-data
```

```bash
docker volume rm mysql-data
```

---

# 72. Volume Backup Concept

A named volume can be backed up by mounting it into a suitable helper container and creating an archive.

Conceptually:

```text
Volume
 ↓
Backup container
 ↓
Archive
 ↓
External storage
```

For production databases, prefer database-aware backup mechanisms rather than treating a live database volume snapshot as automatically consistent.

---

# 73. MySQL Backup vs Volume Backup

Important distinction:

```text
Volume copy
→ Filesystem-level copy

mysqldump / logical backup
→ Database-aware backup
```

A production backup strategy should account for transactional consistency and the database's backup capabilities.

---

# 74. Redis Backup

Depending on Redis configuration:

```text
RDB
AOF
```

can create persistent files.

Those files can then be backed up using an appropriate storage strategy.

---

# 75. Docker Network and Security

Remember:

```text
Network connectivity
≠
Authorization
```

Even if only backend containers can reach MySQL:

```text
Database credentials
+
Database authorization
```

are still required.

Use multiple security layers.

---

# 76. Docker Network and TLS

For some local networks:

```text
Docker private network
```

may be considered sufficient for local development.

Production requirements may still require:

```text
TLS
```

especially across trust boundaries.

Always follow your infrastructure/security requirements.

---

# 77. Spring Boot Connection Flow

MySQL:

```text
Spring Boot
 ↓
mysql:3306
 ↓
Docker DNS
 ↓
MySQL container
```

Redis:

```text
Spring Boot
 ↓
redis:6379
 ↓
Docker DNS
 ↓
Redis container
```

---

# 78. Docker Compose Preview

Compose will make this much easier.

Instead of:

```bash
docker network create ...
docker run mysql ...
docker run redis ...
docker run app ...
```

you will define:

```yaml
services:
  app:
    ...
  mysql:
    ...
  redis:
    ...
```

and run:

```bash
docker compose up
```

Compose gets its own dedicated file in a later topic.

---

# 79. Interview Question

### How do containers communicate?

Answer:

> "Containers can communicate over Docker networks. On a user-defined network, they can generally discover each other through Docker DNS using container or service names."

---

# 80. Interview Question

### Why shouldn't you use container IP addresses?

Answer:

> "Container IPs can change when containers are recreated. Using Docker DNS and stable service/container names avoids hardcoding ephemeral addresses."

---

# 81. Interview Question

### Why doesn't localhost work between containers?

Answer:

> "Because localhost refers to the current container's network namespace. To reach another container, I use its service or container hostname over a shared Docker network."

---

# 82. Interview Question

### Host port vs container port?

Answer:

> "The container port is where the application listens inside the container. The host port is the port published on the host. Container-to-container communication normally uses the container port."

---

# 83. Interview Question

### Do I need to expose MySQL to the host?

Answer:

> "Not if only other containers need MySQL. They can access it through the Docker network. I would publish 3306 only when host-level access is actually required."

---

# 84. Interview Question

### Volume vs bind mount?

Answer:

> "A named volume is managed by Docker and is commonly used for persistent application data. A bind mount maps an explicit host path and is useful when I need direct host filesystem access, especially in development."

---

# 85. Interview Question

### Why use volumes for MySQL?

Answer:

> "Because the container lifecycle should not determine the lifecycle of database data. A named volume lets the database data survive container recreation."

---

# 86. Interview Question

### What happens if MySQL container is deleted?

Answer:

> "If the database stored data only in the container filesystem, the data can be lost. If the data is stored in a persistent volume or external database, deleting the container does not delete the underlying data."

---

# 87. Interview Scenario

### Spring Boot gets `Communications link failure`.

Check:

```text
JDBC hostname
Docker network
MySQL container status
MySQL readiness
Port
Credentials
Database name
```

If MySQL is another container:

```text
jdbc:mysql://mysql:3306/ecommerce
```

not:

```text
jdbc:mysql://localhost:3306/ecommerce
```

---

# 88. Interview Scenario

### Redis connection refused.

Check:

```text
Redis running?
Same network?
Host = redis?
Port = 6379?
Redis ready?
Authentication?
TLS?
Application configuration?
```

---

# 89. Interview Scenario

### Container can connect to MySQL by IP but not by name.

Likely investigate:

```text
Docker DNS
Network configuration
Container network membership
Hostname/service name
```

Don't solve the problem by permanently hardcoding the IP.

---

# 90. Interview Scenario

### MySQL data disappeared after redeployment.

Likely:

```text
No persistent volume
```

or:

```text
Different/new volume mounted
```

Check:

```bash
docker volume ls
docker volume inspect mysql-data
docker inspect mysql
```

---

# 91. Interview Scenario

### Frontend can access MySQL directly.

This may be an architectural/security issue.

Prefer:

```text
Frontend
 ↓
API
 ↓
MySQL
```

rather than:

```text
Frontend
 ↓
MySQL
```

Databases should generally not be exposed directly to browsers.

---

# 92. Interview Scenario

### Need to isolate database from frontend.

Use separate networks:

```text
frontend-net
    |
Frontend
    |
    API
    |
backend-net
 /       \
MySQL   Redis
```

Only API connects to backend services.

---

# 93. Interview Scenario

### API cannot reach Redis after Redis container restart.

Check:

```text
Hostname
Network
Client connection handling
Redis readiness
Application retry/reconnection
```

If using a stable service/container name, the endpoint remains logical even though the container IP may change.

---

# 94. Production Mental Model

```text
Networking
→ Who can talk to whom?

DNS
→ How do they find each other?

Ports
→ Where does each service listen?

Volumes
→ What data must survive?

Configuration
→ What changes between environments?

Security
→ Who is allowed to connect?
```

---

# 95. E-commerce Docker Architecture

A strong local architecture:

```text
                    Browser
                       |
                    :8080
                       |
                Spring Boot API
                       |
                  backend-net
                 /           \
              MySQL         Redis
            :3306           :6379
               |               |
           mysql-data       redis-data
```

For production:

```text
Spring Boot containers
       |
Managed MySQL
       |
Managed Redis / Redis Cluster
```

may be more appropriate depending on scale and requirements.

---

# 96. Practical Exercise

Create:

```bash
docker network create backend-net
```

Run Redis:

```bash
docker run -d \
  --name redis \
  --network backend-net \
  redis
```

Run MySQL:

```bash
docker run -d \
  --name mysql \
  --network backend-net \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=ecommerce \
  -v mysql-data:/var/lib/mysql \
  mysql
```

Then run your Spring Boot image on:

```text
backend-net
```

and configure:

```text
mysql:3306
redis:6379
```

---

# 97. Practical Troubleshooting

Check:

```bash
docker ps
```

Then:

```bash
docker network inspect backend-net
```

Then:

```bash
docker logs mysql
```

Then:

```bash
docker logs redis
```

Then:

```bash
docker logs ecommerce-api
```

This sequence quickly tells you whether the issue is:

```text
Runtime
Network
Dependency readiness
Application configuration
```

---

# 98. File 03 Checklist

You should now understand:

```text
□ Docker networks
□ Default bridge
□ User-defined bridge
□ Docker DNS
□ Container names
□ Service names
□ localhost behavior
□ Container port
□ Host port
□ Port publishing
□ EXPOSE
□ Network isolation
□ Multiple networks
□ Bridge
□ Host
□ None
□ Overlay concept
□ Network inspection
□ Volumes
□ Named volumes
□ Bind mounts
□ Volume lifecycle
□ Volume permissions
□ MySQL persistence
□ Redis persistence
□ Spring Boot → MySQL
□ Spring Boot → Redis
□ Network troubleshooting
□ Health/readiness
□ Docker network security
□ Interview scenarios
```

---

# 99. What Comes Next

```text
File 04 → Docker Compose: Spring Boot + MySQL + Redis
```

This is where the Docker knowledge starts becoming very practical for your backend project.

We'll cover:

```text
docker-compose.yml
services
images
build
ports
environment
volumes
networks
depends_on
healthchecks
service discovery
Spring Boot configuration
MySQL initialization
Redis configuration
profiles
startup order
restart policies
Docker Compose commands
full e-commerce backend stack
interview questions
```

Key takeaway:

> **For Dockerized backend systems, remember three rules: use Docker DNS/service names instead of container IPs, use container ports for container-to-container communication, and put state that must survive container replacement into persistent storage.**

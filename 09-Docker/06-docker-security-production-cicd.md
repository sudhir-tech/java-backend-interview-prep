# Docker — File 06: Security, Production Practices & CI/CD

This file is about taking a Docker setup from:

```text
"It runs on my laptop"
```

to:

```text
"How would I safely build and deploy this?"
```

For backend interviews, this is where Docker questions usually become more practical.

---

# 1. Docker Security Mindset

A container is not automatically secure just because it is isolated.

Think about:

```text
Image
Container
Process
Network
Secrets
Dependencies
Registry
CI/CD
Runtime permissions
```

Security needs to cover the whole chain.

---

# 2. Biggest Docker Security Rules

Remember these first:

```text
1. Use trusted, maintained base images.
2. Keep images small.
3. Don't run as root unnecessarily.
4. Never bake secrets into images.
5. Scan images and dependencies.
6. Keep images updated.
7. Expose only required ports.
8. Limit container resources.
9. Use least privilege.
10. Don't blindly trust third-party images.
```

---

# 3. Don't Run as Root

Bad:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/app.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Depending on the base image, the process may run as root.

Better:

```dockerfile
USER appuser
```

after creating an appropriate non-root user.

---

# 4. Why Root Is Dangerous

If an attacker compromises the application process:

```text
Application vulnerability
        ↓
Container process compromised
        ↓
Root privileges
```

The potential impact is greater.

With a non-root process:

```text
Application compromised
        ↓
Limited user permissions
```

This is not a complete security boundary, but it reduces privileges.

---

# 5. Least Privilege

Give the application only what it needs.

For example, a Spring Boot API usually doesn't need:

```text
Package installation
Docker socket
Host filesystem access
Kernel-level capabilities
Root privileges
```

Avoid unnecessary permissions.

---

# 6. Docker Socket

Be careful with:

```text
/var/run/docker.sock
```

Mounting the Docker socket into a container can effectively give that container powerful control over Docker.

Avoid:

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
```

unless there is a well-understood security reason and the setup is intentionally secured.

---

# 7. Host Filesystem Mounts

Avoid unnecessary mounts such as:

```text
/
```

or sensitive host directories.

A compromised container with excessive host filesystem access can cause serious damage.

Mount only what is required.

---

# 8. Read-Only Root Filesystem

For applications that don't need to write to their root filesystem:

```text
Read-only filesystem
```

can reduce the ability of an attacker or process to modify files.

Conceptually:

```bash
docker run --read-only app
```

If the application needs temporary files, provide an appropriate writable temporary location.

---

# 9. Temporary Filesystem

Some applications need:

```text
/tmp
```

for temporary data.

A read-only root filesystem can still work if writable temporary storage is provided intentionally.

The exact configuration depends on the application.

---

# 10. Linux Capabilities

Linux capabilities divide some root-level privileges into smaller permissions.

A container doesn't necessarily need all capabilities.

Principle:

```text
Remove capabilities you don't need.
```

For example, a normal Spring Boot API shouldn't require broad system capabilities.

---

# 11. `--cap-drop`

Conceptually:

```bash
docker run --cap-drop=ALL app
```

Then selectively add capabilities only if actually required.

Don't blindly use this without testing because some workloads require specific capabilities.

---

# 12. Privileged Containers

Avoid:

```bash
docker run --privileged ...
```

unless there is a very specific reason.

Privileged containers receive significantly expanded access.

For a normal Spring Boot backend:

```text
You should not need privileged mode.
```

---

# 13. Network Exposure

Suppose your stack contains:

```text
Spring Boot
MySQL
Redis
```

Don't automatically publish:

```text
3306
6379
```

to the host.

If only the API needs them:

```text
API → MySQL
API → Redis
```

keep them on the private Docker network.

---

# 14. Public Ports

Only expose what users actually need.

Typical:

```text
Internet
   ↓
API / Load Balancer
   ↓
Application
```

Not:

```text
Internet
 ↓
MySQL
Redis
```

Databases should not be unnecessarily exposed.

---

# 15. Secrets

Never put production secrets into:

```dockerfile
ENV DB_PASSWORD=secret
```

or:

```dockerfile
ARG DB_PASSWORD=secret
```

Images can be inspected.

Build history and metadata can also reveal information depending on how values are used.

---

# 16. Bad Secret Flow

```text
Password
 ↓
Dockerfile
 ↓
Image
 ↓
Registry
 ↓
Production
```

Now the secret may exist in multiple places.

---

# 17. Better Secret Flow

```text
Secret Manager
       ↓
Deployment
       ↓
Container runtime
       ↓
Application
```

Examples include:

```text
Cloud secret managers
Kubernetes Secrets
CI/CD secret stores
Docker secrets in supported environments
```

Use the mechanism appropriate to your platform.

---

# 18. Environment Variables and Secrets

Environment variables are better than hardcoding secrets into source or images, but they aren't automatically a perfect secret-management system.

For sensitive production credentials:

```text
Use proper secret management.
```

Also be aware that processes, debugging tools and platform metadata may expose environment variables.

---

# 19. Secret Rotation

A production secret should be rotatable.

Example:

```text
Old DB password
      ↓
Rotate
      ↓
New DB password
      ↓
Update runtime configuration
```

Your application should not require rebuilding the Docker image just because a password changed.

---

# 20. Image Security

An image can contain vulnerabilities through:

```text
Base OS packages
Java runtime
Application dependencies
Transitive dependencies
Build tools
```

Therefore:

```text
Small image
≠
Automatically secure image
```

You still need scanning and updates.

---

# 21. Vulnerability Scanning

A CI pipeline can:

```text
Build image
    ↓
Scan image
    ↓
Check vulnerabilities
    ↓
Continue / fail
```

Tools vary by organization.

Examples:

```text
Trivy
Docker Scout
Snyk
Grype
Cloud registry scanners
```

Choose according to your organization's tooling.

---

# 22. Dependency Scanning

Don't scan only the OS image.

Your Spring Boot application also has:

```text
Spring Framework
Spring Security
Hibernate
Jackson
Tomcat
MySQL driver
Redis client
```

Scan application dependencies too.

---

# 23. SBOM

SBOM means:

```text
Software Bill of Materials
```

It describes the software components contained in an artifact.

Conceptually:

```text
ecommerce-api
   |
   +-- Java
   +-- Spring
   +-- Hibernate
   +-- Jackson
   +-- MySQL driver
   +-- Other dependencies
```

An SBOM helps with:

```text
Vulnerability response
Software inventory
Supply-chain visibility
Compliance
```

---

# 24. Supply-Chain Security

Your application doesn't only depend on your own code.

It depends on:

```text
Base images
Maven dependencies
Plugins
Build tools
Registry
CI environment
```

A compromised dependency or image can affect the final application.

---

# 25. Trusted Base Images

Prefer maintained sources and official/vendor-supported images.

For Java:

```text
Eclipse Temurin
Other trusted Java distributions
```

depending on organizational requirements.

Avoid random images just because they have convenient names.

---

# 26. Pinning Base Images

Instead of loosely using:

```dockerfile
FROM eclipse-temurin:21-jre
```

organizations may pin an exact digest:

```text
image@sha256:...
```

This improves reproducibility.

But the digest must be updated when you intentionally move to a newer secure image.

---

# 27. Version Tags

Avoid relying only on:

```text
latest
```

Prefer:

```text
ecommerce-api:1.4.0
```

or:

```text
ecommerce-api:git-a1b2c3d
```

This makes deployments and rollbacks predictable.

---

# 28. Immutable Images

Once an image is built and tested:

```text
Don't modify the running image manually.
```

Instead:

```text
Change code
 ↓
Build new image
 ↓
Test
 ↓
Deploy new version
```

This is the immutable deployment model.

---

# 29. Container vs Image

Remember:

```text
Image
→ Immutable application artifact

Container
→ Running instance of the image
```

If you manually edit a running container:

```text
Those changes are not part of your versioned image.
```

They can disappear when the container is recreated.

---

# 30. Don't SSH Into Containers as a Normal Workflow

Traditional servers often use:

```text
SSH
 ↓
modify server
```

Containers are usually managed differently:

```text
Logs
Metrics
Health checks
New image
Redeploy
```

Interactive shell access can still be useful for debugging, but it shouldn't become the deployment strategy.

---

# 31. Container Resource Limits

A container without appropriate limits can consume too many resources.

Consider:

```text
CPU
Memory
PIDs
```

Example:

```bash
docker run \
  --memory=512m \
  --cpus=1 \
  ecommerce-api:1.0
```

The exact values should come from load testing and production requirements.

---

# 32. Memory Limit and Java

Suppose:

```text
Container = 512 MB
```

Java needs memory for:

```text
Heap
Metaspace
Threads
Native memory
Code cache
GC
```

Therefore:

```text
Heap should not blindly consume 512 MB.
```

Leave headroom.

---

# 33. CPU Limits

CPU limits can protect other workloads.

But setting them too low can cause:

```text
Slow requests
Higher latency
Long GC pauses
Timeouts
```

Tune them using actual workload measurements.

---

# 34. Health Checks

A container being:

```text
Running
```

doesn't necessarily mean:

```text
Application is healthy
```

Health should be based on meaningful signals.

For Spring Boot:

```text
/actuator/health
```

can be useful.

---

# 35. Liveness vs Readiness

Remember:

```text
Liveness
→ Should this process be restarted?

Readiness
→ Should this instance receive traffic?
```

A database temporarily unavailable may make an application:

```text
Not ready
```

without necessarily meaning:

```text
Restart the process immediately.
```

---

# 36. Logging

Prefer:

```text
Application
 ↓
stdout/stderr
 ↓
Docker/orchestrator logging
 ↓
Centralized logging
```

Avoid making local container files your only source of production logs.

---

# 37. Logging for Your Backend

Your application might produce:

```text
Request logs
Error logs
Database errors
Authentication events
Cache errors
Startup logs
```

These should be collected centrally in a production environment.

---

# 38. Don't Log Secrets

Never log:

```text
Passwords
JWT secrets
Database credentials
API keys
Private tokens
```

Be careful with request headers too.

For example:

```text
Authorization: Bearer ...
```

should not normally be logged in full.

---

# 39. Docker Registry

The registry stores images.

Flow:

```text
Developer / CI
      ↓
Build image
      ↓
Scan
      ↓
Push registry
      ↓
Deployment
      ↓
Pull image
```

Examples:

```text
Docker Hub
GitHub Container Registry
Amazon ECR
Azure Container Registry
Google Artifact Registry
Private registries
```

---

# 40. Registry Security

Protect the registry with:

```text
Authentication
Authorization
TLS
Image scanning
Access control
Retention policies
```

Don't give every developer push access to production repositories.

---

# 41. CI/CD Pipeline

A typical Docker pipeline:

```text
Git push
   ↓
Compile
   ↓
Unit tests
   ↓
Build image
   ↓
Scan
   ↓
Integration tests
   ↓
Push registry
   ↓
Deploy
```

---

# 42. Build Stage

Example:

```bash
mvn clean verify
```

This can run:

```text
Compilation
Unit tests
Integration checks
Quality checks
```

depending on the Maven configuration.

---

# 43. Docker Build Stage

Then:

```bash
docker build -t ecommerce-api:$VERSION .
```

Example:

```text
VERSION=1.4.0
```

Result:

```text
ecommerce-api:1.4.0
```

---

# 44. Image Scan

After building:

```text
ecommerce-api:1.4.0
```

scan it.

Potential outcomes:

```text
No critical issues
```

or:

```text
Critical vulnerability
```

Your CI policy determines whether deployment is blocked.

---

# 45. Integration Testing

A useful pipeline can start:

```text
Spring Boot
MySQL
Redis
```

using Compose or another test environment.

Then run tests against the real containerized stack.

Example:

```text
Build
 ↓
Compose up
 ↓
Integration tests
 ↓
Compose down
```

---

# 46. Push Image

After successful validation:

```bash
docker push registry.example/ecommerce-api:1.4.0
```

The exact registry command depends on the platform.

The important concept is:

```text
Only push tested artifacts.
```

---

# 47. Deploy

Deployment system pulls:

```text
ecommerce-api:1.4.0
```

and starts it.

The server should not need to:

```text
git clone
mvn package
```

to run the application.

---

# 48. Build Once, Deploy Many

Strong CI/CD principle:

```text
Build
 ↓
Test
 ↓
Scan
 ↓
Publish
 ↓
Deploy same artifact
```

Don't rebuild different images for:

```text
Dev
Test
Production
```

unless there is a deliberate reason.

---

# 49. Configuration Changes Between Environments

Image:

```text
ecommerce-api:1.4.0
```

can stay the same.

Configuration changes:

```text
DEV:
mysql-dev

TEST:
mysql-test

PROD:
managed database
```

This keeps the artifact consistent.

---

# 50. Rollback

Suppose:

```text
Production:
1.3.0 → stable
1.4.0 → broken
```

Rollback:

```text
Deploy 1.3.0
```

No rebuild required.

This is one of the biggest benefits of immutable images.

---

# 51. Deployment Strategies

Common approaches:

```text
Rolling deployment
Blue/green
Canary
Recreate
```

Docker itself doesn't provide the full deployment strategy.

Your orchestrator/platform handles that.

---

# 52. Rolling Deployment

Conceptually:

```text
Old v1
Old v1
Old v1

       ↓

Old v1
New v2
New v2

       ↓

New v2
New v2
New v2
```

Traffic can be shifted gradually depending on the platform.

---

# 53. Blue/Green

```text
Blue → current production
Green → new version
```

Test Green.

Then switch traffic:

```text
Blue
 ↓
Green
```

Rollback can switch traffic back to Blue.

---

# 54. Canary

Send a small percentage of traffic to the new version:

```text
95% → v1
5%  → v2
```

Monitor:

```text
Errors
Latency
CPU
Memory
Business metrics
```

Then increase traffic if healthy.

---

# 55. Docker and Kubernetes

Docker is primarily about:

```text
Images
Containers
Networking
Volumes
Runtime
```

Kubernetes adds:

```text
Scheduling
Scaling
Service discovery
Self-healing
Rolling deployments
Secrets/config
```

The Docker image becomes the deployable artifact.

---

# 56. Docker in Cloud

A common cloud flow:

```text
GitHub
  ↓
CI/CD
  ↓
Docker image
  ↓
Container registry
  ↓
Cloud platform
```

Examples:

```text
AWS ECS / EKS
Azure Container Apps / AKS
Google Cloud Run / GKE
```

The exact choice depends on the application's operational requirements.

---

# 57. Production Database

For a production Spring Boot application:

```text
Application container
       |
       ↓
Managed MySQL
```

is often preferable to managing a MySQL container yourself.

Reasons:

```text
Backups
Failover
Patching
Monitoring
Storage
Recovery
```

---

# 58. Production Redis

Depending on requirements:

```text
Managed Redis
```

may provide:

```text
High availability
Backups
Monitoring
Scaling
Failover
```

If Redis is only a cache, the durability requirements are lower.

---

# 59. Production Statelessness

Your Spring Boot container should ideally be:

```text
Stateless
```

Don't store important business data in:

```text
Container filesystem
```

Store it externally:

```text
MySQL
Object storage
Redis
Managed services
```

depending on the data type.

---

# 60. Container Restart

Containers can restart because of:

```text
Application crash
Host issue
Deployment
Resource pressure
Health failure
```

A well-designed backend should recover without losing important state.

---

# 61. Graceful Shutdown

On deployment:

```text
Old container
     ↓
SIGTERM
     ↓
Stop accepting new requests
     ↓
Finish suitable in-flight work
     ↓
Close connections
     ↓
Exit
```

This reduces failed requests during deployments.

---

# 62. Connection Pool During Shutdown

Spring Boot should close:

```text
HikariCP connections
Redis connections
Other resources
```

during graceful shutdown.

Don't abruptly kill applications if the deployment environment allows graceful termination.

---

# 63. Dockerfile Production Checklist

Example:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/*.jar app.jar

RUN useradd --system appuser
USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Checklist:

```text
□ Maintained base image
□ Appropriate Java version
□ Small runtime image
□ Non-root user
□ No secrets
□ Versioned image
□ Health strategy
□ Logging to stdout/stderr
```

---

# 64. Stronger Production Image

Depending on requirements, you can additionally consider:

```text
Read-only filesystem
Dropped Linux capabilities
Minimal runtime
Pinned base-image digest
SBOM
Image signing
Resource limits
Security scanning
```

Not every application needs every hardening technique, but each should be considered deliberately.

---

# 65. Image Signing

Image signing helps establish:

```text
Who produced this artifact?
Was it altered?
Can the deployment system trust it?
```

Modern supply-chain tooling can support signing and verification.

The exact implementation depends on your registry/platform.

---

# 66. SBOM + Signing

A mature supply-chain flow can look like:

```text
Build
 ↓
Test
 ↓
Scan
 ↓
Generate SBOM
 ↓
Sign image
 ↓
Push registry
 ↓
Verify during deployment
```

This is a strong production concept to understand for interviews.

---

# 67. Dependency Updates

Don't wait for a production incident.

Regularly update:

```text
Spring Boot
Java
Hibernate
Jackson
Tomcat
Database drivers
Redis clients
Base image
```

Test after updates.

---

# 68. Base Image Updates

Even if:

```text
Application source hasn't changed
```

the base image may receive:

```text
Security fixes
JVM updates
OS package updates
```

Rebuild regularly.

---

# 69. Vulnerability Prioritization

Not every scanner finding means:

```text
Immediately stop production
```

Consider:

```text
Severity
Exploitability
Exposure
Whether vulnerable code path is used
Available fix
Compensating controls
```

Critical vulnerabilities should receive urgent attention according to organizational policy.

---

# 70. Don't Ignore Scanner Results

Bad process:

```text
Scanner
 ↓
100 findings
 ↓
Ignore
```

Better:

```text
Scan
 ↓
Prioritize
 ↓
Fix/update
 ↓
Rescan
 ↓
Document exceptions
```

---

# 71. CI/CD Secrets

Don't hardcode registry credentials in:

```text
Jenkinsfile
GitHub Actions YAML
Dockerfile
```

Use:

```text
CI/CD secret store
OIDC/workload identity
Short-lived credentials
```

where supported.

---

# 72. GitHub Actions Example Flow

Conceptually:

```yaml
jobs:
  build:
    steps:
      - checkout
      - setup-java
      - mvn test
      - docker build
      - scan
      - push
```

The exact syntax depends on your pipeline.

---

# 73. Jenkins Example Flow

Given your Java/Jenkins background, think:

```text
Jenkins
  ↓
Checkout
  ↓
Maven build
  ↓
Unit tests
  ↓
Docker build
  ↓
SonarQube / quality checks
  ↓
Image scan
  ↓
Registry
  ↓
Deployment
```

This is a common enterprise backend flow.

---

# 74. Docker + SonarQube

SonarQube checks:

```text
Application source quality
```

Container scanning checks:

```text
Image/dependency vulnerabilities
```

They solve different problems.

You can use both:

```text
Code quality
+
Container security
```

---

# 75. Docker + ELK

For a backend environment:

```text
Spring Boot
   ↓
stdout/stderr
   ↓
Log collector
   ↓
Logstash / Elasticsearch
   ↓
Kibana
```

The exact architecture varies.

Docker doesn't replace centralized logging.

---

# 76. Docker + Monitoring

Monitor:

```text
CPU
Memory
Restart count
Request latency
Error rate
Container health
Database connections
Redis availability
```

Application-level metrics are as important as container metrics.

---

# 77. Production Readiness Checklist

Before deploying a Spring Boot container:

```text
Application
□ Tests passing
□ Dependencies updated
□ Health endpoints
□ Graceful shutdown

Image
□ Small runtime image
□ Non-root
□ No secrets
□ Scanned
□ Versioned
□ Trusted base image

Runtime
□ Resource limits
□ Correct configuration
□ Logs
□ Metrics
□ Network access minimized

Data
□ Managed DB / persistent storage
□ Backup strategy
□ Recovery strategy

CI/CD
□ Automated build
□ Tests
□ Image scan
□ Registry
□ Deployment
□ Rollback
```

---

# 78. Interview Question

### How would you secure a Docker container?

Answer:

> "I'd use a trusted and maintained base image, keep the image minimal, run the application as a non-root user, avoid unnecessary Linux capabilities and privileged mode, keep secrets outside the image, restrict network exposure, scan the image and dependencies, and apply appropriate resource limits."

---

# 79. Interview Question

### Why shouldn't secrets be in a Dockerfile?

Answer:

> "Because the Dockerfile and resulting image can be inspected, and secrets can potentially remain in image layers or metadata. I'd inject sensitive configuration at runtime using a secure secret-management mechanism."

---

# 80. Interview Question

### What is an SBOM?

Answer:

> "An SBOM is a Software Bill of Materials. It provides an inventory of the components and dependencies in an application or image, which helps with vulnerability management and software supply-chain visibility."

---

# 81. Interview Question

### Why shouldn't you use `latest` in production?

Answer:

> "`latest` is mutable and doesn't clearly identify which artifact was deployed. I prefer immutable version or commit-based tags so deployments and rollbacks are predictable."

---

# 82. Interview Question

### What is an immutable image?

Answer:

> "It's a versioned artifact that isn't modified after being built. If we need a change, we build a new image and deploy that version instead of modifying a running container."

---

# 83. Interview Question

### How would you design a Docker CI/CD pipeline?

Answer:

> "I'd run tests and quality checks first, build the Docker image, scan it for vulnerabilities, run integration tests, push the tested image to a registry and deploy that exact image. I'd also keep versioned images so rollback doesn't require rebuilding."

---

# 84. Interview Question

### How would you handle a critical vulnerability in production?

Answer:

> "I'd identify the affected component, check exploitability and exposure, update the vulnerable dependency or base image, rebuild and scan the image, test it, then deploy the patched version. If necessary, I'd use the previous stable version or another mitigation while the fix is prepared."

---

# 85. Interview Question

### Why run containers as non-root?

Answer:

> "It follows least privilege. If the application is compromised, the attacker has fewer permissions inside the container compared with a root process."

---

# 86. Interview Scenario

### Scanner reports a critical vulnerability in the Java base image.

Think:

```text
1. Identify affected component.
2. Check fixed base-image version.
3. Update Dockerfile.
4. Rebuild.
5. Scan again.
6. Run tests.
7. Deploy patched image.
```

If the vulnerability is not immediately patchable:

```text
Assess exposure
+
Apply temporary mitigation
+
Track remediation
```

---

# 87. Interview Scenario

### Developer accidentally committed a password.

Don't simply delete the line.

Do:

```text
1. Rotate the credential.
2. Remove it from source.
3. Check whether it reached CI/history/artifacts.
4. Remove exposed copies where appropriate.
5. Store the new secret securely.
```

Assume an exposed production credential is compromised.

---

# 88. Interview Scenario

### Container is using too much memory.

Investigate:

```text
Container limit
JVM heap
GC
Traffic
Threads
Native memory
Memory leak
Dependency behavior
```

Then use:

```text
Profiling
Metrics
Heap analysis
Load testing
```

rather than blindly increasing the limit.

---

# 89. Interview Scenario

### A container can access every backend service.

Ask:

```text
Does it need to?
```

If not:

```text
Separate networks
Firewall/security groups
Service-level authorization
Least privilege
```

Reduce unnecessary connectivity.

---

# 90. Interview Scenario

### Production image contains Maven and source code.

Not ideal.

Use:

```text
Multi-stage build
```

so:

```text
Builder
→ Maven + JDK + source

Runtime
→ JRE/runtime + JAR
```

This reduces size and attack surface.

---

# 91. Interview Scenario

### Deployment succeeded, but the application immediately crashes.

Check:

```text
Image
Entrypoint
Environment
Secrets
Database connectivity
Redis
Memory
Java version
Application logs
Health checks
```

Deployment success only means the platform started the container; it doesn't prove the application is healthy.

---

# 92. Interview Scenario

### New version causes high error rates.

Use your deployment strategy:

```text
Rollback
```

or:

```text
Shift traffic back
```

depending on:

```text
Rolling
Blue/green
Canary
```

Because the image is versioned, rollback should not require rebuilding.

---

# 93. One-Minute Production Docker Answer

If an interviewer asks:

> "What are the important production practices when using Docker?"

Say:

> "I'd keep the runtime image small and use a maintained base image, run the application as a non-root user, avoid secrets in the image, scan the image and dependencies, restrict network exposure and resources, centralize logs and metrics, and use immutable versioned images. In CI/CD I'd test, build, scan and publish the image, then deploy that exact artifact so rollback is straightforward."

---

# 94. Your Backend Project — Production Mental Model

For the e-commerce backend:

```text
Developer
    |
    ↓
Git
    |
    ↓
CI/CD
    |
    +--> Maven tests
    |
    +--> Docker build
    |
    +--> Security scan
    |
    +--> Integration tests
    |
    ↓
Container Registry
    |
    ↓
Deployment
    |
    ↓
Spring Boot
   /    \
  ↓      ↓
MySQL   Redis
```

The important thing:

```text
Same tested image
        ↓
Environment-specific configuration
```

---

# 95. Quick Revision

```text
Root
→ Avoid when unnecessary

Secrets
→ Never bake into image

Image
→ Small + versioned + scanned

Network
→ Expose only what is required

Runtime
→ Resource limits + health + logs

CI/CD
→ Test → Build → Scan → Push → Deploy

Registry
→ Store versioned images

Rollback
→ Deploy previous known-good image

Production
→ Prefer managed stateful services when appropriate
```

---

# 96. File 06 Checklist

```text
□ Docker security
□ Non-root containers
□ Least privilege
□ Docker socket risk
□ Host filesystem mounts
□ Read-only filesystem
□ Linux capabilities
□ Privileged containers
□ Network exposure
□ Secrets
□ Secret rotation
□ Image scanning
□ Dependency scanning
□ SBOM
□ Supply-chain security
□ Trusted base images
□ Digest pinning
□ Immutable images
□ Resource limits
□ Health checks
□ Logging
□ Registries
□ CI/CD
□ Integration tests
□ Build once, deploy many
□ Rollbacks
□ Rolling deployment
□ Blue/green
□ Canary
□ Docker + Kubernetes
□ Production database
□ Monitoring
□ Jenkins pipeline
□ SonarQube
□ ELK
□ Production checklist
□ Interview scenarios
```

---

# 97. Docker Topic Summary

At this point, the Docker section covers:

```text
01 → Fundamentals
02 → Images, Dockerfile & Build Optimization
03 → Networking, Volumes & Container Communication
04 → Docker Compose
05 → Dockerizing Spring Boot
06 → Security, Production Practices & CI/CD
```

You now have the Docker knowledge needed for a typical Java backend interview.

The next step should be practical:

```text
Build your Spring Boot e-commerce backend image
        ↓
Run MySQL + Redis
        ↓
Connect everything with Compose
        ↓
Debug it
        ↓
Add it to your GitHub project
```

**Key takeaway:**

> **Production Docker is mostly about reducing unnecessary trust: don't trust root privileges, don't trust mutable `latest` tags, don't trust images without scanning, don't trust secrets inside images, and don't trust a container as permanent storage. Build a versioned artifact, test it, scan it, and deploy that exact artifact.**

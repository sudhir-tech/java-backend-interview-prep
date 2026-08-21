# Microservices — Deployment: Docker, Kubernetes & CI/CD

This file covers how microservices are packaged, deployed, scaled and released in production.

Core topics:

```text
Containers
Docker
Dockerfile
Images
Containers
Registries
Multi-stage Builds
Environment Variables
Secrets
Docker Compose
Kubernetes
Pods
Deployments
Services
Ingress
ConfigMaps
Secrets
Namespaces
Probes
Requests/Limits
Autoscaling
Rolling Updates
Rollbacks
Helm
CI/CD
Build Pipeline
Test Pipeline
Artifact
Container Registry
Deployment Strategies
Blue-Green
Canary
GitOps
Production Scenarios
Interview Questions
```

---

# 1. Why Deployment Matters in Microservices

A microservices architecture may contain:

```text
Order Service
Inventory Service
Payment Service
Notification Service
```

Each service needs to be:

```text
Built
Packaged
Tested
Deployed
Monitored
Scaled
Updated
Rolled back
```

Automation becomes extremely important.

---

# 2. Containerization

A container packages an application together with the runtime dependencies needed to execute it.

For a Spring Boot application:

```text
Java Application
+
JRE/JDK
+
Dependencies
+
Configuration
```

can be packaged into a container image.

---

# 3. Why Containers?

Containers provide:

```text
Consistent environments
Isolation
Portable deployment
Fast startup
Easy packaging
Resource controls
```

They reduce:

> "It works on my machine."

problems.

---

# 4. Docker

Docker is commonly used to build and run containers.

Typical flow:

```text
Source Code
   ↓
Dockerfile
   ↓
Docker Image
   ↓
Container Registry
   ↓
Deployment Platform
   ↓
Container
```

---

# 5. Docker Image

An image is a packaged, immutable template from which containers are created.

Example:

```text
order-service:1.4.0
```

---

# 6. Container

A container is a running instance of an image.

```text
Image
  ↓
Container
```

You can run multiple containers from the same image.

---

# 7. Container Registry

Images are stored in registries.

Examples:

```text
Docker Hub
Amazon ECR
Azure Container Registry
Google Artifact Registry
GitHub Container Registry
```

Production pipelines usually:

```text
Build image
 ↓
Push image
 ↓
Deploy image
```

---

# 8. Dockerfile

A Dockerfile defines how the image is built.

Example:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/app.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

# 9. FROM

```dockerfile
FROM eclipse-temurin:21-jre
```

specifies the base image.

Prefer trusted and appropriately maintained base images.

---

# 10. WORKDIR

```dockerfile
WORKDIR /app
```

sets the working directory inside the image.

---

# 11. COPY

```dockerfile
COPY target/app.jar app.jar
```

copies application artifacts into the image.

---

# 12. ENTRYPOINT

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

defines the main process.

---

# 13. CMD

`CMD` can provide default command/arguments.

For example:

```dockerfile
CMD ["java", "-jar", "app.jar"]
```

The exact choice between `ENTRYPOINT` and `CMD` depends on how you want the image to be overridden.

---

# 14. Docker Image Layers

Docker images are composed of layers.

A change to a later layer can invalidate cache for subsequent layers.

Therefore Dockerfile ordering matters.

---

# 15. Multi-Stage Build

For Java applications:

```text
Stage 1
Maven + JDK
 ↓
Build JAR

Stage 2
Small JRE/runtime image
 ↓
Run JAR
```

Example:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre

WORKDIR /app
COPY --from=build /app/target/app.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Benefits:

```text
Smaller runtime image
No Maven in runtime
Reduced attack surface
Cleaner packaging
```

---

# 16. Don't Put Secrets in Dockerfile

Bad:

```dockerfile
ENV DB_PASSWORD=mysecret
```

Secrets should be supplied securely at runtime.

---

# 17. Environment Variables

Configuration can be externalized:

```text
DB_HOST
DB_PORT
DB_USERNAME
DB_PASSWORD
```

The same image can then run in:

```text
Development
Testing
Staging
Production
```

with different configuration.

---

# 18. Immutable Image

Ideally:

```text
Build once
Tag once
Deploy the same image
```

Don't manually modify the container in production.

---

# 19. Image Tagging

Avoid relying only on:

```text
latest
```

Prefer immutable identifiers:

```text
order-service:1.4.2
order-service:git-a81f32c
```

This makes rollback easier.

---

# 20. Docker Compose

Docker Compose is useful for local multi-container development.

Example:

```text
Order Service
Inventory Service
MySQL
Redis
Kafka
```

can be started together.

---

# 21. Compose vs Kubernetes

Compose:

```text
Local development
Simple environments
Testing
```

Kubernetes:

```text
Production orchestration
Scaling
Self-healing
Rolling deployments
Service discovery
```

They solve different levels of the problem.

---

# 22. Kubernetes

Kubernetes is a container orchestration platform.

It manages:

```text
Scheduling
Scaling
Networking
Health checks
Rolling updates
Self-healing
Configuration
```

---

# 23. Kubernetes Cluster

Conceptually:

```text
Kubernetes Cluster
       |
       +→ Control Plane
       |
       +→ Worker Nodes
```

---

# 24. Pod

A Pod is the smallest deployable unit in Kubernetes.

Usually:

```text
1 Pod
 ↓
1 Application Container
```

but a Pod can contain multiple tightly coupled containers.

---

# 25. Pod Is Not the Same as Container

Think:

```text
Pod
 └── Container
```

A Pod provides the execution/networking context for its containers.

---

# 26. Pod IP

Pods receive IP addresses, but Pods are ephemeral.

If a Pod is replaced:

```text
Old Pod
 ↓
Deleted

New Pod
 ↓
New IP
```

Therefore clients should generally not depend on individual Pod IPs.

---

# 27. Kubernetes Service

A Service provides stable networking to a set of Pods.

```text
Service
   ↓
+-- Pod
+-- Pod
+-- Pod
```

It can provide:

```text
Stable DNS
Stable virtual IP
Load distribution
```

---

# 28. Service Discovery

Example:

```text
order-service
```

can resolve through Kubernetes DNS to its Service.

Instead of:

```text
http://10.42.1.17:8080
```

use:

```text
http://order-service:8080
```

within the appropriate cluster/network context.

---

# 29. Deployment

A Deployment manages replicated Pods for an application.

Example:

```yaml
replicas: 3
```

means Kubernetes aims to maintain:

```text
3 Pods
```

for that Deployment.

---

# 30. ReplicaSet

A Deployment manages ReplicaSets, which maintain the desired number of Pods.

Conceptually:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

---

# 31. Desired State

Kubernetes is declarative.

You specify:

```text
I want 3 replicas
```

Kubernetes continuously works toward that state.

If one Pod crashes:

```text
3 desired
2 running
 ↓
Kubernetes creates another
 ↓
3 running
```

---

# 32. Self-Healing

Kubernetes can replace failed Pods.

This is one reason orchestration is useful.

But remember:

> Kubernetes self-healing does not fix application bugs.

A crashing application may simply be restarted repeatedly.

---

# 33. Readiness Probe

Readiness tells Kubernetes:

> "Can this Pod receive traffic?"

Example:

```text
/actuator/health/readiness
```

If readiness fails:

```text
Pod stays running
but
traffic is removed
```

---

# 34. Liveness Probe

Liveness tells Kubernetes:

> "Should this container be restarted?"

Example:

```text
/actuator/health/liveness
```

If liveness fails:

```text
Container may be restarted
```

---

# 35. Startup Probe

Useful for slow-starting applications.

```text
Container starts
 ↓
Startup probe
 ↓
Startup succeeds
 ↓
Liveness/readiness checks
```

---

# 36. Requests and Limits

Kubernetes can define:

```text
CPU request
Memory request
CPU limit
Memory limit
```

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "1"
    memory: "1Gi"
```

---

# 37. Requests

A request tells Kubernetes approximately:

> "Reserve this amount when scheduling."

It helps Kubernetes choose a suitable node.

---

# 38. Limits

A limit caps the resource usage according to Kubernetes/container runtime behavior.

For memory, exceeding the limit can result in the container being killed.

---

# 39. Why Requests and Limits Matter

Without resource planning:

```text
One service
 ↓
Consumes excessive resources
 ↓
Other services affected
```

Resource controls improve isolation.

---

# 40. Horizontal Pod Autoscaler

HPA automatically adjusts the number of Pods based on metrics.

Example:

```text
CPU > target
 ↓
Replicas 3 → 6
```

When load decreases:

```text
6 → 3
```

---

# 41. Autoscaling Isn't Magic

Scaling Pods won't solve:

```text
Database bottleneck
External API rate limit
Single-threaded bottleneck
Partition limitation
Connection pool exhaustion
```

Always identify the actual bottleneck.

---

# 42. Vertical vs Horizontal Scaling

Vertical:

```text
More CPU/RAM
```

Horizontal:

```text
More instances/Pods
```

Microservices commonly benefit from horizontal scaling when workloads are stateless and independently scalable.

---

# 43. Stateless Service

A stateless service does not depend on local instance memory for durable user/session state.

Example:

```text
Pod A
Pod B
Pod C
```

Any request can go to any Pod.

---

# 44. Why Statelessness Helps

```text
Load balancing
Horizontal scaling
Pod replacement
Rolling deployment
```

become easier.

---

# 45. Stateful Data

State should usually live in durable systems such as:

```text
Database
Redis
Object storage
Message broker
```

depending on the use case.

---

# 46. Ingress

Ingress provides HTTP/HTTPS routing into the cluster.

Conceptually:

```text
Internet
   ↓
Ingress
   |
   +→ order-service
   |
   +→ product-service
```

---

# 47. Ingress vs Service

Service:

```text
Stable networking to Pods
```

Ingress:

```text
HTTP/HTTPS routing into services
```

---

# 48. Gateway vs Ingress

API Gateway and Kubernetes Ingress can overlap, but they are not identical concepts.

Gateway may provide:

```text
Authentication
Rate limiting
API policies
Aggregation
Routing
```

Ingress primarily provides cluster-level HTTP routing, depending on the controller/product.

---

# 49. Namespace

Namespaces logically isolate Kubernetes resources.

Example:

```text
dev
staging
prod
```

or:

```text
payments
orders
platform
```

depending on organizational needs.

---

# 50. ConfigMap

ConfigMaps store non-sensitive configuration.

Examples:

```text
Feature flags
URLs
Application settings
Environment-specific configuration
```

---

# 51. Kubernetes Secret

Secrets are intended for sensitive configuration.

Examples:

```text
Passwords
API keys
Certificates
Tokens
```

However, Kubernetes Secret objects require appropriate access controls and encryption-at-rest configuration; they should not be treated as magically secure.

---

# 52. External Secret Management

Production systems may integrate with:

```text
HashiCorp Vault
AWS Secrets Manager
Azure Key Vault
Google Secret Manager
```

This can centralize secret lifecycle management.

---

# 53. Kubernetes Deployment Flow

Typical:

```text
Git
 ↓
CI Pipeline
 ↓
Build
 ↓
Test
 ↓
Docker Image
 ↓
Container Registry
 ↓
Kubernetes Deployment
 ↓
Pods
```

---

# 54. CI/CD

CI/CD means automating software integration and delivery/deployment.

Typical pipeline:

```text
Commit
 ↓
Build
 ↓
Unit Tests
 ↓
Static Analysis
 ↓
Integration Tests
 ↓
Package
 ↓
Build Image
 ↓
Security Scan
 ↓
Push Image
 ↓
Deploy
 ↓
Smoke Tests
```

---

# 55. Continuous Integration

CI means developers frequently integrate changes and automatically validate them.

Typical CI:

```text
git push
 ↓
Build
 ↓
Tests
 ↓
Quality checks
```

---

# 56. Continuous Delivery

Continuous Delivery means the software is kept in a deployable state and can be released through an automated process.

Deployment to production may still require approval.

---

# 57. Continuous Deployment

Continuous Deployment automatically releases validated changes to production.

```text
Commit
 ↓
Pipeline
 ↓
Production
```

with no manual production approval step.

---

# 58. CI vs CD

```text
CI
→ Integrate and validate changes.

Continuous Delivery
→ Keep software ready to release.

Continuous Deployment
→ Automatically release validated changes.
```

---

# 59. Build Artifact

A Java build may produce:

```text
app.jar
```

The pipeline then packages it:

```text
app.jar
 ↓
Docker image
```

---

# 60. Artifact Immutability

Once an artifact is promoted:

```text
Build 123
```

don't modify it between environments.

Use:

```text
Same artifact
Dev → Staging → Production
```

This reduces environment drift.

---

# 61. Maven in CI

Typical:

```bash
./mvnw clean verify
```

This can run:

```text
Compilation
Unit tests
Integration checks
Packaging
```

depending on project configuration.

---

# 62. Static Analysis

Tools such as:

```text
SonarQube
```

can identify:

```text
Code smells
Potential bugs
Security issues
Duplicated code
Coverage information
```

---

# 63. Container Security

Scan images for vulnerabilities.

Consider:

```text
Base image vulnerabilities
OS packages
Application dependencies
Secrets
Configuration
```

Tools vary by organization.

---

# 64. Dependency Scanning

Java dependencies should be checked for known vulnerabilities.

Example concerns:

```text
Spring dependency vulnerability
Log4j vulnerability
Transitive dependency issue
```

Use appropriate SCA/security tooling.

---

# 65. Deployment Strategies

Common strategies:

```text
Rolling
Blue-Green
Canary
Recreate
```

---

# 66. Rolling Deployment

Gradually replace old Pods with new Pods.

```text
v1 v1 v1
 ↓
v2 v1 v1
 ↓
v2 v2 v1
 ↓
v2 v2 v2
```

Advantages:

```text
No full outage in many setups
Gradual replacement
Resource efficient
```

---

# 67. Rolling Deployment Risk

If v2 has a serious bug:

```text
Some traffic
 ↓
v2
 ↓
Errors
```

A rollback may be required.

---

# 68. Blue-Green Deployment

Two environments:

```text
Blue = current
Green = new
```

Traffic initially:

```text
Users
 ↓
Blue
```

After validation:

```text
Users
 ↓
Green
```

Rollback:

```text
Users
 ↓
Blue
```

---

# 69. Blue-Green Benefits

```text
Fast rollback
Clear separation
Easy validation
```

Costs:

```text
Additional infrastructure
Database compatibility considerations
```

---

# 70. Canary Deployment

Send a small percentage of traffic to the new version.

```text
v1 → 95%
v2 → 5%
```

Monitor:

```text
Errors
Latency
Business metrics
```

If healthy:

```text
v1 → 50%
v2 → 50%
```

then eventually:

```text
v2 → 100%
```

---

# 71. Canary Benefits

```text
Lower blast radius
Real production validation
Gradual rollout
```

---

# 72. Database Compatibility During Deployment

Suppose:

```text
v1 expects column A
v2 expects column B
```

During rolling deployment both may run.

Therefore use backward-compatible database migrations.

Pattern:

```text
Expand
 ↓
Deploy compatible code
 ↓
Migrate
 ↓
Contract
```

---

# 73. Rollback

A rollback restores a previously known-good application version.

Example:

```text
v2 bad
 ↓
rollback
 ↓
v1
```

But database rollback can be harder.

Avoid destructive schema changes that cannot coexist with the previous application version.

---

# 74. GitOps

GitOps treats Git as the source of truth for deployment configuration.

Conceptually:

```text
Git
 ↓
Desired deployment state
 ↓
GitOps controller
 ↓
Kubernetes
```

---

# 75. GitOps Benefits

```text
Auditable changes
Versioned configuration
Reproducibility
Automated reconciliation
Easy rollback through Git history
```

---

# 76. Helm

Helm is a package manager/template system commonly used for Kubernetes applications.

It can template:

```text
Deployments
Services
ConfigMaps
Ingress
```

---

# 77. Helm Values

Environment-specific values can be supplied through:

```text
values-dev.yaml
values-staging.yaml
values-prod.yaml
```

This reduces duplicated manifests.

---

# 78. Service Configuration

Don't bake environment-specific values into the application image.

Bad:

```text
Production DB URL
```

hardcoded into image.

Better:

```text
Configuration injected at deployment/runtime.
```

---

# 79. Rolling Deployment and Readiness

A good rolling deployment relies on readiness.

```text
New Pod starts
 ↓
Readiness fails
 ↓
No production traffic
 ↓
Startup completes
 ↓
Readiness passes
 ↓
Traffic begins
```

This prevents traffic from reaching an unready application.

---

# 80. Graceful Shutdown

During deployment:

```text
Pod receives termination signal
 ↓
Stop accepting new requests
 ↓
Finish in-flight work
 ↓
Shutdown
```

Spring Boot supports graceful shutdown configuration.

---

# 81. Why Graceful Shutdown?

Without it:

```text
Request in progress
 ↓
Pod killed
 ↓
Request fails
```

Graceful shutdown reduces dropped requests.

---

# 82. Connection Draining

Load balancers and proxies may need to stop routing new traffic to a terminating instance while allowing existing connections to finish.

This works together with:

```text
Readiness
Termination handling
Graceful shutdown
```

---

# 83. Pod Disruption Budget

A PodDisruptionBudget can help maintain minimum availability during voluntary disruptions.

Example concept:

```text
At least 2 Pods available
```

during planned maintenance.

It does not protect against every possible failure.

---

# 84. Node Failure

If a worker node fails:

```text
Pods on node
 ↓
Unavailable
```

Kubernetes can reschedule workloads onto healthy nodes if the desired state and cluster capacity allow it.

---

# 85. Availability Zones

For higher availability, spread workloads across failure domains.

Example:

```text
Zone A → Order Pods
Zone B → Order Pods
Zone C → Order Pods
```

Avoid placing every replica on one failure domain.

---

# 86. Anti-Affinity

Kubernetes scheduling rules can discourage replicas from being placed on the same node or zone.

Goal:

```text
Pod A → Node 1
Pod B → Node 2
Pod C → Node 3
```

rather than:

```text
Pod A
Pod B
Pod C
 ↓
Same node
```

---

# 87. Service-to-Service Networking

Inside Kubernetes:

```text
Order Pod
 ↓
order-service
 ↓
Inventory Service
```

Kubernetes provides service discovery through DNS.

For external services:

```text
Payment Pod
 ↓
External payment provider
```

network policies, proxies and egress controls may apply.

---

# 88. Network Policies

Network policies can restrict which workloads can communicate.

Example:

```text
Order → Inventory = allowed
Order → Payment DB = denied
```

This supports least-privilege networking.

---

# 89. Namespace Isolation Is Not Enough

Namespaces help organize/isolate resources, but don't assume they automatically prevent all network communication.

Use appropriate:

```text
RBAC
Network Policies
Security Contexts
Admission controls
```

---

# 90. Kubernetes RBAC

RBAC controls:

```text
Who
can do
what
on which resources
```

Example:

```text
CI Service Account
→ Can update Deployments
→ Cannot read all Secrets
```

Follow least privilege.

---

# 91. Service Account

A Pod can run under a Kubernetes ServiceAccount.

The identity can be used for:

```text
Kubernetes API access
Cloud workload identity
Authorization
```

depending on platform integration.

---

# 92. Container Security Context

Security contexts can restrict:

```text
User/group
Capabilities
Privilege escalation
Filesystem access
```

Avoid running containers as root unless there is a justified requirement.

---

# 93. Production Deployment Checklist

Before production:

```text
□ Image scanned
□ Secrets externalized
□ Resource requests configured
□ Resource limits reviewed
□ Readiness configured
□ Liveness configured
□ Startup behavior tested
□ Logging enabled
□ Metrics enabled
□ Tracing enabled
□ Rollback tested
□ Database migration tested
□ Network policies reviewed
□ RBAC reviewed
□ Autoscaling tested
```

---

# 94. Production Scenario

### "New deployment starts but receives traffic immediately and returns 503."

Likely issue:

```text
Readiness not configured correctly
```

Fix:

```text
Readiness probe
+
Correct startup behavior
```

---

# 95. Production Scenario

### "New release causes errors. How do you reduce blast radius?"

Use:

```text
Canary deployment
```

Example:

```text
v1 = 95%
v2 = 5%
```

Monitor:

```text
5xx
Latency
Business metrics
```

---

# 96. Production Scenario

### "Application container keeps restarting."

Check:

```text
Container logs
Exit code
Liveness probe
Startup probe
OOMKilled
Resource limits
Application startup errors
Configuration
Secrets
```

Don't assume Kubernetes is the root cause.

---

# 97. Production Scenario

### "Pod is running but traffic doesn't reach it."

Check:

```text
Readiness probe
Service selector
Service endpoints
Port configuration
Ingress
Network policies
Application listening address
```

---

# 98. Production Scenario

### "Memory keeps increasing."

Check:

```text
JVM heap
GC
Heap dump if appropriate
Traffic patterns
Caching
Object retention
Memory limits
```

A memory limit may turn the symptom into:

```text
OOMKilled
```

rather than fixing the leak.

---

# 99. Production Scenario

### "Rolling deployment breaks because old and new versions use different schemas."

Root issue:

```text
Non-backward-compatible database migration
```

Use:

```text
Expand-and-contract
```

so both versions can operate during rollout.

---

# 100. Production Scenario

### "A deployment must be reversed immediately."

For application code:

```text
Rollback to previous image
```

For database:

```text
Prefer forward-compatible repair
```

Avoid assuming database rollback is as easy as application rollback.

---

# 101. Interview Question

### "What is Docker?"

Answer:

> "Docker is a containerization platform commonly used to package applications and their runtime dependencies into portable images that can be run consistently across environments."

---

# 102. Interview Question

### "Image vs container?"

Answer:

> "An image is an immutable packaged template, while a container is a running instance of that image."

---

# 103. Interview Question

### "Why use multi-stage Docker builds?"

Answer:

> "They let us build the application with a full build environment and then copy only the required runtime artifact into a smaller runtime image. This reduces image size and attack surface."

---

# 104. Interview Question

### "What is Kubernetes?"

Answer:

> "Kubernetes is a container orchestration platform that manages scheduling, networking, scaling, health checks, rolling deployments and self-healing for containerized workloads."

---

# 105. Interview Question

### "What is a Pod?"

Answer:

> "A Pod is Kubernetes' smallest deployable unit. It usually contains one application container, although multiple tightly coupled containers can share a Pod."

---

# 106. Interview Question

### "Deployment vs Pod?"

Answer:

> "A Pod runs the workload, while a Deployment manages the desired number of Pod replicas and handles rollout and replacement through ReplicaSets."

---

# 107. Interview Question

### "Why do we need a Kubernetes Service?"

Answer:

> "Pods are ephemeral and their IPs can change. A Service provides stable networking and service discovery for a group of Pods."

---

# 108. Interview Question

### "Liveness vs readiness?"

Answer:

> "Liveness tells Kubernetes whether the application should be restarted. Readiness tells Kubernetes whether the instance should receive traffic."

---

# 109. Interview Question

### "What is HPA?"

Answer:

> "Horizontal Pod Autoscaler adjusts the number of Pod replicas based on configured metrics such as CPU or custom metrics."

---

# 110. Interview Question

### "What is a ConfigMap?"

Answer:

> "A ConfigMap stores non-sensitive configuration that can be injected into Pods as environment variables or files."

---

# 111. Interview Question

### "How should secrets be handled?"

Answer:

> "Secrets should be externalized from the application image and managed through a controlled secret-management system. Kubernetes Secrets can be used with appropriate RBAC and encryption-at-rest controls, while platforms may also integrate with Vault or cloud secret managers."

---

# 112. Interview Question

### "Rolling vs blue-green vs canary?"

Answer:

> "Rolling replaces instances gradually. Blue-green keeps two environments and switches traffic between them. Canary sends a small percentage of production traffic to the new version before increasing it. I'd choose based on rollback speed, infrastructure cost and required blast-radius control."

---

# 113. Interview Question

### "What is CI/CD?"

Answer:

> "CI automatically builds and validates integrated code changes. Continuous Delivery keeps software deployable, while Continuous Deployment automatically releases validated changes to production."

---

# 114. Interview Question

### "How would you deploy a Spring Boot microservice?"

Answer:

> "I'd build and test the application through CI, package the JAR into a versioned Docker image, scan and push it to a container registry, then deploy it through Kubernetes or another orchestration platform. I'd configure health probes, resources, externalized configuration, observability and a safe rollout strategy such as rolling or canary deployment."

---

# 115. Final Deployment Architecture

```text
                         Git
                          |
                          ↓
                     CI Pipeline
                          |
              +-----------+-----------+
              |                       |
            Tests                  Security
              |                    Scanning
              +-----------+-----------+
                          |
                          ↓
                    Docker Build
                          |
                          ↓
                  Container Registry
                          |
                          ↓
                    CD / GitOps
                          |
                          ↓
                    Kubernetes
                          |
             +------------+------------+
             |            |            |
             ↓            ↓            ↓
          Order       Inventory      Payment
          Service      Service       Service
             |            |            |
             ↓            ↓            ↓
           Pods         Pods         Pods
             |
             ↓
          Database
```

---

# 116. Final Mental Model

```text
Git
 ↓
CI
 ↓
Build + Test
 ↓
Docker Image
 ↓
Registry
 ↓
CD / GitOps
 ↓
Kubernetes
 ↓
Deployment
 ↓
Pods
 ↓
Service
 ↓
Ingress
 ↓
Users
```

And around everything:

```text
Observability
Security
Secrets
Scaling
Health checks
Rollback
```

---

# 117. Interviewer's Real Test

If asked:

> "A new Spring Boot version is ready. How would you safely deploy it?"

Think:

```text
Code
 ↓
Build
 ↓
Unit + Integration Tests
 ↓
Static/Security Checks
 ↓
Build Immutable Docker Image
 ↓
Push Registry
 ↓
Deploy to Kubernetes
 ↓
Startup Probe
 ↓
Readiness Probe
 ↓
Small rollout/canary
 ↓
Monitor errors + latency
 ↓
Increase traffic
 ↓
Complete rollout
```

If something goes wrong:

```text
Stop rollout
 ↓
Rollback application image
 ↓
Investigate
 ↓
Repair forward if DB schema changed
```

The key interview lesson is:

> **A production deployment is not just "put the JAR on a server." A reliable microservices deployment combines immutable artifacts, automated CI/CD, health checks, resource controls, safe rollout strategies, observability, security and a tested rollback path.**

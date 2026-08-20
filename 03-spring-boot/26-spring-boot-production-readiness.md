# Spring Boot — Production Readiness

This file covers the practical production-readiness topics expected from a Java/Spring Boot backend developer.

The goal is to understand how to take a Spring Boot application from:

```text
"It works on my machine"
```

to:

```text
"Safe to run in production"
```

---

# 1. What Is Production Readiness?

Production readiness means the application is prepared to handle:

```text
Real traffic
Failures
Security threats
Operational incidents
Deployments
Scaling
Monitoring
Data recovery
```

A production-ready application should be:

```text
Reliable
Observable
Secure
Scalable
Maintainable
Recoverable
```

---

# 2. Production Readiness Checklist

Before deployment, check:

```text
□ Configuration
□ Secrets
□ Database
□ Connection pools
□ Logging
□ Metrics
□ Tracing
□ Health checks
□ Security
□ Error handling
□ Timeouts
□ Retry policies
□ Resource limits
□ Graceful shutdown
□ Testing
□ CI/CD
□ Deployment strategy
□ Backups
□ Disaster recovery
□ Alerts
□ Documentation
```

---

# 3. Environment Configuration

Never hardcode environment-specific values.

Bad:

```java
String dbUrl =
    "jdbc:mysql://production-db:3306/orders";
```

Better:

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

Use different configuration for:

```text
local
development
test
staging
production
```

---

# 4. Spring Profiles

Spring profiles allow environment-specific configuration.

Example:

```text
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

Activate:

```text
SPRING_PROFILES_ACTIVE=prod
```

Avoid putting production credentials into source control.

---

# 5. Configuration Hierarchy

Spring Boot supports configuration from multiple sources.

Common sources include:

```text
application.yml
Environment variables
Command-line arguments
External configuration
Secret/configuration systems
```

Environment-specific configuration should be externally controllable.

---

# 6. Secrets Management

Sensitive values include:

```text
Database passwords
JWT signing keys
API keys
Cloud credentials
OAuth client secrets
Encryption keys
```

Don't commit them to Git.

Use:

```text
Environment variables
Secret managers
Kubernetes Secrets
Cloud secret-management services
```

---

# 7. Configuration Validation

Fail early if required configuration is missing.

Example:

```java
@ConfigurationProperties(prefix = "payment")
@Validated
public class PaymentProperties {

    @NotBlank
    private String baseUrl;

    @NotBlank
    private String apiKey;
}
```

A bad production configuration should ideally prevent unsafe startup rather than causing mysterious runtime failures.

---

# 8. Database Configuration

Production database configuration should consider:

```text
URL
Credentials
Connection pool
Timeouts
SSL/TLS
Connection limits
Read/write topology
Migration strategy
```

Don't simply copy development settings into production.

---

# 9. HikariCP

Spring Boot commonly uses HikariCP.

Architecture:

```text
Spring Boot
    ↓
HikariCP
    ↓
Database
```

Connections are reused rather than opened for every request.

---

# 10. Connection Pool Sizing

Suppose:

```text
10 application instances
20 connections/instance
```

Potential connections:

```text
200
```

The database must be capable of handling that total.

More connections can actually reduce performance if the database becomes saturated.

---

# 11. Connection Pool Metrics

Monitor:

```text
Active connections
Idle connections
Pending threads
Maximum pool size
Connection acquisition time
Connection timeout
```

If requests are waiting for connections:

```text
Application latency increases
```

---

# 12. Database Migration

Use a migration tool such as:

```text
Flyway
Liquibase
```

Migration files should be version-controlled.

Example:

```text
V1__create_users.sql
V2__create_orders.sql
V3__add_order_status.sql
```

---

# 13. Safe Database Migration

Avoid destructive changes during the same deployment that still depends on the old schema.

Safer pattern:

```text
1. Add compatible schema
2. Deploy compatible application
3. Backfill data
4. Switch application behavior
5. Remove old schema later
```

This is important during rolling deployments.

---

# 14. Backward-Compatible Schema Changes

Example:

Old:

```text
name
```

New requirement:

```text
first_name
last_name
```

Safer rollout:

```text
Add first_name
Add last_name
Deploy code supporting both
Backfill
Switch reads/writes
Remove name later
```

Avoid immediately renaming/removing a column when old application instances may still be running.

---

# 15. Transaction Management

Transactions protect business operations that must succeed or fail together.

Example:

```java
@Transactional
public void createOrder(...) {
    saveOrder();
    saveOrderItems();
}
```

If a failure occurs:

```text
Rollback
```

---

# 16. Transaction Boundary

Put the transaction around the business operation rather than arbitrary individual repository calls.

Usually:

```text
Controller
   ↓
Service
   ↓
@Transactional
   ↓
Repositories
```

The correct boundary depends on the business operation.

---

# 17. Avoid Long Transactions

Long transactions can cause:

```text
Locks
Connection exhaustion
Poor throughput
Deadlocks
Increased latency
```

Avoid doing slow external network calls inside database transactions where possible.

Bad:

```text
BEGIN
 ↓
Database update
 ↓
Call payment API
 ↓
Wait 5 seconds
 ↓
COMMIT
```

Prefer a design that keeps the database transaction focused.

---

# 18. Database Query Optimization

Production applications should monitor:

```text
Slow queries
Missing indexes
N+1 queries
Large result sets
Unnecessary joins
Full table scans
```

Use execution plans when investigating query performance.

---

# 19. Pagination

Never load an unbounded dataset:

```java
repository.findAll();
```

for a potentially huge table.

Use pagination:

```text
page
size
sort
```

For very large datasets, consider keyset/cursor pagination.

---

# 20. N+1 Queries

Example:

```text
1 query → orders
100 queries → customers
```

Total:

```text
101 queries
```

Possible solutions:

```text
Fetch joins
Projections
Batch fetching
Purpose-built queries
```

Measure before choosing.

---

# 21. Indexing

Example:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Indexes can improve reads but increase:

```text
Storage
Write overhead
Maintenance
```

Index based on actual query patterns.

---

# 22. API Timeouts

External calls should have explicit timeouts.

Example:

```text
Connection timeout
Read/response timeout
Overall request timeout
```

Never allow:

```text
Request
 ↓
External service
 ↓
Wait forever
```

---

# 23. Retry Strategy

Retry only transient failures.

Good candidates:

```text
Temporary network failure
Temporary 503
Transient connection failure
```

Usually not:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
Invalid business data
```

---

# 24. Retry With Backoff

Example:

```text
100ms
200ms
400ms
800ms
```

Add jitter where many instances may retry simultaneously.

---

# 25. Retry and Idempotency

Before retrying a write operation, ask:

```text
Can this operation safely run twice?
```

Example:

```text
Payment
```

should use an idempotency mechanism before automatic retries.

---

# 26. Circuit Breaker

If a dependency repeatedly fails:

```text
CLOSED
   ↓
OPEN
   ↓
Fail fast
   ↓
HALF_OPEN
```

The circuit breaker prevents repeated calls from overwhelming an unhealthy dependency.

---

# 27. Bulkhead

Separate resources between critical workloads.

Example:

```text
Payment calls → Pool A
Recommendation calls → Pool B
```

If recommendations become slow, payment processing still has reserved resources.

---

# 28. Rate Limiting

Protect APIs from excessive requests.

Example:

```text
100 requests/minute/user
```

Excess:

```text
429 Too Many Requests
```

Limits should reflect business and capacity requirements.

---

# 29. Load Shedding

When overloaded:

```text
Accept everything
 ↓
Queues grow
 ↓
Latency explodes
 ↓
System collapses
```

Instead:

```text
Reject low-priority work
Protect critical operations
```

Predictable failure is better than total failure.

---

# 30. Graceful Degradation

If a non-critical dependency fails:

```text
Product page
    |
    +--> Product details → available
    |
    +--> Recommendations → fallback
```

The whole request does not necessarily need to fail.

---

# 31. Health Checks

Spring Boot Actuator provides production health endpoints.

Common endpoints:

```text
/actuator/health
/actuator/info
/actuator/metrics
```

Expose only what is appropriate for your environment.

---

# 32. Liveness

Liveness answers:

```text
Is this process alive?
```

If liveness fails, the orchestrator may restart the application.

Do not make liveness depend on every external dependency or a temporary database outage unless that is explicitly intended.

---

# 33. Readiness

Readiness answers:

```text
Should this instance receive traffic?
```

If the application cannot safely serve requests:

```text
Not ready
```

The load balancer/orchestrator can stop routing traffic to it.

---

# 34. Startup vs Readiness

During startup:

```text
Application
 ↓
Initialize
 ↓
Connect dependencies
 ↓
Ready
```

A service should not receive production traffic before it is capable of handling requests correctly.

---

# 35. Graceful Shutdown

During deployment:

```text
Instance receives shutdown
       ↓
Stop accepting new work
       ↓
Finish in-flight requests
       ↓
Close resources
       ↓
Exit
```

This reduces dropped requests.

---

# 36. Connection Cleanup

On shutdown, properly release:

```text
Database connections
Redis connections
HTTP clients
Kafka consumers
Executor threads
File handles
```

Framework-managed resources should generally be allowed to shut down through Spring's lifecycle.

---

# 37. Logging

Production logs should be:

```text
Structured
Searchable
Useful
Safe
Correlated
```

Example:

```text
level=INFO
event=ORDER_CREATED
orderId=1001
traceId=abc123
```

---

# 38. Log Levels

Common levels:

```text
TRACE
DEBUG
INFO
WARN
ERROR
```

Production normally avoids excessive DEBUG/TRACE logging unless temporarily enabled for investigation.

---

# 39. What Not to Log

Never casually log:

```text
Passwords
JWT tokens
API keys
Credit card numbers
Private keys
Secrets
Sensitive personal information
```

Mask sensitive values where logging is necessary.

---

# 40. Structured Logging

Instead of:

```text
Order 1001 failed because payment failed
```

prefer structured fields:

```text
event=PAYMENT_FAILED
orderId=1001
reason=TIMEOUT
traceId=abc123
```

This makes searching and aggregation easier.

---

# 41. Correlation ID

A correlation ID connects logs from one business request.

Example:

```text
Gateway
  ↓ ABC
Order Service
  ↓ ABC
Payment Service
  ↓ ABC
Inventory Service
```

Every service logs:

```text
correlationId=ABC
```

---

# 42. Distributed Tracing

A trace follows a request across services.

Example:

```text
Gateway
  |
  +-- Order Service
        |
        +-- Payment
        |
        +-- Inventory
```

Use tracing to identify where latency occurs.

---

# 43. OpenTelemetry

OpenTelemetry provides standardized instrumentation for:

```text
Traces
Metrics
Logs
Context propagation
```

It can integrate with various observability backends.

---

# 44. Metrics

Track:

```text
Request rate
Error rate
Latency
CPU
Memory
GC
DB connections
DB latency
Redis hit ratio
Kafka lag
External API latency
```

---

# 45. RED Metrics

For APIs:

```text
Rate
Errors
Duration
```

These provide a useful high-level service view.

---

# 46. USE Metrics

For infrastructure:

```text
Utilization
Saturation
Errors
```

Examples:

```text
CPU utilization
DB connection saturation
Disk utilization
Network errors
```

---

# 47. JVM Metrics

Monitor:

```text
Heap usage
Non-heap usage
GC activity
Thread count
Class loading
CPU
```

A Java application can be healthy at the HTTP level while experiencing JVM resource pressure.

---

# 48. Garbage Collection

Excessive GC can cause:

```text
Latency spikes
CPU usage
Throughput reduction
OutOfMemoryError
```

Investigate:

```text
Allocation rate
Heap usage
GC pauses
Object retention
```

Don't simply increase heap size without understanding the cause.

---

# 49. Thread Pool

Spring applications can use thread pools for:

```text
HTTP requests
Async work
Scheduled tasks
Messaging
```

Monitor:

```text
Active threads
Queue size
Rejected tasks
Execution time
```

---

# 50. Thread Pool Exhaustion

Suppose:

```text
Thread pool = 100
100 requests blocked on slow dependency
```

New requests may wait or be rejected.

This is why timeouts and bulkheads are important.

---

# 51. Async Processing

If using:

```java
@Async
```

configure and monitor an appropriate executor.

Don't rely on unlimited thread creation.

---

# 52. Async Pitfall

Bad:

```text
Request
 ↓
@Async
 ↓
Create unlimited background work
```

During traffic spikes this can exhaust:

```text
Memory
Threads
CPU
Downstream capacity
```

Use bounded executors and backpressure where appropriate.

---

# 53. Caching

Production caching requires:

```text
TTL
Invalidation
Memory limits
Eviction policy
Failure handling
Monitoring
```

Do not assume Redis automatically makes every application faster.

---

# 54. Redis Production Configuration

Consider:

```text
Authentication
TLS
Network isolation
Timeouts
Connection pools
Memory limits
Eviction policy
High availability
Monitoring
```

---

# 55. Database Resilience

Consider:

```text
Primary/replica
Failover
Backups
Connection timeout
Query timeout
Retry strategy
Read/write separation
```

Don't retry unsafe transactional operations blindly.

---

# 56. Database Backups

A production backup strategy should define:

```text
Frequency
Retention
Encryption
Storage location
RPO
RTO
Restore procedure
Restore testing
```

A backup is valuable only if it can actually be restored.

---

# 57. RPO

Recovery Point Objective:

```text
Maximum acceptable data loss
```

Example:

```text
RPO = 5 minutes
```

The architecture and backup strategy should support that requirement.

---

# 58. RTO

Recovery Time Objective:

```text
Maximum acceptable recovery time
```

Example:

```text
RTO = 30 minutes
```

This influences failover and disaster recovery architecture.

---

# 59. Disaster Recovery

Consider:

```text
Backups
Replication
Failover
Recovery runbooks
Infrastructure recreation
DNS/load-balancer changes
Data recovery
Testing
```

Don't assume disaster recovery works until it has been tested.

---

# 60. Multi-AZ

Deploy critical components across failure domains.

Example:

```text
Zone A
  App
  DB replica

Zone B
  App
  DB replica

Zone C
  App
```

This reduces the impact of a single availability-zone failure.

---

# 61. Multi-Region

For stronger disaster tolerance:

```text
Region A
   |
   +------ Region B
```

Tradeoffs:

```text
Cost
Latency
Replication complexity
Consistency
Operational complexity
```

Only use it when business requirements justify it.

---

# 62. Security Baseline

Production applications should consider:

```text
HTTPS
Authentication
Authorization
Input validation
CSRF where applicable
CORS configuration
Rate limiting
Secrets management
Dependency security
Audit logging
Least privilege
```

---

# 63. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

Example:

```text
JWT proves identity
Role/permission controls access
```

---

# 64. Password Storage

Never store plaintext passwords.

Use a strong password hashing algorithm such as:

```text
BCrypt
Argon2
```

Store only the resulting password hash and required parameters.

---

# 65. JWT Security

Protect:

```text
Signing key
Token expiration
Issuer
Audience
Algorithm
Refresh token
```

Don't put sensitive information into JWT claims simply because they are encoded.

---

# 66. Input Validation

Use:

```java
@Valid
```

with constraints:

```java
@NotBlank
@Email
@Size
@Min
@Max
```

Validate at the API boundary and enforce business rules in the service/domain layer.

---

# 67. SQL Injection

Use parameterized queries/JPA rather than string concatenation.

Bad:

```java
"SELECT * FROM users WHERE name = '" + name + "'"
```

Better:

```java
repository.findByName(name);
```

---

# 68. Dependency Security

Regularly scan dependencies for known vulnerabilities.

Consider:

```text
Dependency updates
Security advisories
SCA tools
SBOM
Container image scanning
```

Don't blindly upgrade major versions without testing compatibility.

---

# 69. SBOM

Software Bill of Materials describes application components and dependencies.

It helps with:

```text
Vulnerability tracking
Compliance
Incident response
Dependency visibility
```

---

# 70. Docker Image

Typical build:

```text
Source
 ↓
Maven
 ↓
JAR
 ↓
Docker image
 ↓
Container
```

Use a minimal trusted base image and avoid running as root where practical.

---

# 71. Dockerfile Example

Conceptual:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/app.jar app.jar

USER 1001

ENTRYPOINT ["java", "-jar", "app.jar"]
```

The exact image/version should be selected and maintained according to organizational policy.

---

# 72. Container Health

Container orchestration should know:

```text
Is application alive?
Should it receive traffic?
Is it ready?
```

Spring Boot Actuator health endpoints can support these checks.

---

# 73. Kubernetes Deployment

Typical components:

```text
Deployment
Service
Ingress
ConfigMap
Secret
HorizontalPodAutoscaler
```

Spring Boot container:

```text
Pod
  ↓
Application
```

---

# 74. Resource Requests and Limits

Containers should define appropriate:

```text
CPU request
CPU limit
Memory request
Memory limit
```

Badly chosen memory limits can cause:

```text
OOMKilled
```

while insufficient CPU can cause latency and throttling.

---

# 75. Autoscaling

Autoscaling can respond to:

```text
CPU
Memory
Request rate
Queue depth
Custom metrics
```

But scaling the application doesn't fix:

```text
Database bottleneck
Third-party rate limit
Kafka partition limit
```

---

# 76. Deployment Strategies

Common:

```text
Rolling
Blue-Green
Canary
```

---

# 77. Rolling Deployment

Replace instances gradually:

```text
A A A A
↓
B A A A
↓
B B A A
↓
B B B A
↓
B B B B
```

Requires backward compatibility between versions.

---

# 78. Blue-Green Deployment

Two environments:

```text
Blue → current
Green → new
```

After validation:

```text
Traffic
  ↓
Green
```

Rollback can switch traffic back to Blue.

---

# 79. Canary Deployment

Release to a small percentage:

```text
95% → old
5%  → new
```

Monitor:

```text
Errors
Latency
Business metrics
```

Then gradually increase traffic.

---

# 80. Rollback

A deployment should have a rollback strategy.

Possible:

```text
Previous application version
Previous container image
Traffic switch
Feature flag
```

Database changes must also be rollback-compatible or forward-compatible.

---

# 81. Feature Flags

Feature flags allow behavior to be enabled gradually.

Example:

```text
newCheckout = false
```

Then:

```text
5% users → true
```

Feature flags can reduce deployment risk.

But remove stale flags after the rollout is complete.

---

# 82. CI/CD Pipeline

Typical:

```text
Git push
   ↓
Build
   ↓
Unit tests
   ↓
Integration tests
   ↓
Static analysis
   ↓
Security scan
   ↓
Build image
   ↓
Deploy staging
   ↓
Smoke tests
   ↓
Production
```

---

# 83. Quality Gates

Pipeline can fail if:

```text
Tests fail
Code quality threshold fails
Security vulnerability threshold fails
Build fails
Container scan fails
```

Quality gates prevent known problems from reaching production.

---

# 84. Testing Pyramid

```text
        E2E
       /   \
 Integration
    /       \
   Unit Tests
```

Unit tests should generally be fast and numerous.

Integration tests verify real interactions.

End-to-end tests cover critical workflows.

---

# 85. Unit Testing

Test business logic in isolation.

Example:

```java
@Test
void shouldCalculateDiscount() {
    ...
}
```

Use mocks only where they provide value.

---

# 86. Integration Testing

Verify:

```text
Spring context
Database
Redis
Kafka
HTTP integration
```

Testcontainers is useful for realistic infrastructure dependencies.

---

# 87. Contract Testing

For microservices:

```text
Producer
   ↓
API/Event Contract
   ↓
Consumer
```

Contract tests can catch breaking changes before deployment.

---

# 88. Smoke Testing

After deployment, run a small set of critical checks:

```text
Application starts
Health endpoint works
Login works
Read endpoint works
Critical write works
Database reachable
```

Smoke tests should be fast.

---

# 89. Canary Validation

During canary:

```text
5% traffic
```

Compare:

```text
Error rate
Latency
CPU
Memory
Business conversion
Payment failures
```

Rollback if meaningful regression occurs.

---

# 90. Observability During Deployment

Watch:

```text
5xx
p95/p99
Database latency
Connection pool
CPU
Memory
GC
Kafka lag
External API errors
```

Deployment should be monitored, not just executed.

---

# 91. Incident Response

When production breaks:

```text
Detect
 ↓
Triage
 ↓
Mitigate
 ↓
Recover
 ↓
Communicate
 ↓
Root cause
 ↓
Prevent recurrence
```

Focus first on restoring service.

---

# 92. Runbook

A runbook documents operational procedures.

Example:

```text
Kafka lag increasing
```

Steps:

```text
Check consumer health
Check downstream DB
Check error logs
Check partition distribution
Check recent deployments
Scale consumers if appropriate
Rollback if deployment caused issue
```

---

# 93. Root Cause Analysis

After an incident:

```text
What happened?
Why did it happen?
Why wasn't it detected?
Why wasn't it prevented?
What will we change?
```

Avoid focusing only on blaming an individual.

---

# 94. Error Budget

If availability target is:

```text
99.9%
```

some downtime/error is within the service objective.

Error budgets help balance:

```text
Reliability
Feature velocity
```

---

# 95. SLI

Service Level Indicator:

```text
Measured reliability metric
```

Example:

```text
Successful requests / total requests
```

---

# 96. SLO

Service Level Objective:

```text
Target for the SLI
```

Example:

```text
99.9% successful requests
```

---

# 97. SLA

Service Level Agreement:

```text
Contractual commitment
```

SLA may include:

```text
Availability
Response times
Support
Compensation
```

Don't treat SLI, SLO, and SLA as interchangeable.

---

# 98. Production Performance

Measure:

```text
p50
p95
p99
```

Example:

```text
p50 = 80 ms
p95 = 200 ms
p99 = 600 ms
```

A high p99 can indicate tail-latency problems even when average latency looks fine.

---

# 99. Performance Investigation

If an API becomes slow:

```text
Check metrics
 ↓
Check traces
 ↓
Identify slow dependency
 ↓
Check logs
 ↓
Check DB queries
 ↓
Check connection pools
 ↓
Check CPU/GC
```

Optimize the actual bottleneck.

---

# 100. Thread Dump

Thread dumps help investigate:

```text
Thread starvation
Deadlocks
Blocked threads
Long-running operations
Unexpected concurrency
```

Use them when the JVM shows thread-related problems.

---

# 101. Heap Dump

Heap dumps help investigate:

```text
Memory leaks
Unexpected object retention
Large collections
OutOfMemoryError
```

Analyze carefully because heap dumps can contain sensitive application data.

---

# 102. Deadlocks

A database deadlock can occur when:

```text
Transaction A locks row 1
Transaction B locks row 2

A waits for row 2
B waits for row 1
```

Database detects the cycle and usually aborts one transaction.

Investigate:

```text
Lock ordering
Transaction duration
Query patterns
Indexes
```

---

# 103. Application Deadlock

Threads can also deadlock:

```text
Thread A holds Lock 1
waits for Lock 2

Thread B holds Lock 2
waits for Lock 1
```

Use thread dumps to investigate.

Prefer simpler concurrency designs where possible.

---

# 104. Graceful Failure

A production system should fail in controlled ways.

Example:

```text
Payment unavailable
```

Possible response:

```text
Order → PENDING
```

instead of:

```text
Entire platform → unavailable
```

---

# 105. Dependency Isolation

External dependencies include:

```text
Payment provider
Email provider
Shipping provider
Database
Redis
Kafka
```

Each should have:

```text
Timeout
Failure handling
Monitoring
Capacity consideration
```

---

# 106. External API Client

A robust client should consider:

```text
Timeouts
Retry
Backoff
Circuit breaker
Rate limits
Idempotency
Error mapping
Observability
```

Don't make raw HTTP calls without considering failure behavior.

---

# 107. API Error Handling

Use centralized exception handling:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    ...
}
```

Map domain errors to consistent API responses.

Avoid returning:

```text
stack trace
database exception
internal class names
```

to clients.

---

# 108. Error Response

Example:

```json
{
  "code": "ORDER_NOT_FOUND",
  "message": "Order was not found",
  "traceId": "abc123"
}
```

Don't expose internal implementation details.

---

# 109. HTTP Status Strategy

Common production statuses:

```text
200 → successful read/update
201 → created
204 → successful no-content operation
400 → invalid request
401 → unauthenticated
403 → unauthorized
404 → resource not found
409 → business/state conflict
429 → rate limited
500 → unexpected server failure
503 → temporary service unavailable
```

---

# 110. API Security Headers

Depending on the architecture, consider:

```text
HSTS
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
```

Security headers should be configured appropriately for the application and clients.

---

# 111. CORS

CORS controls which browser origins can make cross-origin requests.

Avoid:

```text
Allow-Origin: *
```

for sensitive authenticated APIs unless the design explicitly requires it.

Configure allowed origins deliberately.

---

# 112. CSRF

CSRF protection is particularly relevant to browser-based applications using cookie-based authentication.

For stateless token-based APIs, the CSRF threat model is different.

Do not disable CSRF blindly; understand the authentication mechanism first.

---

# 113. Session Security

If using sessions:

```text
Secure cookies
HttpOnly
SameSite
Session expiration
Session invalidation
```

Avoid storing sensitive data in client-accessible cookies.

---

# 114. Secrets Rotation

Production secrets should be rotatable.

Examples:

```text
Database password
API key
JWT signing key
Cloud credentials
```

Applications should support safe rotation without unnecessary downtime where practical.

---

# 115. Least Privilege

A service should have only the permissions it needs.

Example:

```text
Order Service
→ read/write orders

Not:
→ full database administrator
```

Apply least privilege to:

```text
Database users
Cloud IAM
Kafka ACLs
Redis access
Kubernetes service accounts
```

---

# 116. Audit Logging

For sensitive operations, record:

```text
Who
What
When
Resource
Result
Correlation/trace ID
```

Examples:

```text
Admin changed product price
Admin cancelled order
User changed email
Payment status changed
```

Audit logs should be protected from unauthorized modification.

---

# 117. PII

Personally identifiable information should be handled carefully.

Examples:

```text
Email
Phone number
Address
Government identifiers
```

Use:

```text
Data minimization
Access control
Masking
Encryption where appropriate
Retention policies
```

---

# 118. Data Encryption

At rest:

```text
Database/storage encryption
```

In transit:

```text
TLS/HTTPS
```

Sensitive data may require additional application-level protection depending on the threat model.

---

# 119. Production Cache Checklist

```text
TTL
Invalidation
Stampede protection
Memory
Eviction
Security
HA
Monitoring
Fallback
```

---

# 120. Production Kafka Checklist

```text
Replication
Retention
Partitions
Consumer lag
Retry
DLT
Idempotency
Schema compatibility
Security
Monitoring
```

---

# 121. Production Database Checklist

```text
Indexes
Backups
Migrations
Connection pool
Timeouts
Slow queries
Replication
Failover
Monitoring
Restore testing
```

---

# 122. Production API Checklist

```text
Authentication
Authorization
Validation
Rate limiting
Timeouts
Error handling
Pagination
Idempotency
Observability
Versioning
```

---

# 123. Production JVM Checklist

```text
Heap
GC
Threads
CPU
Memory
Startup time
Shutdown behavior
JVM flags
Container limits
```

Use evidence and measurements rather than arbitrary JVM tuning.

---

# 124. Production Spring Boot Checklist

```text
Profiles
Externalized configuration
Actuator
Health checks
Structured logging
Metrics
Tracing
Security
Graceful shutdown
Database pool
HTTP timeouts
Dependency resilience
```

---

# 125. Production Readiness Example

Suppose an ecommerce backend is ready for production.

Architecture:

```text
                 Internet
                    |
                   WAF
                    |
              Load Balancer
                    |
              API Gateway
                    |
          +---------+---------+
          |         |         |
       App 1      App 2     App 3
          |         |         |
          +---------+---------+
                    |
          +---------+---------+
          |                   |
        Redis               MySQL
          |                   |
          |               Read Replica
          |
        Kafka
          |
   +------+------+------+
   |      |      |      |
Payment Inventory Notify Analytics
```

Operational layer:

```text
Metrics
Logs
Tracing
Alerts
Backups
CI/CD
```

---

# 126. Production Deployment Flow

```text
Developer
   ↓
Git
   ↓
CI
   ↓
Unit Tests
   ↓
Integration Tests
   ↓
Security Scan
   ↓
Build Docker Image
   ↓
Deploy Staging
   ↓
Smoke Tests
   ↓
Canary
   ↓
Production
   ↓
Monitor
```

---

# 127. Deployment Failure Flow

If canary shows:

```text
5xx ↑
p99 ↑
```

then:

```text
Stop rollout
     ↓
Rollback
     ↓
Restore service
     ↓
Investigate
```

Don't continue deployment hoping the problem disappears.

---

# 128. Production Incident Example

Problem:

```text
API latency suddenly increases
```

Investigation:

```text
Metrics
 ↓
p99 increased
 ↓
Tracing
 ↓
DB span slow
 ↓
DB metrics
 ↓
Connection pool saturated
 ↓
Slow query identified
```

Fix:

```text
Optimize query
Add appropriate index
Reduce unnecessary load
Monitor
```

---

# 129. Production Incident: Kafka Lag

Problem:

```text
Consumer lag increasing
```

Investigate:

```text
Consumer errors
 ↓
Processing latency
 ↓
Database latency
 ↓
Partition distribution
```

Possible fix:

```text
Optimize DB
Scale consumers
Increase partitions if justified
Fix downstream dependency
```

---

# 130. Production Incident: Redis Down

Problem:

```text
Redis unavailable
```

Potential response:

```text
Short timeout
 ↓
Controlled fallback
 ↓
Protect database
 ↓
Restore Redis
 ↓
Warm critical cache
 ↓
Monitor recovery
```

---

# 131. Production Incident: Database Down

Potential response:

```text
Detect
 ↓
Failover if configured
 ↓
Reduce non-critical traffic
 ↓
Protect retry storms
 ↓
Restore/repair
 ↓
Validate consistency
```

Do not blindly retry every request.

---

# 132. Production Incident: Memory Increase

Symptoms:

```text
Heap usage ↑
GC ↑
Latency ↑
```

Investigate:

```text
Heap dump
Allocation patterns
Caches
Large collections
Object retention
Recent deployment
```

Do not immediately increase heap without identifying why memory is growing.

---

# 133. Production Incident: Thread Pool Exhaustion

Symptoms:

```text
Requests waiting
Latency ↑
Rejected tasks
```

Investigate:

```text
Slow dependency
Blocked threads
Connection pool
Executor configuration
Long-running tasks
```

Fix the bottleneck rather than only increasing the thread count.

---

# 134. Production Incident: Deployment Breaks Old Clients

Problem:

```text
New API
 ↓
Breaking contract
 ↓
Old clients fail
```

Prevention:

```text
Backward-compatible changes
API versioning
Contract tests
Gradual rollout
```

---

# 135. Production Readiness Questions

Before shipping, ask:

```text
What happens if the DB goes down?
What happens if Redis goes down?
What happens if Kafka is unavailable?
What happens if payment times out?
What happens if the same request is retried?
What happens during deployment?
What happens if traffic doubles?
What happens if one instance dies?
What happens if an entire zone fails?
How do we detect the problem?
How do we recover?
```

If these questions have no answer, the system probably isn't production-ready.

---

# 136. Interview: What Makes a Spring Boot Application Production Ready?

> I look beyond whether the API works. I check externalized configuration and secrets, database and connection-pool settings, health checks, structured logging, metrics and tracing, security, timeouts, retry and failure handling, graceful shutdown, testing, CI/CD, deployment strategy, backups, and operational monitoring.

---

# 137. Interview: How Do You Monitor a Spring Boot Application?

> I use metrics for request rate, errors, latency, JVM resources, database pools, and dependency health. I use structured logs with correlation IDs and distributed tracing to follow requests across services. Spring Boot Actuator is useful for exposing health and metrics endpoints, while the actual monitoring backend depends on the environment.

---

# 138. Interview: Liveness vs Readiness?

> Liveness answers whether the application process should be considered alive and potentially restarted. Readiness answers whether the instance is currently capable of receiving traffic. I avoid making liveness depend on every external dependency because a temporary dependency failure shouldn't necessarily cause the application to restart repeatedly.

---

# 139. Interview: How Do You Handle Graceful Shutdown?

> During shutdown, the instance should stop receiving new traffic, allow in-flight requests to complete within a defined period, stop background processing safely, and release resources such as database connections, HTTP clients, and Kafka consumers.

---

# 140. Interview: How Do You Handle Production Secrets?

> I never commit secrets to source control. I externalize them through environment-specific configuration and a secret-management mechanism such as a cloud secret manager or Kubernetes Secret. Access should follow least privilege and secrets should support rotation.

---

# 141. Interview: How Do You Handle a Slow External API?

> I configure explicit timeouts, classify failures, use bounded retries with backoff only for transient errors, and use a circuit breaker or bulkhead when appropriate. For non-critical dependencies I also consider graceful degradation so the external failure doesn't take down the whole request.

---

# 142. Interview: How Do You Deploy Without Downtime?

> I run multiple application instances behind a load balancer, use readiness and health checks, support graceful shutdown, and make API and database changes backward compatible. Then I use a rolling, blue-green, or canary deployment strategy depending on the risk and infrastructure.

---

# 143. Interview: How Do You Handle Database Migrations During Deployment?

> I prefer backward-compatible migrations. For example, I can add a new nullable column first, deploy code that supports both schemas, backfill the data, switch the application to the new field, and remove the old field only after old versions are no longer running.

---

# 144. Interview: What Would You Monitor in Production?

> At the application level I'd monitor request rate, 4xx/5xx, p95/p99 latency, JVM memory and GC, thread pools, database connection pools, Redis health and hit ratio, Kafka lag, and external dependency latency. I would also configure alerts around sustained error rates, saturation, and availability.

---

# 145. Interview: What Is Your Approach to a Production Incident?

> First I focus on mitigation and restoring service. I check recent deployments, metrics, logs, and traces to identify the failing component. After stabilizing the system, I investigate root cause, document the incident, and add preventive actions such as tests, alerts, capacity changes, or code fixes.

---

# 146. Interview: How Do You Optimize a Slow API?

> I start with measurements rather than guessing. I look at p95/p99 latency, distributed traces, database queries, connection pools, external calls, CPU, GC, and thread pools. Once I identify the bottleneck, I optimize that specific area and compare metrics before and after the change.

---

# 147. Interview: What Happens If Redis Goes Down?

> If the data is cacheable from the database, I can use a controlled fallback, but I need to protect the database from a cache-miss storm. I would use short timeouts, appropriate circuit-breaking or fallback controls, restore Redis, and monitor recovery.

---

# 148. Interview: What Happens If Kafka Goes Down?

> I first determine whether the operation can continue without publishing the event. For important database-backed workflows, an Outbox Pattern allows the business transaction and event record to commit together. The event can then be published when Kafka becomes available.

---

# 149. Interview: What Happens If One Application Instance Dies?

> If the application is stateless and behind a load balancer, traffic can be routed to healthy instances. Readiness and liveness checks help detect unhealthy instances, while graceful shutdown reduces disruption during planned termination.

---

# 150. Interview: How Do You Scale a Spring Boot Application?

> I first identify the bottleneck. Stateless Spring Boot instances can usually be scaled horizontally behind a load balancer. Then I evaluate database capacity, caching, connection pools, Kafka partitions, external dependencies, and network resources. Scaling the application tier alone doesn't solve a database or downstream bottleneck.

---

# 151. Interview: How Do You Protect Against Traffic Spikes?

> I use horizontal scaling, autoscaling, caching, rate limiting, and load shedding where appropriate. For workloads that can be asynchronous, queues or Kafka can absorb bursts. I also verify that downstream systems have enough capacity because scaling the API tier can otherwise overload them.

---

# 152. Interview: What Is Your Production Logging Strategy?

> I use structured logs with useful business identifiers, correlation or trace IDs, appropriate log levels, and centralized collection. I avoid logging passwords, tokens, secrets, and unnecessary personal information.

---

# 153. Interview: What Is an SLO?

> An SLO is a target for a service-level indicator. For example, an API might have an SLO of 99.9% successful requests or a p95 latency target. SLOs help define what reliability the service is expected to provide.

---

# 154. Interview: RPO vs RTO?

> RPO defines how much data loss is acceptable after a disaster. RTO defines how quickly the service must be restored. Both affect backup, replication, and disaster-recovery architecture.

---

# 155. Final Production Checklist

```text
APPLICATION
□ Stateless where possible
□ Configuration externalized
□ Profiles/environment separation
□ Graceful shutdown
□ Timeouts
□ Error handling

DATABASE
□ Migrations
□ Indexes
□ Connection pool
□ Slow-query monitoring
□ Backups
□ Restore testing
□ Failover strategy

CACHE
□ TTL
□ Invalidation
□ Memory limits
□ Failure strategy
□ Monitoring

KAFKA
□ Partitions
□ Replication
□ Consumer lag
□ Retry
□ DLT
□ Idempotency
□ Schema compatibility

SECURITY
□ HTTPS
□ Authentication
□ Authorization
□ Secrets
□ Least privilege
□ Input validation
□ Dependency scanning

OBSERVABILITY
□ Logs
□ Metrics
□ Traces
□ Correlation IDs
□ Dashboards
□ Alerts

DEPLOYMENT
□ CI/CD
□ Automated tests
□ Security scan
□ Smoke tests
□ Rolling/canary strategy
□ Rollback
□ Backward-compatible DB changes

RECOVERY
□ RPO
□ RTO
□ Backups
□ Disaster recovery
□ Runbooks
□ Incident process
```

---

# 156. Final Mental Model

```text
                    PRODUCTION SYSTEM
                           |
        +------------------+------------------+
        |                  |                  |
      Code               Data             Dependencies
        |                  |                  |
   Spring Boot         MySQL/Redis          Kafka/APIs
        |                  |                  |
        +------------------+------------------+
                           |
                       Reliability
                           |
       +-------------------+-------------------+
       |                   |                   |
    Timeout              Retry             Isolation
       |                   |                   |
   Circuit Breaker     Backoff            Bulkhead
                           |
                      Observability
                           |
             Logs + Metrics + Traces
                           |
                        Security
                           |
                  Auth + Secrets + TLS
                           |
                       Operations
                           |
             Deploy + Monitor + Recover
```

---

# 157. Final Rule

> **Production readiness means designing for what happens when things go wrong, not only when everything works. A strong Spring Boot backend should be observable, secure, resilient, horizontally scalable, safely deployable, and recoverable.**

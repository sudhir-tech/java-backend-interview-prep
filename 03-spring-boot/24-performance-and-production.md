# Spring Boot — Performance and Production

This file covers the practical performance and production-readiness topics expected from a Java/Spring Boot backend developer.

The goal is to understand how to build, measure, optimize, deploy, monitor, and operate a Spring Boot application in production.

---

# 1. What Is Production Readiness?

Production readiness means the application is prepared for:

```text
Real traffic
Failures
Security issues
Deployments
Scaling
Monitoring
Recovery
```

A production-ready backend should be:

```text
Reliable
Observable
Secure
Scalable
Maintainable
Recoverable
```

---

# 2. Performance vs Scalability

Performance asks:

```text
How fast can the system handle a request?
```

Scalability asks:

```text
How well does the system handle increasing load?
```

Example:

```text
100 requests/sec → fast
1000 requests/sec → still acceptable
```

A system can be fast for a small workload but fail to scale.

---

# 3. Measure Before Optimizing

Never optimize based only on assumptions.

Measure:

```text
Latency
Throughput
CPU
Memory
GC
Database latency
Connection pools
Thread pools
External API latency
Cache hit ratio
```

A good rule:

> Find the bottleneck first, then optimize it.

---

# 4. Latency

Latency is the time required to complete an operation.

Example:

```text
Request → 120 ms → Response
```

Monitor:

```text
p50
p95
p99
```

Average latency can hide slow requests.

---

# 5. Throughput

Throughput measures how much work the system completes in a period.

Examples:

```text
Requests/sec
Orders/sec
Messages/sec
Transactions/sec
```

A performance improvement should ideally increase useful throughput or reduce latency.

---

# 6. CPU Utilization

High CPU can indicate:

```text
Expensive computation
Excessive serialization
Large JSON processing
Poor algorithms
Too many threads
Busy loops
GC pressure
```

But low CPU does not necessarily mean the application is healthy. It may be waiting on the database or external services.

---

# 7. Memory

Monitor:

```text
Heap
Non-heap
Container memory
Direct memory
```

High memory usage can result from:

```text
Large objects
Caches
Memory leaks
Large responses
Unbounded collections
```

---

# 8. JVM Heap

The JVM heap stores application objects.

If the heap is too small:

```text
Frequent GC
OutOfMemoryError
```

If it is too large:

```text
Longer GC cycles
Higher memory requirements
```

Tune based on measurements.

---

# 9. Garbage Collection

GC reclaims memory occupied by objects that are no longer reachable.

Excessive GC can cause:

```text
CPU increase
Latency spikes
Reduced throughput
```

Monitor GC activity before changing JVM settings.

---

# 10. Memory Leak Investigation

Symptoms:

```text
Heap steadily increases
GC becomes frequent
Application eventually crashes
```

Investigate:

```text
Heap dump
Object retention
Static collections
Unbounded caches
Listeners
ThreadLocal usage
```

---

# 11. Thread Pools

Spring applications use threads for:

```text
HTTP requests
Async processing
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

# 12. Thread Pool Exhaustion

Suppose:

```text
Thread pool = 100
```

and all 100 threads are blocked on slow database or HTTP calls.

New requests may:

```text
Wait
Queue
Timeout
Get rejected
```

This can make the entire application appear unavailable.

---

# 13. Don't Fix Thread Problems by Only Increasing Threads

Increasing:

```text
100 → 500 threads
```

may make the situation worse.

More threads can cause:

```text
Context switching
Memory overhead
Database overload
CPU contention
```

Find the underlying blocking operation first.

---

# 14. Database Connection Pool

Spring Boot commonly uses HikariCP.

Architecture:

```text
Spring Boot
     ↓
HikariCP
     ↓
Database
```

The pool reuses database connections.

---

# 15. Connection Pool Sizing

Suppose:

```text
10 application instances
20 DB connections each
```

Potential total:

```text
200 connections
```

The database must support that workload.

More connections do not automatically mean better performance.

---

# 16. Connection Pool Metrics

Monitor:

```text
Active
Idle
Pending
Maximum
Acquisition time
Timeouts
```

If requests wait for connections:

```text
Latency increases
```

---

# 17. Database Query Performance

Check:

```text
Slow queries
Indexes
Execution plans
N+1 queries
Large joins
Full table scans
Large result sets
```

Optimize the query before adding unnecessary infrastructure.

---

# 18. Indexes

Example:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Indexes improve reads but can increase:

```text
Storage
Write cost
Maintenance
```

Create indexes based on actual query patterns.

---

# 19. N+1 Queries

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

---

# 20. Pagination

Avoid:

```java
repository.findAll();
```

for large tables.

Use:

```text
Page
Size
Sort
```

For very large datasets, consider:

```text
Keyset pagination
Cursor pagination
```

---

# 21. Caching

Caching can reduce:

```text
Database load
Latency
Repeated computation
```

Typical flow:

```text
Request
 ↓
Redis
 |
 +-- hit → response
 |
 +-- miss
      ↓
   Database
      ↓
    Redis
      ↓
   response
```

Cache only data where the tradeoffs make sense.

---

# 22. Cache Hit Ratio

Formula:

```text
hits
-------------------
hits + misses
```

Example:

```text
900 hits
100 misses
```

Hit ratio:

```text
90%
```

Also measure actual:

```text
API latency
Database load
Cache latency
```

---

# 23. Cache Stampede

If a popular key expires:

```text
10,000 requests
      ↓
Cache miss
      ↓
10,000 DB requests
```

This can overload the database.

Mitigation:

```text
TTL jitter
Request coalescing
Distributed lock
Background refresh
Cache warming
```

---

# 24. HTTP Timeouts

Every external dependency should have sensible timeouts.

Examples:

```text
Connection timeout
Read timeout
Request timeout
```

Never allow:

```text
External service
      ↓
Wait forever
```

---

# 25. Retry

Retry only failures likely to be transient.

Good candidates:

```text
Temporary network failure
503
Transient connection failure
```

Usually not:

```text
400
401
403
Invalid business request
```

---

# 26. Retry With Backoff

Example:

```text
100 ms
200 ms
400 ms
800 ms
```

Add jitter where appropriate to avoid synchronized retry storms.

---

# 27. Retry and Idempotency

Before retrying a write operation:

```text
Can this operation safely execute twice?
```

For payments and order creation, idempotency keys are often important.

---

# 28. Circuit Breaker

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

A circuit breaker prevents repeatedly calling an unhealthy dependency.

---

# 29. Bulkhead

Separate resources for different workloads.

Example:

```text
Payment → Pool A
Recommendations → Pool B
```

If recommendations become slow, payment processing can retain capacity.

---

# 30. Rate Limiting

Protect APIs from excessive traffic.

Example:

```text
100 requests/minute/user
```

Excess traffic:

```text
429 Too Many Requests
```

---

# 31. Load Shedding

During extreme overload:

```text
Accept everything
 ↓
Queues grow
 ↓
Latency increases
 ↓
System collapses
```

Load shedding rejects lower-priority work to protect critical operations.

---

# 32. Graceful Degradation

If a non-critical dependency fails:

```text
Product details → available
Recommendations → fallback
```

The entire request does not always need to fail.

---

# 33. Health Checks

Spring Boot Actuator can expose health information.

Common endpoints:

```text
/actuator/health
/actuator/info
/actuator/metrics
```

Expose only appropriate endpoints in production.

---

# 34. Liveness

Liveness answers:

```text
Is this process alive?
```

A failed liveness check may cause the orchestrator to restart the application.

Avoid making liveness depend unnecessarily on every external dependency.

---

# 35. Readiness

Readiness answers:

```text
Should this instance receive traffic?
```

If the instance cannot safely serve requests:

```text
Not ready
```

The load balancer/orchestrator can stop routing traffic to it.

---

# 36. Graceful Shutdown

During shutdown:

```text
Stop accepting new traffic
        ↓
Finish in-flight requests
        ↓
Stop background work
        ↓
Close resources
        ↓
Exit
```

This reduces dropped requests during deployments.

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

Useful fields:

```text
timestamp
level
service
traceId
requestId
user/business identifier
event
```

---

# 38. What Not to Log

Never casually log:

```text
Passwords
JWT tokens
API keys
Credit card data
Private keys
Secrets
Sensitive personal data
```

Mask sensitive values where necessary.

---

# 39. Correlation ID

A correlation ID connects logs for one request.

Example:

```text
Gateway
  ↓ ABC123
Order Service
  ↓ ABC123
Payment Service
  ↓ ABC123
Inventory Service
```

All services log:

```text
correlationId=ABC123
```

---

# 40. Distributed Tracing

Tracing follows a request across services.

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

It helps identify where latency is introduced.

---

# 41. Metrics

Important production metrics:

```text
Request rate
Error rate
p95/p99 latency
CPU
Memory
GC
Thread count
DB connections
DB latency
Redis hit ratio
Kafka lag
External API latency
```

---

# 42. RED Metrics

For request-driven services:

```text
Rate
Errors
Duration
```

These provide a useful high-level view of API health.

---

# 43. USE Metrics

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

# 44. Spring Boot Actuator

Actuator provides operational features such as:

```text
Health
Metrics
Application information
Environment/configuration information
```

Secure sensitive actuator endpoints.

---

# 45. Production Configuration

Do not hardcode:

```text
Database URLs
Passwords
API keys
JWT secrets
Environment-specific settings
```

Use:

```text
Environment variables
External configuration
Secret managers
Kubernetes Secrets
```

---

# 46. Spring Profiles

Common environments:

```text
local
dev
test
staging
prod
```

Example:

```text
application.yml
application-dev.yml
application-prod.yml
```

Activate the appropriate profile externally.

---

# 47. Database Migrations

Use tools such as:

```text
Flyway
Liquibase
```

Example:

```text
V1__create_users.sql
V2__create_orders.sql
V3__add_order_status.sql
```

Migrations should be version-controlled.

---

# 48. Backward-Compatible Database Changes

During rolling deployments:

```text
Old application
New application
```

may run simultaneously.

Safer migration:

```text
Add new column
 ↓
Deploy compatible code
 ↓
Backfill
 ↓
Switch behavior
 ↓
Remove old column later
```

---

# 49. Transaction Duration

Long transactions can cause:

```text
Locks
Connection exhaustion
Deadlocks
Latency
Reduced throughput
```

Avoid slow external calls inside database transactions where possible.

---

# 50. API Error Handling

Use centralized exception handling:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    ...
}
```

Return consistent error responses.

Don't expose:

```text
Stack traces
SQL errors
Internal class names
```

---

# 51. HTTP Status Codes

Common statuses:

```text
200 → success
201 → created
204 → no content
400 → bad request
401 → unauthenticated
403 → forbidden
404 → not found
409 → conflict
429 → rate limited
500 → server error
503 → unavailable
```

---

# 52. Security Baseline

Production applications should consider:

```text
HTTPS
Authentication
Authorization
Input validation
CORS
CSRF where applicable
Rate limiting
Secrets management
Dependency scanning
Least privilege
Audit logging
```

---

# 53. Dependency Security

Regularly check dependencies for known vulnerabilities.

Consider:

```text
Dependency scanning
Security advisories
SCA
SBOM
Container image scanning
```

---

# 54. Docker

Typical flow:

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

Use a maintained, minimal base image and avoid unnecessary privileges.

---

# 55. Kubernetes

Common Kubernetes components:

```text
Deployment
Service
Ingress
ConfigMap
Secret
HorizontalPodAutoscaler
```

Application:

```text
Pod
 ↓
Spring Boot
```

---

# 56. Resource Requests and Limits

Configure appropriate:

```text
CPU request
CPU limit
Memory request
Memory limit
```

Incorrect memory limits can cause:

```text
OOMKilled
```

---

# 57. Horizontal Scaling

A stateless Spring Boot service can often scale horizontally:

```text
Instance 1
Instance 2
Instance 3
Instance 4
```

behind a load balancer.

But verify that:

```text
Database
Redis
Kafka
External APIs
```

can handle the increased load.

---

# 58. Autoscaling

Autoscaling can use:

```text
CPU
Memory
Request rate
Queue depth
Custom metrics
```

Scaling the application tier alone does not fix a database bottleneck.

---

# 59. Deployment Strategies

Common strategies:

```text
Rolling
Blue-Green
Canary
```

---

# 60. Rolling Deployment

Instances are replaced gradually:

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

Requires backward-compatible application and database changes.

---

# 61. Blue-Green Deployment

Two environments:

```text
Blue → current
Green → new
```

After validation:

```text
Traffic → Green
```

Rollback can switch traffic back to Blue.

---

# 62. Canary Deployment

Start with a small percentage:

```text
95% old
5% new
```

Monitor:

```text
Error rate
Latency
CPU
Memory
Business metrics
```

Increase gradually if healthy.

---

# 63. Rollback

A deployment should have a clear rollback strategy.

Examples:

```text
Previous container image
Previous application version
Traffic switch
Feature flag
```

Database migrations must also be considered.

---

# 64. CI/CD

Typical pipeline:

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

# 65. Testing Pyramid

```text
       E2E
      /   \
 Integration
    /       \
 Unit Tests
```

Unit tests should generally be fast.

Integration tests verify real component interactions.

E2E tests should focus on critical workflows.

---

# 66. Integration Testing

Test interactions with:

```text
Spring context
Database
Redis
Kafka
HTTP services
```

Testcontainers can provide realistic infrastructure for integration tests.

---

# 67. Smoke Testing

After deployment:

```text
Application starts
Health works
Login works
Critical read works
Critical write works
Database works
```

Keep smoke tests fast.

---

# 68. Production Monitoring During Deployment

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
External API failures
```

A deployment should be monitored, not just executed.

---

# 69. Incident Response

Typical flow:

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

First restore service, then investigate deeply.

---

# 70. Root Cause Analysis

Ask:

```text
What happened?
Why?
Why wasn't it detected?
Why wasn't it prevented?
What should change?
```

Create preventive actions.

---

# 71. Runbooks

A runbook documents operational procedures.

Example:

```text
Kafka lag increasing
```

Steps:

```text
Check consumer health
Check consumer errors
Check DB latency
Check partition distribution
Check recent deployments
Scale if appropriate
Rollback if required
```

---

# 72. RPO

Recovery Point Objective means:

```text
Maximum acceptable data loss
```

Example:

```text
RPO = 5 minutes
```

---

# 73. RTO

Recovery Time Objective means:

```text
Maximum acceptable recovery time
```

Example:

```text
RTO = 30 minutes
```

---

# 74. Backups

A production backup strategy should define:

```text
Frequency
Retention
Encryption
Storage
RPO
RTO
Restore process
Restore testing
```

A backup is useful only if it can actually be restored.

---

# 75. Disaster Recovery

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

---

# 76. Performance Investigation Example

Problem:

```text
API p99 increased from 300ms → 2s
```

Investigation:

```text
Metrics
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
Monitor again
```

---

# 77. Memory Incident

Symptoms:

```text
Heap ↑
GC ↑
Latency ↑
```

Investigate:

```text
Heap dump
Object retention
Caches
Large collections
Recent deployment
```

Don't immediately increase heap without finding the cause.

---

# 78. Thread Pool Incident

Symptoms:

```text
Requests waiting
Latency ↑
Rejected tasks
```

Investigate:

```text
Slow external API
Blocked threads
DB connection pool
Executor configuration
Long-running tasks
```

---

# 79. Database Incident

If the database becomes unavailable:

```text
Detect
 ↓
Failover if available
 ↓
Protect retries
 ↓
Reduce non-critical traffic
 ↓
Restore
 ↓
Validate
```

Avoid uncontrolled retry storms.

---

# 80. Redis Incident

If Redis goes down:

```text
Redis unavailable
 ↓
Short timeout
 ↓
Controlled fallback
 ↓
Protect database
 ↓
Restore Redis
 ↓
Monitor recovery
```

---

# 81. Kafka Incident

If Kafka is unavailable:

```text
Determine whether request can continue
 ↓
Use retry/DLQ where appropriate
 ↓
Protect downstream systems
 ↓
Restore Kafka
 ↓
Process backlog safely
```

For important database-backed events, consider the Outbox Pattern.

---

# 82. External API Failure

For payment or shipping services:

```text
Timeout
 ↓
Retry transient failures
 ↓
Circuit breaker
 ↓
Fallback / pending state
```

Avoid holding database transactions open while waiting on slow external services.

---

# 83. Performance Anti-Patterns

Avoid:

```text
Unbounded queries
Unbounded caches
Unlimited threads
Missing timeouts
Blind retries
Excessive logging
N+1 queries
Huge transactions
Premature optimization
```

---

# 84. Production Checklist

```text
□ Externalized configuration
□ Secrets protected
□ Database pool configured
□ Queries optimized
□ Pagination
□ Cache strategy
□ Timeouts
□ Retry strategy
□ Circuit breaker
□ Rate limiting
□ Health checks
□ Graceful shutdown
□ Structured logs
□ Metrics
□ Tracing
□ Security
□ Tests
□ CI/CD
□ Rollback
□ Backups
□ Disaster recovery
□ Alerts
□ Runbooks
```

---

# 85. Interview: How Do You Optimize a Slow Spring Boot API?

> I start with measurements instead of guessing. I check p95 and p99 latency, traces, database queries, connection pools, external calls, CPU, GC, and thread pools. Once I identify the actual bottleneck, I optimize that area and compare the metrics before and after the change.

---

# 86. Interview: What Would You Monitor in Production?

> I monitor request rate, 4xx and 5xx errors, p95/p99 latency, JVM memory and GC, thread pools, database connection pools and latency, Redis metrics, Kafka lag, and external dependency latency. I also configure alerts for sustained errors, saturation, and availability problems.

---

# 87. Interview: Liveness vs Readiness?

> Liveness tells us whether the process is alive and potentially needs restarting. Readiness tells us whether the instance should receive traffic. I avoid making liveness depend on every external dependency because a temporary dependency failure shouldn't necessarily restart the application.

---

# 88. Interview: How Do You Handle a Slow External Service?

> I configure explicit timeouts, retry only transient failures with bounded backoff, and use a circuit breaker or bulkhead where appropriate. For non-critical dependencies, I also use graceful degradation so one failure doesn't take down the complete request.

---

# 89. Interview: How Do You Deploy Without Downtime?

> I run multiple instances behind a load balancer, use readiness checks and graceful shutdown, keep database and API changes backward compatible, and use rolling, blue-green, or canary deployment depending on the risk.

---

# 90. Interview: How Do You Handle Production Incidents?

> First I focus on mitigation and restoring service. I check recent deployments, metrics, logs, and traces to identify the failing component. Once the system is stable, I investigate root cause and add preventive actions such as code fixes, tests, alerts, or capacity changes.

---

# 91. Interview: How Do You Scale Spring Boot?

> I first identify the bottleneck. Stateless application instances can usually be scaled horizontally, but I also check database capacity, connection pools, Redis, Kafka, and downstream services. Adding application instances doesn't help if the real bottleneck is the database.

---

# 92. Interview: How Do You Prevent Database Overload?

> I use appropriate indexes and pagination, optimize expensive queries, cache suitable read-heavy data, configure connection pools correctly, protect against retry storms, and use rate limiting or load shedding when necessary.

---

# 93. Interview: What Is p99 Latency?

> p99 means 99 percent of requests complete at or below that latency, while the slowest one percent are above it. It is useful for identifying tail-latency problems that an average can hide.

---

# 94. Interview: RPO vs RTO?

> RPO defines how much data loss is acceptable after a failure. RTO defines how quickly the system must be restored. Both influence backup, replication, and disaster-recovery design.

---

# 95. Final Mental Model

```text
                 PRODUCTION SPRING BOOT
                         |
        +----------------+----------------+
        |                |                |
     Performance      Reliability       Security
        |                |                |
   DB / JVM / Cache   Timeout / Retry   Auth / TLS
        |            Circuit Breaker       |
        |              Bulkhead          Secrets
        +----------------+----------------+
                         |
                    Observability
                         |
              Logs + Metrics + Traces
                         |
                     Operations
                         |
          Deploy + Monitor + Recover
```

---

# 96. Final Rule

> **Don't optimize what you haven't measured, don't deploy what you can't observe, and don't depend on a component without planning for its failure. A production-ready Spring Boot application is designed not only for the happy path, but also for traffic spikes, slow dependencies, failed deployments, database problems, and recovery.**

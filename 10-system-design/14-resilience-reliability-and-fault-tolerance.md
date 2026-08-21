# System Design — File 14: Resilience, Reliability & Fault Tolerance

A production backend must remain useful when servers, networks, databases, or dependencies fail. Resilience is about isolating failures, recovering safely, and protecting critical functionality.

## 1. Reliability vs Availability

**Availability** = whether the system is operational and serving requests.

**Reliability** = whether the system continues to perform correctly over time.

A system can be available but still unreliable if it returns incorrect results.

---

## 2. Fault Tolerance

Fault tolerance means continuing to operate despite component failures.

```text
             Load Balancer
              /         \\
             v           v
          Server 1    Server 2
             X
          failed
```

Traffic continues through healthy instances.

---

## 3. Failure Domains

A failure domain is infrastructure that can fail together.

Examples:

```text
Instance
Rack
Availability Zone
Region
```

Spread critical replicas across independent failure domains.

---

## 4. Single Point of Failure

A single point of failure is a component whose failure can bring down the system.

Ask during design:

> "What happens if this component disappears?"

Typical protections:

```text
Multiple API instances
Multiple database replicas
Multiple AZs
Multiple regions
```

---

## 5. SLI, SLO and SLA

### SLI

What we measure:

```text
Latency
Error rate
Availability
Queue delay
```

### SLO

What we target:

```text
99.9% successful requests
p95 latency < 300 ms
```

### SLA

What we contractually promise customers.

Easy interview summary:

```text
SLI -> Measure
SLO -> Target
SLA -> Promise
```

---

## 6. Error Budget

If:

```text
SLO = 99.9%
```

then the allowed failure/unavailability is:

```text
0.1%
```

This is the error budget.

If the budget is repeatedly exhausted, teams should prioritize reliability work over risky releases.

---

## 7. Timeouts

Never wait indefinitely for a remote dependency.

```text
Service A
   |
   v
Service B
   |
 timeout
   |
   v
Fallback / Error
```

Timeouts protect threads, connections and memory.

---

## 8. Timeout Budget

Suppose:

```text
Client timeout = 2 seconds
```

Do not give every downstream call a 2-second timeout. Every dependency should fit inside the overall request deadline.

---

## 9. Retries

Retries can recover from transient failures such as:

```text
Temporary network error
Timeout
Short-lived overload
Leader transition
```

Use:

```text
Maximum attempts
Backoff
Jitter
Overall deadline
```

---

## 10. Exponential Backoff

Instead of retrying immediately:

```text
100 ms
200 ms
400 ms
800 ms
```

The delay increases after each failure.

---

## 11. Retry Jitter

Without jitter:

```text
10,000 clients
    |
    v
Retry at exactly the same time
```

With jitter:

```text
Base delay + random variation
```

Retries are spread over time.

---

## 12. Retry Only Safe Operations

Be careful retrying side-effecting operations.

Potentially safe:

```text
GET
```

Potentially dangerous:

```text
POST /payments
POST /orders
```

For important non-idempotent operations, use idempotency keys where appropriate.

---

## 13. Retry Storm

```text
Dependency fails
      |
      v
Clients retry
      |
      v
More traffic
      |
      v
Dependency becomes more overloaded
```

Use:

```text
Backoff
Jitter
Retry limits
Circuit breakers
Rate limiting
```

---

## 14. Circuit Breaker

A circuit breaker prevents repeated calls to an unhealthy dependency.

Typical states:

```text
CLOSED
   |
failures
   v
OPEN
   |
cooldown
   v
HALF-OPEN
```

### Closed

Requests flow normally.

### Open

Requests fail fast without calling the dependency.

### Half-Open

A small number of test requests determine whether the dependency recovered.

---

## 15. Circuit Breaker Benefit

Without one:

```text
Slow dependency
      |
      v
Threads wait
      |
      v
Connection pools fill
      |
      v
Application becomes unhealthy
```

With one:

```text
Dependency unhealthy
      |
      v
Circuit opens
      |
      v
Fast failure / fallback
```

A circuit breaker protects your service; it does not fix the dependency.

---

## 16. Bulkhead Pattern

Separate resources for different workloads.

```text
Payment -> Pool A
Search  -> Pool B
```

If Search becomes overloaded, Payment can still use its dedicated pool.

This prevents one workload from consuming all resources.

---

## 17. Cascading Failure

Example:

```text
API
 |
 v
Recommendation Service
 |
 v
Slow Database
```

The database becomes slow, recommendation threads wait, API threads wait, and the API becomes overloaded.

Protections:

```text
Timeouts
Circuit breakers
Bulkheads
Rate limits
Fallbacks
```

---

## 18. Load Shedding

When overloaded, reject or degrade lower-priority work.

```text
Keep:
Payment
Checkout
Order creation

Degrade:
Recommendations
Analytics
Personalization
```

Protect critical business operations first.

---

## 19. Graceful Degradation

Instead of:

```text
Product page -> 500
```

allow:

```text
Product page
+
Recommendations unavailable
```

Other examples:

```text
Serve stale cache
Use default values
Reduce search quality
Disable expensive features
```

---

## 20. Fallbacks

A fallback provides an alternative when a dependency fails.

Example:

```text
Recommendation Service
        X
        |
        v
Popular products
```

Good fallbacks are:

```text
Simple
Fast
Reliable
Non-cascading
```

---

## 21. Health Checks

Two important concepts:

### Liveness

```text
Is the process alive?
```

### Readiness

```text
Can the instance safely receive traffic?
```

An instance can be alive but not ready.

---

## 22. Avoid Bad Health Checks

Avoid making health checks depend on every external dependency:

```text
/health
  |
  +--> DB
  +--> Redis
  +--> Kafka
  +--> External API
```

If one dependency temporarily fails, every application instance could become unhealthy and create another outage.

---

## 23. Graceful Shutdown

Safe shutdown:

```text
Stop new traffic
      |
      v
Drain active requests
      |
      v
Stop background work
      |
      v
Close connections
      |
      v
Terminate
```

Useful during deployments, autoscaling, maintenance and node replacement.

---

## 24. Connection Draining

A load balancer should stop sending new requests to an instance being removed.

```text
New requests
     |
     +----> Healthy instances

Existing requests
     |
     +----> Draining instance
```

This reduces dropped requests.

---

## 25. Zero-Downtime Deployment

Typical process:

```text
Old version
    |
    v
Start new version
    |
    v
Readiness passes
    |
    v
Send traffic to new version
    |
    v
Drain old version
    |
    v
Terminate old version
```

---

## 26. Backward-Compatible Changes

During rolling deployments, v1 and v2 may run simultaneously.

Ensure compatibility for:

```text
Database schema
REST APIs
Events
Messages
```

---

## 27. Expand-and-Contract Database Migration

Instead of immediately removing a column:

```text
1. Add new column
2. Deploy code writing both
3. Backfill data
4. Deploy code reading new column
5. Stop writing old column
6. Remove old column later
```

This reduces deployment risk.

---

## 28. Dead Letter Queue

For message processing:

```text
Queue
  |
  v
Consumer
  |
  X repeated failure
  |
  v
DLQ
```

A DLQ stores messages that cannot be processed after the configured retry policy.

Useful for:

```text
Poison messages
Bad payloads
Permanent processing errors
```

---

## 29. Poison Message

A poison message repeatedly fails:

```text
Message
  |
Consumer -> failure
  |
Retry -> failure
  |
Retry -> failure
```

Use:

```text
Bounded retries
Backoff
DLQ
Error classification
```

---

## 30. Transient vs Permanent Errors

### Transient

```text
Timeout
Temporary network error
Temporary overload
```

Potentially retry.

### Permanent

```text
Invalid payload
Unsupported schema
Business rule violation
```

Usually do not retry forever.

---

## 31. Disaster Recovery

A disaster recovery plan answers:

> What happens if a major failure occurs?

Examples:

```text
Region failure
Database corruption
Accidental deletion
Infrastructure outage
```

---

## 32. Backups

Important backup properties:

```text
Frequency
Retention
Encryption
Geographic separation
Restore testing
```

A backup that has never been restored is not fully trusted.

---

## 33. Restore Testing

Test the complete path:

```text
Backup
  |
  v
Restore
  |
  v
Application validation
```

This can reveal corrupt backups, missing dependencies, wrong procedures and unexpected restore times.

---

## 34. RTO vs RPO

### RTO

Recovery Time Objective:

> How quickly must the system recover?

### RPO

Recovery Point Objective:

> How much data loss, measured in time, is acceptable?

Example:

```text
RTO = 30 minutes
RPO = 5 minutes
```

---

## 35. Active-Active Disaster Recovery

```text
Global Routing
   /       \\
Region A  Region B
Active    Active
```

Advantages:

```text
Fast failover
Traffic sharing
Low regional latency
```

Challenges:

```text
Cost
Data consistency
Operational complexity
```

---

## 36. Active-Passive Disaster Recovery

```text
Region A -> Active
Region B -> Standby
```

If Region A fails:

```text
Traffic -> Region B
```

Simpler than active-active but usually has slower failover and underutilized standby resources.

---

## 37. Chaos Engineering

Deliberately introduce controlled failures:

```text
Kill an instance
Introduce latency
Stop a dependency
Fill disk
Increase traffic
```

Goal:

> Find weaknesses before real incidents expose them.

---

## 38. Game Days

A planned failure exercise.

Example:

```text
"Primary database is unavailable."
```

The team practices:

```text
Detection
Communication
Mitigation
Failover
Recovery
Validation
```

---

## 39. Observability

Three major pillars:

```text
Logs
Metrics
Traces
```

### Logs
Detailed events.

### Metrics
Numerical measurements.

### Traces
Request flow across services.

---

## 40. Golden Signals

Common signals:

```text
Latency
Traffic
Errors
Saturation
```

These help identify user-impacting problems.

---

## 41. Saturation

Saturation shows how close a resource is to capacity.

Examples:

```text
CPU = 90%
DB connections = 95%
Queue depth = growing
Disk = 90%
```

High saturation can be an early warning.

---

## 42. Alerting

Good alerts are:

```text
Actionable
Relevant
Based on user impact
```

Prefer alerts such as:

```text
Error rate > threshold
p95 latency > SLO
Queue age > threshold
Connection pool exhausted
```

over alerts that don't indicate actual user impact.

---

## 43. Incident Response

Typical lifecycle:

```text
Detect
  |
Triage
  |
Mitigate
  |
Recover
  |
Root cause analysis
  |
Prevent recurrence
```

During an active incident:

> Restore service safely first; investigate deeply after the immediate impact is controlled.

---

## 44. Runbooks

A runbook documents known operational procedures.

Example:

```text
Database replica lag

1. Check replication status
2. Check DB load
3. Check network
4. Reduce traffic if necessary
5. Fail over if required
6. Validate recovery
```

Runbooks reduce response time.

---

## 45. Rate Limiting

Rate limiting protects systems from excessive traffic.

Example:

```text
100 requests/minute/user
```

Common algorithms:

```text
Fixed Window
Sliding Window
Token Bucket
Leaky Bucket
```

---

## 46. Token Bucket

Conceptually:

```text
Bucket
[● ● ● ● ●]
```

Tokens are added at a fixed rate. A request consumes a token.

If no token is available:

```text
Reject
or
Wait
```

Token bucket can allow controlled bursts.

---

## 47. Load Shedding vs Rate Limiting

### Rate Limiting

Controls traffic before the system becomes overwhelmed.

### Load Shedding

Rejects or degrades work when the system is already under pressure.

Both protect availability.

---

## 48. Dependency Budget

Every synchronous dependency adds:

```text
Latency
Failure probability
Operational complexity
```

Example:

```text
API
 |
 +--> User
 +--> Inventory
 +--> Pricing
 +--> Recommendation
 +--> Payment
```

Minimize unnecessary synchronous dependencies.

---

## 49. Synchronous vs Asynchronous Resilience

Synchronous:

```text
API -> Service B -> Service C
```

If C is slow, API may become slow.

Asynchronous:

```text
API -> Queue -> Worker
```

The API can finish without waiting for background processing.

Use asynchronous processing when the business workflow allows it.

---

## 50. Resilience Checklist

```text
□ Remove single points of failure
□ Use independent failure domains
□ Configure timeouts
□ Bound retries
□ Use exponential backoff
□ Add jitter
□ Use idempotency
□ Use circuit breakers where appropriate
□ Use bulkheads
□ Rate-limit traffic
□ Shed low-priority load
□ Gracefully degrade
□ Configure correct health checks
□ Drain connections during shutdown
□ Use DLQs for failed messages
□ Test backups and restores
□ Define RTO/RPO
□ Monitor latency, errors and saturation
□ Maintain runbooks
□ Practice disaster recovery
```

---

## 51. Interview Question — What Is a Circuit Breaker?

> "A circuit breaker prevents repeated calls to an unhealthy dependency. It normally stays closed, opens after failures cross a threshold and later enters half-open to test whether the dependency has recovered."

---

## 52. Interview Question — Retry vs Circuit Breaker?

> "Retries are useful for short-lived transient failures. A circuit breaker prevents us from repeatedly calling a dependency that is consistently failing. They work well together with bounded retries, backoff and jitter."

---

## 53. Interview Question — What Is a Bulkhead?

> "A bulkhead isolates resources for different workloads so one overloaded dependency cannot consume all threads, connections or capacity and take down unrelated functionality."

---

## 54. Interview Question — What Is Graceful Degradation?

> "Graceful degradation means keeping critical functionality available while temporarily reducing or disabling non-critical features when dependencies fail or the system is overloaded."

---

## 55. Interview Question — RTO vs RPO?

> "RTO is how quickly the system needs to recover. RPO is how much data loss, measured in time, the business can tolerate."

---

## 56. Interview Question — How Would You Design for High Availability?

> "I'd remove single points of failure, run services across multiple failure domains, use health-aware load balancing, database replication and automated failover where appropriate. I'd also use graceful deployments, monitoring and a tested disaster-recovery plan."

---

## 57. Interview Question — How Do You Prevent Cascading Failures?

> "I'd use timeouts, bounded retries with exponential backoff and jitter, circuit breakers, bulkheads, rate limiting and graceful degradation. I'd also minimize synchronous dependencies and monitor downstream latency and saturation."

---

## 58. Practical Scenario — Payment Service Is Down

A safer design:

```text
Order Service
    |
    v
Payment Service
    X
    |
Timeout
    |
    v
Order = PAYMENT_PENDING
```

Then process or retry asynchronously when appropriate.

For payment operations, idempotency and durable transaction records are especially important.

---

## 59. Practical Scenario — Recommendation Service Is Slow

Use:

```text
Short timeout
Circuit breaker
Cached recommendations
Fallback to popular products
```

Do not allow recommendations to consume all API resources.

---

## 60. Practical Scenario — One Availability Zone Goes Down

```text
Load Balancer
 /          \\
AZ-A        AZ-B
 API         API
```

If AZ-A fails:

```text
Traffic -> AZ-B
```

The remaining zone must have enough capacity for the required workload.

---

## 61. Practical Scenario — Region Goes Down

A multi-region design must consider more than traffic routing:

```text
Database state
Cache state
Message queues
Secrets/config
External dependencies
Capacity
DNS/global routing
```

---

## 62. One-Minute Interview Answer

### "How would you make a Spring Boot microservice resilient?"

> "I'd start with timeouts for every remote call and use bounded retries with exponential backoff and jitter only for transient failures. I'd add circuit breakers and bulkheads so an unhealthy dependency can't exhaust application resources. I'd keep the service stateless, use health checks and graceful shutdown, and provide fallbacks or graceful degradation for non-critical dependencies. For asynchronous processing I'd use retries and a DLQ. Finally, I'd monitor latency, errors and saturation and define clear SLO, RTO and RPO requirements."

---

## 63. Key Takeaway

> **Resilience is not about preventing every failure. It is about isolating failures, detecting them quickly, recovering safely, protecting critical functionality and preventing one failure from becoming a cascading outage.**

**File 14 complete.**

# Microservices — Resilience Patterns & Fault Tolerance

This file covers how microservices continue operating when dependencies fail.

Core topics:

```text
Resilience
Fault Tolerance
Timeouts
Retries
Exponential Backoff
Jitter
Circuit Breaker
Bulkhead
Rate Limiting
Load Shedding
Fallback
Graceful Degradation
Health Checks
Backpressure
Idempotency
Failure Isolation
Cascading Failure
Resilience4j
Spring Boot Integration
Production Scenarios
Interview Questions
```

---

# 1. What Is Resilience?

Resilience is the ability of a system to continue providing acceptable behavior when components fail or become unreliable.

Example:

```text
Payment Service
      ↓
     DOWN
```

A resilient checkout system should not necessarily cause:

```text
Gateway
 ↓
Order Service
 ↓
Entire platform
 ↓
DOWN
```

Instead, failure should be isolated.

---

# 2. What Is Fault Tolerance?

Fault tolerance means the system can continue operating despite certain faults.

Examples:

```text
One service instance fails
One dependency times out
One Kafka consumer crashes
One Redis node becomes unavailable
```

The exact tolerance level depends on system requirements.

---

# 3. Why Resilience Matters in Microservices

Microservices create many network boundaries:

```text
Order
 ↓
Inventory
 ↓
Payment
 ↓
Notification
```

Every network call can fail because of:

```text
Timeout
Network failure
Service crash
Overload
DNS issue
Database failure
Third-party outage
```

Therefore:

> Network calls should be treated as failure-prone operations.

---

# 4. Failure Is Normal

Don't design assuming:

```text
Service always responds
```

Design for:

```text
Service responds
Service responds slowly
Service returns an error
Service is unavailable
Network request times out
```

---

# 5. Timeout

A timeout limits how long a caller waits.

Example:

```text
Order Service
 ↓
Payment Service
 ↓
Timeout = 3 seconds
```

If payment doesn't respond:

```text
Fail
```

rather than waiting forever.

---

# 6. Why Timeouts Matter

Without timeouts:

```text
Payment hangs
 ↓
Order threads wait
 ↓
More requests arrive
 ↓
Threads exhausted
 ↓
Order Service becomes unavailable
```

One slow dependency can therefore cause cascading failure.

---

# 7. Types of Timeout

Common categories:

```text
Connection timeout
Read timeout
Write timeout
Overall request timeout
```

Exact timeout controls depend on the HTTP client and infrastructure.

---

# 8. Connection Timeout

How long to wait to establish a connection.

Example:

```text
Connect to Payment Service
 ↓
No connection
 ↓
Connection timeout
```

---

# 9. Read Timeout

Connection exists, but the response doesn't arrive within the configured period.

```text
Connected
 ↓
Waiting for response
 ↓
Read timeout
```

---

# 10. Timeout Budget

If an API has:

```text
Overall budget = 2 seconds
```

don't configure every downstream call independently to:

```text
5 seconds
```

The downstream timeout strategy should fit within the overall latency budget.

---

# 11. Retry

A retry attempts a failed operation again.

Example:

```text
Request
 ↓
Timeout
 ↓
Retry
 ↓
Success
```

Retries can help with transient failures.

---

# 12. What Is a Transient Failure?

A transient failure may disappear shortly.

Examples:

```text
Temporary network issue
Short service restart
Temporary connection refusal
Transient overload
```

Not every failure is transient.

---

# 13. What Should Not Be Retried Blindly?

Avoid blindly retrying:

```text
Invalid request
Authentication failure
Authorization failure
Business validation error
Permanent not-found condition
Non-idempotent operation without protection
```

---

# 14. Retry Storm

Suppose:

```text
1,000 requests
```

each retries 3 times.

Potential attempts:

```text
1,000 × 4 = 4,000
```

If the downstream service is already failing, retries can make the outage worse.

---

# 15. Exponential Backoff

Instead of immediately retrying:

```text
Retry 1 → 100ms
Retry 2 → 200ms
Retry 3 → 400ms
Retry 4 → 800ms
```

This gives the dependency time to recover.

Exact values should be tuned to the system.

---

# 16. Jitter

Jitter adds randomness to retry delays.

Without jitter:

```text
1000 clients
 ↓
all retry at 1 second
```

With jitter:

```text
Client A → 0.9 sec
Client B → 1.1 sec
Client C → 1.3 sec
...
```

Retries are spread out.

---

# 17. Retry Budget

Don't retry forever.

Example:

```text
Maximum attempts = 2
```

or:

```text
Maximum elapsed retry time = 1 second
```

The correct budget depends on the API's latency and business requirements.

---

# 18. Retry + Timeout

These work together.

Example:

```text
Overall request budget = 3 sec

Attempt 1
 ↓
Timeout
 ↓
Backoff
 ↓
Attempt 2
 ↓
Success
```

The retry policy must respect the overall request budget.

---

# 19. Idempotency

An operation is idempotent if repeating it produces the same intended business result.

Examples:

```text
PUT /users/101
```

with the same desired representation is generally designed to be idempotent.

But:

```text
POST /payments
```

may not be safe to retry blindly.

---

# 20. Idempotency Key

For operations such as payment:

```http
Idempotency-Key: checkout-123
```

The server can remember the result associated with that key.

If the same request is retried:

```text
Same key
 ↓
Return existing result
```

instead of performing the operation again.

---

# 21. Retryable HTTP Methods

Don't assume HTTP method alone guarantees business-level retry safety.

Commonly:

```text
GET
PUT
DELETE
```

are designed to be idempotent according to HTTP semantics, while:

```text
POST
```

is not inherently idempotent.

But implementation matters.

---

# 22. Circuit Breaker

A circuit breaker prevents continuous calls to an unhealthy dependency.

States:

```text
CLOSED
OPEN
HALF_OPEN
```

---

# 23. CLOSED

Normal operation:

```text
Service A
 ↓
Service B
```

Failures are monitored.

---

# 24. OPEN

Failure threshold is reached:

```text
Service B failing
 ↓
Circuit OPEN
```

New calls fail fast instead of reaching Service B.

---

# 25. HALF_OPEN

After a waiting period:

```text
OPEN
 ↓
HALF_OPEN
```

A limited number of test calls are allowed.

If healthy:

```text
CLOSED
```

If unhealthy:

```text
OPEN
```

---

# 26. Circuit Breaker Example

```text
Order Service
      |
      ↓
Payment Service

Payment starts failing
      |
      ↓
Circuit opens
      |
      ↓
Order fails fast
```

This protects both services.

---

# 27. Circuit Breaker Threshold

A circuit may open based on metrics such as:

```text
Failure rate
Slow-call rate
Number of calls
Sliding window
```

Don't simply say:

> "After one failure the circuit opens."

Real implementations are configurable.

---

# 28. Slow Calls

A dependency doesn't have to return errors to be dangerous.

Example:

```text
Payment response = 20 seconds
```

Even if it returns HTTP 200, it can consume resources and cause cascading latency.

Circuit breakers can be configured to consider slow calls.

---

# 29. Bulkhead Pattern

Bulkhead isolates resources so one dependency cannot consume everything.

Example:

```text
Order Service

Payment calls
→ Thread/connection pool A

Inventory calls
→ Pool B
```

If Payment becomes slow:

```text
Pool A exhausted
```

Inventory can still operate.

---

# 30. Why "Bulkhead"?

The name comes from ship compartments.

If one compartment floods:

```text
Other compartments remain protected.
```

The same principle applies to application resources.

---

# 31. Bulkhead Types

Possible isolation dimensions:

```text
Thread pools
Semaphore limits
Connection pools
Queues
Process instances
Containers
```

---

# 32. Semaphore Bulkhead

Limit concurrent calls:

```text
Payment max concurrent = 20
```

Request 21:

```text
Rejected / queued
```

This prevents unlimited concurrency.

---

# 33. Thread Pool Bulkhead

Separate execution resources:

```text
Payment pool
Inventory pool
Notification pool
```

One dependency cannot consume the entire shared pool.

---

# 34. Rate Limiting

Rate limiting controls how much traffic is accepted.

Example:

```text
100 requests/sec/client
```

Excess:

```text
HTTP 429
```

---

# 35. Rate Limiting vs Bulkhead

Rate limiting:

```text
Controls incoming traffic rate
```

Bulkhead:

```text
Controls resource/concurrency isolation
```

They solve related but different problems.

---

# 36. Load Shedding

Load shedding deliberately rejects some work when the system is overloaded.

Example:

```text
System overloaded
 ↓
Reject low-priority requests
 ↓
Protect critical checkout operations
```

This is better than allowing everything to fail slowly.

---

# 37. Graceful Degradation

When a dependency fails, return a reduced but useful experience.

Example:

```text
Recommendation Service DOWN
```

Product page can still show:

```text
Product details
Price
Availability
```

without:

```text
Recommendations
```

---

# 38. Fallback

A fallback provides an alternative response when a dependency fails.

Example:

```text
Recommendation Service
 ↓
failure
 ↓
Return empty recommendations
```

Fallbacks should be business-appropriate.

---

# 39. Bad Fallback

Don't hide serious failures.

Example:

```text
Payment Service DOWN
 ↓
Pretend payment succeeded
```

This is dangerous.

A fallback must preserve business correctness.

---

# 40. Graceful Degradation Example

For an e-commerce site:

```text
Search Service DOWN
 ↓
Use cached popular products
```

Potentially acceptable.

But:

```text
Payment Service DOWN
 ↓
Pretend payment succeeded
```

not acceptable.

---

# 41. Backpressure

Backpressure means slowing or limiting producers when consumers cannot keep up.

Example:

```text
Producer
 ↓
Queue
 ↓
Consumer
```

Consumer is slow.

Without backpressure:

```text
Queue grows forever
 ↓
Memory exhaustion
```

With backpressure:

```text
Producer slows
or
Requests are rejected
```

---

# 42. Queue-Based Backpressure

Example:

```text
API
 ↓
Queue
 ↓
Worker
```

If workers are overloaded:

```text
Queue length ↑
```

You can:

```text
Limit queue size
Scale workers
Slow producers
Reject excess work
```

---

# 43. Kafka Backpressure

Kafka consumers control their consumption pace.

If producers generate faster than consumers process:

```text
Consumer lag ↑
```

Possible actions:

```text
Scale consumers
Optimize processing
Increase partitions where appropriate
Fix downstream bottleneck
```

Don't blindly increase consumers if the real bottleneck is the database.

---

# 44. Connection Pool Protection

A dependency can exhaust connection pools.

Example:

```text
Hikari maxPoolSize = 20
```

If 20 connections are blocked:

```text
New requests wait
```

Use:

```text
Timeouts
Pool sizing
Bulkheads
Query optimization
```

---

# 45. Thread Pool Protection

If all worker threads are waiting on a slow dependency:

```text
Thread pool exhausted
```

The service can stop handling unrelated requests.

Use:

```text
Timeouts
Bulkheads
Async processing where appropriate
Resource limits
```

---

# 46. Cascading Failure

Example:

```text
Payment Service slows
 ↓
Order waits
 ↓
Order threads exhausted
 ↓
Gateway waits
 ↓
Gateway resources exhausted
 ↓
Entire API becomes unavailable
```

This is a cascading failure.

---

# 47. Preventing Cascading Failure

Use:

```text
Timeouts
Circuit breakers
Bulkheads
Rate limiting
Load shedding
Backpressure
Bounded retries
Caching
Autoscaling
```

No single pattern solves every failure.

---

# 48. Fail Fast

Fail fast means don't wait unnecessarily when success is unlikely.

Example:

```text
Circuit OPEN
 ↓
Immediately return failure
```

instead of:

```text
Wait 30 seconds
 ↓
Timeout
```

---

# 49. Fail Fast vs Fail Silent

Fail fast:

```text
Clearly report failure
```

Fail silently:

```text
Hide the failure
```

Fail silently is dangerous for critical business operations.

---

# 50. Resilience and User Experience

A resilient API should provide meaningful responses.

Example:

```http
503 Service Unavailable
```

or:

```json
{
  "status": "TEMPORARILY_UNAVAILABLE",
  "message": "Payment service is temporarily unavailable."
}
```

Don't expose internal stack traces.

---

# 51. HTTP Status Codes

Useful examples:

```text
429 → Too Many Requests
502 → Bad Gateway
503 → Service Unavailable
504 → Gateway Timeout
```

Choose based on what actually happened.

---

# 52. 502 vs 504

### 502 Bad Gateway

Gateway/proxy received an invalid response from an upstream server or couldn't get a valid upstream response.

### 504 Gateway Timeout

Gateway/proxy did not receive a timely response from the upstream service.

Exact behavior can depend on the proxy/gateway implementation.

---

# 53. Resilience4j

Resilience4j is a lightweight fault-tolerance library commonly used with Java/Spring applications.

It provides modules such as:

```text
CircuitBreaker
Retry
RateLimiter
Bulkhead
TimeLimiter
```

---

# 54. Resilience4j Circuit Breaker

Conceptually:

```java
@CircuitBreaker(
    name = "paymentService",
    fallbackMethod = "paymentFallback"
)
public PaymentResponse pay(PaymentRequest request) {
    return paymentClient.pay(request);
}
```

The exact configuration belongs outside the business method where practical.

---

# 55. Resilience4j Retry

Conceptually:

```java
@Retry(name = "paymentService")
public PaymentResponse pay(...) {
    ...
}
```

Use retry selectively.

Don't retry every exception.

---

# 56. Resilience4j Rate Limiter

Conceptually:

```java
@RateLimiter(name = "paymentService")
public PaymentResponse pay(...) {
    ...
}
```

Limits permitted calls.

---

# 57. Resilience4j Bulkhead

Conceptually:

```java
@Bulkhead(
    name = "paymentService"
)
public PaymentResponse pay(...) {
    ...
}
```

Limits concurrent executions according to the configured bulkhead type.

---

# 58. TimeLimiter

TimeLimiter can control asynchronous execution duration.

Conceptually:

```text
Async operation
 ↓
Time limit
 ↓
Timeout
```

Use it with appropriate asynchronous execution models.

---

# 59. Fallback Method

Conceptually:

```java
public PaymentResponse paymentFallback(
        PaymentRequest request,
        Exception ex) {

    return PaymentResponse.unavailable();
}
```

The fallback should not accidentally hide critical business failures.

---

# 60. Resilience4j Configuration

Typical configuration concepts:

```text
Failure threshold
Slow-call threshold
Sliding window
Wait duration
Retry count
Backoff
Timeout
Bulkhead capacity
Rate limit
```

Tune these based on real traffic and dependency behavior.

---

# 61. Circuit Breaker Metrics

Monitor:

```text
Current state
Failure rate
Slow-call rate
Rejected calls
Successful calls
Failed calls
```

A circuit breaker without monitoring is difficult to operate.

---

# 62. Retry Metrics

Monitor:

```text
Retry count
Retry success
Retry failure
Retry latency
```

If retry counts explode:

```text
Investigate dependency
```

Don't simply increase retry limits.

---

# 63. Bulkhead Metrics

Monitor:

```text
Concurrent calls
Rejected calls
Queue size
Execution duration
```

---

# 64. Rate Limiter Metrics

Monitor:

```text
Accepted requests
Rejected requests
Current limits
Traffic by client
```

---

# 65. Resilience Configuration Per Dependency

Don't use one global configuration for everything.

Example:

```text
Payment
timeout = 2s
retry = 1

Inventory
timeout = 500ms
retry = 2

Recommendations
timeout = 200ms
retry = 0
fallback = empty
```

These are illustrative values, not universal defaults.

---

# 66. Dependency Criticality

Classify dependencies:

```text
Critical
Important
Optional
```

Example:

```text
Payment = Critical
Inventory = Critical
Recommendations = Optional
Analytics = Optional
```

Resilience behavior can then match business importance.

---

# 67. Critical Dependency Failure

Payment fails:

```text
Checkout cannot complete
```

Return:

```text
Failure
```

Don't fake success.

---

# 68. Optional Dependency Failure

Recommendations fail:

```text
Product page still works
```

Use:

```text
Fallback
```

and continue.

---

# 69. Timeout Budget Example

Suppose:

```text
Gateway timeout = 3 seconds
```

Order calls:

```text
Inventory = 500ms
Payment = 2s
```

You need to account for:

```text
Gateway overhead
Network latency
Retries
```

Otherwise downstream operations can exceed the overall budget.

---

# 70. Retry Multiplication

Suppose:

```text
Gateway retries = 2
Order retries = 2
Payment client retries = 2
```

Potential attempts can multiply dramatically.

Avoid stacking retries at every layer.

Prefer deciding:

```text
Which layer owns retries?
```

and keep policies bounded.

---

# 71. Retry Ownership

A useful approach:

```text
Client
 ↓
Gateway
 ↓
Service
 ↓
Dependency
```

Don't have all four layers blindly retry.

Choose one or a small number of controlled layers.

---

# 72. Circuit Breaker Placement

Circuit breakers are commonly placed near the caller of the dependency.

Example:

```text
Order Service
 ↓
Circuit Breaker
 ↓
Payment Service
```

This protects the Order Service from repeatedly calling Payment.

---

# 73. Circuit Breaker Is Not a Health Check

Health check:

```text
Is service healthy?
```

Circuit breaker:

```text
Should this caller continue making requests based on observed failures?
```

They are related but different.

---

# 74. Circuit Breaker Is Not a Retry

Retry:

```text
Try again
```

Circuit breaker:

```text
Stop sending requests temporarily
```

They can be combined.

---

# 75. Bulkhead Is Not a Rate Limiter

Bulkhead:

```text
Limit concurrent resource usage
```

Rate limiter:

```text
Limit request rate
```

Example:

```text
Rate limiter = 100 req/sec
Bulkhead = max 20 concurrent payment calls
```

Both can be useful.

---

# 76. Load Shedding vs Rate Limiting

Rate limiting:

```text
Reject traffic above a configured rate
```

Load shedding:

```text
Reject work when the system is overloaded
```

Load shedding can be dynamic and priority-aware.

---

# 77. Priority-Based Load Shedding

Example:

```text
Checkout = HIGH
Search = MEDIUM
Recommendations = LOW
```

During overload:

```text
Keep checkout
Reduce recommendations
```

This protects critical business operations.

---

# 78. Graceful Shutdown

A resilient service should handle shutdown cleanly.

Conceptually:

```text
Stop accepting new traffic
 ↓
Finish in-flight work
 ↓
Commit/rollback safely
 ↓
Close resources
```

In Kubernetes, readiness can become false before termination completes.

---

# 79. Deployment and Resilience

During deployment:

```text
Instance removed
```

Traffic should move to healthy instances.

Use:

```text
Readiness
Graceful shutdown
Load balancing
Connection draining
```

to reduce failed requests.

---

# 80. Autoscaling

Autoscaling can help when demand increases.

Possible signals:

```text
CPU
Memory
Request rate
Queue depth
Kafka lag
Custom business metrics
```

Autoscaling is not a replacement for resilience patterns.

---

# 81. Retry vs Autoscaling

If Payment is slow because:

```text
Payment capacity insufficient
```

scaling may help.

If Payment is down because:

```text
Bug
```

adding instances won't fix it.

Understand the failure before scaling.

---

# 82. Resilience Testing

Don't assume resilience works.

Test:

```text
Dependency unavailable
Dependency slow
Network timeout
Redis failure
Kafka delay
Database exhaustion
Instance crash
```

---

# 83. Chaos Testing

Chaos testing intentionally introduces failures.

Example:

```text
Kill one service instance
 ↓
Observe system
```

or:

```text
Add network latency
```

Goal:

```text
Validate failure handling
```

---

# 84. Chaos Engineering

A mature approach asks:

```text
What happens if this dependency disappears?
```

before production discovers the answer.

---

# 85. Resilience Testing Example

Test:

```text
Payment Service response delay = 5 sec
```

Expected:

```text
Order timeout
 ↓
Circuit breaker eventually opens
 ↓
No thread exhaustion
 ↓
Meaningful failure response
```

---

# 86. Production Scenario

### Payment Service becomes slow.

Bad architecture:

```text
Order waits 30 sec
 ↓
Retries
 ↓
More requests
 ↓
Threads exhausted
```

Better:

```text
Timeout
 ↓
Bounded retry if appropriate
 ↓
Circuit breaker
 ↓
Fail fast
 ↓
Protect resources
```

---

# 87. Production Scenario

### Recommendation Service is down.

Possible:

```text
Timeout
 ↓
Circuit breaker
 ↓
Fallback
 ↓
Show page without recommendations
```

This is graceful degradation.

---

# 88. Production Scenario

### Payment Service is down.

Possible:

```text
Timeout
 ↓
Circuit breaker
 ↓
Fail checkout
 ↓
Order remains appropriate PENDING/FAILED state
 ↓
Do not pretend payment succeeded
```

The exact workflow may use asynchronous retry/Saga recovery.

---

# 89. Production Scenario

### Traffic suddenly increases 10x.

Consider:

```text
Rate limiting
Autoscaling
Caching
Load shedding
Backpressure
Queueing
Database protection
Connection pool limits
```

---

# 90. Production Scenario

### Redis is unavailable.

If Redis is only cache:

```text
Fallback to DB
```

but protect DB with:

```text
Rate limiting
Request coalescing
Connection limits
```

If Redis is critical state:

```text
High-availability strategy
```

may be required.

---

# 91. Production Scenario

### Database is slow.

Use:

```text
Timeouts
Connection pool protection
Caching
Circuit breaker where appropriate
Query optimization
Load shedding
```

Avoid blindly increasing connection pool size.

---

# 92. Production Scenario

### External payment provider returns 500.

Potential:

```text
Bounded retry
Exponential backoff
Jitter
Idempotency key
Circuit breaker
```

But confirm whether the provider actually processed the request before retrying payment operations.

---

# 93. Ambiguous Payment Outcome

This is a very important distributed-systems problem.

Request:

```text
Payment → provider
```

Provider processes payment.

Network fails before response reaches us.

Our system sees:

```text
TIMEOUT
```

But actual payment may be:

```text
SUCCESS
```

Don't simply retry without an idempotency strategy.

Use:

```text
Provider transaction ID
Idempotency key
Payment status query
Reconciliation
```

---

# 94. Reconciliation

If local state and payment provider state disagree:

```text
Local = UNKNOWN
Provider = SUCCESS
```

A reconciliation process can detect and resolve the mismatch.

This is especially important in financial workflows.

---

# 95. Resilience and Data Consistency

Fault tolerance must not violate business correctness.

Bad:

```text
Payment timeout
 ↓
Assume success
```

Better:

```text
Payment timeout
 ↓
UNKNOWN/PENDING
 ↓
Query provider/reconcile
 ↓
Determine final state
```

---

# 96. Observability for Resilience

Every resilience mechanism should be observable.

Track:

```text
Timeouts
Retries
Circuit state
Rejected calls
Fallbacks
Rate-limit rejections
Bulkhead rejections
```

Otherwise you won't know when resilience mechanisms are masking a dependency problem.

---

# 97. Alerting

Useful alerts:

```text
Circuit breaker open
Retry rate unusually high
Timeout rate high
Fallback rate high
Bulkhead rejected calls increasing
Rate-limit rejection spike
```

---

# 98. Resilience Dashboard

Example:

```text
Payment Service

Requests/sec
Error rate
p95 latency
Timeouts
Retries
Circuit state
Fallback count
Bulkhead rejected calls
```

---

# 99. Common Mistakes

```text
❌ No timeouts
❌ Infinite retries
❌ Retry every exception
❌ Retry every layer
❌ No jitter
❌ No idempotency
❌ Circuit breaker without monitoring
❌ Fake successful fallback for critical operations
❌ Unlimited concurrency
❌ One shared resource pool for everything
❌ No graceful shutdown
❌ No failure testing
❌ Blindly scaling instead of finding root cause
```

---

# 100. Resilience Pattern Comparison

| Pattern | Main purpose |
|---|---|
| Timeout | Stop waiting indefinitely |
| Retry | Recover from transient failures |
| Backoff | Spread retries |
| Jitter | Avoid synchronized retries |
| Circuit Breaker | Stop calls to failing dependency |
| Bulkhead | Isolate resources |
| Rate Limiter | Control request rate |
| Load Shedding | Reject work under overload |
| Backpressure | Prevent producer overload |
| Fallback | Provide alternative behavior |
| Graceful Degradation | Keep useful functionality |
| Idempotency | Make retries safe |

---

# 101. Interview Question

### "What is fault tolerance?"

Answer:

> "Fault tolerance is the ability of a system to continue providing acceptable service despite certain component failures. In microservices this usually involves isolating failures and preventing one dependency from taking down the entire system."

---

# 102. Interview Question

### "Why are timeouts important?"

Answer:

> "Without timeouts, a slow dependency can hold threads, connections and other resources indefinitely. That can cause cascading failures. Timeouts provide a bounded waiting period and help the system fail fast."

---

# 103. Interview Question

### "How do retries make an outage worse?"

Answer:

> "If a dependency is already overloaded, retries multiply incoming traffic. For example, 1,000 failed requests with three retries can create up to 4,000 attempts. I'd use bounded retries, exponential backoff, jitter and retry only transient failures."

---

# 104. Interview Question

### "Explain circuit breaker."

Answer:

> "A circuit breaker monitors calls to a dependency. After configured failure or slow-call thresholds are reached it opens and fails calls fast. After a wait period it enters half-open mode to test recovery. If successful it closes again."

---

# 105. Interview Question

### "What is bulkhead?"

Answer:

> "Bulkhead isolates resources so one failing or slow dependency cannot consume all application resources. For example, payment calls can use a limited pool separate from inventory calls."

---

# 106. Interview Question

### "Retry vs circuit breaker?"

Answer:

> "Retry is useful for transient failures and tries the operation again. A circuit breaker stops sending requests when a dependency is consistently failing or slow. They solve different problems and can be combined carefully."

---

# 107. Interview Question

### "What is graceful degradation?"

Answer:

> "It means maintaining the core user experience when optional functionality fails. For example, if recommendations are unavailable, the product page can still show product information without recommendations."

---

# 108. Interview Question

### "What is load shedding?"

Answer:

> "Load shedding deliberately rejects lower-priority or excess work during overload so critical functionality remains available instead of allowing the entire system to fail."

---

# 109. Interview Question

### "What is backpressure?"

Answer:

> "Backpressure prevents a producer from overwhelming a slower consumer. The producer may slow down, queue work, or reject requests when downstream capacity is exhausted."

---

# 110. Interview Question

### "What is Resilience4j?"

Answer:

> "Resilience4j is a Java fault-tolerance library commonly integrated with Spring Boot. It provides patterns such as circuit breakers, retries, rate limiting, bulkheads and time limiters."

---

# 111. Interview Scenario

### "Payment API is taking 20 seconds. What would you do?"

Answer:

> "I'd enforce a bounded timeout so requests don't wait indefinitely. I'd avoid aggressive retries because the dependency is already slow. I'd use a circuit breaker based on failure/slow-call behavior, isolate payment resources with a bulkhead if needed, and expose an appropriate pending/failure state. I'd monitor the dependency and investigate its root cause."

---

# 112. Interview Scenario

### "Recommendation service is down. Should checkout fail?"

Answer:

> "No, if recommendations are optional. I'd use a timeout and circuit breaker and return a fallback such as an empty recommendation section. The core checkout flow should continue."

---

# 113. Interview Scenario

### "Payment times out. Should we retry?"

Answer:

> "Not blindly. A timeout can mean the provider processed the payment but our response was lost. I'd use an idempotency key and provider transaction/status mechanism so a retry cannot create a duplicate payment."

---

# 114. Interview Scenario

### "All service calls have three retries. Is that good?"

Answer:

> "Not necessarily. Stacking retries across gateway, service and client layers can multiply traffic and create retry storms. I'd decide where retries belong, keep them bounded, use backoff and jitter, and make retryable operations idempotent."

---

# 115. Interview Scenario

### "How do you prevent one dependency from consuming all threads?"

Answer:

> "I'd use timeouts and bulkhead isolation, potentially with separate thread pools or concurrency limits for different dependencies. That way a slow payment service cannot consume the resources needed for inventory or other critical operations."

---

# 116. Interview Scenario

### "How do you test resilience?"

Answer:

> "I'd test dependency timeouts, service crashes, slow responses, connection exhaustion, Redis/Kafka failures and traffic spikes. I'd verify that timeouts, circuit breakers, retries, bulkheads and fallbacks behave as expected. Chaos testing can also validate failure behavior."

---

# 117. E-Commerce Resilience Design

Example:

```text
                    Gateway
                       |
                +------+------+
                |             |
             Order          Product
                |
        +-------+-------+
        |               |
    Inventory         Payment
        |               |
      MySQL          Provider
```

Policies:

```text
Product:
  Cache
  Short timeout
  Fallback where appropriate

Inventory:
  Timeout
  Limited retries
  Bulkhead

Payment:
  Timeout
  Idempotency
  Limited retry
  Circuit breaker
  Reconciliation

Gateway:
  Rate limiting
  Timeouts
  Circuit breakers
```

---

# 118. Resilience Hierarchy

Think from outer to inner:

```text
Traffic control
 ↓
Rate limiting
 ↓
Timeout
 ↓
Retry/backoff
 ↓
Circuit breaker
 ↓
Bulkhead
 ↓
Fallback/degradation
 ↓
Business recovery
```

Not every request needs every pattern.

---

# 119. Resilience Decision Tree

Ask:

```text
Is failure transient?
 ├─ YES → bounded retry?
 └─ NO  → fail fast

Is dependency consistently failing?
 ├─ YES → circuit breaker
 └─ NO  → normal flow

Can feature be degraded?
 ├─ YES → fallback
 └─ NO  → business failure

Can dependency consume shared resources?
 ├─ YES → bulkhead
 └─ NO  → shared pool may be acceptable

Is traffic excessive?
 ├─ YES → rate limit/load shed
 └─ NO  → normal traffic
```

---

# 120. Final Mental Model

Remember:

```text
Timeout
→ Don't wait forever.

Retry
→ Try transient failures again.

Backoff
→ Wait longer between retries.

Jitter
→ Randomize retry timing.

Circuit Breaker
→ Stop calling a failing dependency.

Bulkhead
→ Isolate resources.

Rate Limiter
→ Control traffic.

Load Shedding
→ Reject work under overload.

Backpressure
→ Slow producers when consumers can't keep up.

Fallback
→ Alternative behavior.

Graceful Degradation
→ Preserve useful functionality.

Idempotency
→ Make retries safe.

Observability
→ Know when resilience mechanisms activate.
```

---

# 121. Final Interview Answer

If asked:

> "How would you design a resilient Spring Boot microservice?"

Use:

> "I'd start with strict timeouts on all network calls so resources aren't held indefinitely. For transient failures I'd use bounded retries with exponential backoff and jitter, but only for operations that are safe to retry. I'd use circuit breakers to stop repeatedly calling unhealthy dependencies and bulkheads to isolate resources between critical dependencies. Rate limiting and load shedding would protect the service during overload. For optional features I'd use graceful degradation, while critical operations such as payments would preserve business correctness using idempotency and explicit pending/reconciliation states. Finally, I'd monitor timeouts, retries, circuit states and fallbacks so the resilience mechanisms themselves are observable."

---

# 122. Revision Checklist

```text
□ Resilience
□ Fault tolerance
□ Failure isolation
□ Timeout
□ Connection timeout
□ Read timeout
□ Timeout budget
□ Retry
□ Transient failure
□ Retry storm
□ Exponential backoff
□ Jitter
□ Retry budget
□ Idempotency
□ Idempotency key
□ Circuit breaker
□ CLOSED
□ OPEN
□ HALF_OPEN
□ Slow calls
□ Bulkhead
□ Semaphore bulkhead
□ Thread-pool bulkhead
□ Rate limiting
□ Load shedding
□ Backpressure
□ Fallback
□ Graceful degradation
□ Cascading failure
□ Fail fast
□ HTTP 429
□ HTTP 502
□ HTTP 503
□ HTTP 504
□ Resilience4j
□ TimeLimiter
□ Resilience metrics
□ Dependency criticality
□ Retry ownership
□ Circuit placement
□ Graceful shutdown
□ Autoscaling
□ Chaos testing
□ Reconciliation
□ Production scenarios
```

---

# 123. The Interviewer's Real Test

If asked:

> "Payment Service is slow, users are retrying, and your Order Service is running out of threads. How do you stop the cascade?"

Think:

```text
Payment slow
      ↓
Strict timeout
      ↓
Stop long waits
      ↓
Bounded retry + backoff/jitter
      ↓
Circuit breaker
      ↓
Fail fast when dependency remains unhealthy
      ↓
Bulkhead payment calls
      ↓
Protect Order threads
      ↓
Rate limit / load shed if traffic remains high
      ↓
Keep critical resources available
      ↓
Monitor recovery
```

The key interview lesson is:

> **Resilience is not about making failures disappear. It is about containing failures, protecting resources, and preserving the most important business functionality.**

# Microservices — Resilience4j: Circuit Breaker, Retry, Timeout & Bulkhead

This file covers the resilience patterns you should know for Java/Spring Boot microservices interviews.

The main goal is simple:

> A failure in one service should not bring down the entire system.

---

# 1. Why Resilience Matters

In a monolith:

```text
Order Service
   ↓
Payment logic
```

A method call can fail, but everything is inside one process.

In microservices:

```text
Order Service
      |
      ↓
Payment Service
      |
      ↓
External Payment Provider
```

Now there are multiple failure points.

Possible failures:

```text
Network failure
Timeout
Service unavailable
Database failure
External API failure
High latency
Connection pool exhaustion
Traffic spike
```

---

# 2. What Is Resilience?

Resilience means the system can:

```text
Detect failures
Limit failure impact
Recover from temporary failures
Continue providing useful functionality
```

Common resilience patterns:

```text
Timeout
Retry
Circuit Breaker
Bulkhead
Rate Limiting
Fallback
Idempotency
Caching
Load Shedding
```

---

# 3. The Four Most Important Patterns

For interviews, remember:

```text
Timeout
→ Stop waiting forever

Retry
→ Try again for transient failures

Circuit Breaker
→ Stop calling an unhealthy dependency

Bulkhead
→ Prevent one dependency from consuming all resources
```

These patterns solve different problems.

---

# 4. Timeout

A timeout defines how long a caller waits for a dependency.

Example:

```text
Order Service
      |
      | timeout = 2 seconds
      ↓
Payment Service
```

If Payment doesn't respond within 2 seconds:

```text
Timeout
```

---

# 5. Why Timeouts Are Critical

Without timeouts:

```text
Request
 ↓
Payment
 ↓
wait...
 ↓
wait...
 ↓
wait...
```

Threads/connections can remain occupied.

Under load:

```text
100 requests
 ↓
100 waiting threads
 ↓
resource exhaustion
```

A timeout limits this damage.

---

# 6. Timeout Is Not a Failure Recovery Strategy

A timeout answers:

> "How long should I wait?"

It does not answer:

> "What should I do next?"

After timeout you might:

```text
Return an error
Retry
Open circuit
Use fallback
Queue work
```

depending on the business operation.

---

# 7. Retry

Retry means attempting an operation again after a failure.

Example:

```text
Call Payment
   ↓
Temporary failure
   ↓
Retry
   ↓
Success
```

Useful for transient failures such as:

```text
Temporary network issue
Connection reset
503 Service Unavailable
Temporary infrastructure failure
```

---

# 8. Don't Retry Everything

Bad:

```text
Every failure
 ↓
Retry 10 times
```

This can make an outage worse.

Some failures are permanent:

```text
400 Bad Request
Invalid input
Authentication failure
Authorization failure
Business rule violation
```

Retrying these usually doesn't help.

---

# 9. Exponential Backoff

Instead of immediately retrying:

```text
Retry
Retry
Retry
```

use increasing delays:

```text
Attempt 1
 ↓
100 ms
 ↓
Attempt 2
 ↓
200 ms
 ↓
Attempt 3
 ↓
400 ms
```

This gives a struggling dependency time to recover.

---

# 10. Jitter

If thousands of clients retry at exactly the same time:

```text
100 ms
100 ms
100 ms
100 ms
...
```

they can create another traffic spike.

Jitter introduces randomness:

```text
87 ms
113 ms
94 ms
121 ms
```

This spreads retries over time.

---

# 11. Retry Storm

Imagine:

```text
1,000 requests
```

Each retries 3 times.

Potentially:

```text
1,000 initial calls
+
3,000 retry attempts
```

The failing service receives even more traffic.

This is called retry amplification/storm behavior.

---

# 12. Idempotency and Retry

Critical rule:

> Don't blindly retry operations that can create duplicate side effects.

Example:

```http
POST /payments
```

Payment succeeds.

Response is lost.

Client retries.

Without idempotency:

```text
Charge #1
Charge #2
```

---

# 13. Idempotency Key

Client sends:

```http
Idempotency-Key: abc123
```

Payment Service records:

```text
abc123 → PaymentResult
```

If the same request arrives again:

```text
abc123
```

the service returns the previous result rather than charging again.

---

# 14. Circuit Breaker

A circuit breaker prevents repeated calls to a failing dependency.

Conceptually:

```text
Order Service
      |
      ↓
Payment Service
```

If Payment repeatedly fails:

```text
Circuit opens
```

Future calls fail fast instead of continuing to hammer Payment.

---

# 15. Circuit Breaker States

The classic model has three states:

```text
CLOSED
OPEN
HALF_OPEN
```

---

# 16. CLOSED

Normal operation:

```text
Order
 ↓
Payment
 ↓
Response
```

The circuit monitors failures.

---

# 17. OPEN

Failure threshold is reached:

```text
Payment unhealthy
      ↓
Circuit OPEN
```

Calls are rejected immediately.

The application doesn't keep making calls to Payment.

---

# 18. HALF_OPEN

After a configured wait period:

```text
OPEN
 ↓
wait
 ↓
HALF_OPEN
```

A small number of test calls are allowed.

If successful:

```text
HALF_OPEN
 ↓
CLOSED
```

If failures continue:

```text
HALF_OPEN
 ↓
OPEN
```

---

# 19. Circuit Breaker State Diagram

```text
                 failures
      +--------------------------+
      |                          ↓
   CLOSED --------------------> OPEN
      ↑                          |
      |                          | wait duration
      |                          ↓
      +--------------------- HALF_OPEN
                                |
                           test calls
                           /        \
                      success      failure
                        |             |
                        ↓             ↓
                     CLOSED         OPEN
```

---

# 20. What Circuit Breaker Solves

It protects the caller from:

```text
Repeated failures
Long waits
Resource exhaustion
Cascading failures
Unhealthy dependencies
```

---

# 21. Circuit Breaker Does NOT Fix the Dependency

Important interview point:

> A circuit breaker doesn't repair Payment Service.

It protects the rest of the system while Payment is unhealthy.

---

# 22. Fallback

When a dependency fails, the caller may provide an alternative response.

Example:

```text
Recommendation Service unavailable
        ↓
Fallback
        ↓
Return popular products
```

Fallback should be meaningful.

---

# 23. Bad Fallback

Payment fails:

```text
Fallback → "Payment successful"
```

Obviously unacceptable.

Fallback must never falsely report business success.

---

# 24. Good Fallback

Recommendation service fails:

```text
Return:
"Popular products"
```

or:

```text
Return empty recommendations
```

if the business allows it.

---

# 25. Bulkhead

Bulkhead prevents one dependency from consuming all resources.

The name comes from ship compartments.

If one compartment floods:

```text
Other compartments remain protected.
```

---

# 26. Microservices Bulkhead

Suppose Order Service calls:

```text
Payment
Recommendation
Shipping
```

Without isolation:

```text
Recommendation becomes slow
 ↓
all threads become occupied
 ↓
Order Service becomes unhealthy
```

With bulkheads:

```text
Payment pool
Recommendation pool
Shipping pool
```

Failure in one area is contained.

---

# 27. Thread Pool Bulkhead

Conceptually:

```text
Order Service
 |
 +--- Payment → 20 threads
 |
 +--- Shipping → 10 threads
 |
 +--- Recommendation → 5 threads
```

Recommendation cannot consume all available threads.

---

# 28. Semaphore Bulkhead

Instead of separate thread pools, limit concurrent calls:

```text
Maximum concurrent calls = 10
```

If all 10 are occupied:

```text
New call rejected
```

This is another bulkhead strategy.

---

# 29. Bulkhead vs Circuit Breaker

Very common interview question.

### Circuit Breaker

Protects against:

```text
Repeated dependency failures
```

### Bulkhead

Protects against:

```text
Resource exhaustion caused by one dependency/workload
```

They can be used together.

---

# 30. Timeout vs Circuit Breaker

### Timeout

```text
Stop waiting after X seconds.
```

### Circuit Breaker

```text
Stop making calls after repeated failures.
```

Usually:

```text
Timeout
+
Circuit Breaker
```

work well together.

---

# 31. Retry vs Circuit Breaker

Retry:

```text
"This failure may be temporary.
Try again."
```

Circuit breaker:

```text
"This dependency appears unhealthy.
Stop calling it for now."
```

They complement each other.

---

# 32. Typical Resilience Flow

A simplified request:

```text
Client
  ↓
Order Service
  ↓
Timeout
  ↓
Retry
  ↓
Circuit Breaker
  ↓
Payment
```

The exact ordering of resilience decorators depends on the framework/configuration.

Don't memorize a single universal order.

---

# 33. Resilience4j

Resilience4j is a lightweight fault-tolerance library commonly used with Java applications.

It provides modules for:

```text
Circuit Breaker
Retry
Rate Limiter
Bulkhead
Time Limiter
```

---

# 34. Why Resilience4j?

It is:

```text
Lightweight
Modular
Functional
Java-friendly
Spring Boot friendly
```

It is commonly used instead of older Hystrix-based approaches.

---

# 35. Resilience4j Circuit Breaker

Conceptually:

```java
@CircuitBreaker(
    name = "paymentService",
    fallbackMethod = "paymentFallback"
)
public PaymentResponse processPayment(
        PaymentRequest request) {

    return paymentClient.process(request);
}
```

The exact configuration belongs outside the business method.

---

# 36. Fallback Method

Conceptually:

```java
public PaymentResponse paymentFallback(
        PaymentRequest request,
        Throwable throwable) {

    return new PaymentResponse(
        "PAYMENT_SERVICE_UNAVAILABLE"
    );
}
```

The fallback signature needs to match the protected method appropriately.

---

# 37. Retry Annotation

Conceptually:

```java
@Retry(name = "paymentService")
public PaymentResponse processPayment(
        PaymentRequest request) {

    return paymentClient.process(request);
}
```

Resilience4j can retry according to configured rules.

---

# 38. Time Limiter

A TimeLimiter controls how long an asynchronous operation is allowed to run.

It is particularly relevant with asynchronous/reactive operations.

Conceptually:

```text
Async operation
      ↓
TimeLimiter
      ↓
timeout
```

For blocking HTTP clients, configure the underlying HTTP client's connection/read/write timeouts appropriately as well.

---

# 39. Rate Limiter

Rate limiting controls how many calls are allowed over a period.

Example:

```text
Payment Service
maximum = 100 requests/sec
```

Additional calls can be rejected or delayed depending on configuration.

---

# 40. Rate Limiter vs Bulkhead

Rate limiter:

```text
Controls request rate
```

Bulkhead:

```text
Controls concurrent resource usage
```

Example:

```text
Rate = 100 requests/sec
Concurrency = 20
```

These solve different problems.

---

# 41. Circuit Breaker Configuration

Important concepts include:

```text
Failure rate threshold
Slow call rate threshold
Sliding window
Minimum number of calls
Wait duration in OPEN state
Permitted calls in HALF_OPEN
```

---

# 42. Failure Rate Threshold

Example:

```text
Failure threshold = 50%
```

If enough calls are measured and roughly half fail:

```text
Circuit may OPEN
```

The exact behavior depends on the configured sliding window and minimum call requirements.

---

# 43. Sliding Window

Circuit breaker metrics can use a sliding window.

Common types:

```text
COUNT_BASED
TIME_BASED
```

### Count-based

Evaluate the last:

```text
N calls
```

### Time-based

Evaluate calls during:

```text
last N seconds
```

---

# 44. Minimum Number of Calls

Suppose:

```text
Threshold = 50%
```

and only one request has happened.

That one request fails.

You usually don't want the circuit to immediately conclude:

```text
Payment is definitely unhealthy.
```

A minimum call threshold prevents premature decisions.

---

# 45. Slow Call Rate

A dependency can be unhealthy because it is:

```text
Slow
```

even when requests eventually succeed.

Circuit breakers can be configured to consider slow calls.

Example:

```text
Calls > 2 seconds
→ considered slow
```

If slow-call rate becomes too high, the circuit may open.

---

# 46. Circuit Breaker Example

Suppose:

```text
Minimum calls = 10
Failure threshold = 50%
```

Measured results:

```text
10 calls
6 failures
```

Failure rate:

```text
60%
```

Circuit can transition toward:

```text
OPEN
```

depending on other configuration.

---

# 47. OPEN State Behavior

When open:

```text
Call does not reach Payment
```

Instead:

```text
CircuitBreakerOpenException
```

or framework-specific behavior can occur.

A fallback may then execute.

---

# 48. HALF_OPEN Behavior

After:

```text
waitDurationInOpenState
```

the breaker moves toward:

```text
HALF_OPEN
```

A limited number of calls test recovery.

Example:

```text
5 permitted calls
```

If healthy:

```text
CLOSED
```

---

# 49. Why Not Immediately Close?

Because the dependency may still be unhealthy.

Testing with a small number of calls avoids immediately sending full traffic back to a recovering service.

---

# 50. Combining Patterns

Example:

```text
Client
 ↓
Order Service
 ↓
Rate Limiter
 ↓
Bulkhead
 ↓
Timeout
 ↓
Retry
 ↓
Circuit Breaker
 ↓
Payment Service
```

This is conceptual.

Actual decorator ordering should be chosen based on the desired semantics and framework configuration.

---

# 51. Don't Add Every Pattern Everywhere

A common mistake:

```text
Every endpoint
+ retry
+ circuit breaker
+ bulkhead
+ rate limiter
+ fallback
```

This can make the system difficult to understand.

Use resilience mechanisms where they solve a real problem.

---

# 52. Retry Configuration

A reasonable retry policy should define:

```text
Maximum attempts
Wait duration
Backoff
Jitter
Retryable exceptions
Non-retryable exceptions
```

---

# 53. Retryable Exceptions

Potentially retry:

```text
ConnectException
TimeoutException
503
Temporary network failure
```

Potentially don't retry:

```text
400
401
403
404
Business validation exception
```

Exact choices depend on the operation and downstream contract.

---

# 54. Circuit Breaker + Retry Interaction

Imagine:

```text
1 user request
 ↓
Retry 3 times
```

The circuit breaker may observe multiple failed attempts depending on how the resilience decorators are composed.

This matters when setting thresholds.

Don't assume:

```text
1 user request = 1 breaker call
```

in every configuration.

---

# 55. Retry and Circuit Breaker Danger

Bad configuration:

```text
Retries = 10
Timeout = 30 sec
```

One user request could wait for a very long time.

Then multiply by:

```text
1,000 concurrent requests
```

You can exhaust resources quickly.

Resilience configuration must consider the **end-to-end latency budget**.

---

# 56. Fallback and Business Semantics

Fallback is safest for operations where stale/partial data is acceptable.

Good:

```text
Recommendations
Product reviews
Non-critical analytics
```

Riskier:

```text
Payment
Inventory reservation
Order creation
```

For critical transactions, a fallback should usually communicate failure rather than pretend success.

---

# 57. Graceful Degradation

Graceful degradation means the system continues providing reduced functionality when some components fail.

Example:

```text
Recommendation Service DOWN
```

Still allow:

```text
Product browsing
Cart
Checkout
```

but hide:

```text
Recommendations
```

---

# 58. Fail Fast

Fail fast means returning quickly when continuing is unlikely to succeed.

Example:

```text
Circuit OPEN
 ↓
immediate failure
```

This is better than:

```text
wait 30 seconds
 ↓
retry
 ↓
wait
```

for an already-known unhealthy dependency.

---

# 59. Cascading Failure

Example:

```text
Payment slows
 ↓
Order waits
 ↓
Order threads exhausted
 ↓
Gateway waits
 ↓
Gateway resources exhausted
 ↓
Entire system becomes unhealthy
```

Resilience patterns are designed to break this chain.

---

# 60. Connection Pool Exhaustion

Suppose:

```text
HTTP connection pool = 100
```

Payment becomes slow.

100 connections become occupied.

New requests wait.

Eventually:

```text
Timeouts
 ↓
More retries
 ↓
More load
```

This is why timeouts and bulkheads matter.

---

# 61. Thread Pool Exhaustion

Similarly:

```text
Thread pool = 200
```

A slow dependency can cause threads to remain occupied.

Eventually:

```text
No available threads
```

Requests queue or fail.

---

# 62. Bulkhead Example

Without bulkhead:

```text
Payment calls = 190 threads
Other work = 10 threads
```

Payment consumes almost everything.

With isolation:

```text
Payment = max 50
Other workloads = remaining capacity
```

The exact mechanism depends on the application.

---

# 63. Resilience4j Bulkhead

Conceptually:

```java
@Bulkhead(
    name = "paymentService",
    type = Bulkhead.Type.SEMAPHORE
)
public PaymentResponse processPayment(...) {
    ...
}
```

Another strategy can use a dedicated thread-pool bulkhead.

---

# 64. Semaphore Bulkhead

A semaphore limits concurrent executions.

Example:

```text
maxConcurrentCalls = 10
```

Only 10 calls can execute concurrently through that bulkhead.

The next call can be rejected if the limit is reached.

---

# 65. Thread-Pool Bulkhead

A thread-pool bulkhead isolates execution using a dedicated executor.

Conceptually:

```text
Main workload
      |
      +---- Payment executor
      |
      +---- Other executor
```

This can provide stronger execution isolation but introduces more resource/complexity considerations.

---

# 66. Rate Limiting + Bulkhead

Suppose:

```text
Rate limit = 100/sec
Concurrency = 20
```

Then:

```text
Rate Limiter
→ controls how quickly calls enter

Bulkhead
→ controls how many are executing concurrently
```

They can work together.

---

# 67. Timeout + Retry

Suppose:

```text
Timeout = 500 ms
Retries = 2
```

Potential maximum waiting time can exceed 500 ms because there may be multiple attempts.

You must design the total latency budget.

---

# 68. Timeout Budget Example

Suppose API SLA:

```text
1 second
```

You can't casually configure:

```text
3 retries
×
1 second timeout
```

and expect the API to remain within 1 second.

The resilience policy must fit inside the overall request budget.

---

# 69. Resilience at Multiple Layers

Resilience can exist at:

```text
Client
 ↓
Gateway
 ↓
Service
 ↓
HTTP client
 ↓
Database
 ↓
External provider
```

Avoid stacking conflicting policies blindly.

For example:

```text
Gateway retries 3 times
Service retries 3 times
HTTP client retries 3 times
```

can create huge retry amplification.

---

# 70. Retry Ownership

A good architecture should make it clear:

```text
Which layer owns retries?
```

Ideally avoid uncontrolled retries at every layer.

---

# 71. External API Example

Order Service calls:

```text
Shipping Provider
```

Possible design:

```text
Timeout
+
small retry policy
+
circuit breaker
+
idempotency if supported
```

If shipping request is not safe to retry, use a provider-supported idempotency key or a durable workflow.

---

# 72. Database Resilience

Resilience isn't only for HTTP.

Database issues can include:

```text
Connection exhaustion
Slow queries
Deadlocks
Transient network failures
```

Use appropriate:

```text
Connection pool limits
Query timeouts
Transaction boundaries
Monitoring
```

Don't blindly retry database writes.

---

# 73. Fallback Cache

Example:

```text
Product Service
 ↓
Redis
 ↓
Database
```

If the database is temporarily unavailable:

```text
Cache may serve recently cached product data
```

This is graceful degradation.

But stale data must be acceptable.

---

# 74. Cache + Resilience

Caching can reduce dependency traffic:

```text
Request
 ↓
Cache HIT
 ↓
No database call
```

This can indirectly improve resilience.

But caching introduces:

```text
Staleness
Invalidation complexity
Memory usage
Cache failure scenarios
```

---

# 75. Circuit Breaker Metrics

Monitor:

```text
Circuit state
Failure rate
Slow call rate
Call count
Rejected calls
Fallback count
```

Without monitoring, you may not know whether your resilience configuration is actually helping.

---

# 76. Operational Example

Suppose:

```text
Payment Service
failure rate = 70%
```

Circuit opens.

Monitor:

```text
Payment failure rate
Circuit state
Order fallback rate
Checkout success rate
```

The circuit breaker is only one part of the overall observability story.

---

# 77. Alerting

Useful alerts:

```text
Circuit opened repeatedly
High downstream latency
High retry count
High timeout rate
Bulkhead rejection rate
Fallback rate increased
```

Don't alert on every single transient failure.

Focus on meaningful trends.

---

# 78. Testing Resilience

You should test:

```text
Dependency unavailable
Dependency slow
Dependency returns 503
Network timeout
Repeated failures
Recovery
Duplicate requests
High concurrency
```

---

# 79. Fault Injection

You can intentionally introduce:

```text
Latency
Exceptions
Service shutdown
Network failures
```

and observe whether the system behaves correctly.

This is part of resilience/chaos testing.

---

# 80. Recovery Testing

Don't test only:

```text
Service fails
```

Also test:

```text
Service recovers
```

For example:

```text
Payment DOWN
 ↓
Circuit OPEN
 ↓
Payment recovers
 ↓
HALF_OPEN
 ↓
successful probes
 ↓
CLOSED
```

---

# 81. Resilience4j Configuration Concept

A configuration might conceptually contain:

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        failureRateThreshold: 50
        minimumNumberOfCalls: 10
        slidingWindowSize: 20
        waitDurationInOpenState: 10s

  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 500ms
```

Exact property names and defaults depend on the Resilience4j/Spring Boot version.

---

# 82. Configuration Should Be Externalized

Don't hardcode resilience values throughout Java code.

Prefer:

```text
application.yml
environment variables
configuration management
```

This allows tuning without changing business logic.

---

# 83. Don't Hide Failures

Bad fallback:

```text
return empty success response
```

when payment actually failed.

This creates data inconsistency and makes debugging difficult.

A resilience mechanism should preserve correct business semantics.

---

# 84. Resilience vs Reliability

Reliability:

```text
System performs correctly over time.
```

Resilience:

```text
System continues or recovers when failures occur.
```

They are related but not identical.

---

# 85. Availability vs Resilience

Availability:

```text
Is the system accessible?
```

Resilience:

```text
How does it behave when components fail?
```

A highly available system can still have poor resilience if failures cascade.

---

# 86. CAP and Resilience

In distributed systems, network partitions create trade-offs around:

```text
Consistency
Availability
Partition tolerance
```

Microservices often accept eventual consistency where appropriate.

Resilience mechanisms don't eliminate these fundamental distributed-system trade-offs.

---

# 87. Interview Question

### "What is a circuit breaker?"

Answer:

> "A circuit breaker monitors calls to a dependency and opens when failures or slow calls cross configured thresholds. While open, calls fail fast instead of continuing to hit the unhealthy dependency. After a wait period, it allows limited test calls in half-open state to determine whether the dependency has recovered."

---

# 88. Interview Question

### "What are the circuit breaker states?"

Answer:

> "Closed means normal traffic is allowed, open means calls are rejected quickly, and half-open allows a limited number of test calls to determine whether the dependency has recovered."

---

# 89. Interview Question

### "Retry vs circuit breaker?"

Answer:

> "Retry is useful for transient failures where another attempt may succeed. A circuit breaker prevents repeated calls when a dependency is consistently unhealthy. They can be combined, but retry counts must be bounded to avoid amplifying load."

---

# 90. Interview Question

### "Why use exponential backoff?"

Answer:

> "It spaces retries out instead of immediately sending repeated requests to an unhealthy dependency. This gives the dependency time to recover and reduces retry storms."

---

# 91. Interview Question

### "What is jitter?"

Answer:

> "Jitter adds randomness to retry delays so many clients don't retry at exactly the same time and create another traffic spike."

---

# 92. Interview Question

### "What is bulkhead?"

Answer:

> "Bulkhead isolates resources so one dependency or workload cannot consume the entire capacity of a service. For example, we can limit concurrent payment calls so a slow payment provider doesn't exhaust all application resources."

---

# 93. Interview Question

### "What is graceful degradation?"

Answer:

> "It means the system continues providing useful functionality with reduced capabilities when a non-critical dependency fails. For example, product recommendations can be unavailable while product browsing and checkout continue."

---

# 94. Interview Question

### "Should we retry payment requests?"

Answer:

> "Only with careful idempotency protection and a clear understanding of the payment provider's semantics. A lost response doesn't necessarily mean the payment failed, so blindly retrying could create duplicate charges."

---

# 95. Interview Scenario

### "Payment Service is down. How does your Order Service behave?"

Good answer:

> "I'd use a bounded timeout and circuit breaker to avoid waiting indefinitely and repeatedly calling the unhealthy service. If payment is mandatory for order completion, I'd return a clear failure rather than pretending the order was paid. For asynchronous workflows, I could persist the order state and retry payment through a durable workflow if the business process allows it."

---

# 96. Interview Scenario

### "Recommendation Service is down."

Good answer:

> "Since recommendations are usually non-critical, I'd fail gracefully. The circuit breaker can stop repeated calls and the application can return the product page without recommendations, possibly using cached popular products."

---

# 97. Interview Scenario

### "A downstream service is slow, not failing."

Answer:

> "I'd configure timeouts and consider slow-call detection in the circuit breaker. I'd also inspect connection/thread pools and latency metrics. If the dependency is consistently slow, the circuit breaker or bulkhead can prevent the latency from exhausting resources."

---

# 98. Interview Scenario

### "Your retries are increasing traffic dramatically."

Answer:

> "I'd check whether multiple layers are retrying the same request. I'd reduce retry attempts, add exponential backoff and jitter, restrict retries to transient failures, and use circuit breakers to stop retrying an unhealthy dependency."

---

# 99. Interview Scenario

### "How would you design resilience for an e-commerce checkout?"

Possible approach:

```text
Checkout
   |
   +--> Inventory
   |      |
   |   timeout
   |   bounded retry if safe
   |   circuit breaker
   |
   +--> Payment
   |      |
   |   timeout
   |   idempotency
   |   bounded retry if safe
   |   circuit breaker
   |
   +--> Notification
          |
       asynchronous event
```

The exact workflow depends on business consistency requirements.

---

# 100. Common Mistakes

```text
❌ Infinite retries
❌ Retrying all exceptions
❌ No timeout
❌ Retry without idempotency
❌ Circuit breaker with no monitoring
❌ Huge fallback logic
❌ Fallback that lies about success
❌ No resource isolation
❌ Retry at every layer
❌ Ignoring recovery behavior
❌ Using every resilience pattern everywhere
```

---

# 101. Practical Resilience Checklist

For a synchronous dependency, ask:

```text
1. What is the timeout?
2. Which failures are retryable?
3. Is the operation idempotent?
4. How many retries?
5. What backoff?
6. Is jitter needed?
7. When should circuit open?
8. What resources need isolation?
9. Is fallback possible?
10. How will recovery be detected?
11. What metrics will we monitor?
```

---

# 102. Final Mental Model

Remember:

```text
Timeout
→ Don't wait forever.

Retry
→ Try transient failures again.

Backoff
→ Don't retry immediately.

Jitter
→ Don't let everyone retry together.

Circuit Breaker
→ Stop calling an unhealthy dependency.

Bulkhead
→ Don't let one dependency consume everything.

Fallback
→ Continue with reduced functionality when safe.

Idempotency
→ Make retries safe for side effects.
```

---

# 103. Final Interview Answer

If asked:

> "How do you make a Spring Boot microservice resilient?"

Use:

> "I'd start with sensible timeouts for every remote dependency. For transient failures I'd use bounded retries with exponential backoff and jitter, but only where the operation is safe to retry. I'd use circuit breakers to stop repeatedly calling unhealthy dependencies and bulkheads to prevent one dependency from exhausting service resources. Where business requirements allow it, I'd use graceful fallbacks or asynchronous processing. I'd also make important operations idempotent and monitor timeout, retry, circuit-breaker and fallback metrics."

---

# 104. Revision Checklist

```text
□ Resilience
□ Timeout
□ Retry
□ Exponential backoff
□ Jitter
□ Retry storm
□ Idempotency
□ Idempotency key
□ Circuit breaker
□ CLOSED
□ OPEN
□ HALF_OPEN
□ Failure threshold
□ Slow-call threshold
□ Sliding window
□ Minimum calls
□ Fallback
□ Graceful degradation
□ Fail fast
□ Bulkhead
□ Semaphore bulkhead
□ Thread-pool bulkhead
□ Rate limiter
□ Timeout vs circuit breaker
□ Retry vs circuit breaker
□ Bulkhead vs circuit breaker
□ Resilience4j
□ Resilience4j annotations
□ TimeLimiter
□ Configuration
□ Retry ownership
□ Cascading failure
□ Connection pool exhaustion
□ Thread pool exhaustion
□ Monitoring
□ Fault injection
□ Recovery testing
□ Checkout resilience scenario
```

---

# 105. The Interviewer's Real Test

When an interviewer asks:

> "What happens if Payment Service goes down?"

Don't just say:

```text
Use Circuit Breaker.
```

Walk through the failure:

```text
Payment becomes unavailable
        ↓
Timeout prevents indefinite waiting
        ↓
Bounded retry handles transient failures
        ↓
Circuit breaker opens after repeated failures
        ↓
Calls fail fast
        ↓
Bulkhead protects service resources
        ↓
Fallback / async workflow if business rules allow
        ↓
Metrics + alerts detect the incident
        ↓
Half-open probes detect recovery
        ↓
Circuit closes
```

That demonstrates actual production-level reasoning.

# Microservices — Distributed Logging, Monitoring & Tracing

This file covers observability for Java/Spring Boot microservices.

Core topics:

```text
Observability
Logging
Structured logging
Centralized logging
ELK Stack
Correlation ID
Trace ID
Metrics
Monitoring
Health checks
Distributed tracing
OpenTelemetry
Micrometer
Prometheus
Grafana
Alerts
SLO / SLA / SLI
Golden Signals
Troubleshooting
Production scenarios
Interview questions
```

---

# 1. What Is Observability?

Observability is the ability to understand the internal state and behavior of a system from its external outputs.

The three classic pillars are:

```text
Logs
Metrics
Traces
```

A production microservices system should use all three.

---

# 2. Logs

Logs record individual events or messages.

Example:

```text
Order 101 created successfully
```

Logs are useful for:

```text
Debugging
Error investigation
Audit information
Operational diagnosis
```

---

# 3. Metrics

Metrics are numerical measurements collected over time.

Examples:

```text
HTTP request count
Request latency
CPU usage
Memory usage
Error rate
Kafka consumer lag
Database connection pool usage
```

Metrics are especially useful for dashboards and alerting.

---

# 4. Traces

A trace follows a request or business operation across multiple services.

Example:

```text
Client
 ↓
API Gateway
 ↓
Order Service
 ↓
Inventory Service
 ↓
Payment Service
```

A distributed trace connects these operations into one request journey.

---

# 5. Logs vs Metrics vs Traces

| Type | Best for |
|---|---|
| Logs | Detailed events |
| Metrics | Trends, health, alerting |
| Traces | Request flow across services |

Use them together.

---

# 6. Why Observability Matters in Microservices

In a monolith:

```text
Request
 ↓
One application
```

Debugging may be relatively straightforward.

In microservices:

```text
Gateway
 ↓
Order
 ↓
Inventory
 ↓
Payment
 ↓
Notification
```

A failure can occur anywhere.

Without observability:

```text
"Something is slow."
```

With observability:

```text
Order Service = 120ms
Inventory = 80ms
Payment = 4.2s
```

Now the bottleneck is visible.

---

# 7. Centralized Logging

Instead of storing logs independently on every machine:

```text
Service A → local logs
Service B → local logs
Service C → local logs
```

centralize them:

```text
Service A ─┐
Service B ─┼→ Log Pipeline → Central Store → Search/UI
Service C ─┘
```

This makes production investigation much easier.

---

# 8. ELK Stack

A common centralized logging stack is:

```text
Elasticsearch
Logstash
Kibana
```

ELK:

```text
E = Elasticsearch
L = Logstash
K = Kibana
```

---

# 9. Elasticsearch

Elasticsearch is commonly used to index and search logs and other documents.

Example query concepts:

```text
service = order-service
level = ERROR
traceId = abc123
```

This makes searching across distributed logs much easier.

---

# 10. Logstash

Logstash can collect, transform and route data.

Conceptually:

```text
Application logs
 ↓
Logstash
 ↓
Transform / Parse
 ↓
Elasticsearch
```

In modern architectures, other agents such as Beats or Fluent Bit may also be used.

---

# 11. Kibana

Kibana provides visualization and search capabilities for Elasticsearch data.

Example:

```text
Errors in last 15 minutes
Top failing endpoints
Requests by service
Logs for traceId
```

---

# 12. Structured Logging

Avoid logs like:

```text
Order failed for user 101
```

Prefer structured data such as:

```json
{
  "timestamp": "2026-08-21T10:30:00Z",
  "level": "ERROR",
  "service": "order-service",
  "orderId": "101",
  "userId": "20",
  "traceId": "abc123",
  "message": "Order creation failed"
}
```

Structured logs are easier to search and analyze.

---

# 13. Why JSON Logs?

JSON or another structured format allows systems to query fields directly.

Instead of searching text:

```text
"order 101"
```

you can query:

```text
orderId = 101
```

This is much more useful at scale.

---

# 14. Log Levels

Common levels:

```text
TRACE
DEBUG
INFO
WARN
ERROR
```

Use appropriate levels.

---

# 15. TRACE

Very detailed diagnostic information.

Usually not enabled broadly in production because of volume.

---

# 16. DEBUG

Useful for development and detailed troubleshooting.

Production DEBUG logging should be controlled because it can generate large volumes.

---

# 17. INFO

Normal significant application events.

Examples:

```text
Application started
Order created
Payment completed
```

Don't log every tiny internal operation at INFO.

---

# 18. WARN

Something unexpected happened, but the application may continue.

Example:

```text
Cache unavailable, falling back to database
```

---

# 19. ERROR

A failure that requires attention.

Example:

```text
Payment processing failed
Database connection failure
```

---

# 20. What Not to Log

Never casually log:

```text
Passwords
JWT secrets
API keys
Credit card data
Database credentials
Private tokens
Sensitive personal data
```

Production logs are valuable but can also become a security risk.

---

# 21. Correlation ID

A correlation ID identifies a logical request/workflow across services.

Example:

```text
Request
correlationId = req-123
```

Then:

```text
Order Service → req-123
Inventory → req-123
Payment → req-123
```

You can search all related logs.

---

# 22. Trace ID

A trace ID identifies a distributed trace.

Example:

```text
traceId = abc123
```

Every span belonging to that trace shares the trace ID.

---

# 23. Trace ID vs Correlation ID

They can be used similarly, but they are conceptually different.

```text
Trace ID
→ identifies distributed trace

Correlation ID
→ application-level identifier used to correlate related operations
```

Modern tracing systems can often provide the trace context needed for correlation.

---

# 24. Span

A span represents one unit of work within a trace.

Example:

```text
Trace: checkout-123

Span 1 → API Gateway
Span 2 → Order Service
Span 3 → Inventory Service
Span 4 → Payment Service
```

A span usually has:

```text
Operation name
Start time
Duration
Attributes
Status
Parent relationship
```

---

# 25. Trace Structure

Conceptually:

```text
Trace
 |
 +-- Gateway span
       |
       +-- Order span
             |
             +-- Inventory span
             |
             +-- Payment span
```

This creates a distributed call tree.

---

# 26. Distributed Tracing

Distributed tracing follows requests across service boundaries.

Useful for:

```text
Latency analysis
Failure localization
Dependency analysis
Request flow
Performance troubleshooting
```

---

# 27. OpenTelemetry

OpenTelemetry is an open standard/tooling ecosystem for collecting:

```text
Traces
Metrics
Logs
```

It can export telemetry to supported backends.

Think:

```text
Application
 ↓
OpenTelemetry instrumentation
 ↓
Collector / Exporter
 ↓
Observability backend
```

---

# 28. OpenTelemetry Collector

The Collector can receive telemetry, process it and export it to one or more backends.

Conceptually:

```text
Services
  ↓
OTel Collector
  ↓
+---------+---------+
↓         ↓         ↓
Tracing  Metrics   Logs
Backend  Backend   Backend
```

This can reduce direct coupling between applications and observability vendors.

---

# 29. Micrometer

Micrometer provides a vendor-neutral metrics instrumentation facade commonly used in Spring applications.

Spring Boot can integrate Micrometer with monitoring systems such as Prometheus.

---

# 30. Prometheus

Prometheus is commonly used for metrics collection and querying.

Typical architecture:

```text
Spring Boot
 ↓
Micrometer
 ↓
Prometheus
 ↓
Grafana
```

---

# 31. Grafana

Grafana is commonly used to visualize metrics and build dashboards.

Example dashboard:

```text
Request rate
Error rate
p95 latency
CPU
Memory
JVM
DB pool
Kafka lag
```

---

# 32. Actuator

Spring Boot Actuator provides production-oriented endpoints for application monitoring and management.

Common endpoints include:

```text
/actuator/health
/actuator/metrics
```

The exact exposed endpoints depend on configuration.

---

# 33. Health Check

A health endpoint helps determine whether an application is healthy.

Example:

```text
/actuator/health
```

Possible response concept:

```json
{
  "status": "UP"
}
```

---

# 34. Liveness vs Readiness

Very important in Kubernetes/container environments.

### Liveness

Answers:

> "Should this application process be restarted?"

### Readiness

Answers:

> "Is this instance ready to receive traffic?"

They solve different problems.

---

# 35. Liveness Example

Suppose:

```text
Application deadlocked
```

A liveness probe may fail.

The orchestrator can restart the instance.

---

# 36. Readiness Example

Suppose:

```text
Application started
but database connection initialization is incomplete
```

Readiness can remain false.

The service should not receive normal traffic yet.

---

# 37. Don't Confuse Them

Bad design:

```text
Database temporarily unavailable
 ↓
Liveness fails
 ↓
Container repeatedly restarts
```

This can create a restart loop.

Readiness and liveness should reflect different purposes.

---

# 38. Metrics Types

Common metric types:

```text
Counter
Gauge
Timer
Histogram
```

---

# 39. Counter

A counter generally increases over time.

Examples:

```text
orders_created_total
http_requests_total
errors_total
```

It may reset when the process restarts.

---

# 40. Gauge

A gauge represents a current value.

Examples:

```text
JVM memory used
Queue size
Active connections
CPU usage
```

It can increase or decrease.

---

# 41. Timer

A timer measures durations and can also provide count-related information depending on the implementation.

Examples:

```text
HTTP request duration
Database query duration
Payment processing time
```

---

# 42. Histogram

A histogram groups observed values into buckets.

Useful for:

```text
Latency distributions
Request durations
Response sizes
```

This supports percentile-style analysis depending on the metrics backend/configuration.

---

# 43. Percentiles

Common latency metrics:

```text
p50
p90
p95
p99
```

Example:

```text
p95 = 500 ms
```

means approximately 95% of measured requests were at or below 500 ms, subject to the metric calculation method.

---

# 44. Why p99 Matters

Average latency can hide slow requests.

Example:

```text
99 requests = 50 ms
1 request = 10 seconds
```

Average:

```text
~149.5 ms
```

but some users experience 10 seconds.

Percentiles reveal tail latency.

---

# 45. Golden Signals

A common observability framework uses:

```text
Latency
Traffic
Errors
Saturation
```

These are called the four golden signals.

---

# 46. Latency

How long requests take.

Example:

```text
p50 = 50ms
p95 = 200ms
p99 = 500ms
```

---

# 47. Traffic

How much demand the system receives.

Examples:

```text
Requests/sec
Orders/minute
Kafka records/sec
```

---

# 48. Errors

How often requests fail.

Examples:

```text
HTTP 5xx
Payment failures
Kafka consumer errors
Database failures
```

---

# 49. Saturation

How "full" a resource is.

Examples:

```text
CPU
Memory
Connection pool
Disk
Kafka consumer capacity
Thread pool
```

---

# 50. SLI

SLI means:

```text
Service Level Indicator
```

It is a measurement of service behavior.

Example:

```text
Percentage of successful requests
```

---

# 51. SLO

SLO means:

```text
Service Level Objective
```

It is the target.

Example:

```text
99.9% successful requests
```

---

# 52. SLA

SLA means:

```text
Service Level Agreement
```

A formal commitment, often with business/customer implications.

Simple relationship:

```text
SLI → measurement
SLO → target
SLA → agreement
```

---

# 53. Alerting

Don't alert on every log.

Good alerts indicate actionable problems.

Examples:

```text
Error rate > threshold
p99 latency too high
Kafka lag growing
DB connection pool exhausted
Disk nearly full
Service unavailable
```

---

# 54. Bad Alert

```text
CPU > 70%
```

by itself may not indicate a real problem.

A service can be healthy at 80% CPU.

Alerts should reflect user impact or meaningful system risk where possible.

---

# 55. Alert Fatigue

If engineers receive:

```text
100 alerts/day
```

many will eventually be ignored.

Better:

```text
Fewer
Actionable
Prioritized
Well-defined
```

alerts.

---

# 56. Log Correlation Example

Request:

```text
POST /orders
```

Correlation:

```text
traceId = abc123
```

Logs:

```text
gateway      abc123
order        abc123
inventory    abc123
payment      abc123
```

Search:

```text
traceId = abc123
```

Now the complete flow can be investigated.

---

# 57. Logging MDC

In Java, SLF4J/Logback's MDC can store contextual values for logs.

Conceptually:

```java
MDC.put("correlationId", correlationId);
```

Then log statements can automatically include that context.

Always clean up request-scoped MDC values appropriately in thread-pool environments.

---

# 58. Spring Boot Logging

Spring Boot commonly uses:

```text
SLF4J
Logback
```

Example:

```java
private static final Logger log =
        LoggerFactory.getLogger(OrderService.class);

log.info("Order created: {}", orderId);
log.error("Payment failed for order {}", orderId, exception);
```

---

# 59. Don't Log Huge Objects

Avoid:

```java
log.info("request={}", entireRequestObject);
```

if the object is:

```text
Huge
Sensitive
Expensive to serialize
```

Log useful identifiers and relevant fields.

---

# 60. Exception Logging

Bad:

```java
log.error("Something went wrong");
```

Better:

```java
log.error(
    "Payment failed for orderId={}",
    orderId,
    exception
);
```

You want:

```text
Context
Exception
Identifiers
```

---

# 61. Error Log Deduplication

A retry loop can generate thousands of identical errors.

Observability should help distinguish:

```text
One failure
```

from:

```text
10,000 repeated failures
```

Use:

```text
Metrics
Rate limiting
Sampling
Structured fields
```

where appropriate.

---

# 62. Sampling

Distributed tracing can generate huge amounts of data.

Sampling reduces the amount collected.

Example:

```text
100,000 requests
 ↓
sample 10%
 ↓
10,000 traces
```

For critical errors, systems may use special sampling strategies to retain more useful traces.

---

# 63. Head vs Tail Sampling

### Head sampling

Decision is made near the beginning of the trace.

### Tail sampling

Decision can be made after observing more of the trace.

Tail sampling can retain:

```text
Errors
Slow traces
Interesting requests
```

more intelligently, at the cost of more infrastructure complexity.

---

# 64. Trace Context Propagation

When Service A calls Service B:

```text
Service A
 ↓
HTTP request + trace context
 ↓
Service B
```

Service B continues the same distributed trace.

This is essential for cross-service tracing.

---

# 65. Async Trace Propagation

With Kafka:

```text
Service A
 ↓
Kafka event
 ↓
Service B
```

trace/correlation context can be propagated through message headers according to the tracing setup.

This allows event-driven workflows to remain observable.

---

# 66. Database Metrics

Monitor:

```text
Connection pool size
Active connections
Idle connections
Wait time
Query latency
Slow queries
Errors
```

For HikariCP:

```text
Active
Idle
Pending
Max
Min
```

can be especially useful.

---

# 67. JVM Metrics

Java applications should monitor:

```text
Heap
Non-heap
GC
Threads
CPU
Class loading
JVM uptime
```

---

# 68. GC Problems

If garbage collection becomes excessive:

```text
Latency increases
CPU increases
Throughput decreases
```

Metrics and traces can help correlate GC pauses with request latency.

---

# 69. Thread Pool Monitoring

Monitor:

```text
Active threads
Queue size
Rejected tasks
Pool size
Task duration
```

A full thread pool can cause:

```text
Request queueing
Timeouts
5xx errors
```

---

# 70. Connection Pool Exhaustion

Example:

```text
DB pool max = 20
20 connections busy
```

New requests:

```text
wait
```

Eventually:

```text
timeout
```

Metrics can reveal this quickly.

---

# 71. Kafka Observability

Important metrics:

```text
Consumer lag
Records consumed
Processing latency
Error count
Retry count
DLT records
Rebalances
```

---

# 72. Redis Observability

Monitor:

```text
Hit ratio
Memory
Evictions
Command latency
Connection count
Errors
Hot keys
Replication health
```

---

# 73. API Gateway Observability

Monitor:

```text
Request count
Latency
4xx
5xx
Rate limits
Upstream failures
Timeouts
```

The gateway is often a useful first place to determine whether a problem is global or service-specific.

---

# 74. Dependency Health

Don't monitor only your service.

Monitor:

```text
Database
Redis
Kafka
External APIs
Other services
```

A service can be healthy internally while its dependencies are failing.

---

# 75. Dependency Map

A useful observability dashboard can show:

```text
Gateway
  |
  +--> Order
          |
          +--> Inventory
          |
          +--> Payment
          |
          +--> Redis
          |
          +--> MySQL
```

This makes failures easier to localize.

---

# 76. Timeout Metrics

Track:

```text
Request timeouts
Connection timeouts
Read timeouts
Circuit breaker opens
```

A timeout often indicates:

```text
Dependency latency
Network problem
Resource saturation
```

---

# 77. Monitoring vs Observability

Monitoring asks:

> "Is the system behaving according to known expectations?"

Observability helps answer:

> "Why is the system behaving this way?"

Both are important.

---

# 78. Health Check vs Business Monitoring

Health:

```text
Service process is alive
```

Business:

```text
Orders completed per minute
Payment success rate
Checkout conversion
```

A service can be:

```text
UP
```

while business functionality is broken.

---

# 79. Business Metrics

Examples:

```text
Orders created
Orders cancelled
Payment success rate
Cart abandonment
Average order value
Inventory reservation failures
```

Technical metrics alone don't show business impact.

---

# 80. Error Budget

If SLO is:

```text
99.9% availability
```

allowed failure budget is approximately:

```text
0.1%
```

This is the error budget.

Teams can use it to balance:

```text
Reliability
vs
Release velocity
```

---

# 81. Incident Investigation Flow

When users report:

> "Checkout is slow."

Start with:

```text
1. Traffic
2. Error rate
3. Latency
4. Saturation
5. Dependency health
6. Traces
7. Logs
```

---

# 82. Step 1 — Check Traffic

Ask:

```text
Did traffic suddenly increase?
```

Example:

```text
100 req/s
→
1,000 req/s
```

Maybe the system is overloaded.

---

# 83. Step 2 — Check Errors

Look at:

```text
4xx
5xx
Dependency errors
Timeouts
```

If errors increased at the same time as latency, investigate the failing dependency.

---

# 84. Step 3 — Check Latency

Compare:

```text
p50
p95
p99
```

If p99 suddenly increased:

```text
Tail latency problem
```

---

# 85. Step 4 — Check Saturation

Look at:

```text
CPU
Memory
Threads
DB pool
Redis
Kafka lag
```

Find the constrained resource.

---

# 86. Step 5 — Trace a Slow Request

Example:

```text
Gateway = 50ms
Order = 100ms
Inventory = 120ms
Payment = 4,000ms
```

Root cause candidate:

```text
Payment dependency
```

---

# 87. Step 6 — Search Logs

Use:

```text
traceId
orderId
requestId
```

Search for:

```text
ERROR
WARN
timeout
exception
```

Now inspect the exact failure.

---

# 88. Root Cause vs Symptom

Example:

```text
API latency ↑
```

is a symptom.

Actual cause:

```text
DB connection pool exhausted
```

or:

```text
Payment provider latency
```

Observability helps connect them.

---

# 89. Production Scenario

### "One endpoint suddenly becomes slow."

Check:

```text
Endpoint latency
Dependency latency
DB queries
Cache hit ratio
Thread pool
Connection pool
GC
Recent deployments
Traffic
```

---

# 90. Production Scenario

### "5xx errors increased after deployment."

Check:

```text
Deployment timestamp
Error logs
Trace samples
Exception types
Dependency failures
Configuration changes
Database migrations
```

Compare:

```text
Before deployment
vs
After deployment
```

---

# 91. Production Scenario

### "Database CPU is 95%."

Don't immediately increase DB capacity.

Check:

```text
Query latency
Slow queries
Traffic increase
Missing indexes
N+1 queries
Cache hit ratio
Connection pool
Recent code changes
```

---

# 92. Production Scenario

### "Kafka lag keeps increasing."

Check:

```text
Consumer processing time
Partition count
Consumer count
Downstream dependency latency
Consumer errors
Rebalances
Traffic increase
```

---

# 93. Production Scenario

### "Redis memory keeps increasing."

Check:

```text
TTL
Key count
Evictions
Large values
Unexpected key patterns
Memory fragmentation
Hot keys
Cache leaks
```

---

# 94. Production Scenario

### "Everything looks healthy but users report failures."

Check business metrics:

```text
Payment success rate
Order completion
Checkout success
External provider response
```

Technical health endpoints may still show:

```text
UP
```

while business operations fail.

---

# 95. Alert Design

Good alert:

```text
Payment failure rate > 10% for 5 minutes
```

Potentially better than:

```text
One payment failed
```

because it indicates a systemic issue.

---

# 96. Alert Severity

Possible levels:

```text
INFO
WARNING
CRITICAL
```

Severity should reflect:

```text
User impact
Business impact
Urgency
```

---

# 97. Runbooks

A good production alert should link to a runbook.

Example:

```text
Alert:
High Kafka Consumer Lag

Runbook:
1. Check consumer health
2. Check downstream DB
3. Check traffic
4. Check consumer errors
5. Scale consumers if appropriate
6. Investigate partition distribution
```

---

# 98. Observability Cardinality

Be careful with metric labels.

Bad:

```text
userId
orderId
requestId
```

as high-cardinality metric labels.

Millions of unique values can make the metrics backend expensive or unhealthy.

Use high-cardinality identifiers primarily in logs/traces rather than unrestricted metric labels.

---

# 99. Logs vs Metrics for IDs

Good:

```text
Log:
orderId=123456789
```

Potentially bad metric:

```text
http_requests_total{orderId="123456789"}
```

because there may be millions of unique order IDs.

---

# 100. Trace Sampling Strategy

Keep more traces for:

```text
Errors
High latency
Important business operations
Specific customers/tenants when appropriate and safe
```

Sample routine successful traffic more aggressively when volume is high.

---

# 101. Observability Costs

Telemetry consumes:

```text
CPU
Memory
Network
Storage
Money
```

Don't log everything at DEBUG forever.

Design:

```text
Retention
Sampling
Log levels
Metric cardinality
```

carefully.

---

# 102. Sensitive Data in Observability

Observability systems often have broad access.

Avoid exposing:

```text
Passwords
Tokens
Payment details
Secrets
Unnecessary PII
```

Use:

```text
Masking
Redaction
Access controls
Encryption
Retention policies
```

---

# 103. Correlation ID Middleware

A common Spring Boot approach:

```text
Incoming request
 ↓
Filter/interceptor
 ↓
Read or generate correlation ID
 ↓
Put in MDC
 ↓
Continue request
 ↓
Response header
```

This makes logs consistently searchable.

---

# 104. Example Correlation Flow

```text
POST /orders
X-Correlation-ID: req-123
```

Then:

```text
Gateway
req-123

Order Service
req-123

Inventory Service
req-123

Payment Service
req-123
```

---

# 105. Trace Context

With distributed tracing, the propagation mechanism carries trace information between services.

Conceptually:

```text
Service A
 ↓
trace context
 ↓
Service B
```

This allows spans to be connected.

---

# 106. OpenTelemetry Architecture

Conceptually:

```text
Spring Boot Service
       |
       ↓
OpenTelemetry SDK/Instrumentation
       |
       ↓
OTel Collector
       |
       +----> Tracing backend
       +----> Metrics backend
       +----> Log backend
```

The exact deployment can vary.

---

# 107. Observability in Kubernetes

Common components:

```text
Application
Actuator
Prometheus
Grafana
OpenTelemetry
Centralized logs
Kubernetes probes
```

Typical flow:

```text
Pod
 ↓
/actuator/health
 ↓
Readiness/Liveness

Metrics
 ↓
Prometheus
 ↓
Grafana

Traces
 ↓
OpenTelemetry
 ↓
Tracing backend

Logs
 ↓
Log collector
 ↓
Central log system
```

---

# 108. Deployment Observability

After deployment monitor:

```text
Error rate
Latency
Traffic
CPU
Memory
DB
Kafka
Redis
Business metrics
```

Compare:

```text
Before deployment
vs
After deployment
```

---

# 109. Canary Deployment

Observability is especially important during canary releases.

Example:

```text
95% → old version
5%  → new version
```

Compare:

```text
Error rate
Latency
Business success rate
```

If new version is worse:

```text
Stop rollout
```

---

# 110. Production Dashboard

A useful service dashboard might contain:

```text
Request rate
p50 latency
p95 latency
p99 latency
4xx rate
5xx rate
CPU
Memory
JVM GC
Thread pool
DB pool
Redis
Kafka lag
```

---

# 111. Service-Level Dashboard

For Order Service:

```text
Orders/minute
Order success rate
Order failure rate
Checkout latency
Payment dependency latency
Inventory dependency latency
DB latency
Kafka lag
```

This combines technical and business visibility.

---

# 112. Interview Question

### "What are the three pillars of observability?"

Answer:

> "Logs, metrics and distributed traces. Logs provide detailed events, metrics provide numerical trends and alerting signals, and traces show how a request flows across services."

---

# 113. Interview Question

### "What is distributed tracing?"

Answer:

> "Distributed tracing follows a request across multiple services using a trace ID and spans. It helps identify where latency or failures occur in a distributed workflow."

---

# 114. Interview Question

### "What is a correlation ID?"

Answer:

> "It's an identifier used to correlate logs and operations belonging to the same logical request or workflow. I propagate it across service boundaries so production issues can be investigated end to end."

---

# 115. Interview Question

### "What is OpenTelemetry?"

Answer:

> "OpenTelemetry is an open-source observability framework and standard for generating, collecting and exporting telemetry such as traces, metrics and logs. It helps instrument applications without tightly coupling them to one observability vendor."

---

# 116. Interview Question

### "Prometheus vs Grafana?"

Answer:

> "Prometheus is primarily a metrics collection and querying system, while Grafana is primarily used to visualize metrics and build dashboards. They are commonly used together."

---

# 117. Interview Question

### "What is ELK?"

Answer:

> "ELK stands for Elasticsearch, Logstash and Kibana. Elasticsearch stores and searches indexed data, Logstash can collect and transform logs, and Kibana provides search and visualization."

---

# 118. Interview Question

### "What is the difference between liveness and readiness?"

Answer:

> "Liveness determines whether an instance should be restarted, while readiness determines whether the instance should receive traffic. A temporary dependency problem should not automatically cause unnecessary restarts."

---

# 119. Interview Question

### "How would you troubleshoot a slow API?"

Answer:

> "I'd first check traffic, error rate, p95/p99 latency and resource saturation. Then I'd inspect distributed traces to identify the slow service or dependency, followed by logs using the trace or correlation ID. I'd also check database, Redis, Kafka and external dependency metrics and recent deployments."

---

# 120. Interview Question

### "Why are metrics better than logs for alerting?"

Answer:

> "Metrics are compact numerical time series that are easier to aggregate and evaluate against thresholds. Logs provide richer detail for investigation. I would generally use metrics for alerts and logs/traces for diagnosis."

---

# 121. Interview Question

### "Why not put userId as a Prometheus label?"

Answer:

> "Because userId can have extremely high cardinality. Millions of unique label values can create a huge number of time series and put significant load on the metrics system. I'd keep such identifiers in logs or traces instead."

---

# 122. Interview Scenario

### "API p99 increased from 300 ms to 5 seconds."

Approach:

```text
Check deployment
 ↓
Check traffic
 ↓
Check error rate
 ↓
Inspect traces
 ↓
Find slow span
 ↓
Check dependency metrics
 ↓
Inspect logs
 ↓
Identify root cause
```

Possible root causes:

```text
DB
Redis
External API
Thread pool
GC
Network
Kafka
```

---

# 123. Interview Scenario

### "All services are UP but checkout is failing."

Answer:

> "I'd check business metrics rather than relying only on health checks. Then I'd trace a failed checkout and inspect the payment/inventory dependencies and correlated logs. A service can report UP while a specific business capability is failing."

---

# 124. Interview Scenario

### "Logs show thousands of errors but you don't know which requests are related."

Answer:

> "I'd introduce structured logging with traceId/correlationId and key business identifiers. Then I can search the entire request flow across services instead of manually correlating timestamps."

---

# 125. Interview Scenario

### "Your logging volume doubled after deployment."

Check:

```text
Log level
New logging statements
Exception loops
Retry loops
Duplicate logging
Request payload logging
```

Then:

```text
Reduce unnecessary logs
Change levels
Fix repeated failures
Use sampling where appropriate
```

---

# 126. Interview Scenario

### "Kafka lag is high but CPU is only 30%."

Don't assume the consumer needs more CPU.

Check:

```text
Downstream DB latency
External API latency
Thread pool
Connection pool
Partition assignment
Consumer errors
Rebalances
```

The bottleneck may be I/O rather than CPU.

---

# 127. Interview Scenario

### "Database latency increased after a new release."

Check:

```text
New queries
Query plans
Indexes
N+1 behavior
Connection pool
Traffic
Transaction duration
Lock contention
```

Use traces to identify which API calls are waiting on DB operations.

---

# 128. Common Observability Mistakes

```text
❌ Logging passwords/tokens
❌ No correlation IDs
❌ Only monitoring CPU
❌ No distributed tracing
❌ High-cardinality metrics
❌ Alerting on every log
❌ No business metrics
❌ No readiness/liveness distinction
❌ No dashboards
❌ No runbooks
❌ Unlimited DEBUG logs
❌ No telemetry retention strategy
```

---

# 129. Practical Observability Checklist

For every production service, ask:

```text
1. Can I search its logs?
2. Are logs structured?
3. Can I correlate a request across services?
4. Do I have request rate?
5. Do I have error rate?
6. Do I have p95/p99 latency?
7. Do I have resource saturation?
8. Can I trace a slow request?
9. Are dependencies monitored?
10. Do I have readiness/liveness?
11. Are alerts actionable?
12. Do alerts have runbooks?
13. Are sensitive values redacted?
14. Is metric cardinality controlled?
15. Is telemetry cost controlled?
```

---

# 130. Final Mental Model

Remember:

```text
Logs
→ What happened?

Metrics
→ How often/how much?

Traces
→ Where did the request spend time?

Correlation ID
→ Which operations belong together?

Trace ID
→ Which spans belong to this distributed trace?

Span
→ One unit of work

Prometheus
→ Metrics collection/querying

Grafana
→ Visualization

ELK
→ Centralized log search/analysis

OpenTelemetry
→ Telemetry instrumentation/collection

Actuator
→ Spring Boot operational endpoints

Liveness
→ Should this instance restart?

Readiness
→ Should this instance receive traffic?
```

---

# 131. Final Interview Answer

If asked:

> "How do you monitor and troubleshoot a Spring Boot microservices application in production?"

Use:

> "I'd use centralized structured logging, metrics and distributed tracing. For Spring Boot I'd use Actuator and Micrometer for application metrics, Prometheus and Grafana for metrics and dashboards, and OpenTelemetry for distributed tracing. Logs would include correlation or trace IDs so I can follow a request across services. For an incident I'd start with traffic, errors, latency and saturation, then use traces to locate the slow or failing dependency and logs to identify the exact exception. I'd also monitor business metrics such as order and payment success rates because technical health alone doesn't guarantee the business flow is working."

---

# 132. Revision Checklist

```text
□ Observability
□ Logs
□ Metrics
□ Traces
□ Centralized logging
□ ELK
□ Elasticsearch
□ Logstash
□ Kibana
□ Structured logging
□ Log levels
□ Sensitive data
□ Correlation ID
□ Trace ID
□ Span
□ Distributed tracing
□ OpenTelemetry
□ OTel Collector
□ Micrometer
□ Prometheus
□ Grafana
□ Actuator
□ Health checks
□ Liveness
□ Readiness
□ Counters
□ Gauges
□ Timers
□ Histograms
□ Percentiles
□ p50
□ p95
□ p99
□ Golden signals
□ SLI
□ SLO
□ SLA
□ Error budget
□ Alerting
□ Alert fatigue
□ Runbooks
□ Sampling
□ Trace propagation
□ MDC
□ JVM monitoring
□ HikariCP monitoring
□ Kafka monitoring
□ Redis monitoring
□ Dependency monitoring
□ Business metrics
□ High-cardinality metrics
□ Incident investigation
□ Production troubleshooting
□ Canary monitoring
```

---

# 133. The Interviewer's Real Test

If asked:

> "The checkout API suddenly became slow. How would you troubleshoot it?"

Don't say:

```text
Check logs.
```

Give a systematic answer:

```text
Checkout latency ↑
        |
        ↓
Check traffic
        |
        ↓
Check error rate
        |
        ↓
Check p95/p99
        |
        ↓
Check saturation
        |
        ↓
Trace a slow request
        |
        +---- Order Service
        |
        +---- Inventory Service
        |
        +---- Payment Service
        |
        +---- Database
        |
        +---- Redis
        |
        ↓
Find slow span
        |
        ↓
Search correlated logs
        |
        ↓
Check recent deployment/config change
        |
        ↓
Identify root cause
        |
        ↓
Mitigate
        |
        ↓
Verify recovery
```

The key interview lesson is:

> **Don't troubleshoot microservices by looking at one service in isolation. Follow the request across the entire system.**

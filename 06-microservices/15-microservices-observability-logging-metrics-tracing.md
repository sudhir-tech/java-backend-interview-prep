# Microservices — Observability: Logging, Metrics, Monitoring & Distributed Tracing

This file covers how to understand what is happening inside a distributed microservices system.

Core topics:

```text
Observability
Logging
Structured Logging
Log Levels
Correlation ID
Trace ID
Metrics
Counters
Gauges
Histograms
Percentiles
RED Method
USE Method
Monitoring
Alerting
Distributed Tracing
Spans
OpenTelemetry
Prometheus
Grafana
ELK Stack
Centralized Logging
Health Checks
Actuator
SLO
SLI
SLA
Error Budget
Production Debugging
Interview Questions
```

---

# 1. What Is Observability?

Observability is the ability to understand a system's internal state by examining the information it produces.

The three traditional pillars are:

```text
Logs
Metrics
Traces
```

In modern systems, profiling and continuous debugging can also complement these signals.

---

# 2. Why Observability Matters in Microservices

Consider:

```text
Client
 ↓
Gateway
 ↓
Order Service
 ↓
Inventory Service
 ↓
Payment Service
```

A request may cross several services.

If the request fails:

```text
Which service failed?
Why?
How long did each service take?
Was there a timeout?
Was the database slow?
```

Observability helps answer these questions.

---

# 3. Logs

Logs record discrete events.

Example:

```text
2026-08-21 10:20:15
Order 500 created
```

Useful for:

```text
Detailed debugging
Errors
Business events
Security events
Operational investigation
```

---

# 4. Metrics

Metrics are numerical measurements collected over time.

Examples:

```text
HTTP requests/sec
Error count
CPU usage
Memory usage
Request latency
Kafka consumer lag
Database connections
```

Metrics are excellent for:

```text
Dashboards
Alerts
Capacity planning
Trend analysis
```

---

# 5. Traces

A trace follows one request or distributed operation across services.

Example:

```text
Trace
 |
 +→ Gateway
 |
 +→ Order
 |
 +→ Inventory
 |
 +→ Payment
```

A trace contains multiple spans.

---

# 6. Span

A span represents one unit of work within a trace.

Example:

```text
Trace: abc123

Gateway Span
    |
Order Span
    |
Inventory Span
    |
Payment Span
```

Each span can contain:

```text
Start time
Duration
Service name
Operation
Attributes
Status
Events
```

---

# 7. Trace vs Span

Remember:

```text
Trace
→ Entire distributed request

Span
→ One operation inside the trace
```

---

# 8. Example Trace

```text
Trace ID = abc123

Gateway
  20ms
    ↓
Order Service
  80ms
    ↓
Inventory Service
  150ms
    ↓
Payment Service
  900ms
```

The trace immediately suggests:

```text
Payment is the slowest dependency.
```

---

# 9. Correlation ID

A correlation ID identifies a logical request across services.

Example:

```text
X-Correlation-ID: abc123
```

Propagate it:

```text
Gateway
 ↓ abc123
Order
 ↓ abc123
Inventory
 ↓ abc123
Payment
```

It makes logs easier to search.

---

# 10. Trace ID vs Correlation ID

They can serve similar troubleshooting purposes, but they are not necessarily identical.

Distributed tracing systems generally provide:

```text
Trace ID
Span ID
Trace context
```

A correlation ID may be an application-level identifier.

---

# 11. Structured Logging

Instead of:

```text
Order 500 created by user 101
```

use structured data:

```json
{
  "level": "INFO",
  "event": "ORDER_CREATED",
  "orderId": "500",
  "userId": "101",
  "traceId": "abc123"
}
```

Structured logs are easier to search and analyze.

---

# 12. Why Structured Logs?

They allow queries such as:

```text
orderId = 500
```

or:

```text
traceId = abc123
```

instead of parsing free-form text.

---

# 13. Log Levels

Common levels:

```text
TRACE
DEBUG
INFO
WARN
ERROR
```

Use them intentionally.

---

# 14. TRACE

Very detailed diagnostic information.

Usually:

```text
Disabled or restricted in production
```

unless temporarily needed.

---

# 15. DEBUG

Useful for developer-level troubleshooting.

Example:

```text
Calling inventory service
```

Avoid excessive DEBUG logging in production.

---

# 16. INFO

Normal important application events.

Example:

```text
Order created
Application started
Consumer connected
```

---

# 17. WARN

Something unexpected happened, but the application may continue.

Example:

```text
Cache unavailable, using database
```

---

# 18. ERROR

An operation failed.

Example:

```text
Payment processing failed
```

Include enough context to investigate the failure.

---

# 19. Don't Log Secrets

Never casually log:

```text
Passwords
Access tokens
Refresh tokens
Client secrets
Database passwords
Credit card data
Private keys
```

Sensitive data should be redacted or excluded.

---

# 20. Logging PII

Personally identifiable information should be handled carefully.

Examples:

```text
Email
Phone number
Address
Government identifiers
```

Only log what is necessary and follow organizational privacy requirements.

---

# 21. Centralized Logging

In microservices, logs are distributed:

```text
Order Service
 → Order logs

Payment Service
 → Payment logs

Inventory Service
 → Inventory logs
```

Searching every server manually is impractical.

Centralize logs.

---

# 22. ELK Stack

A common logging architecture:

```text
Application
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

Historically, ELK refers to:

```text
Elasticsearch
Logstash
Kibana
```

---

# 23. Elasticsearch

Stores and indexes log data for searching and analysis.

---

# 24. Logstash

Collects/processes/transforms logs before sending them onward.

Other lightweight collectors such as Beats or Fluent Bit are also commonly used.

---

# 25. Kibana

Provides visualization and search capabilities for Elasticsearch data.

Example:

```text
Search:
traceId:abc123
```

---

# 26. Centralized Logging Flow

```text
Order ──────┐
Inventory ──┼→ Log Collector → Elasticsearch → Kibana
Payment ────┘
```

---

# 27. Logging Architecture in Kubernetes

A common pattern:

```text
Container stdout/stderr
        ↓
Log Agent
        ↓
Central Log Platform
```

The exact tools vary.

Examples:

```text
Fluent Bit
Fluentd
Filebeat
OpenTelemetry Collector
```

---

# 28. Metrics

Metrics should answer:

```text
Is the service healthy?
How much traffic is it handling?
How many errors occur?
How slow are requests?
Are resources exhausted?
```

---

# 29. Counter

A counter only increases, except when reset.

Examples:

```text
HTTP requests
Errors
Orders created
Messages processed
```

---

# 30. Gauge

A gauge represents a current value that can go up or down.

Examples:

```text
CPU usage
Memory usage
Active connections
Queue depth
```

---

# 31. Histogram

A histogram records observations into buckets.

Useful for:

```text
Request latency
Response sizes
Processing duration
```

---

# 32. Why Histograms Matter

Suppose average latency is:

```text
100ms
```

That doesn't tell you whether:

```text
99% = 50ms
1% = 5 seconds
```

Latency distributions are often more useful.

---

# 33. Percentiles

Common percentiles:

```text
p50
p90
p95
p99
```

Meaning:

```text
p95 = 95% of observations are at or below this value
```

---

# 34. Average vs Percentile

Average:

```text
Overall arithmetic mean
```

p95:

```text
Useful for tail latency
```

For APIs, p95/p99 can reveal slow user experiences hidden by averages.

---

# 35. RED Method

For request-driven services:

```text
R = Rate
E = Errors
D = Duration
```

Example:

```text
Requests/sec
5xx/sec
p95 latency
```

---

# 36. USE Method

For resources:

```text
U = Utilization
S = Saturation
E = Errors
```

Example:

```text
CPU utilization
Thread pool saturation
Disk errors
```

---

# 37. RED vs USE

RED:

```text
Service/request perspective
```

USE:

```text
Resource perspective
```

Both are useful.

---

# 38. Important Application Metrics

For an API:

```text
Request rate
Error rate
p50 latency
p95 latency
p99 latency
Active requests
Timeouts
```

---

# 39. Database Metrics

Monitor:

```text
Connection pool usage
Active connections
Pending connections
Query latency
Slow queries
Connection errors
CPU
Disk
Replication lag
```

---

# 40. Kafka Metrics

Monitor:

```text
Consumer lag
Throughput
Producer errors
Consumer errors
Partition health
Under-replicated partitions
Processing latency
```

---

# 41. JVM Metrics

For Java services:

```text
Heap usage
Non-heap usage
GC activity
Thread count
CPU
Class loading
Memory pools
```

---

# 42. Spring Boot Actuator

Spring Boot Actuator provides production-oriented endpoints and application metrics.

Common endpoint:

```text
/actuator/health
```

Other endpoints can include:

```text
/actuator/metrics
/actuator/prometheus
```

depending on dependencies and configuration.

---

# 43. Actuator Health

Example:

```text
GET /actuator/health
```

Response may indicate:

```json
{
  "status": "UP"
}
```

Health checks can also include dependency status depending on configuration.

---

# 44. Liveness

Liveness answers:

> "Should this application instance be restarted?"

A liveness failure generally means the process is unhealthy enough that restarting it may help.

---

# 45. Readiness

Readiness answers:

> "Should this instance receive traffic?"

An instance can be:

```text
Alive
but
Not Ready
```

Example:

```text
Application starting
 ↓
Liveness = UP
Readiness = NOT READY
```

Traffic should wait until readiness succeeds.

---

# 46. Liveness vs Readiness

```text
Liveness
→ Restart decision

Readiness
→ Traffic routing decision
```

This distinction is very important in Kubernetes.

---

# 47. Startup Probe

For applications that take a long time to start, Kubernetes can use a startup probe.

Conceptually:

```text
Container starts
 ↓
Startup probe
 ↓
Application ready
 ↓
Readiness/liveness checks continue
```

This prevents aggressive liveness failures during slow startup.

---

# 48. Monitoring

Monitoring focuses on collecting and presenting operational signals.

Examples:

```text
CPU dashboard
API latency dashboard
Error dashboard
Kafka lag dashboard
Database dashboard
```

---

# 49. Alerting

Monitoring tells you:

```text
Something changed.
```

Alerting tells the team:

```text
This requires attention.
```

---

# 50. Good Alerts

Good alerts are:

```text
Actionable
Relevant
Stable
Meaningful
```

Bad:

```text
Alert on every single warning
```

This creates alert fatigue.

---

# 51. Alert Example

Bad:

```text
CPU > 70%
```

for every service, regardless of impact.

Better:

```text
High CPU
+
Sustained duration
+
Request latency increasing
```

Use signals that represent actual user/system impact.

---

# 52. SLI

SLI means:

```text
Service Level Indicator
```

It is a measured value representing service performance.

Examples:

```text
Successful request percentage
p95 latency
Availability
```

---

# 53. SLO

SLO means:

```text
Service Level Objective
```

It defines the target.

Example:

```text
99.9% successful requests
```

---

# 54. SLA

SLA means:

```text
Service Level Agreement
```

It is a formal commitment, often contractual.

---

# 55. SLI vs SLO vs SLA

```text
SLI
→ What we measure

SLO
→ What target we aim for

SLA
→ What we formally promise
```

---

# 56. Error Budget

If SLO is:

```text
99.9% availability
```

allowed failure is approximately:

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

# 57. Error Budget Concept

If reliability is excellent:

```text
Error budget available
 ↓
Can potentially release faster
```

If reliability is poor:

```text
Error budget exhausted
 ↓
Prioritize reliability work
```

---

# 58. Distributed Tracing

A trace follows an operation across services.

Example:

```text
Trace abc123

Gateway
 ↓
Order
 ↓
Inventory
 ↓
Payment
```

---

# 59. Span Hierarchy

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

This reveals where time is spent.

---

# 60. OpenTelemetry

OpenTelemetry is an open-source observability framework for collecting and exporting telemetry.

It supports:

```text
Traces
Metrics
Logs
```

and integrates with many backends.

---

# 61. OpenTelemetry Architecture

Conceptually:

```text
Application
   ↓
OpenTelemetry SDK/Instrumentation
   ↓
OpenTelemetry Collector
   ↓
Backend
```

---

# 62. OpenTelemetry Collector

The Collector can:

```text
Receive telemetry
Process telemetry
Batch data
Filter data
Export telemetry
```

It decouples applications from a specific backend.

---

# 63. Trace Context

Distributed tracing needs context propagation.

Common standards use headers such as:

```text
traceparent
```

A trace context can travel:

```text
HTTP
 ↓
Service
 ↓
Message
```

---

# 64. Trace Sampling

Tracing every request can be expensive at high traffic.

Sampling decides which traces to retain.

Examples:

```text
Always sample errors
Sample a percentage of normal traffic
Use adaptive strategies
```

The strategy should preserve useful debugging information.

---

# 65. Logs + Traces

A powerful pattern is including:

```text
traceId
spanId
```

in logs.

Then:

```text
Find trace
 ↓
Open span
 ↓
Open related logs
```

This dramatically improves debugging.

---

# 66. Metrics + Traces

Suppose:

```text
p95 latency increased
```

Metrics tell you:

```text
Something is wrong.
```

Trace data can show:

```text
Payment Service consumes 80% of request time.
```

Together they provide much stronger diagnosis.

---

# 67. Logs + Metrics + Traces

Think:

```text
Metrics
→ Detect

Traces
→ Locate

Logs
→ Explain
```

This is a useful interview mental model.

---

# 68. Golden Signals

Common service health signals:

```text
Latency
Traffic
Errors
Saturation
```

These are often called the four golden signals.

---

# 69. Saturation

Saturation measures how "full" a resource is.

Examples:

```text
CPU
Thread pool
Connection pool
Queue
Disk
```

High saturation often predicts failure.

---

# 70. Example: Thread Pool Saturation

```text
Thread pool size = 100
Active = 100
Queue = 500
```

The service is saturated.

Even if:

```text
CPU = 50%
```

the service may still be unable to handle more requests.

---

# 71. Example: DB Connection Saturation

```text
Max connections = 50
Active = 50
Pending = 200
```

This is a strong signal of database/resource contention.

---

# 72. Observability During Incident

Suppose:

```text
Users report slow checkout.
```

Start with:

```text
Metrics
```

Check:

```text
Request rate
Error rate
p95/p99 latency
```

Then:

```text
Trace
```

Find slow dependency.

Then:

```text
Logs
```

Find the exact failure/error.

---

# 73. Production Debugging Example

```text
Checkout latency ↑
       ↓
p95 = 4 sec
       ↓
Trace shows Payment = 3 sec
       ↓
Payment logs show DB timeout
       ↓
DB connection pool saturated
```

Root cause:

```text
Database/resource contention
```

not simply:

```text
Checkout API is slow.
```

---

# 74. Observability vs Monitoring

Monitoring:

```text
Known questions
```

Example:

```text
Is CPU high?
```

Observability:

```text
Ability to investigate unknown problems
```

Example:

```text
Why did checkout become slow only for one region?
```

Observability provides richer diagnostic context.

---

# 75. Health Checks Are Not Full Observability

Health:

```text
UP
```

doesn't mean:

```text
Everything is performing well.
```

A service can be:

```text
UP
but
p99 latency = 8 seconds
```

---

# 76. Business Metrics

Don't monitor only infrastructure.

Useful business metrics:

```text
Orders/minute
Payment success rate
Checkout conversion
Cart abandonment
Inventory reservation failures
```

Business metrics help identify user impact.

---

# 77. Custom Metrics

Example:

```text
payment_success_total
payment_failure_total
orders_created_total
inventory_reservation_failure_total
```

Use clear names and labels.

---

# 78. Metric Cardinality

Be careful with high-cardinality labels.

Bad:

```text
http_requests_total{
    userId="every-user"
}
```

Millions of unique values can create huge metric storage costs.

Better labels:

```text
method
route
status
service
```

with controlled cardinality.

---

# 79. Avoid Raw URL Labels

Bad:

```text
/path/orders/123
/path/orders/124
/path/orders/125
```

This creates many time series.

Better:

```text
route="/orders/{id}"
```

---

# 80. Prometheus

Prometheus is commonly used for:

```text
Metrics collection
Time-series storage
PromQL queries
Alerting integrations
```

---

# 81. Prometheus Pull Model

A common Prometheus architecture:

```text
Prometheus
    |
    | scrape
    ↓
Application /metrics
```

The application exposes metrics.

Prometheus periodically scrapes them.

---

# 82. Prometheus + Spring Boot

A Spring Boot service can expose Prometheus metrics through Actuator when the appropriate Micrometer Prometheus registry is configured.

Conceptually:

```text
Spring Boot
 ↓
Micrometer
 ↓
/actuator/prometheus
 ↓
Prometheus
```

---

# 83. Micrometer

Micrometer provides an instrumentation facade commonly used in Spring applications.

Conceptually:

```text
Application
 ↓
Micrometer
 ↓
Prometheus / other monitoring systems
```

This reduces vendor-specific instrumentation.

---

# 84. Grafana

Grafana is commonly used for:

```text
Dashboards
Visualization
Metrics exploration
Alerts
```

Example:

```text
Prometheus
 ↓
Grafana
 ↓
Dashboard
```

---

# 85. Prometheus vs Grafana

Prometheus:

```text
Collect/store/query metrics
```

Grafana:

```text
Visualize/query data and build dashboards
```

They often work together.

---

# 86. ELK vs Prometheus

ELK:

```text
Primarily logs
```

Prometheus:

```text
Primarily metrics
```

Tracing:

```text
OpenTelemetry + trace backend
```

Modern platforms often combine all three signal types.

---

# 87. Example Observability Stack

```text
Applications
    |
    +→ Logs → Log Platform
    |
    +→ Metrics → Prometheus
    |
    +→ Traces → OpenTelemetry → Trace Backend
                       |
                       ↓
                    Grafana
```

The exact backend can vary.

---

# 88. Spring Boot Observability

A production Spring Boot service may expose:

```text
Health
Metrics
Logs
Traces
```

through a combination of:

```text
Actuator
Micrometer
OpenTelemetry
Logging framework
```

---

# 89. Log Correlation

Example:

```text
traceId = 8f3a
spanId = 44aa
```

Log:

```json
{
  "level": "ERROR",
  "traceId": "8f3a",
  "spanId": "44aa",
  "service": "payment-service",
  "message": "Database timeout"
}
```

Search:

```text
traceId = 8f3a
```

and find related logs.

---

# 90. Error Tracking

Track:

```text
Exception type
Error count
Affected endpoint
Affected service
Trace ID
Release version
```

This makes errors actionable.

---

# 91. Deployment Correlation

Suppose errors increase immediately after:

```text
Release v2.4.1
```

Include:

```text
service version
deployment version
environment
```

in observability data.

This makes regression detection easier.

---

# 92. Environment Labels

Useful dimensions:

```text
environment
region
service
version
```

Example:

```text
service=order
environment=prod
region=ap-south-1
version=2.4.1
```

Keep label cardinality controlled.

---

# 93. Alert on Symptoms

Prefer alerts that represent user impact.

Example:

```text
High 5xx rate
High p95 latency
Low payment success rate
```

rather than alerting on every internal metric independently.

---

# 94. Alert on Causes Too

Some infrastructure alerts are useful:

```text
Database unavailable
Disk nearly full
Kafka consumer lag critical
Certificate expiring
```

The key is to keep alerts actionable.

---

# 95. Runbook

An alert should ideally have a runbook.

Example:

```text
Alert:
Payment API 5xx > 5%

Runbook:
1. Check dashboard
2. Check recent deployment
3. Inspect trace
4. Check payment logs
5. Check DB health
6. Roll back if confirmed regression
```

---

# 96. Incident Timeline

During an incident record:

```text
10:00 deployment
10:05 latency increased
10:07 error rate increased
10:10 rollback
10:12 metrics recovered
```

This helps post-incident analysis.

---

# 97. Post-Incident Review

Ask:

```text
What happened?
Why?
Why wasn't it detected earlier?
How did users experience it?
What prevented faster recovery?
What should we change?
```

Avoid focusing only on individual blame.

---

# 98. Observability Cost

Telemetry has cost.

Consider:

```text
Log volume
Metric cardinality
Trace sampling
Retention
Storage
Network traffic
```

Collect useful data, not everything indiscriminately.

---

# 99. Sampling Strategy

At high traffic:

```text
Normal requests
→ sample 5%

Errors
→ sample 100%

Slow requests
→ sample 100%
```

This is only an example strategy; production values should be based on requirements.

---

# 100. Sensitive Telemetry

Observability data can contain sensitive information.

Protect:

```text
Logs
Traces
Metrics
Dashboards
```

Apply:

```text
Access control
Redaction
Encryption
Retention policies
```

---

# 101. Production Scenario

### "API latency increased suddenly."

Answer:

```text
1. Check request rate and p95/p99 latency
2. Check error rate
3. Check recent deployments
4. Inspect distributed traces
5. Identify slow dependency
6. Check service logs
7. Check DB/cache/message-broker metrics
8. Mitigate
9. Confirm recovery
10. Perform root-cause analysis
```

---

# 102. Production Scenario

### "CPU is normal, but API is very slow."

Possible causes:

```text
Database connection pool exhaustion
Thread pool saturation
External dependency latency
Network issues
Lock contention
Queue backlog
```

Lesson:

> CPU alone is not a reliable measure of application health.

---

# 103. Production Scenario

### "Only one endpoint is slow."

Check:

```text
Route-specific latency
Trace
Database queries
Downstream calls
Payload size
Cache hit rate
```

Avoid looking only at service-wide CPU.

---

# 104. Production Scenario

### "Only one region is affected."

Check dimensions:

```text
region
availability zone
service version
network path
database replica
dependency endpoint
```

Distributed tracing and regional metrics are especially useful.

---

# 105. Production Scenario

### "Kafka lag is increasing."

Investigate:

```text
Producer rate
Consumer processing time
Consumer count
Partition count
Consumer errors
Downstream database latency
Rebalances
```

Don't automatically add consumers if the bottleneck is downstream.

---

# 106. Production Scenario

### "Logs are too noisy."

Improve:

```text
Log levels
Structured logging
Sampling
Deduplication
Sensitive-data filtering
```

Don't simply delete all logs.

---

# 107. Production Scenario

### "A trace shows 2 seconds latency, but service logs show 100ms."

Possible explanation:

```text
Network latency
Downstream service
Queue wait
Connection acquisition
Uninstrumented work
```

Tracing provides context beyond a single service's log duration.

---

# 108. Interview Question

### "What are the three pillars of observability?"

Answer:

> "Logs, metrics and traces. Logs provide detailed events, metrics provide numerical trends and health signals, and traces show how a request moves through distributed services."

---

# 109. Interview Question

### "What is distributed tracing?"

Answer:

> "Distributed tracing follows a request across multiple services using a trace made up of spans. It helps identify where latency and failures occur in a distributed workflow."

---

# 110. Interview Question

### "What is OpenTelemetry?"

Answer:

> "OpenTelemetry is an open-source observability framework for generating, collecting and exporting telemetry such as traces, metrics and logs. It helps standardize instrumentation and reduces dependence on a single observability vendor."

---

# 111. Interview Question

### "What is the difference between logs and metrics?"

Answer:

> "Logs contain detailed event information and are useful for investigating individual failures. Metrics are numerical time-series data and are better for dashboards, trends, alerts and capacity analysis."

---

# 112. Interview Question

### "What is p95 latency?"

Answer:

> "p95 means 95 percent of measured requests completed at or below that latency value. It is useful for understanding tail latency that an average can hide."

---

# 113. Interview Question

### "What is the difference between liveness and readiness?"

Answer:

> "Liveness determines whether an instance should be restarted, while readiness determines whether it should receive traffic. A service can be alive but not ready, for example while it is still starting."

---

# 114. Interview Question

### "What is Prometheus?"

Answer:

> "Prometheus is a monitoring and time-series system commonly used to collect, store and query metrics. It commonly scrapes metrics endpoints exposed by applications."

---

# 115. Interview Question

### "What is Grafana?"

Answer:

> "Grafana is a visualization and dashboarding platform commonly used with systems such as Prometheus. It helps teams explore metrics and build operational dashboards and alerts."

---

# 116. Interview Question

### "What is ELK?"

Answer:

> "ELK traditionally refers to Elasticsearch, Logstash and Kibana. Elasticsearch stores and indexes logs, Logstash processes and transports them, and Kibana provides search and visualization."

---

# 117. Interview Question

### "How do you debug a slow microservice request?"

Answer:

> "I'd first check traffic, error rate and p95/p99 latency. Then I'd inspect the distributed trace to find which service or dependency is consuming time. After locating the slow component, I'd inspect its logs and resource metrics such as database connections, thread pools, cache and downstream calls."

---

# 118. Interview Question

### "Why is correlation ID useful?"

Answer:

> "It lets us associate logs and operations belonging to the same request across multiple services. This makes distributed troubleshooting much easier."

---

# 119. Interview Question

### "Why shouldn't you log everything?"

Answer:

> "Excessive logs increase storage and processing costs, create noise and can expose sensitive information. I'd use structured logs, appropriate levels, redaction and controlled retention."

---

# 120. Interview Question

### "What metrics would you monitor for a Spring Boot API?"

Answer:

> "I'd monitor request rate, 4xx/5xx errors, p95/p99 latency, active requests, timeouts, JVM memory and GC, thread pools, database connection pools, downstream call latency and business metrics such as payment success rate."

---

# 121. Interview Question

### "How do you monitor Kafka consumers?"

Answer:

> "I'd monitor consumer lag, processing latency, consumer errors, rebalance activity, throughput and downstream dependencies. Increasing lag can mean consumers are slower than producers or are blocked by another dependency."

---

# 122. Interview Scenario

### "Production checkout is slow. What is your first step?"

Answer:

> "I wouldn't immediately restart services. I'd check the service's traffic, error rate and p95/p99 latency, then use distributed tracing to locate the slow part of the request. From there I'd inspect the relevant logs and dependency metrics and apply the appropriate mitigation."

---

# 123. Interview Scenario

### "The service is UP but users report failures."

Answer:

> "Health status alone doesn't prove the service is healthy from a user perspective. I'd check request error rates, latency, dependency health, database connectivity, saturation and business-level success metrics."

---

# 124. Interview Scenario

### "CPU is only 30%, but requests are timing out."

Answer:

> "I'd look beyond CPU. Possible causes include database connection pool exhaustion, thread starvation, network latency, downstream timeouts, lock contention or queue saturation. Distributed tracing and resource-specific metrics would help locate the issue."

---

# 125. Final Observability Architecture

```text
                    +----------------+
                    |  Microservices |
                    +--------+-------+
                             |
            +----------------+----------------+
            |                |                |
            ↓                ↓                ↓
          Logs            Metrics           Traces
            |                |                |
            ↓                ↓                ↓
      Log Platform       Prometheus      OpenTelemetry
            |                |                |
            ↓                ↓                ↓
         Kibana           Grafana        Trace Backend
            \                |                /
             \_______________|_______________/
                             ↓
                       Observability
                          Platform
```

Spring Boot ecosystem:

```text
Spring Boot
    |
    +→ Actuator
    |
    +→ Micrometer
    |
    +→ OpenTelemetry
    |
    +→ Structured Logging
```

---

# 126. Incident Debugging Mental Model

Remember:

```text
Metrics
→ DETECT

Traces
→ LOCATE

Logs
→ EXPLAIN

Dashboards
→ VISUALIZE

Alerts
→ NOTIFY

Runbooks
→ RESPOND

Postmortems
→ IMPROVE
```

---

# 127. Golden Signals

For a service:

```text
Traffic
Latency
Errors
Saturation
```

For a database:

```text
Connections
Query latency
CPU
Disk
Replication lag
```

For Kafka:

```text
Throughput
Consumer lag
Processing latency
Errors
Partition health
```

---

# 128. Final Interview Answer

If asked:

> "How would you design observability for a Spring Boot microservices platform?"

Use:

> "I'd implement centralized structured logging, application and infrastructure metrics, and distributed tracing. Spring Boot Actuator and Micrometer can expose health and metrics, Prometheus can collect metrics, Grafana can visualize them, and OpenTelemetry can provide standardized tracing and telemetry collection. I'd propagate trace context across HTTP and messaging, include trace IDs in logs, and define actionable alerts around latency, errors, saturation and business metrics. During incidents I'd use metrics to detect the problem, traces to locate it and logs to understand the root cause."

---

# 129. Revision Checklist

```text
□ Observability
□ Logs
□ Metrics
□ Traces
□ Structured logging
□ Log levels
□ Correlation ID
□ Trace ID
□ Span ID
□ Distributed tracing
□ Centralized logging
□ ELK
□ Elasticsearch
□ Logstash
□ Kibana
□ Counters
□ Gauges
□ Histograms
□ Percentiles
□ p50
□ p95
□ p99
□ RED
□ USE
□ Golden signals
□ JVM metrics
□ Database metrics
□ Kafka metrics
□ Spring Boot Actuator
□ Liveness
□ Readiness
□ Startup probe
□ Monitoring
□ Alerting
□ SLI
□ SLO
□ SLA
□ Error budget
□ OpenTelemetry
□ OpenTelemetry Collector
□ Trace propagation
□ Sampling
□ Prometheus
□ Micrometer
□ Grafana
□ Metric cardinality
□ Business metrics
□ Runbooks
□ Incident response
□ Postmortems
□ Production debugging
```

---

# 130. The Interviewer's Real Test

If asked:

> "Users say checkout is slow. CPU and memory look normal. How do you investigate?"

Think:

```text
User report
    ↓
Check RED metrics
    ↓
p95/p99 latency ↑
    ↓
Find affected route
    ↓
Open distributed trace
    ↓
Payment span = 2.5 sec
    ↓
Payment logs
    ↓
DB connection pool saturated
    ↓
DB query latency ↑
    ↓
Identify root cause
```

The key interview lesson is:

> **Observability is not just collecting logs. It is the ability to connect metrics, traces and logs to understand what the system is doing and why it is doing it.**

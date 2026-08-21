# System Design — File 16: Observability & Monitoring

A production system needs more than code and infrastructure. We need to know:

```text
Is the system healthy?
What is failing?
Why is it failing?
Which users are affected?
How quickly can we recover?
```

Observability gives us the ability to understand a system from its external behavior.

---

## 1. Monitoring vs Observability

### Monitoring

Monitoring answers questions we already know to ask.

Examples:

```text
CPU > 80%
Error rate > 5%
Database connections > 90%
```

### Observability

Observability helps investigate unexpected problems.

Example:

```text
Why did checkout latency increase
only for users in one region?
```

Simple interview answer:

> "Monitoring tells us when something is wrong. Observability helps us understand why it is wrong."

---

## 2. Three Pillars of Observability

The classic three pillars are:

```text
Logs
Metrics
Traces
```

Modern observability can also include:

```text
Profiles
Events
Continuous profiling
```

But logs, metrics and traces are the core interview topics.

---

## 3. Logs

Logs record application events.

Example:

```text
2026-08-21 10:30:22
OrderService
orderId=1234
status=PAYMENT_PENDING
```

Logs are useful for:

```text
Debugging
Auditing
Error investigation
Business events
Operational analysis
```

---

## 4. Structured Logging

Avoid logs like:

```text
Something went wrong with order
```

Prefer structured information:

```text
{
  "level": "ERROR",
  "service": "order-service",
  "orderId": "1234",
  "traceId": "abc123",
  "errorCode": "PAYMENT_TIMEOUT"
}
```

Benefits:

```text
Searchable
Filterable
Machine-readable
Easier aggregation
```

---

## 5. Log Levels

Common levels:

```text
TRACE
DEBUG
INFO
WARN
ERROR
```

Typical production usage:

```text
INFO  -> important application events
WARN  -> unusual but recoverable conditions
ERROR -> failures requiring investigation
DEBUG -> detailed troubleshooting
```

Don't log everything at ERROR.

---

## 6. What Not to Log

Avoid sensitive information:

```text
Passwords
Access tokens
Refresh tokens
Private keys
Credit-card secrets
Sensitive personal data
```

Logs are frequently copied into multiple systems.

---

## 7. Centralized Logging

In a microservice system:

```text
Order Service  \
Payment Service ---> Log Platform
Inventory      /
```

Instead of checking individual servers.

Common technologies:

```text
ELK / Elastic Stack
OpenSearch
Cloud logging platforms
```

---

## 8. ELK Stack

ELK traditionally refers to:

```text
Elasticsearch
Logstash
Kibana
```

Typical flow:

```text
Application
    |
    v
Logstash / Collector
    |
    v
Elasticsearch
    |
    v
Kibana
```

Kibana can be used for:

```text
Search
Dashboards
Visualization
Log analysis
```

---

## 9. Log Aggregation

A centralized platform lets you query:

```text
service=payment
level=ERROR
traceId=abc123
```

This is especially useful when one request crosses many services.

---

## 10. Metrics

Metrics are numerical measurements over time.

Examples:

```text
Request count
Error count
Latency
CPU usage
Memory usage
Queue depth
Database connections
Cache hit ratio
```

Metrics are usually cheaper to aggregate than raw logs.

---

## 11. Counter

A counter only increases, except when reset.

Examples:

```text
http_requests_total
payment_failures_total
orders_created_total
```

Useful for:

```text
Request counts
Errors
Events
```

---

## 12. Gauge

A gauge represents a value that can increase or decrease.

Examples:

```text
CPU usage
Memory usage
Queue depth
Active connections
```

Example:

```text
active_connections = 42
```

---

## 13. Histogram

A histogram measures distributions.

Useful for:

```text
Request latency
Response size
Processing time
```

It helps answer:

```text
How many requests completed under 100 ms?
How many under 500 ms?
```

---

## 14. Summary

A summary can calculate quantiles on the client/application side depending on the metrics system.

Common distinction:

```text
Histogram -> buckets, aggregatable
Summary   -> quantiles often calculated per instance
```

For distributed systems, histograms are often more useful for aggregating latency distributions across instances.

---

## 15. Prometheus

Prometheus is a popular metrics monitoring system.

Conceptually:

```text
Application
    |
    v
/metrics
    |
    v
Prometheus
    |
    v
Queries / Alerts / Dashboards
```

Applications expose metrics.

Prometheus periodically scrapes them.

---

## 16. Micrometer + Spring Boot

Spring Boot applications commonly use:

```text
Micrometer
```

to expose application metrics.

Typical integration:

```text
Spring Boot
    |
    v
Micrometer
    |
    v
Prometheus
```

Metrics can include:

```text
HTTP requests
JVM memory
GC
Threads
Database pools
Custom business metrics
```

---

## 17. Grafana

Grafana is commonly used to visualize metrics.

Example:

```text
Prometheus
    |
    v
Grafana
```

Dashboards can show:

```text
Request rate
p95 latency
Error rate
CPU
Memory
DB connections
```

---

## 18. Logs vs Metrics

### Logs

Good for:

```text
Detailed events
Specific failures
Debugging
```

### Metrics

Good for:

```text
Trends
Alerting
Aggregated health
Capacity
```

Example:

```text
Metric:
Error rate = 5%

Logs:
Payment timeout for order 1234
```

They complement each other.

---

## 19. Distributed Tracing

A request may travel:

```text
Client
  |
  v
API Gateway
  |
  v
Order Service
  |
  +--> Inventory
  |
  +--> Payment
  |
  +--> Kafka
```

Tracing helps reconstruct the complete request path.

---

## 20. Trace ID

A trace ID identifies the overall request.

Example:

```text
traceId = abc123
```

The same trace ID can appear across services.

---

## 21. Span

A span represents one operation within a trace.

Example:

```text
Trace: abc123

Span 1 -> API Gateway
Span 2 -> Order Service
Span 3 -> Payment Service
Span 4 -> Database
```

A trace is made up of spans.

---

## 22. Parent and Child Spans

Example:

```text
Order Request
     |
     +---- Payment
     |
     +---- Inventory
     |
     +---- Database
```

The child spans help understand where time was spent.

---

## 23. Trace Example

Suppose total request latency:

```text
900 ms
```

Trace shows:

```text
Gateway       20 ms
Order         100 ms
Inventory     100 ms
Payment       650 ms
Database       30 ms
```

The bottleneck is obvious:

```text
Payment = 650 ms
```

Without tracing, investigating the 900 ms request is much harder.

---

## 24. OpenTelemetry

OpenTelemetry is a common standard/tooling ecosystem for collecting:

```text
Traces
Metrics
Logs
```

It can export telemetry to different observability backends.

Conceptually:

```text
Spring Boot
    |
    v
OpenTelemetry
    |
    +----> Tracing backend
    +----> Metrics backend
    +----> Log backend
```

---

## 25. Correlation ID

A correlation ID helps connect related operations.

Example:

```text
correlationId = req-123
```

It can appear in:

```text
API logs
Service logs
Queue messages
Database audit records
```

Trace IDs and correlation IDs can overlap in purpose, but they are not necessarily identical.

---

## 26. Trace ID vs Correlation ID

### Trace ID

Usually represents:

```text
One distributed trace
```

### Correlation ID

A broader application-defined identifier used to associate related activity.

Don't assume they must always be the same value.

---

## 27. Golden Signals

Four common service health signals:

```text
Latency
Traffic
Errors
Saturation
```

### Latency

How long requests take.

### Traffic

How much demand the service receives.

### Errors

How many requests fail.

### Saturation

How close resources are to capacity.

---

## 28. RED Method

For request-driven services:

```text
R = Rate
E = Errors
D = Duration
```

Example dashboard:

```text
Request rate
Error rate
p95/p99 latency
```

---

## 29. USE Method

Useful for infrastructure/resources:

```text
U = Utilization
S = Saturation
E = Errors
```

Example:

```text
CPU utilization
CPU saturation
Disk errors
```

RED and USE complement each other.

---

## 30. Latency Percentiles

Average latency can hide bad experiences.

Example:

```text
99 requests -> 50 ms
1 request   -> 10 seconds
```

Average may not communicate the tail problem well.

Use:

```text
p50
p95
p99
```

---

## 31. p50

p50 is the median.

Roughly:

```text
50% of requests are at or below this latency
```

Useful for typical user experience.

---

## 32. p95

p95 means approximately:

```text
95% of requests are at or below this latency
```

The slowest 5% are above it.

Useful for detecting tail degradation.

---

## 33. p99

p99 means approximately:

```text
99% of requests are at or below this latency
```

The slowest 1% are above it.

Important for high-scale systems because even a small percentage can represent many users.

---

## 34. Why Average Latency Can Mislead

Example:

```text
Average = 100 ms
p99 = 2 seconds
```

The average looks good.

But some users experience:

```text
2 seconds
```

For large systems, tail latency matters.

---

## 35. Error Rate

A useful metric:

```text
Error Rate =
Failed Requests / Total Requests
```

Example:

```text
500 errors = 200
Total requests = 10,000

Error rate = 2%
```

Track by:

```text
Endpoint
Service
Region
Status code
Dependency
```

---

## 36. Saturation Metrics

Examples:

```text
CPU utilization
Memory usage
Thread pool usage
Connection pool usage
Queue depth
Disk utilization
```

Saturation often provides an early warning before users see complete failure.

---

## 37. Database Observability

Monitor:

```text
Query latency
Slow queries
Connection pool usage
Active connections
CPU
Memory
Replication lag
Locks
Deadlocks
Disk usage
```

For a Spring Boot application, also inspect:

```text
HikariCP metrics
```

---

## 38. Redis Observability

Monitor:

```text
Hit ratio
Miss ratio
Latency
Memory
Evictions
Connected clients
Commands/sec
Hot keys
Replication health
```

A cache problem can become a database problem.

---

## 39. Kafka Observability

Important metrics:

```text
Consumer lag
Producer errors
Throughput
Partition distribution
Under-replicated partitions
Request latency
Broker health
```

Consumer lag is particularly important.

---

## 40. Consumer Lag

Example:

```text
Produced:
10,000 messages

Consumed:
8,000 messages
```

Backlog:

```text
2,000 messages
```

If lag keeps growing:

```text
Consumers cannot keep up
```

Possible actions:

```text
Scale consumers
Optimize processing
Increase partitions when appropriate
Throttle producers
```

---

## 41. Alerting

Good alerts should be:

```text
Actionable
Relevant
Based on user impact
```

Bad:

```text
CPU > 70%
```

if this is normal.

Better:

```text
Error rate > SLO
p95 latency > threshold
Queue age > threshold
Database connections exhausted
```

---

## 42. Alert Severity

Example:

```text
INFO
WARNING
CRITICAL
```

Critical alerts should represent issues requiring immediate action.

Avoid too many critical alerts.

---

## 43. Alert Fatigue

If engineers receive:

```text
100 alerts/day
```

and most are irrelevant:

```text
Important alert gets ignored
```

This is alert fatigue.

Reduce:

```text
Noisy alerts
Duplicate alerts
Non-actionable alerts
```

---

## 44. SLI

A Service Level Indicator is a measurable representation of user experience.

Examples:

```text
Successful request ratio
p95 API latency
Checkout success rate
```

---

## 45. SLO

A Service Level Objective is the target.

Example:

```text
99.9% successful checkout requests
p95 latency < 300 ms
```

---

## 46. SLA

A Service Level Agreement is typically a contractual commitment.

Remember:

```text
SLI -> Measure
SLO -> Target
SLA -> Contract
```

---

## 47. Error Budget

If:

```text
SLO = 99.9%
```

the allowed unreliability is:

```text
0.1%
```

That allowance is the error budget.

If releases repeatedly consume the budget:

```text
Prioritize reliability
```

---

## 48. Incident Detection

A typical path:

```text
Metric
  |
  v
Alert
  |
  v
On-call engineer
  |
  v
Investigate logs/traces
  |
  v
Mitigate
```

Automated alerts should lead to actionable investigation.

---

## 49. Incident Investigation

A useful sequence:

```text
1. What changed?
2. When did it start?
3. How many users are affected?
4. Which endpoints are affected?
5. Which dependency is slow/failing?
6. Is there a recent deployment?
7. Can we mitigate quickly?
```

---

## 50. Recent Deployment

When an incident starts shortly after a deployment:

```text
Check release
Check error rate
Check latency
Compare old/new version
Rollback if appropriate
```

Observability should make version comparison easy.

---

## 51. Canary Monitoring

For a canary deployment:

```text
95% -> Old
5%  -> New
```

Compare:

```text
Error rate
Latency
CPU
Business metrics
```

If the new version performs badly:

```text
Stop rollout
Rollback
```

---

## 52. Business Metrics

Technical metrics are not enough.

Examples:

```text
Orders/minute
Payment success rate
Checkout completion
Search success
Registration success
```

A service can look healthy technically while a business workflow is failing.

---

## 53. Technical + Business Observability

Example:

```text
HTTP 200 rate = 99.9%
```

Looks healthy.

But:

```text
Payment success = 70%
```

Business is clearly unhealthy.

Always connect infrastructure health to user outcomes.

---

## 54. Dashboard Design

A useful service dashboard might contain:

```text
Request rate
Error rate
p50/p95/p99 latency
CPU
Memory
Thread pools
DB connections
Dependency latency
Business success rate
```

Avoid dashboards containing hundreds of unrelated charts.

---

## 55. Service Dependency Map

Visualize:

```text
API Gateway
   |
   +--> Order
          |
          +--> Payment
          |
          +--> Inventory
          |
          +--> Redis
          |
          +--> MySQL
```

Dependency maps help identify:

```text
Critical dependencies
Bottlenecks
Failure propagation
```

---

## 56. Health Endpoint

A basic health endpoint can expose application health.

Example:

```text
GET /actuator/health
```

In Spring Boot, Actuator provides production-oriented endpoints and metrics integration.

Be careful not to expose sensitive diagnostic information publicly.

---

## 57. Spring Boot Actuator

Common capabilities include:

```text
Health
Metrics
Info
Environment/config diagnostics
Application mappings
```

Expose only what is appropriate for the environment and secure sensitive endpoints.

---

## 58. JVM Observability

For Java applications monitor:

```text
Heap usage
GC activity
Thread count
Thread states
Class loading
CPU
JVM pauses
```

A memory leak may show:

```text
Heap usage keeps increasing
+
GC activity increases
+
Eventually OutOfMemoryError
```

---

## 59. Garbage Collection

High GC activity can cause:

```text
Latency spikes
CPU usage
Throughput reduction
```

Monitor:

```text
GC frequency
Pause duration
Heap occupancy
```

Don't tune GC blindly; first understand the workload.

---

## 60. Thread Pool Monitoring

A Spring Boot service may use:

```text
Tomcat threads
Async executors
Database connection pools
HTTP client pools
```

Monitor:

```text
Active threads
Queued tasks
Pool utilization
Rejected tasks
```

Thread exhaustion can cause cascading failures.

---

## 61. HikariCP Observability

For JDBC connection pools, monitor:

```text
Active connections
Idle connections
Pending threads
Maximum pool size
Connection acquisition time
```

A common failure pattern:

```text
Slow DB
   |
Connections held longer
   |
Pool exhausted
   |
Requests wait
   |
Latency increases
```

---

## 62. Distributed Trace Sampling

Tracing every request can be expensive at huge scale.

Sampling strategies:

```text
Head-based sampling
Tail-based sampling
Error-based sampling
Rate-based sampling
```

A practical approach may sample normal traffic while retaining more traces for errors or slow requests.

---

## 63. Log Sampling

High-volume services can produce enormous logs.

Possible approaches:

```text
Sample repetitive INFO logs
Keep ERROR logs
Increase sampling during incidents
```

Never sample away information required for critical auditing or compliance.

---

## 64. High Cardinality

Metric labels can become dangerous when they contain highly unique values.

Bad:

```text
http_requests{userId="123456"}
```

Millions of users can create huge numbers of time series.

Better labels:

```text
service
endpoint
method
status
region
```

Be careful with:

```text
userId
requestId
traceId
```

as metric labels.

These are better suited to logs/traces.

---

## 65. Logs vs Metrics vs Traces

| Tool | Best For |
|---|---|
| Logs | Detailed events |
| Metrics | Trends, alerting, capacity |
| Traces | Request path and latency |
| Profiles | CPU/memory/code-level performance |

Use all of them together.

---

## 66. Observability Architecture

Example:

```text
                 Spring Boot
                /     |      \
               v      v       v
             Logs  Metrics   Traces
               |      |        |
               v      v        v
             Log    Prometheus OTel/
           Platform    |       Trace DB
               |       |
               v       v
            Dashboards / Alerts
```

---

## 67. Cost of Observability

Observability has costs:

```text
Storage
Network
CPU
Indexing
Retention
Query infrastructure
```

Control cost through:

```text
Sampling
Retention policies
Log levels
Aggregation
Filtering
Tiered storage
```

---

## 68. Observability Security

Telemetry can contain sensitive information.

Protect:

```text
Logs
Traces
Metrics
Dashboards
Alert systems
```

Use:

```text
Access control
Redaction
Encryption
Retention policies
```

---

## 69. Observability During an Outage

When a production outage occurs:

```text
Metrics -> detect scope
Traces  -> find bottleneck
Logs    -> understand error
Deployments -> identify change
Business metrics -> measure impact
```

This combination dramatically reduces investigation time.

---

## 70. Interview — What Are the Three Pillars?

> "The traditional three pillars are logs, metrics and distributed traces. Logs provide detailed events, metrics provide aggregated measurements and traces show how a request moves through distributed services."

---

## 71. Interview — Why Is p99 Important?

> "Average latency can hide tail behavior. p99 shows the latency experienced by roughly the slowest one percent of requests, which becomes important at scale because even a small percentage can represent many users."

---

## 72. Interview — What Is Consumer Lag?

> "Consumer lag is the amount of unprocessed data between producers and consumers. If it keeps growing, consumers aren't keeping up with incoming traffic, so I'd investigate processing time, consumer capacity and partitioning."

---

## 73. Interview — How Would You Debug a Slow API?

> "I'd start with p95/p99 latency and error rate, then use distributed tracing to identify the slow dependency. I'd check application logs, database query latency, connection pools, Redis and downstream services, and compare the timing with recent deployments."

---

## 74. Interview — How Would You Monitor a Spring Boot Application?

> "I'd use Actuator and Micrometer for application and JVM metrics, Prometheus for metric collection and Grafana for dashboards. I'd centralize structured logs and use distributed tracing with OpenTelemetry. I'd monitor HTTP latency, error rate, JVM health, HikariCP, database performance, Redis and business metrics."

---

## 75. Practical Scenario — API p99 Suddenly Increases

Check:

```text
Recent deployment
Downstream latency
Database queries
Redis
Connection pools
CPU
GC
Thread pools
Network
```

Use traces to identify where the extra latency appears.

---

## 76. Practical Scenario — Error Rate Increases After Deployment

Process:

```text
Compare versions
   |
   v
Check affected endpoint
   |
   v
Inspect traces/logs
   |
   v
Rollback or mitigate
   |
   v
Investigate root cause
```

---

## 77. Practical Scenario — Kafka Lag Keeps Growing

Check:

```text
Consumer processing time
Consumer count
Partition distribution
Broker health
Database latency
Downstream dependencies
```

Possible solutions:

```text
Scale consumers
Optimize processing
Increase partitions when appropriate
Reduce producer rate
```

---

## 78. Practical Scenario — DB Connection Pool Exhausted

Likely causes:

```text
Slow queries
Long transactions
Database overload
Connection leaks
Pool too small
Traffic spike
```

Investigate before simply increasing pool size.

---

## 79. Practical Scenario — CPU Is Normal but API Is Slow

CPU alone doesn't prove health.

Check:

```text
Database latency
Network latency
Thread waiting
Connection pools
External dependencies
Lock contention
GC
Queue delays
```

This is why observability needs multiple signals.

---

## 80. Final Checklist

```text
□ Monitoring vs observability
□ Logs
□ Structured logging
□ Log levels
□ Centralized logging
□ ELK
□ Metrics
□ Counters
□ Gauges
□ Histograms
□ Prometheus
□ Micrometer
□ Grafana
□ Distributed tracing
□ Trace IDs
□ Spans
□ OpenTelemetry
□ Correlation IDs
□ Golden signals
□ RED
□ USE
□ p50/p95/p99
□ Error rate
□ Saturation
□ DB monitoring
□ Redis monitoring
□ Kafka consumer lag
□ Alerting
□ Alert fatigue
□ SLI/SLO/SLA
□ Error budgets
□ Incident investigation
□ Canary monitoring
□ Business metrics
□ Dashboards
□ Dependency maps
□ Spring Boot Actuator
□ JVM monitoring
□ GC monitoring
□ Thread pools
□ HikariCP
□ Trace sampling
□ Log sampling
□ High cardinality
□ Observability cost
□ Telemetry security
```

---

## 81. One-Minute Interview Answer

### "How would you design observability for a Spring Boot microservice?"

> "I'd use Actuator and Micrometer for application and JVM metrics, Prometheus for collecting metrics and Grafana for dashboards. I'd use structured centralized logs with correlation or trace IDs, and distributed tracing through OpenTelemetry to follow requests across services. I'd monitor the golden signals—latency, traffic, errors and saturation—along with database, Redis, Kafka, connection-pool and business metrics. Alerts should be actionable and tied to SLOs, while logs and traces should be protected from sensitive-data leakage."

---

## 82. Key Takeaway

> **Observability turns a distributed system from a black box into something engineers can understand. Metrics tell you that a problem exists, traces help locate it, and logs help explain what happened. The strongest production monitoring connects all three with user-facing business outcomes.**

**File 16 complete.**

# Spring Boot — Actuator and Production Monitoring

Spring Boot Actuator provides production-ready features for monitoring and managing Spring Boot applications.

It helps expose information about:

```text
Application health
Metrics
Environment
Beans
Mappings
Loggers
Application info
HTTP exchanges
Thread information
```

Typical production flow:

```text
Spring Boot Application
        ↓
Spring Boot Actuator
        ↓
Health + Metrics + Monitoring
        ↓
Prometheus / Grafana / APM
        ↓
Alerts
```

---

# 1. Why Actuator?

Without monitoring, an application may fail silently.

Actuator helps answer:

```text
Is the application healthy?
Is the database reachable?
How many requests are failing?
How long do requests take?
How much memory is being used?
Are there too many active requests?
```

---

# 2. Add Actuator

Maven dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

---

# 3. Basic Configuration

By default, only selected endpoints are exposed.

Example:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

Do not expose every endpoint blindly in production.

---

# 4. Health Endpoint

Common endpoint:

```text
/actuator/health
```

Example response:

```json
{
  "status": "UP"
}
```

Health checks answer whether the application and its important dependencies are available.

---

# 5. Health Details

You can configure health details:

```yaml
management:
  endpoint:
    health:
      show-details: when_authorized
```

Possible values include:

```text
never
when-authorized
always
```

Avoid exposing sensitive health details publicly.

---

# 6. Database Health

Spring Boot can automatically contribute database health information when a supported datasource is configured.

Conceptually:

```text
Application
    ↓
Database Health Indicator
    ↓
Database reachable?
    ↓
UP / DOWN
```

This helps detect database connectivity problems.

---

# 7. Custom Health Indicator

You can create a custom health indicator.

Example:

```java
@Component
public class PaymentServiceHealthIndicator
        implements HealthIndicator {

    @Override
    public Health health() {

        boolean available =
            checkPaymentService();

        if (available) {
            return Health.up().build();
        }

        return Health.down()
            .withDetail(
                "paymentService",
                "unavailable"
            )
            .build();
    }

    private boolean checkPaymentService() {
        return true;
    }
}
```

Use health checks carefully. A health check should be fast and meaningful.

---

# 8. Liveness vs Readiness

Modern applications commonly distinguish:

```text
Liveness
Readiness
```

Liveness answers:

```text
Should this application instance be restarted?
```

Readiness answers:

```text
Should this instance receive traffic?
```

---

# 9. Liveness

Example endpoint:

```text
/actuator/health/liveness
```

A liveness failure may cause an orchestrator such as Kubernetes to restart the application.

Do not include external dependency failures in liveness unless restarting the application is genuinely the correct response.

---

# 10. Readiness

Example:

```text
/actuator/health/readiness
```

Readiness can indicate whether the application is ready to serve traffic.

For example:

```text
Application started
Database available
Required initialization complete
        ↓
READY
```

---

# 11. Why Liveness and Readiness Matter

Consider:

```text
Application process is alive
        ↓
Database temporarily unavailable
```

The application may still be alive but not ready to handle requests correctly.

Therefore:

```text
Liveness ≠ Readiness
```

---

# 12. Metrics

Actuator exposes metrics through:

```text
/actuator/metrics
```

Examples:

```text
JVM memory
CPU
HTTP requests
Threads
Garbage collection
Database connection pool
```

---

# 13. Micrometer

Spring Boot uses Micrometer as its metrics abstraction.

Conceptually:

```text
Application
     ↓
Micrometer
     ↓
Metrics backend
```

Possible backends include:

```text
Prometheus
Datadog
New Relic
CloudWatch
Other monitoring systems
```

---

# 14. HTTP Metrics

Spring Boot can expose HTTP server metrics.

Typical metric:

```text
http.server.requests
```

Useful information can include:

```text
Request count
Request duration
HTTP status
HTTP method
URI
```

These metrics help identify slow or failing endpoints.

---

# 15. JVM Metrics

Useful JVM metrics include:

```text
Heap memory
Non-heap memory
Garbage collection
Threads
Class loading
CPU
```

Example concepts:

```text
jvm.memory.used
jvm.threads.live
jvm.gc
```

Exact metric names can vary by version and configuration.

---

# 16. HikariCP Metrics

For applications using HikariCP, metrics can help monitor:

```text
Active connections
Idle connections
Pending connection requests
Maximum pool size
Minimum idle
```

This is especially useful for diagnosing database connection pool exhaustion.

---

# 17. Database Connection Pool Problem

Example:

```text
Requests increase
      ↓
Database connections become busy
      ↓
Connection pool exhausted
      ↓
Requests wait
      ↓
Latency increases
      ↓
Timeouts
```

Monitoring Hikari metrics can help detect this before it becomes a major outage.

---

# 18. Metrics Endpoint

Example:

```text
/actuator/metrics/jvm.memory.used
```

The endpoint can provide information about a particular metric.

Do not assume metric names from memory; inspect `/actuator/metrics` in the running application or use the configured monitoring backend.

---

# 19. Info Endpoint

Endpoint:

```text
/actuator/info
```

You can expose application information.

Example:

```yaml
info:
  app:
    name: ecommerce-backend
    version: 1.0.0
```

Response can contain:

```json
{
  "app": {
    "name": "ecommerce-backend",
    "version": "1.0.0"
  }
}
```

Do not expose secrets or sensitive infrastructure information.

---

# 20. Environment Endpoint

Endpoint:

```text
/actuator/env
```

It can expose application environment and configuration information.

This endpoint is sensitive.

Do not expose it publicly without strict access control.

---

# 21. Beans Endpoint

Endpoint:

```text
/actuator/beans
```

It provides information about Spring beans.

Useful for debugging configuration problems.

It can expose implementation details, so protect it in production.

---

# 22. Mappings Endpoint

Endpoint:

```text
/actuator/mappings
```

It provides information about request mappings.

Useful for:

```text
Debugging routing
Understanding registered endpoints
```

It should generally be protected because it reveals application structure.

---

# 23. Loggers Endpoint

Endpoint:

```text
/actuator/loggers
```

It can expose logger configuration.

Depending on configuration, it can also be used to change logging levels at runtime.

This is powerful and should be restricted.

---

# 24. Thread Dump

Actuator can provide thread information.

Useful for diagnosing:

```text
Blocked threads
Thread contention
Deadlocks
Unexpected thread growth
```

Thread diagnostics are especially useful when an application becomes slow or unresponsive.

---

# 25. Heap Dump

Some Actuator setups can expose heap dump functionality.

Heap dumps are powerful diagnostic artifacts.

They may contain:

```text
Objects
Application data
Sensitive information
```

Therefore, access must be tightly restricted.

---

# 26. Shutdown Endpoint

Spring Boot can support a shutdown endpoint when explicitly enabled.

Do not expose it publicly.

A remote shutdown capability can be extremely dangerous if improperly secured.

---

# 27. Endpoint Exposure

Example:

```yaml
management:
  endpoints:
    web:
      exposure:
        include:
          - health
          - info
          - metrics
```

Prefer an allowlist of endpoints rather than exposing everything.

---

# 28. Don't Expose *

Avoid:

```yaml
include: "*"
```

in production unless you fully understand the security implications and have appropriate access controls.

Better:

```yaml
include: health,info,metrics,prometheus
```

when those are actually required.

---

# 29. Management Port

You can separate management traffic:

```yaml
management:
  server:
    port: 8081
```

Application:

```text
8080
```

Management:

```text
8081
```

This can make network-level access control easier.

---

# 30. Management Base Path

Default:

```text
/actuator
```

You can customize it:

```yaml
management:
  endpoints:
    web:
      base-path: /management
```

Then:

```text
/management/health
```

would be used instead.

Changing the path is not a substitute for authentication and authorization.

---

# 31. Securing Actuator

Actuator endpoints can expose sensitive operational information.

Use Spring Security to restrict access.

Example concept:

```java
.requestMatchers(
    "/actuator/health"
).permitAll()

.requestMatchers(
    "/actuator/**"
).hasRole("ACTUATOR")
```

Exact authorization should match your deployment architecture.

---

# 32. Public Health vs Protected Details

A common design:

```text
/actuator/health
    ↓
Public or internal load-balancer access

/actuator/metrics
    ↓
Protected

/actuator/env
    ↓
Protected

/actuator/beans
    ↓
Protected
```

Only expose what monitoring infrastructure actually needs.

---

# 33. Prometheus

Prometheus is a popular monitoring system.

Spring Boot can expose metrics in Prometheus format using the appropriate Micrometer registry dependency.

Typical endpoint:

```text
/actuator/prometheus
```

Conceptual flow:

```text
Spring Boot
    ↓
Micrometer
    ↓
Prometheus endpoint
    ↓
Prometheus server
    ↓
Grafana
```

---

# 34. Prometheus Dependency

Typical Maven dependency:

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

The exact dependency version is normally managed by Spring Boot's dependency management.

---

# 35. Prometheus Configuration

Example:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
```

Then monitoring infrastructure can scrape:

```text
/actuator/prometheus
```

---

# 36. Grafana

Grafana is commonly used to visualize metrics.

Flow:

```text
Spring Boot
     ↓
Micrometer
     ↓
Prometheus
     ↓
Grafana
```

Grafana dashboards can display:

```text
Request rate
Error rate
Latency
CPU
Memory
Database pool
JVM metrics
```

---

# 37. RED Metrics

For request-driven services, RED is useful:

```text
Rate
Errors
Duration
```

Example:

```text
Rate:
1000 requests/min

Errors:
2%

Duration:
p95 = 250ms
```

These metrics provide a useful high-level view of API health.

---

# 38. USE Method

For infrastructure/resource monitoring:

```text
Utilization
Saturation
Errors
```

Example for a database connection pool:

```text
Utilization
→ 85%

Saturation
→ 20 waiting requests

Errors
→ connection timeout
```

---

# 39. Latency Percentiles

Average latency can hide slow requests.

Instead monitor:

```text
p50
p90
p95
p99
```

Example:

```text
p50 = 80 ms
p95 = 300 ms
p99 = 1.2 s
```

The p99 shows the experience of the slowest portion of requests more clearly than the average.

---

# 40. Error Rate

Monitor:

```text
4xx
5xx
```

A high 5xx rate often indicates server-side failures.

4xx errors can indicate:

```text
Invalid client requests
Authentication failures
Authorization failures
Missing resources
```

The correct interpretation depends on the endpoint and traffic pattern.

---

# 41. Availability

A simple availability concept:

```text
Successful requests
-------------------
Total requests
```

For example:

```text
99.9%
```

Availability targets should be defined according to the system's requirements.

---

# 42. Health Check vs Monitoring

Health check:

```text
Is the application currently healthy?
```

Monitoring:

```text
How has the application behaved over time?
```

You need both.

---

# 43. Health Check vs Metrics

Health:

```text
UP / DOWN
```

Metrics:

```text
Latency
Traffic
Memory
CPU
Errors
Connections
```

A service can be:

```text
UP
```

while still having:

```text
Very high latency
High error rate
Memory pressure
```

---

# 44. Logging

Logs provide detailed event information.

Example:

```text
2026-08-20 18:20:01
INFO
OrderService
Order 101 created
```

Logs answer:

```text
What happened?
```

Metrics answer:

```text
How much/how often?
```

Traces answer:

```text
Where did the request spend time?
```

---

# 45. Observability

Observability commonly combines:

```text
Logs
Metrics
Traces
```

Conceptual model:

```text
                Observability
                      │
       ┌──────────────┼──────────────┐
       │              │              │
      Logs          Metrics        Traces
       │              │              │
    Details        Trends        Request path
```

---

# 46. Distributed Tracing

Microservices:

```text
Client
 ↓
API Gateway
 ↓
Order Service
 ↓
Payment Service
 ↓
Inventory Service
```

A trace can show the entire request path.

Useful information:

```text
Service
Duration
Errors
Database calls
External calls
```

---

# 47. Correlation ID

A correlation ID can connect logs across services:

```text
Request
 ↓
X-Correlation-Id: ABC123
 ↓
Order Service
 ↓
Payment Service
 ↓
Inventory Service
```

Every service logs:

```text
ABC123
```

This makes troubleshooting easier.

---

# 48. Structured Logging

Instead of only plain text:

```text
Order created
```

structured logs can include:

```json
{
  "event": "order_created",
  "orderId": "101",
  "userId": "55",
  "correlationId": "ABC123"
}
```

Be careful not to log sensitive information.

---

# 49. Alerting

Monitoring becomes useful when it triggers actionable alerts.

Examples:

```text
5xx rate > 5%
p95 latency > 1 second
Database connection pool exhausted
Memory usage > threshold
Application readiness DOWN
Disk space low
```

Avoid alerting on every minor event.

---

# 50. Good Alerts

A good alert should be:

```text
Actionable
Relevant
Specific
Urgent enough to require attention
```

Bad:

```text
Every single 4xx request
```

This can create alert fatigue.

---

# 51. SLI

SLI means:

```text
Service Level Indicator
```

It is a measured value representing service performance.

Examples:

```text
Request success rate
Request latency
Availability
```

---

# 52. SLO

SLO means:

```text
Service Level Objective
```

Example:

```text
99.9% of requests should succeed
```

or:

```text
95% of requests should complete within 300 ms
```

---

# 53. SLA

SLA means:

```text
Service Level Agreement
```

It is a formal agreement with customers or stakeholders.

Simple distinction:

```text
SLI → What we measure
SLO → Target
SLA → Agreement
```

---

# 54. Production Incident Flow

Example:

```text
Alert
 ↓
Check health
 ↓
Check error rate
 ↓
Check latency
 ↓
Check logs
 ↓
Check database metrics
 ↓
Check recent deployment
 ↓
Identify root cause
 ↓
Mitigate
 ↓
Fix
 ↓
Add regression test
```

---

# 55. Common Production Metrics

For a Java Spring Boot backend:

```text
HTTP request rate
HTTP error rate
HTTP latency
JVM heap
JVM non-heap
GC activity
CPU
Threads
Hikari active connections
Hikari pending connections
Database latency
External API latency
```

---

# 56. Monitoring Ecommerce Backend

For your ecommerce backend, useful metrics include:

```text
Product API latency
Product API errors
Login failures
Order creation rate
Order failure rate
Payment failures
Inventory update failures
Database connection pool
JVM memory
CPU
```

Business metrics can also be useful:

```text
Orders/minute
Cart additions/minute
Checkout conversion
Payment success rate
```

---

# 57. Actuator and ELK

If using ELK:

```text
Spring Boot
    ↓
Logs
    ↓
Logstash
    ↓
Elasticsearch
    ↓
Kibana
```

Actuator complements ELK:

```text
Actuator → metrics/health
ELK → logs/search
```

They solve different observability needs.

---

# 58. Actuator and Jenkins

Jenkins is a CI/CD tool.

A typical pipeline:

```text
Git Push
 ↓
Jenkins
 ↓
Build
 ↓
Unit Tests
 ↓
SonarQube
 ↓
Package
 ↓
Deploy
 ↓
Health Check
```

After deployment, Jenkins or deployment tooling can verify:

```text
/actuator/health
```

---

# 59. Health Check During Deployment

Example:

```text
Deploy new version
      ↓
Application starts
      ↓
Readiness check
      ↓
READY
      ↓
Traffic enabled
```

If readiness fails:

```text
Traffic not sent
```

This helps prevent unhealthy instances from receiving production traffic.

---

# 60. Graceful Shutdown

During deployment:

```text
Old instance
     ↓
Stop receiving new traffic
     ↓
Finish active requests
     ↓
Shutdown
```

Graceful shutdown reduces dropped requests during deployments.

---

# 61. Kubernetes Integration

In Kubernetes:

```text
Pod
 ↓
Liveness Probe
Readiness Probe
Startup Probe
```

Spring Boot Actuator can provide health endpoints that can be used for these probes.

Conceptually:

```text
Kubernetes
   ↓
/actuator/health/liveness

Kubernetes
   ↓
/actuator/health/readiness
```

---

# 62. Startup Probe

Startup probes are useful when an application takes a long time to initialize.

Conceptually:

```text
Application starting
       ↓
Startup probe
       ↓
Application ready
       ↓
Liveness/readiness become important
```

This prevents liveness checks from restarting an application simply because it has not finished starting yet.

---

# 63. Monitoring Database Connections

A common production issue:

```text
Too many requests
      ↓
Connection pool exhausted
      ↓
Threads wait
      ↓
Latency increases
      ↓
Timeouts
```

Monitor:

```text
Active
Idle
Pending
Max
```

and investigate slow database queries.

---

# 64. Monitoring Garbage Collection

High GC activity can cause:

```text
CPU spikes
Latency spikes
Throughput reduction
```

Monitor:

```text
GC frequency
GC pause duration
Heap usage
Old-generation behavior
```

Do not tune JVM GC based on a single metric; correlate it with application performance.

---

# 65. Memory Leak Investigation

Symptoms:

```text
Heap continuously increases
GC becomes frequent
Application slows
OutOfMemoryError
```

Possible tools:

```text
Actuator metrics
JVM metrics
Heap dump
JFR
APM
```

---

# 66. Thread Pool Monitoring

Important thread pools may include:

```text
Tomcat request threads
@Async executor
Database connection pool
HTTP client pool
```

Too many blocked threads can cause:

```text
Request queueing
Latency
Timeouts
```

---

# 67. Production Debugging Example

Problem:

```text
API latency increased
```

Check:

```text
1. HTTP p95/p99
2. 5xx rate
3. CPU
4. JVM memory
5. GC
6. Hikari connections
7. Database latency
8. External API latency
9. Recent deployment
10. Logs/traces
```

This is a much stronger debugging approach than immediately restarting the application.

---

# 68. Actuator Security Best Practices

```text
□ Expose only required endpoints
□ Protect sensitive endpoints
□ Do not expose env publicly
□ Do not expose beans publicly
□ Protect mappings
□ Protect loggers
□ Protect heap/thread diagnostics
□ Use HTTPS
□ Use authentication/authorization
□ Avoid exposing secrets
```

---

# 69. Monitoring Checklist

```text
□ Health endpoint
□ Liveness
□ Readiness
□ JVM metrics
□ HTTP metrics
□ Database metrics
□ Connection pool metrics
□ Error rate
□ Latency percentiles
□ Request rate
□ Logs
□ Correlation IDs
□ Distributed tracing
□ Alerts
□ Dashboards
□ Deployment health checks
```

---

# 70. Interview Questions

## What is Spring Boot Actuator?

> Spring Boot Actuator provides production-ready endpoints and metrics for monitoring and managing a Spring Boot application.

---

## What is /actuator/health?

> It provides the current health status of the application and can include health information from configured dependencies such as the database.

---

## What is the difference between liveness and readiness?

> Liveness tells the platform whether the application instance should remain running, while readiness tells it whether the instance is ready to receive traffic.

---

## What is Micrometer?

> Micrometer is the metrics instrumentation and abstraction layer commonly used by Spring Boot to expose application metrics to monitoring systems such as Prometheus.

---

## What is Prometheus?

> Prometheus is a monitoring and time-series database commonly used to collect and query application metrics.

---

## What is Grafana?

> Grafana is a visualization and dashboarding platform that can display metrics from systems such as Prometheus.

---

## What metrics would you monitor in a Spring Boot application?

> I would monitor request rate, error rate, latency percentiles, JVM memory, garbage collection, CPU, thread usage, database connection pool metrics, and important business metrics.

---

## Why are p95 and p99 useful?

> Average latency can hide slow requests. Percentiles such as p95 and p99 show the experience of the slower portion of traffic and help identify tail-latency problems.

---

## How would you debug a slow API?

> I would first check request latency and error rate, then correlate that with CPU, JVM and GC metrics, database connection pool and query latency, external service latency, logs, traces, and recent deployments.

---

## How do you monitor a database connection pool?

> I monitor active, idle, pending, and maximum connections and correlate them with request latency and database performance to identify pool exhaustion or slow queries.

---

## Why shouldn't we expose all Actuator endpoints?

> Some endpoints expose sensitive application and infrastructure information, such as environment properties, beans, mappings, and logger configuration. I expose only the endpoints required by monitoring and protect the rest.

---

# 71. Production Monitoring Mental Model

```text
                 Application
                      │
       ┌──────────────┼──────────────┐
       │              │              │
      Logs          Metrics        Traces
       │              │              │
       └──────────────┼──────────────┘
                      │
                Observability
                      │
              ┌───────┴───────┐
              │               │
          Dashboard         Alerts
              │               │
              └───────┬───────┘
                      │
                 Investigation
                      │
                   Recovery
```

---

# 72. Final Interview Rule

> **I use Spring Boot Actuator for health checks and operational metrics, Micrometer with a monitoring backend such as Prometheus for metrics, and centralized logs and tracing for troubleshooting. In production, I expose only the required Actuator endpoints and protect sensitive operational information.**

Next:

```text
12 Spring Boot Actuator & Production Monitoring
      ↓
13 Spring Boot Microservices
```

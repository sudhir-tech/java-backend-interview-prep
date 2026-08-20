# Spring Boot — Spring Cloud and Distributed Systems

Spring Cloud provides tools for building distributed systems with Spring Boot.

It commonly addresses problems such as:

```text
Service discovery
Configuration management
API gateways
Resilience
Load balancing
Distributed tracing
Messaging
```

A typical architecture:

```text
                         Client
                           |
                           v
                    API Gateway
                           |
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
   User Service      Order Service      Product Service
        |                  |                  |
        +------------------+------------------+
                           |
                    Message Broker
                           |
                +----------+----------+
                |                     |
                v                     v
          Payment Service      Inventory Service
```

---

# 1. Why Spring Cloud?

Spring Boot makes it easy to build individual services.

Spring Cloud adds tools for distributed-system concerns.

Think:

```text
Spring Boot
    ↓
Build an application

Spring Cloud
    ↓
Connect and operate multiple applications
```

---

# 2. Common Spring Cloud Components

Important concepts include:

```text
Spring Cloud Gateway
Spring Cloud Config
Spring Cloud LoadBalancer
Service Discovery
OpenFeign
Resilience4j integration
Distributed tracing / observability
Spring Cloud Stream
```

The exact Spring Cloud components used depend on the architecture and current project versions.

---

# 3. Service Discovery

In a microservice environment, instances may change dynamically.

Instead of:

```text
http://10.0.0.20:8081
```

a service can discover:

```text
PAYMENT-SERVICE
```

Conceptually:

```text
                 Service Registry
                /       |       \
               /        |        \
              v         v         v
          User        Order     Payment
         Service     Service    Service
```

---

# 4. Why Service Discovery?

Suppose Payment Service has:

```text
Instance 1 → 10.0.0.20
Instance 2 → 10.0.0.21
Instance 3 → 10.0.0.22
```

If the Order Service hardcodes one IP:

```text
Order → 10.0.0.20
```

it becomes difficult to handle:

```text
Scaling
Failure
Deployment
Dynamic infrastructure
```

Service discovery provides a logical service identity instead.

---

# 5. Eureka

Eureka is a service discovery solution associated with the Spring Cloud ecosystem.

Conceptually:

```text
                    Eureka Server
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
   USER-SERVICE    ORDER-SERVICE    PAYMENT-SERVICE
```

Services register themselves with Eureka.

Consumers can discover available instances.

---

# 6. Eureka Server

Conceptually:

```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryApplication {

    public static void main(String[] args) {
        SpringApplication.run(
            DiscoveryApplication.class,
            args
        );
    }
}
```

The exact setup depends on the Spring Cloud version used by the project.

---

# 7. Eureka Client

A service registers with the discovery server using Spring Cloud configuration and dependencies.

Example configuration:

```yaml
spring:
  application:
    name: order-service
```

The service name becomes its logical identity.

---

# 8. Modern Cloud-Native Discovery

Eureka is not mandatory for every modern architecture.

For example:

```text
Kubernetes Service
Cloud service discovery
DNS-based discovery
Service mesh
```

may already provide service discovery.

Interview answer:

> I understand Eureka-based discovery, but in modern Kubernetes environments I would also consider platform-native service discovery before adding another registry.

---

# 9. Client-Side Load Balancing

Suppose:

```text
PAYMENT-SERVICE

Instance 1
Instance 2
Instance 3
```

A client-side load balancer can select an available instance.

Conceptually:

```text
Order Service
      |
      v
Load Balancer
   /   |   \
  v    v    v
 P1   P2   P3
```

Spring Cloud LoadBalancer is one option.

---

# 10. Server-Side Load Balancing

Another approach:

```text
Order Service
      |
      v
Load Balancer
      |
  +---+---+
  |   |   |
  v   v   v
 P1  P2  P3
```

The infrastructure handles instance selection.

Examples:

```text
Kubernetes Service
AWS Load Balancer
NGINX
Cloud load balancer
```

---

# 11. OpenFeign

OpenFeign provides a declarative HTTP client.

Example:

```java
@FeignClient(
    name = "payment-service"
)
public interface PaymentClient {

    @PostMapping("/api/payments")
    PaymentResponse pay(
        @RequestBody PaymentRequest request
    );
}
```

Usage:

```java
PaymentResponse response =
    paymentClient.pay(request);
```

The service code does not need to manually construct every HTTP request.

---

# 12. Feign and Service Discovery

With suitable Spring Cloud configuration:

```text
Order Service
     |
PaymentClient
     |
PAYMENT-SERVICE
     |
Service Discovery
     |
Available instances
```

The logical service name can be resolved to an available instance.

---

# 13. Feign Timeout

A network call should not wait forever.

Conceptually configure:

```text
Connect timeout
Read/response timeout
```

For example:

```text
Payment call
     |
Timeout after defined limit
     |
Failure handling
```

The exact configuration depends on the HTTP client and Spring Cloud version.

---

# 14. Feign Error Handling

Remote failures should be translated into appropriate application behavior.

Possible failures:

```text
404
400
401
403
429
500
503
Timeout
Connection failure
```

Do not blindly convert every remote error into:

```text
500 Internal Server Error
```

Understand what the downstream failure means to your business operation.

---

# 15. API Gateway

Spring Cloud Gateway can act as the edge gateway.

```text
Client
  |
  v
Gateway
  |
  +------> User Service
  |
  +------> Product Service
  |
  +------> Order Service
```

---

# 16. Gateway Responsibilities

Typical responsibilities:

```text
Routing
Authentication integration
Authorization policies
Rate limiting
CORS
Request filtering
Observability
TLS termination
```

Keep business logic inside business services.

---

# 17. Gateway Route

Conceptually:

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: product-service
          uri: http://product-service
          predicates:
            - Path=/api/products/**
```

Exact configuration varies by Spring Cloud version and deployment environment.

---

# 18. Gateway Filters

Gateway filters can modify or inspect requests/responses.

Examples:

```text
Add headers
Remove headers
Rewrite path
Authenticate request
Rate limit
Log request metadata
```

Example concept:

```text
Request
  |
Gateway Filter
  |
Authentication
  |
Route
  |
Service
```

---

# 19. Gateway Authentication

A common flow:

```text
Client
  |
  | JWT
  v
API Gateway
  |
Validate token
  |
  v
Service
```

Whether authorization is enforced only at the gateway or additionally at each service depends on the security architecture.

For sensitive systems, services should not blindly trust client-controlled headers.

---

# 20. Gateway vs Service Security

Gateway:

```text
Edge authentication
Global policies
Rate limiting
```

Service:

```text
Business authorization
Resource-level permissions
Defense in depth
```

Example:

```text
Gateway → authenticated
Order Service → user can access THIS order?
```

---

# 21. Spring Cloud Config

Centralized configuration:

```text
                 Config Server
                      |
       +--------------+--------------+
       |              |              |
       v              v              v
     User           Order         Payment
```

Configuration can be stored centrally and loaded by services.

---

# 22. Configuration Server

Conceptually:

```text
Config Repository
       |
       v
Config Server
       |
       +------> User Service
       +------> Order Service
       +------> Payment Service
```

This can separate configuration from application binaries.

---

# 23. Externalized Configuration

Prefer:

```text
Environment variables
Configuration files
Config server
Secret manager
Platform configuration
```

over hardcoding:

```java
String password =
    "production-password";
```

---

# 24. Secrets vs Configuration

Not all configuration is equally sensitive.

Normal configuration:

```text
service timeout
feature flag
port
log level
```

Secrets:

```text
database password
API key
JWT private key
cloud credentials
```

Secrets should use dedicated secret-management mechanisms where possible.

---

# 25. Refreshing Configuration

Some distributed systems need configuration changes without full redeployment.

Spring Cloud has historically provided refresh mechanisms.

However, runtime configuration refresh adds complexity.

Use it only when the operational benefit justifies it.

---

# 26. Resilience

Distributed systems fail.

Common failures:

```text
Timeout
Connection refused
Service unavailable
Slow dependency
Network partition
Database unavailable
Message broker unavailable
```

Resilience patterns help limit the impact.

---

# 27. Timeout

Always define sensible timeouts.

```text
Order Service
     |
     v
Payment Service
     |
     X
No response
```

Without a timeout, resources may remain occupied too long.

---

# 28. Retry

Retry temporary failures:

```text
Request
  |
Failure
  |
Wait
  |
Retry
```

Use:

```text
Exponential backoff
Jitter
Maximum attempts
Appropriate exception/status filtering
```

---

# 29. Retry Is Not Always Safe

Do not automatically retry:

```text
POST payment
```

if the operation is not idempotent.

Otherwise:

```text
First payment succeeds
Response lost
Retry
Second payment occurs
```

This is why idempotency keys can be important.

---

# 30. Idempotency Key

Example:

```http
Idempotency-Key: ORDER-123-ABC
```

The server can record the key and ensure repeated requests do not produce duplicate business effects.

---

# 31. Circuit Breaker

Circuit breaker states:

```text
CLOSED
OPEN
HALF_OPEN
```

CLOSED:

```text
Normal traffic
```

OPEN:

```text
Fail fast
```

HALF_OPEN:

```text
Test recovery
```

---

# 32. Resilience4j Circuit Breaker

Conceptual example:

```java
@CircuitBreaker(
    name = "paymentService",
    fallbackMethod = "fallback"
)
public PaymentResponse pay(
        PaymentRequest request) {

    return paymentClient.pay(request);
}
```

The exact annotation and configuration depend on the Resilience4j/Spring Boot versions used.

---

# 33. Bulkhead

Bulkhead isolates resources.

```text
Payment calls
   |
Pool A

Notification calls
   |
Pool B
```

If Notification becomes slow:

```text
Pool B exhausted
```

Payment can continue using:

```text
Pool A
```

---

# 34. Rate Limiter

Example:

```text
100 requests/second
```

After the limit:

```text
429 Too Many Requests
```

Useful for protecting:

```text
Login
Search
Payment
Public APIs
Expensive operations
```

---

# 35. Distributed Systems and Partial Failure

Consider:

```text
Order Service    → UP
Payment Service  → DOWN
Inventory        → UP
```

The application must decide:

```text
Can order creation continue?
Should it wait?
Should it fail fast?
Should it queue the operation?
```

The answer depends on business requirements.

---

# 36. Asynchronous Communication

Instead of:

```text
Order → Payment
```

synchronously:

```text
Order
  |
  v
Event
  |
  v
Broker
  |
  v
Payment
```

This can reduce direct coupling.

---

# 37. Kafka

Kafka provides:

```text
Topics
Partitions
Producers
Consumers
Consumer groups
Offsets
```

Basic architecture:

```text
Producer
    |
    v
Kafka Topic
    |
    +----> Consumer Group A
    |
    +----> Consumer Group B
```

---

# 38. Kafka Partitions

A topic can have multiple partitions:

```text
orders
 ├── Partition 0
 ├── Partition 1
 └── Partition 2
```

Partitions allow parallel processing.

Ordering is generally guaranteed only within a partition.

---

# 39. Consumer Groups

Within one consumer group:

```text
Partition 0 → Consumer A
Partition 1 → Consumer B
Partition 2 → Consumer C
```

This enables horizontal scaling.

Different consumer groups can independently consume the same topic for different purposes.

---

# 40. Kafka Offset

Consumers track offsets representing their progress through partitions.

Conceptually:

```text
Partition
0 1 2 3 4 5 6 7
        ^
      offset
```

Offsets help consumers resume processing.

---

# 41. Event Delivery

A distributed messaging system may provide:

```text
At-most-once
At-least-once
Exactly-once-related guarantees
```

Business-level exactly-once effects are more complicated than simply configuring a messaging system.

---

# 42. Idempotent Consumer

If:

```text
OrderCreated
OrderCreated
```

arrives twice:

```java
if (processed(eventId)) {
    return;
}
```

Then process only new events.

This is important for reliable event-driven systems.

---

# 43. Dead Letter Queue

Failed messages can eventually go to:

```text
Dead Letter Queue
```

Flow:

```text
Message
  |
Processing fails
  |
Retry
  |
Retry
  |
Retry
  |
DLQ
```

The DLQ should be monitored and operationally managed.

---

# 44. Outbox Pattern

Problem:

```text
DB transaction succeeds
Event publish fails
```

Solution:

```text
DB transaction
  |
  +-- Business record
  |
  +-- Outbox event
```

Then:

```text
Outbox Publisher
      |
      v
Message Broker
```

This makes database state and event creation atomic within the local database transaction.

---

# 45. Eventual Consistency

Example:

```text
Order DB
  |
Order created
  |
Event
  |
Inventory Service
  |
Inventory DB
```

There may be a short period where:

```text
Order = created
Inventory = not updated yet
```

This is eventual consistency.

---

# 46. Distributed Transactions

Avoid trying to use one normal local transaction across:

```text
Order DB
Payment DB
Inventory DB
```

Instead consider:

```text
Saga
Outbox
Events
Compensation
Idempotency
```

---

# 47. Saga

Example:

```text
Create Order
     |
Reserve Inventory
     |
Process Payment
     |
Confirm Order
```

Failure:

```text
Payment fails
     |
Cancel Order
     |
Release Inventory
```

---

# 48. Choreography vs Orchestration

Choreography:

```text
Events drive the workflow
```

Orchestration:

```text
Coordinator drives the workflow
```

Use choreography carefully as event chains can become difficult to understand.

For complex business workflows, orchestration may provide clearer control.

---

# 49. Distributed Tracing

Example:

```text
Client
  |
  +-- Gateway: 20ms
  |
  +-- Order: 80ms
  |
  +-- Payment: 500ms
  |
  +-- Inventory: 40ms
```

Tracing helps identify where latency is introduced.

---

# 50. OpenTelemetry

OpenTelemetry provides standard instrumentation for:

```text
Traces
Metrics
Logs
```

It can propagate context between services.

Typical flow:

```text
Request
  |
Trace ID
  |
Gateway
  |
Order Service
  |
Payment Service
```

---

# 51. Correlation ID

A correlation ID can connect logs:

```text
X-Correlation-Id: ABC123
```

Logs:

```text
Order Service   ABC123
Payment Service ABC123
Inventory       ABC123
```

This is useful even when tracing is unavailable or incomplete.

---

# 52. Observability

Observability combines:

```text
Logs
Metrics
Traces
```

Think:

```text
Logs
→ What happened?

Metrics
→ How much/how often?

Traces
→ Where did the request spend time?
```

---

# 53. Health Checks

Services should expose health endpoints.

With Spring Boot Actuator:

```text
/actuator/health
/actuator/health/liveness
/actuator/health/readiness
```

These can integrate with:

```text
Kubernetes
Load balancers
Monitoring systems
Deployment pipelines
```

---

# 54. Kubernetes Service Discovery

In Kubernetes:

```text
Order Pod
   |
   v
order-service
   |
   v
Order Pods
```

The Kubernetes Service provides stable service discovery and load distribution.

This can eliminate the need for a separate Eureka deployment in many Kubernetes architectures.

---

# 55. Kubernetes Probes

Common probes:

```text
Startup probe
Liveness probe
Readiness probe
```

Startup:

```text
Has application finished starting?
```

Liveness:

```text
Should the container be restarted?
```

Readiness:

```text
Should traffic be sent here?
```

---

# 56. Configuration in Kubernetes

Common mechanisms:

```text
ConfigMap
Secret
Environment variables
Mounted configuration
```

Use appropriate secret-management controls for sensitive values.

---

# 57. Horizontal Scaling

Example:

```text
Order Service
   |
   +-- Pod 1
   +-- Pod 2
   +-- Pod 3
```

A Kubernetes Service can distribute traffic among ready pods.

---

# 58. Deployment Strategies

Common approaches:

```text
Rolling
Blue-Green
Canary
```

Spring Cloud is not responsible for every deployment concern; deployment platforms such as Kubernetes often provide these capabilities.

---

# 59. Centralized Logging

Typical architecture:

```text
Service
  |
  v
Log Collector
  |
  v
Elasticsearch / Cloud Logging
  |
  v
Kibana / Dashboard
```

For your existing ELK experience:

```text
Spring Boot
   ↓
Logstash
   ↓
Elasticsearch
   ↓
Kibana
```

This is especially useful for production troubleshooting.

---

# 60. Metrics

Typical metrics:

```text
Request rate
Error rate
p95 latency
p99 latency
CPU
Memory
GC
Threads
Database connections
External API latency
```

Prometheus is a common metrics backend.

---

# 61. API Gateway Security

A gateway can perform:

```text
JWT validation
Rate limiting
CORS
Request filtering
```

But sensitive authorization should still be enforced by the service responsible for the resource.

Example:

```text
Gateway:
User authenticated

Order Service:
Does user own order 101?
```

---

# 62. Secret Management

Do not put:

```text
password=admin123
```

in Git.

Use:

```text
Vault
AWS Secrets Manager
Azure Key Vault
Kubernetes Secrets
```

and apply appropriate access control.

---

# 63. API Versioning

Examples:

```text
/api/v1/orders
/api/v2/orders
```

Good API evolution should consider:

```text
Backward compatibility
Deprecation
Consumer migration
Contract testing
```

---

# 64. Contract Testing

Contract testing verifies that consumers and providers agree on:

```text
Request
Response
Schema
Status
Headers
```

This can catch breaking changes before deployment.

---

# 65. Spring Cloud Contract

Spring Cloud Contract is one option in the Spring ecosystem for contract testing.

Conceptually:

```text
Consumer expectations
        |
        v
Contract
        |
        v
Provider verification
```

Use it where contract testing provides value.

---

# 66. Microservice Failure Example

Suppose:

```text
Order Service
     |
     v
Payment Service
     |
     X
Timeout
```

A resilient flow could be:

```text
Timeout
   |
Retry if safe
   |
Circuit breaker
   |
Business failure / async retry
```

The exact response depends on whether the payment operation is safely retryable.

---

# 67. Avoid Cascading Failures

Bad architecture:

```text
A
|
B
|
C
|
D
|
E
```

Every request synchronously waits for every dependency.

If E fails:

```text
E fails
↓
D waits
↓
C waits
↓
B waits
↓
A waits
```

Resources can be exhausted across the system.

---

# 68. Better Failure Isolation

Use:

```text
Timeouts
Circuit breakers
Bulkheads
Async messaging
Caching
Fallbacks where appropriate
```

to limit cascading failures.

---

# 69. Stateless Services

A stateless service can be scaled horizontally:

```text
Request 1 → Instance A
Request 2 → Instance B
Request 3 → Instance C
```

Shared state should normally be externalized to:

```text
Database
Redis
Object storage
Message broker
```

depending on the use case.

---

# 70. Distributed Locks

For tasks that must execute once across multiple instances:

```text
Instance A
Instance B
Instance C
```

a distributed lock may coordinate execution.

But where possible, design operations to be idempotent rather than relying heavily on distributed locking.

---

# 71. Scheduled Jobs

Example:

```java
@Scheduled(cron = "0 0 * * * *")
public void processOrders() {
}
```

With multiple service instances, this can execute multiple times.

Possible approaches:

```text
Distributed lock
Dedicated scheduler
Platform scheduler
Idempotent processing
```

---

# 72. Service Boundaries

Good:

```text
Order Service
→ owns orders

Inventory Service
→ owns inventory

Payment Service
→ owns payments
```

Bad:

```text
Order Service
→ directly updates Inventory DB

Order Service
→ directly updates Payment DB
```

---

# 73. Avoid Shared Database Coupling

Bad:

```text
Order Service
       |
       v
Shared Database
       ^
       |
Payment Service
```

Better:

```text
Order Service → Order DB
Payment Service → Payment DB
```

and communicate through:

```text
APIs
Events
```

---

# 74. Service Contract Example

Payment API:

```http
POST /api/payments
```

Request:

```json
{
  "orderId": 101,
  "amount": 999.00
}
```

Response:

```json
{
  "paymentId": "PAY-1001",
  "status": "SUCCESS"
}
```

Contract should define:

```text
Schema
Status codes
Errors
Authentication
Idempotency behavior
```

---

# 75. Business Metrics

Technical metrics are not enough.

For ecommerce, also monitor:

```text
Orders/minute
Payment success rate
Checkout failure rate
Inventory reservation failures
Cart conversion
```

These connect technical health to business outcomes.

---

# 76. SLI / SLO / SLA

SLI:

```text
Measured indicator
```

Example:

```text
Request success rate
```

SLO:

```text
Target
```

Example:

```text
99.9% successful requests
```

SLA:

```text
Formal agreement
```

Simple interview model:

```text
SLI → What we measure
SLO → What we target
SLA → What we promise
```

---

# 77. Production Architecture

A practical cloud-native architecture might be:

```text
                         Client
                           |
                           v
                    Load Balancer
                           |
                           v
                    API Gateway
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
      User Service     Order Service    Product Service
          |                |                |
        User DB          Order DB       Product DB
                           |
                   +-------+-------+
                   |               |
                   v               v
             Payment Service  Inventory Service
                   |               |
               Payment DB      Inventory DB

                    Message Broker
                         |
                    Event Consumers

Observability:
Logs + Metrics + Traces + Alerts
```

---

# 78. Production Checklist

```text
□ Clear service boundaries
□ Service-owned data
□ API contracts
□ Timeouts
□ Controlled retries
□ Circuit breakers
□ Rate limiting
□ Bulkheads where useful
□ Idempotency
□ Saga where appropriate
□ Outbox where appropriate
□ Dead-letter handling
□ Centralized logs
□ Metrics
□ Distributed tracing
□ Correlation IDs
□ Health checks
□ Secret management
□ API versioning
□ Contract testing
□ CI/CD
□ Containerization
□ Horizontal scaling
□ Failure testing
```

---

# 79. Interview: What Is Spring Cloud?

> Spring Cloud provides tools and integrations that help build distributed systems with Spring, including service discovery, API gateways, configuration management, load balancing, resilience, and cloud-native communication patterns.

---

# 80. Interview: What Is Eureka?

> Eureka is a service discovery mechanism where services register themselves and clients can discover available service instances dynamically. In Kubernetes environments, I would also consider native Kubernetes service discovery.

---

# 81. Interview: What Is OpenFeign?

> OpenFeign provides a declarative HTTP client. Instead of manually constructing HTTP requests, I define an interface with mappings and call the interface like a normal Java dependency.

---

# 82. Interview: What Is an API Gateway?

> An API Gateway is a common entry point for clients. It handles routing and cross-cutting concerns such as authentication integration, rate limiting, and observability, while business logic remains in the services.

---

# 83. Interview: How Do You Handle Service Failure?

> I start with sensible timeouts, then use retries only for safe transient failures. Depending on the dependency, I can add circuit breakers, bulkheads, asynchronous messaging, and safe fallbacks to prevent cascading failures.

---

# 84. Interview: How Do You Handle Distributed Transactions?

> I avoid trying to make one local database transaction span multiple services. I use patterns such as Saga, events, compensation, idempotency, and the Outbox Pattern depending on the workflow.

---

# 85. Interview: Why Is Idempotency Important?

> A request or event can be delivered more than once because of retries or at-least-once messaging. Idempotent processing ensures repeated delivery doesn't create duplicate business effects.

---

# 86. Interview: How Do You Debug a Slow Microservice Request?

> I start with request rate, error rate, and latency percentiles, then use distributed tracing to identify the slow service. After that I check database latency, connection pools, JVM metrics, external dependencies, logs, and recent deployments.

---

# 87. Interview: Why Use Kafka?

> Kafka is useful for high-throughput event streaming and asynchronous communication. It provides topics, partitions, consumer groups, and durable offsets, which allow services to process events independently and scale consumers.

---

# 88. Interview: What Is Eventual Consistency?

> Eventual consistency means different services can temporarily have different views of data because updates propagate asynchronously. The system eventually converges once the relevant events are processed.

---

# 89. Interview: What Is the Outbox Pattern?

> The Outbox Pattern stores the business change and its corresponding event in the same local database transaction. A separate process publishes the stored event to the broker, reducing the chance of losing an event after a successful database update.

---

# 90. Final Mental Model

```text
                    DISTRIBUTED SYSTEM
                           |
       +-------------------+-------------------+
       |                   |                   |
 Configuration        Communication        Resilience
       |                   |                   |
 Config Server        REST / Feign        Timeout
 Environment          Kafka / Events       Retry
 Secrets              Gateway             Circuit Breaker
                                             |
                                             v
                                      Failure Isolation
       |
       +-------------------+-------------------+
                           |
                    Observability
                           |
             +-------------+-------------+
             |             |             |
            Logs         Metrics       Traces
             |             |             |
             +-------------+-------------+
                           |
                         Alerts
```

---

# 91. Final Interview Rule

> **I treat Spring Cloud as a toolbox for distributed-system concerns rather than simply a collection of annotations. I focus on service discovery, API routing, configuration, resilient communication, asynchronous events, data ownership, observability, and failure handling. The goal is to build services that can evolve and fail independently without turning the system into a distributed monolith.**

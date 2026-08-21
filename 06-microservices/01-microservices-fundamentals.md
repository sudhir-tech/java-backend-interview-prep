# Microservices — Fundamentals

This is the first file in the Microservices interview-preparation series.

The goal is to understand not just the definition of microservices, but why companies use them, what problems they solve, and what new problems they introduce.

---

# 1. What Are Microservices?

Microservices is an architectural style where an application is divided into multiple small, independently deployable services.

Each service:

```text
Owns a specific business capability
Has a clear responsibility
Can be developed independently
Can be deployed independently
Can often scale independently
```

Example e-commerce system:

```text
                    API Gateway
                         |
        +----------------+----------------+
        |                |                |
   User Service     Product Service   Order Service
        |                |                |
     User DB         Product DB        Order DB
```

Instead of one huge application handling everything, business capabilities are separated.

---

# 2. Simple Example

A monolithic e-commerce application might contain:

```text
E-commerce Application
│
├── User
├── Product
├── Cart
├── Order
├── Payment
└── Notification
```

All modules may run inside:

```text
one application
one deployment
possibly one database
```

A microservices architecture could become:

```text
User Service
Product Service
Cart Service
Order Service
Payment Service
Notification Service
```

Each service can be deployed separately.

---

# 3. Why Do Companies Use Microservices?

Common reasons:

```text
Independent deployment
Independent scaling
Team ownership
Technology flexibility
Fault isolation
Faster development for large organizations
```

For example:

```text
Product Service → heavily read
Payment Service → highly sensitive
Notification Service → asynchronous
```

Each can be designed and scaled according to its workload.

---

# 4. Microservices Are Not Just "Many Small APIs"

A common interview trap:

> "If I split my application into five REST APIs, is it microservices?"

Not necessarily.

Good microservices are usually organized around:

```text
Business capabilities
+
Independent ownership
+
Independent deployment
+
Clear boundaries
```

Simply creating many small services without good boundaries can create a distributed monolith.

---

# 5. What Is a Monolith?

A monolithic application is typically deployed as one application unit.

Example:

```text
              E-commerce App
                    |
       +------------+------------+
       |            |            |
     Users       Products      Orders
       |            |            |
       +------------+------------+
                    |
                Database
```

A change to one part may require redeploying the whole application.

---

# 6. Monolith vs Microservices

| Monolith | Microservices |
|---|---|
| One deployment unit | Multiple deployment units |
| Usually centralized codebase | Multiple services |
| Often simpler initially | More operational complexity |
| Easier local development | Distributed development |
| Simple transactions | Distributed transactions are harder |
| Scaling often at app level | Services can scale independently |
| Failure often within app | Failures can propagate across services |
| Simpler debugging | Distributed tracing often required |

Important:

> Microservices are not automatically better than monoliths.

---

# 7. When Is a Monolith Better?

A monolith can be a good choice when:

```text
Small team
Small product
Unclear requirements
Low operational complexity
Early-stage startup
Simple deployment needs
```

A well-structured modular monolith can be an excellent architecture.

---

# 8. What Is a Modular Monolith?

A modular monolith is one deployable application with strong internal boundaries.

Example:

```text
                 Application
                     |
      +--------------+--------------+
      |              |              |
   User Module   Order Module   Product Module
      |              |              |
      +--------------+--------------+
                     |
                 Database
```

It can provide many organizational benefits of microservices without immediately introducing distributed-system complexity.

---

# 9. Microservices Trade-Off

Microservices solve some problems but introduce others.

### Benefits

```text
Independent deployment
Independent scaling
Team autonomy
Fault isolation
Technology flexibility
```

### Costs

```text
Network calls
Distributed failures
Observability complexity
Deployment complexity
Data consistency challenges
Distributed transactions
More infrastructure
```

A strong interview answer always mentions both sides.

---

# 10. Service Boundary

One of the most important microservices decisions is:

```text
Where should one service end
and another service begin?
```

Good boundaries usually follow:

```text
Business capability
Domain ownership
Data ownership
Team ownership
```

---

# 11. Example: E-Commerce Boundaries

Possible services:

```text
User Service
    → users, profiles

Product Service
    → catalog, inventory

Order Service
    → orders, order lifecycle

Payment Service
    → payments

Notification Service
    → email/SMS/push notifications
```

Each service has a focused responsibility.

---

# 12. Don't Split Too Early

Bad decomposition:

```text
Name Service
Price Service
Address Service
Email Service
```

just because each table becomes a service.

This can create:

```text
Too many network calls
Hard deployment
Complex transactions
Operational overhead
```

The boundary should be based on business responsibility, not merely database tables.

---

# 13. What Is a Distributed System?

Once your application has multiple services communicating over a network, you're dealing with distributed-system problems.

Example:

```text
Order Service
      |
      | HTTP
      ↓
Payment Service
```

The payment service can:

```text
Be slow
Be unavailable
Return errors
Timeout
Return duplicate responses
Have network problems
```

A method call inside a monolith doesn't normally have these network failure modes.

---

# 14. Network Call vs Method Call

Monolith:

```java
paymentService.processPayment();
```

Microservice:

```text
Order Service
      |
      | HTTP
      ↓
Payment Service
```

The second is fundamentally different because:

```text
Network can fail
Latency exists
Serialization exists
Timeouts exist
Retries can happen
Authentication is required
```

---

# 15. Synchronous Communication

Example:

```text
Order Service
      |
      | REST
      ↓
Payment Service
      |
      ↓
Response
```

The caller waits for the response.

Common technologies:

```text
REST/HTTP
gRPC
```

---

# 16. Asynchronous Communication

Example:

```text
Order Service
      |
      | Event
      ↓
Kafka
      |
      +------→ Notification Service
      |
      +------→ Analytics Service
```

The producer doesn't need to wait for every consumer.

Common technologies:

```text
Kafka
RabbitMQ
AWS SQS/SNS
```

---

# 17. Synchronous vs Asynchronous

### Synchronous

```text
Request
 ↓
Service B
 ↓
Wait
 ↓
Response
```

Advantages:

```text
Simple request/response
Easy to understand
Immediate result
```

Disadvantages:

```text
Tight runtime dependency
Latency
Timeouts
Failure propagation
```

### Asynchronous

```text
Producer
 ↓
Message/Event
 ↓
Broker
 ↓
Consumer
```

Advantages:

```text
Loose coupling
Better decoupling
Buffering
Independent processing
```

Disadvantages:

```text
Eventual consistency
More complex debugging
Duplicate messages
Ordering concerns
Retry handling
```

---

# 18. What Is an API Gateway?

An API Gateway is an entry point for client requests.

Example:

```text
Mobile/Web Client
       |
       ↓
 API Gateway
       |
 +-----+------+------+
 |            |      |
User        Product  Order
Service     Service  Service
```

It can handle:

```text
Routing
Authentication
Authorization
Rate limiting
Request transformation
Load balancing integration
Observability
```

---

# 19. Why Use an API Gateway?

Without a gateway:

```text
Client
 ├── User Service
 ├── Product Service
 ├── Order Service
 └── Payment Service
```

The client must know every service.

With a gateway:

```text
Client
   ↓
Gateway
   ↓
Services
```

The client gets a simpler entry point.

---

# 20. What Is Service Discovery?

In dynamic environments, service instances can change.

Example:

```text
Order Service
   ↓
Where is Payment Service?
```

Service discovery provides a way to locate service instances.

Examples:

```text
Eureka
Consul
Kubernetes Service Discovery
```

---

# 21. Client-Side Discovery

Conceptually:

```text
Order Service
      |
      ↓
Service Registry
      |
      ↓
Payment instances
```

The client/service selects an instance.

---

# 22. Server-Side Discovery

Conceptually:

```text
Order Service
      |
      ↓
Load Balancer
      |
   +--+--+
   |     |
Payment Payment
   1       2
```

The client doesn't need to know individual service instances.

---

# 23. Load Balancing

Suppose Payment Service has:

```text
Payment-1
Payment-2
Payment-3
```

Traffic can be distributed across them.

Possible strategies:

```text
Round robin
Least connections
Weighted routing
Hash-based routing
```

The exact algorithm depends on the infrastructure.

---

# 24. Fault Isolation

One benefit of microservices is limiting failures.

Example:

```text
Notification Service DOWN
        ↓
Order Service
        ↓
Order can still succeed
```

if notification is asynchronous and non-critical.

But if:

```text
Order Service
      ↓
Payment Service
```

is a required synchronous dependency, payment failure may prevent order completion.

Good service design identifies which dependencies are critical.

---

# 25. Cascading Failure

A dangerous pattern:

```text
Service A
 ↓
Service B
 ↓
Service C
 ↓
Service D
```

If D becomes slow:

```text
D slow
 ↓
C waits
 ↓
B waits
 ↓
A waits
 ↓
Thread pools exhausted
```

This can cause cascading failure.

---

# 26. How Do You Prevent Cascading Failure?

Common resilience mechanisms:

```text
Timeout
Circuit breaker
Retry with backoff
Bulkhead
Rate limiting
Fallback
Async processing
```

These will be covered in detail later.

---

# 27. What Is a Circuit Breaker?

A circuit breaker prevents repeated calls to an unhealthy dependency.

Typical states:

```text
CLOSED
  ↓
OPEN
  ↓
HALF_OPEN
  ↓
CLOSED
```

---

# 28. Circuit Breaker States

### CLOSED

Requests flow normally.

```text
A → B
```

Failures are monitored.

### OPEN

Too many failures.

```text
A → X
```

Calls fail fast instead of hitting B.

### HALF_OPEN

After a recovery period:

```text
A → B
```

A limited number of test requests are allowed.

If successful:

```text
HALF_OPEN → CLOSED
```

If failures continue:

```text
HALF_OPEN → OPEN
```

---

# 29. What Is a Retry?

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

Retries are useful for transient failures.

But retries can make outages worse if used blindly.

---

# 30. Retry with Backoff

Bad:

```text
Retry immediately
Retry immediately
Retry immediately
```

Better:

```text
Attempt 1
 ↓
wait
 ↓
Attempt 2
 ↓
longer wait
 ↓
Attempt 3
```

Exponential backoff is common.

Adding jitter helps prevent many clients from retrying simultaneously.

---

# 31. What Is Bulkhead?

Bulkhead isolation prevents one dependency from consuming all resources.

Example:

```text
Payment calls → Thread Pool A
Notification calls → Thread Pool B
```

If notification becomes slow:

```text
Notification pool exhausted
```

Payment can still have available resources.

The idea comes from compartmentalizing a ship.

---

# 32. What Is Timeout?

Never allow an external service call to wait indefinitely.

Example:

```text
Order Service
   |
   | timeout = 2 sec
   ↓
Payment Service
```

If Payment doesn't respond:

```text
Timeout
```

Then apply the appropriate fallback/error strategy.

---

# 33. Idempotency

An operation is idempotent when repeating it produces the same intended final result.

Example:

```text
PUT /users/10
```

setting:

```json
{
  "name": "Sudhir"
}
```

Repeated requests can result in the same final state.

---

# 34. Why Is Idempotency Important in Microservices?

Suppose:

```text
Order Service
 ↓
Payment Service
```

Payment succeeds but the response is lost.

Order Service retries.

Without idempotency:

```text
Payment charged twice
```

With an idempotency key:

```text
payment request ID = ABC123
```

Payment Service can recognize the duplicate.

---

# 35. Database per Service

A common microservices principle:

```text
User Service
   ↓
User DB

Order Service
   ↓
Order DB

Payment Service
   ↓
Payment DB
```

Each service owns its data.

---

# 36. Why Database-per-Service?

It provides:

```text
Data ownership
Loose coupling
Independent schema evolution
Independent scaling
Service autonomy
```

But it makes cross-service queries and transactions harder.

---

# 37. Shared Database Problem

Suppose:

```text
Order Service
      |
      ↓
Shared DB
      ↑
      |
Payment Service
```

Now both services can depend on the same tables.

Problems:

```text
Tight coupling
Schema coordination
Independent deployment becomes harder
Cross-service ownership becomes unclear
```

A shared database can be useful in some transitional architectures, but it weakens service autonomy.

---

# 38. How Do Services Get Data from Another Service?

Avoid directly querying another service's database.

Instead:

```text
Order Service
     |
     | API
     ↓
User Service
```

or:

```text
User Service
     |
     | Event
     ↓
Message Broker
     ↓
Order Service
```

The correct approach depends on consistency and access requirements.

---

# 39. Distributed Transactions

In a monolith:

```text
Order
Payment
Inventory
```

could potentially share one database transaction.

In microservices:

```text
Order DB
Payment DB
Inventory DB
```

one ACID transaction cannot normally span all of them without distributed transaction infrastructure.

This is one of the major microservices challenges.

---

# 40. Saga Pattern

Saga manages a business transaction as a sequence of local transactions.

Example:

```text
Create Order
   ↓
Reserve Inventory
   ↓
Process Payment
   ↓
Confirm Order
```

If payment fails:

```text
Cancel Order
+
Release Inventory
```

These are compensating actions.

---

# 41. Choreography vs Orchestration

### Choreography

Services react to events:

```text
OrderCreated
     ↓
Inventory Service

InventoryReserved
     ↓
Payment Service

PaymentCompleted
     ↓
Order Service
```

No central coordinator.

### Orchestration

A central orchestrator controls the workflow:

```text
             Order Saga
                 |
       +---------+---------+
       ↓         ↓         ↓
    Order    Inventory   Payment
```

Both approaches have trade-offs.

---

# 42. Eventual Consistency

In distributed systems, different services may temporarily have different views of data.

Example:

```text
Order DB:
PAID

Analytics DB:
still PROCESSING
```

After event propagation:

```text
Analytics DB:
PAID
```

This is eventual consistency.

---

# 43. Strong vs Eventual Consistency

Strong consistency:

```text
Read immediately reflects the committed update.
```

Eventual consistency:

```text
Updates propagate asynchronously.
```

Microservices often use eventual consistency where appropriate.

---

# 44. Distributed Tracing

A request can travel:

```text
Client
 ↓
Gateway
 ↓
Order Service
 ↓
Payment Service
 ↓
Notification Service
```

A trace ID allows you to follow the request across services.

Common technologies:

```text
OpenTelemetry
Jaeger
Zipkin
```

---

# 45. Correlation ID

A correlation/request ID can be passed across service calls.

Example:

```text
X-Correlation-ID: abc-123
```

Logs across services can then be connected.

---

# 46. Centralized Logging

Instead of checking:

```text
Service A logs
Service B logs
Service C logs
```

centralized logging collects logs into one system.

Common stack:

```text
Application
 ↓
Log collector
 ↓
Elasticsearch/OpenSearch
 ↓
Kibana/OpenSearch Dashboards
```

---

# 47. Health Checks

Microservices should expose health information.

Spring Boot commonly provides:

```text
Spring Boot Actuator
```

Useful concepts:

```text
Liveness
Readiness
Health
Metrics
```

---

# 48. Liveness vs Readiness

Liveness:

```text
Should this instance be restarted?
```

Readiness:

```text
Can this instance receive traffic?
```

Example:

```text
Application started
but database unavailable
```

It may be:

```text
Alive
but not ready
```

depending on the health configuration.

---

# 49. Configuration Management

Microservices often need:

```text
Database URL
Service URLs
Timeouts
Feature flags
Credentials
```

Don't hardcode environment-specific configuration.

Common approaches:

```text
Environment variables
Spring Cloud Config
Kubernetes ConfigMaps/Secrets
Cloud secret managers
```

---

# 50. Secrets

Never commit:

```text
Database password
JWT secret
API keys
Cloud credentials
```

to Git.

Use:

```text
Secret manager
Environment variables
Kubernetes Secrets
Cloud secret services
```

with appropriate security controls.

---

# 51. Independent Deployment

One major microservices benefit:

```text
Product Service v5
```

can be deployed while:

```text
Order Service v3
Payment Service v7
```

continue running.

But this requires backward-compatible APIs/events.

---

# 52. Backward Compatibility

Suppose Product Service changes:

```json
{
  "productName": "Phone"
}
```

to:

```json
{
  "name": "Phone"
}
```

If Order Service still expects:

```text
productName
```

the system can break.

Therefore:

```text
API versioning
Backward-compatible changes
Consumer-driven contracts
```

are important.

---

# 53. API Versioning

Common approaches:

```text
/api/v1/products
/api/v2/products
```

or:

```text
Header-based versioning
```

or:

```text
Content negotiation
```

The best approach depends on the API governance strategy.

---

# 54. Contract Testing

Contract testing verifies that:

```text
Provider
```

and:

```text
Consumer
```

agree on the API contract.

A common technology:

```text
Pact
```

This helps catch breaking changes before deployment.

---

# 55. Microservices and Testing

Testing levels:

```text
Unit tests
 ↓
Integration tests
 ↓
Contract tests
 ↓
End-to-end tests
```

Don't rely entirely on end-to-end tests because they can become slow and fragile.

---

# 56. Service Ownership

A healthy microservices architecture usually has clear ownership.

Example:

```text
Team A → User Service
Team B → Product Service
Team C → Order Service
Team D → Payment Service
```

Ownership includes:

```text
Code
Deployment
Monitoring
On-call
Data
API contract
```

---

# 57. Conway's Law

A useful architecture principle:

> Organizations tend to design systems that mirror their communication structures.

If teams are tightly coupled, service boundaries may become tightly coupled too.

Microservices work best when team ownership and service boundaries are aligned.

---

# 58. Distributed Monolith

A distributed monolith looks like:

```text
Service A
 ↓
Service B
 ↓
Service C
 ↓
Service D
```

and:

```text
Every deployment requires every service
Every request depends on every service
All services share a database
```

You have distributed complexity without independent benefits.

---

# 59. Signs of a Distributed Monolith

```text
Many synchronous dependencies
Shared database
Coupled releases
Cross-service transactions everywhere
Circular dependencies
Services cannot run independently
```

---

# 60. Microservice Granularity

Too large:

```text
Mega Service
```

Too small:

```text
100 tiny services
```

The goal is an appropriate business boundary.

There is no universal number of microservices.

---

# 61. Microservices and Team Size

Microservices introduce operational overhead.

For a small team:

```text
10 services
```

may create unnecessary complexity.

For a large organization:

```text
20 teams
50 services
```

may provide valuable independent ownership.

Architecture should match organizational scale and business needs.

---

# 62. Common Microservices Technologies

### Spring ecosystem

```text
Spring Boot
Spring Cloud
Spring Cloud Gateway
Spring Cloud Config
Spring Cloud OpenFeign
Resilience4j
Spring Kafka
Spring Security
Actuator
```

### Infrastructure

```text
Docker
Kubernetes
Kafka
Redis
Prometheus
Grafana
OpenTelemetry
Jaeger
```

You don't need every tool to build microservices.

---

# 63. Spring Cloud OpenFeign

Feign can simplify HTTP service-to-service communication.

Conceptually:

```java
@FeignClient(name = "payment-service")
public interface PaymentClient {

    @PostMapping("/payments")
    PaymentResponse process(
        @RequestBody PaymentRequest request
    );
}
```

The application calls an interface instead of manually creating every HTTP request.

Resilience and timeouts still need explicit consideration.

---

# 64. REST Communication

Example:

```text
Order Service
POST /payments
        ↓
Payment Service
```

Typical concerns:

```text
Timeout
Authentication
Retries
Idempotency
Error handling
Versioning
Tracing
```

---

# 65. Error Handling Between Services

Don't simply return:

```text
500 Internal Server Error
```

for every failure.

Distinguish:

```text
4xx → client/request problem
5xx → service/server problem
Timeout → dependency problem
```

Use consistent error contracts.

---

# 66. Retry Only Appropriate Operations

Safe candidates often include operations that are:

```text
Idempotent
```

or:

```text
Known transient failures
```

Danger:

```text
POST payment
```

being retried without an idempotency mechanism.

Could result in duplicate side effects.

---

# 67. Microservices Security

Common layers:

```text
API Gateway
 ↓
Authentication
 ↓
Authorization
 ↓
Service-to-service security
 ↓
Database security
```

Common technologies:

```text
OAuth 2.0
OpenID Connect
JWT
mTLS
Spring Security
```

---

# 68. JWT in Microservices

A gateway or authentication service can validate a JWT.

The token may contain:

```text
user ID
roles
scopes
expiration
```

Services can validate the token or rely on trusted gateway/service security depending on architecture.

Don't blindly trust arbitrary headers from clients.

---

# 69. Service-to-Service Authentication

Options include:

```text
OAuth2 client credentials
mTLS
Service identity
Signed tokens
Cloud IAM
```

The correct choice depends on infrastructure and threat model.

---

# 70. Microservices Deployment

Typical flow:

```text
Code
 ↓
Git
 ↓
CI
 ↓
Build
 ↓
Test
 ↓
Docker image
 ↓
Container registry
 ↓
Kubernetes
 ↓
Service
```

---

# 71. Horizontal Scaling

If Order Service has high traffic:

```text
Order-1
Order-2
Order-3
Order-4
```

A load balancer distributes requests.

The service should generally be stateless or externalize state so instances can scale horizontally.

---

# 72. Stateless Services

Avoid storing user session state only in:

```text
local memory
```

because:

```text
Request 1 → Instance A
Request 2 → Instance B
```

Instance B won't have A's memory.

Use:

```text
JWT
Redis
Database
External session store
```

when state must be shared.

---

# 73. Service Resilience Checklist

For every synchronous dependency ask:

```text
Timeout?
Retry?
Backoff?
Circuit breaker?
Bulkhead?
Fallback?
Idempotency?
Tracing?
```

This is a great interview checklist.

---

# 74. Microservices Architecture Example

E-commerce:

```text
                         Client
                           |
                           ↓
                     API Gateway
                           |
          +----------------+----------------+
          |                |                |
          ↓                ↓                ↓
     User Service    Product Service    Order Service
          |                |                |
       User DB          Product DB       Order DB
                                             |
                                             ↓
                                      Payment Service
                                             |
                                         Payment DB

                    Order Service
                         |
                         ↓
                       Kafka
                    /     |      \
                   ↓      ↓       ↓
            Notification  Analytics  Audit
```

---

# 75. Example Order Flow

A possible flow:

```text
1. Client → POST /orders
2. Gateway authenticates request
3. Order Service validates request
4. Inventory is reserved
5. Payment is processed
6. Order is confirmed
7. OrderCreated/OrderConfirmed event published
8. Notification consumes event
9. Analytics consumes event
```

Depending on requirements, payment/inventory may be synchronous or part of a Saga workflow.

---

# 76. What Happens if Payment Fails?

Possible flow:

```text
Create Order
   ↓
Reserve Inventory
   ↓
Payment fails
   ↓
Release Inventory
   ↓
Cancel Order
```

Those release/cancel operations are compensating actions.

---

# 77. What Happens if Notification Fails?

If notification isn't business-critical:

```text
Order succeeds
 ↓
Event published
 ↓
Notification fails
 ↓
Retry asynchronously
```

The customer order doesn't necessarily need to fail.

This is a good example of separating critical and non-critical dependencies.

---

# 78. Microservices Interview Question

### "What are the disadvantages of microservices?"

Strong answer:

> Microservices introduce distributed-system complexity. We have network latency and failures, harder debugging, distributed transactions, eventual consistency, deployment and monitoring overhead, and more infrastructure. I'd choose microservices when the business and organizational benefits justify that complexity.

---

# 79. Microservices Interview Question

### "Why not use microservices for everything?"

Strong answer:

> Because microservices aren't free. For a small application or team, a modular monolith can be simpler and more productive. I'd introduce microservices when independent scaling, deployment, ownership or domain boundaries provide a clear benefit.

---

# 80. Microservices Interview Question

### "How do microservices communicate?"

Answer:

> They can communicate synchronously using REST or gRPC, or asynchronously using messaging systems such as Kafka or RabbitMQ. The choice depends on latency, coupling, reliability and consistency requirements.

---

# 81. Microservices Interview Question

### "How do you handle service failure?"

Answer:

> I'd use timeouts, retries with backoff where appropriate, circuit breakers, bulkheads and graceful fallbacks. For non-critical workflows I'd prefer asynchronous communication where possible.

---

# 82. Microservices Interview Question

### "How do you maintain data consistency?"

Answer:

> Within a service I use local database transactions. Across services I avoid tightly coupled distributed transactions where possible and use patterns such as Saga, transactional outbox and idempotent consumers depending on the workflow.

---

# 83. Microservices Interview Question

### "Why database per service?"

Answer:

> It establishes clear data ownership and allows each service to evolve and scale independently. The trade-off is that cross-service queries and transactions become more difficult.

---

# 84. Microservices Interview Question

### "What is a Saga?"

Answer:

> A Saga is a distributed business transaction implemented as a sequence of local transactions. If a later step fails, compensating actions are executed to undo the business effect of earlier steps.

---

# 85. Microservices Interview Question

### "What is API Gateway?"

Answer:

> An API Gateway provides a common entry point for clients and can handle routing, authentication, rate limiting, request transformation and observability before forwarding requests to backend services.

---

# 86. Microservices Interview Question

### "What is service discovery?"

Answer:

> Service discovery allows services to locate available instances dynamically rather than relying on hardcoded host addresses. It can be implemented using systems such as Eureka, Consul or Kubernetes service discovery.

---

# 87. Microservices Interview Question

### "What is circuit breaker?"

Answer:

> A circuit breaker prevents repeated calls to an unhealthy dependency. It opens after a failure threshold, fails fast while open, and later allows limited requests in half-open state to test recovery.

---

# 88. Microservices Interview Question

### "What is idempotency?"

Answer:

> Idempotency means repeating the same operation doesn't create additional unintended side effects. It's especially important for payment, order and event-processing workflows where retries can occur.

---

# 89. Microservices Interview Question

### "What is eventual consistency?"

Answer:

> Eventual consistency means different services or replicas may temporarily have different states, but the system converges to a consistent state after updates propagate.

---

# 90. Microservices Interview Question

### "How do you monitor microservices?"

Answer:

> I use centralized logs, metrics, distributed tracing and health checks. I'd monitor latency, error rate, throughput, resource utilization, dependency failures and database performance.

---

# 91. Microservices Interview Question

### "How do you debug a request across services?"

Answer:

> I'd use a correlation ID or distributed trace ID propagated through the request. Then I can follow the request from gateway to each service and identify where latency or failure occurred.

---

# 92. Microservices Interview Question

### "How do you deploy microservices?"

Answer:

> A typical pipeline is Git → CI → tests → build Docker image → push to registry → deploy to Kubernetes or another container platform. Each service can have its own deployment pipeline and scaling configuration.

---

# 93. Microservices Interview Question

### "How do you avoid breaking other services?"

Answer:

> I maintain backward-compatible APIs and events, use versioning when necessary, contract testing, consumer/provider testing and gradual deployment strategies.

---

# 94. Microservices Interview Question

### "What is a distributed monolith?"

Answer:

> It's an architecture where services are technically separate but remain tightly coupled through shared databases, synchronous dependencies and coordinated releases. It gets the complexity of microservices without many of their benefits.

---

# 95. Microservices Interview Question

### "How would you split an e-commerce monolith?"

Answer:

> I'd first identify business capabilities and domain boundaries rather than simply splitting database tables. I'd consider users, catalog, orders, payments and notifications as potential boundaries, then extract services gradually based on business value and team ownership.

---

# 96. Important Rule: Don't Say This

Avoid:

> "Microservices are always better because they are scalable."

Better:

> "Microservices can provide independent scaling, but they introduce distributed-system complexity. I'd use them when the independent scaling, deployment and ownership benefits justify that complexity."

---

# 97. Important Rule: Don't Say This

Avoid:

> "Each microservice must have its own server."

A service can have:

```text
Multiple instances
Containers
Pods
Virtual machines
Serverless infrastructure
```

The important concept is independent service boundaries and deployment, not one physical server per service.

---

# 98. Important Rule: Don't Say This

Avoid:

> "Microservices communicate only through REST."

They can use:

```text
REST
gRPC
Kafka
RabbitMQ
SQS/SNS
Other messaging systems
```

---

# 99. Important Rule: Don't Say This

Avoid:

> "Every service should have its own database server."

The principle is generally:

```text
Database/schema ownership
```

rather than requiring a physically separate database server for every service.

Infrastructure choices depend on scale and operational needs.

---

# 100. Final Mental Model

Think about microservices in five layers:

```text
1. BUSINESS
   ↓
   Service boundaries

2. COMMUNICATION
   ↓
   REST / gRPC / Events

3. DATA
   ↓
   Database per service
   Transactions
   Saga
   Eventual consistency

4. RESILIENCE
   ↓
   Timeout
   Retry
   Circuit breaker
   Bulkhead
   Idempotency

5. OPERATIONS
   ↓
   Docker
   Kubernetes
   Logging
   Metrics
   Tracing
   Deployment
```

---

# 101. Final Revision Checklist

Before moving to the next file, make sure you can explain:

```text
□ What are microservices?
□ Monolith vs microservices
□ Modular monolith
□ Advantages
□ Disadvantages
□ Service boundaries
□ Business capabilities
□ Distributed systems
□ Synchronous communication
□ Asynchronous communication
□ REST
□ gRPC
□ API Gateway
□ Service discovery
□ Load balancing
□ Fault isolation
□ Cascading failure
□ Timeout
□ Retry
□ Backoff
□ Circuit breaker
□ Bulkhead
□ Idempotency
□ Database per service
□ Shared database problems
□ Distributed transactions
□ Saga
□ Choreography
□ Orchestration
□ Eventual consistency
□ API versioning
□ Contract testing
□ Distributed tracing
□ Correlation IDs
□ Centralized logging
□ Health checks
□ Liveness
□ Readiness
□ Configuration management
□ Secrets
□ Independent deployment
□ Backward compatibility
□ DTO/service communication
□ Optimistic concurrency
□ Stateless services
□ Horizontal scaling
□ Docker
□ Kubernetes
□ Kafka
□ Redis
□ Resilience patterns
□ Microservices security
□ Distributed monolith
```

---

# 102. Interview-Ready Summary

If the interviewer asks:

> "Explain microservices."

Use this:

> "Microservices is an architectural style where an application is divided into independently deployable services aligned with business capabilities. Each service owns a focused responsibility and ideally its data. Services communicate through APIs or messaging. The main benefits are independent deployment, scaling and team ownership, but the trade-off is distributed-system complexity such as network failures, eventual consistency, distributed transactions and observability. I would choose microservices when those benefits justify the additional operational complexity."

That is the foundation for the rest of the microservices section.

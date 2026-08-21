# Microservices — Inter-Service Communication: REST, OpenFeign, WebClient & Messaging

This file covers how Spring Boot microservices communicate with each other.

Core topics:

```text
Synchronous Communication
Asynchronous Communication
REST
HTTP
OpenFeign
RestClient
WebClient
WebFlux
Kafka
RabbitMQ
Request/Response
Events
Message Broker
Serialization
DTOs
Timeouts
Retries
Circuit Breakers
Idempotency
Correlation IDs
API Versioning
Service-to-Service Calls
Production Scenarios
Interview Questions
```

---

# 1. Why Do Microservices Communicate?

A microservice rarely works completely alone.

Example:

```text
Order Service
 ↓
Inventory Service
 ↓
Payment Service
```

Services communicate to:

```text
Retrieve data
Validate operations
Trigger business workflows
Publish events
Notify other services
```

---

# 2. Two Major Communication Styles

Microservices communication can broadly be:

```text
Synchronous
Asynchronous
```

---

# 3. Synchronous Communication

The caller waits for a response.

Example:

```text
Order Service
      ↓
Inventory Service
      ↓
Response
      ↓
Order Service
```

Common technologies:

```text
HTTP/REST
OpenFeign
RestClient
WebClient
gRPC
```

---

# 4. Asynchronous Communication

The sender doesn't need to wait for the consumer to complete the work.

Example:

```text
Order Service
      ↓
Message Broker
      ↓
Inventory Consumer
```

Common technologies:

```text
Kafka
RabbitMQ
AWS SQS/SNS
Azure Service Bus
```

---

# 5. Synchronous vs Asynchronous

| Synchronous | Asynchronous |
|---|---|
| Caller waits | Caller can continue |
| Immediate response | Response may be later |
| Simpler flow | More distributed complexity |
| Tighter runtime coupling | Looser temporal coupling |
| Failure can propagate | Broker can buffer work |

---

# 6. REST Communication

A common synchronous approach:

```text
Order Service
 ↓ HTTP
Inventory Service
```

Example:

```http
GET /inventory/products/101
```

Response:

```json
{
  "productId": 101,
  "available": true
}
```

---

# 7. HTTP Methods

Common methods:

```text
GET
POST
PUT
PATCH
DELETE
```

Use methods according to their semantics.

---

# 8. REST Client Options in Spring

Modern Spring applications can use:

```text
RestClient
WebClient
OpenFeign
```

Older applications may also contain:

```text
RestTemplate
```

For new synchronous code, `RestClient` is often a simpler choice when reactive/non-blocking behavior isn't required.

---

# 9. RestClient

`RestClient` is Spring's synchronous HTTP client API.

Conceptually:

```java
ProductResponse response =
    restClient.get()
              .uri("/products/{id}", id)
              .retrieve()
              .body(ProductResponse.class);
```

It provides a fluent API for synchronous HTTP calls.

---

# 10. WebClient

`WebClient` is Spring's reactive HTTP client.

Conceptually:

```java
Mono<ProductResponse> response =
    webClient.get()
             .uri("/products/{id}", id)
             .retrieve()
             .bodyToMono(ProductResponse.class);
```

It is useful for:

```text
Reactive applications
Non-blocking I/O
Streaming
High-concurrency I/O workloads
```

---

# 11. WebClient Doesn't Automatically Make the Whole Application Reactive

Using:

```text
WebClient
```

doesn't automatically make a blocking application non-blocking.

If you immediately call:

```java
.block()
```

you are introducing blocking behavior.

The surrounding architecture should be designed appropriately.

---

# 12. OpenFeign

OpenFeign provides a declarative HTTP client.

Instead of writing request-building code manually:

```java
client.get(...)
```

you define an interface:

```java
@FeignClient(name = "inventory-service")
public interface InventoryClient {

    @GetMapping("/inventory/{productId}")
    InventoryResponse getInventory(
        @PathVariable Long productId
    );
}
```

The framework handles the HTTP client plumbing.

---

# 13. Why Use OpenFeign?

Advantages:

```text
Less boilerplate
Readable interfaces
Easy Spring integration
Declarative API
Easy request mapping
```

It can make service-to-service REST calls feel similar to calling a local interface.

But remember:

> A remote call is not a local method call.

It can still fail, timeout and experience network latency.

---

# 14. Feign Mental Model

```text
Order Service
     |
InventoryClient interface
     |
OpenFeign
     |
HTTP
     |
Inventory Service
```

---

# 15. Feign Service Name

Example:

```java
@FeignClient(name = "inventory-service")
```

The name can be used with service discovery/load-balancing infrastructure depending on the application's setup.

---

# 16. Feign DTOs

Don't expose internal entity classes across services.

Prefer:

```text
Service A
 ↓
DTO
 ↓
HTTP
 ↓
DTO
 ↓
Service B
```

Example:

```java
public record InventoryResponse(
    Long productId,
    boolean available
) {}
```

---

# 17. Why Not Share Entity Classes?

Sharing JPA entities across services creates tight coupling.

Example:

```text
Order Service
     |
Shared Product Entity
     |
Product Service
```

A database/entity change can force unrelated services to change.

Prefer independent contracts.

---

# 18. API Contract

Service communication should have a defined contract:

```text
Endpoint
HTTP method
Request
Response
Errors
Authentication
Version
Timeout expectations
```

---

# 19. Contract Example

```text
GET /inventory/{productId}

200:
{
  "productId": 101,
  "available": true
}

404:
Product not found

503:
Inventory temporarily unavailable
```

---

# 20. Error Handling

Don't treat every HTTP error as a generic exception.

Distinguish:

```text
400 → invalid request
401 → authentication failure
403 → authorization failure
404 → resource not found
409 → conflict
429 → rate limit
500 → server error
503 → service unavailable
```

---

# 21. Remote Exception Mapping

Suppose:

```text
Inventory Service
 → 404
```

Order Service should translate that response into meaningful domain behavior.

Don't leak:

```text
Remote stack trace
Internal database details
```

to clients.

---

# 22. Timeout

Every synchronous remote call should have a bounded timeout.

Example:

```text
Order
 ↓
Inventory
 ↓
Timeout = 500ms
```

The exact value depends on the application's latency budget.

---

# 23. Connection Pool

HTTP clients can maintain connection pools.

Monitor:

```text
Active connections
Idle connections
Pending requests
Connection acquisition time
```

A badly configured client can exhaust connections.

---

# 24. Retry

Retries can help with transient errors.

Example:

```text
Request
 ↓
Temporary network failure
 ↓
Retry
 ↓
Success
```

But don't retry everything.

---

# 25. Retryable Failures

Potential examples:

```text
Temporary network error
Connection reset
Transient 503
```

Not generally:

```text
400 validation error
401
403
Business rule failure
```

---

# 26. Retry and Idempotency

Be careful with:

```text
POST /payments
```

A timeout doesn't tell you whether the server processed the request.

Use:

```text
Idempotency-Key
```

when the business operation supports it.

---

# 27. Circuit Breaker

Protect a caller from a consistently failing dependency.

```text
Order
 ↓
Circuit Breaker
 ↓
Payment
```

When Payment becomes unhealthy:

```text
Circuit OPEN
 ↓
Fail fast
```

---

# 28. Bulkhead

Isolate resources between dependencies.

Example:

```text
Payment calls
→ Pool A

Inventory calls
→ Pool B
```

Payment cannot consume all resources needed by Inventory.

---

# 29. Correlation ID

Propagate a request identifier:

```text
Gateway
traceId = abc123
 ↓
Order
traceId = abc123
 ↓
Inventory
traceId = abc123
```

This makes distributed troubleshooting easier.

---

# 30. Authentication Between Services

Options:

```text
OAuth2 client credentials
mTLS
Platform workload identity
Signed credentials
```

The service should authenticate the caller rather than blindly trusting network location.

---

# 31. Synchronous Chain

Example:

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

If every call is synchronous:

```text
Gateway latency
≈
sum of downstream latencies
```

plus network/processing overhead.

Long chains increase latency and failure propagation.

---

# 32. Distributed Transaction Problem

Suppose:

```text
Order DB
Inventory DB
Payment DB
```

and one request changes all three.

A single local database transaction cannot normally span all services safely.

This creates a distributed transaction problem.

---

# 33. Saga Pattern

Saga breaks a distributed business transaction into local transactions.

Example:

```text
Create Order
 ↓
Reserve Inventory
 ↓
Process Payment
```

If Payment fails:

```text
Release Inventory
 ↓
Cancel Order
```

These are compensating actions.

---

# 34. Orchestration

A central orchestrator coordinates the workflow.

```text
Order Saga Orchestrator
       |
       +→ Inventory
       |
       +→ Payment
       |
       +→ Notification
```

The orchestrator decides what happens next.

---

# 35. Choreography

Services react to events without a central orchestrator.

```text
OrderCreated
     ↓
Inventory
     ↓
InventoryReserved
     ↓
Payment
     ↓
PaymentCompleted
```

Each service reacts to events.

---

# 36. Orchestration vs Choreography

| Orchestration | Choreography |
|---|---|
| Central coordinator | Event-driven collaboration |
| Easier workflow visibility | Less central control |
| Coordinator can become complex | Event chains can become hard to understand |
| Explicit process | More decoupled |

---

# 37. Event-Driven Communication

Example:

```text
Order Service
 ↓
OrderCreated event
 ↓
Kafka
 ↓
Inventory Service
```

The producer doesn't need a direct HTTP call to Inventory.

---

# 38. Message Broker

A broker transports messages between producers and consumers.

Examples:

```text
Kafka
RabbitMQ
SQS
Azure Service Bus
```

---

# 39. Kafka Mental Model

```text
Producer
 ↓
Kafka Topic
 ↓
Consumer Group
 ↓
Consumer
```

Kafka is especially useful for:

```text
Event streaming
High-throughput messaging
Durable event logs
Decoupled consumers
```

---

# 40. RabbitMQ Mental Model

RabbitMQ is commonly used for messaging with:

```text
Producer
 ↓
Exchange
 ↓
Queue
 ↓
Consumer
```

It is often useful for task/message delivery patterns.

---

# 41. Kafka vs RabbitMQ

| Kafka | RabbitMQ |
|---|---|
| Event streaming/log | Message broker |
| Partitioned topics | Exchanges/queues |
| Strong replay model | Queue-centric consumption |
| High throughput | Flexible routing |
| Consumer offsets | Message acknowledgements |

The right choice depends on the workload.

---

# 42. Event vs Command

Command:

```text
"Do this."
```

Example:

```text
ReserveInventory
```

Event:

```text
"This happened."
```

Example:

```text
InventoryReserved
```

---

# 43. Event Design

Good event:

```json
{
  "eventId": "evt-123",
  "eventType": "OrderCreated",
  "occurredAt": "2026-08-21T10:00:00Z",
  "orderId": "500",
  "customerId": "101"
}
```

Include useful metadata.

---

# 44. Event Versioning

Events evolve.

Example:

```text
OrderCreated v1
OrderCreated v2
```

Consumers should tolerate compatible evolution.

Avoid breaking all consumers for a small producer change.

---

# 45. Schema Evolution

Prefer:

```text
Add optional field
```

over:

```text
Remove required field unexpectedly
```

Use a schema strategy appropriate to the messaging platform.

---

# 46. Serialization

Messages need to be serialized.

Common formats:

```text
JSON
Avro
Protobuf
```

---

# 47. JSON

Advantages:

```text
Human-readable
Easy to debug
Widely supported
```

Disadvantages:

```text
Larger payloads
Less strict schema by default
```

---

# 48. Avro

Avro provides schema-based serialization.

Useful for:

```text
Event streaming
Schema evolution
Compact messages
```

---

# 49. Protobuf

Protocol Buffers provide:

```text
Compact binary format
Strong schemas
Efficient serialization
```

Common in:

```text
gRPC
High-performance service communication
```

---

# 50. REST vs Messaging

Use REST when:

```text
Need immediate response
Request/response fits business flow
Caller needs result now
```

Use messaging when:

```text
Work can happen asynchronously
Need decoupling
Need buffering
Need event propagation
```

---

# 51. Example: REST

User checks product availability:

```text
GET /inventory/101
```

User expects:

```text
Available: true
```

REST is a natural fit.

---

# 52. Example: Messaging

After order creation:

```text
OrderCreated
```

Multiple consumers may react:

```text
Inventory
Analytics
Notification
Fraud Detection
```

Messaging is a natural fit.

---

# 53. Hybrid Architecture

Real systems often use both.

```text
Client
 ↓
REST
 ↓
Order Service
 ↓
Kafka
 ├→ Inventory
 ├→ Notification
 └→ Analytics
```

The initial command is synchronous, while downstream side effects are asynchronous.

---

# 54. Request/Reply over Messaging

Messaging can also implement request/reply:

```text
Service A
 ↓
Message
 ↓
Broker
 ↓
Service B
 ↓
Response message
```

But this can become more complex than ordinary HTTP request/response.

Use it when asynchronous infrastructure provides real value.

---

# 55. Eventual Consistency

With asynchronous communication, data may not update everywhere immediately.

Example:

```text
Order Created
 ↓
Inventory event processing
 ↓
Analytics update
```

There can be a temporary difference between services.

This is eventual consistency.

---

# 56. Why Eventual Consistency?

It enables:

```text
Loose coupling
Independent scaling
Asynchronous processing
Higher resilience
```

But applications must tolerate delayed state.

---

# 57. Outbox Pattern

Problem:

```text
Save Order to DB
Publish event
```

What if:

```text
DB commit succeeds
Event publish fails
```

Now:

```text
Order exists
Event missing
```

The Outbox Pattern solves this by storing the event in the same local DB transaction.

---

# 58. Outbox Flow

```text
Order Service
      |
      +--> Orders table
      |
      +--> Outbox table
             |
             ↓
       Event Publisher
             |
             ↓
           Kafka
```

Order and outbox record commit atomically in the same database transaction.

---

# 59. Outbox Publisher

A background process reads:

```text
outbox
```

and publishes events.

After successful publication:

```text
mark as published
```

or use another delivery-state strategy.

---

# 60. Duplicate Events

Messaging systems can produce duplicates.

Consumers should be designed to tolerate them where required.

Example:

```text
OrderCreated
OrderCreated
```

Consumer should not accidentally:

```text
Charge customer twice
```

---

# 61. Idempotent Consumer

Store processed event IDs:

```text
eventId = evt-123
```

Before processing:

```text
Already processed?
```

If yes:

```text
Ignore duplicate
```

---

# 62. At-Least-Once Delivery

Many messaging systems provide at-least-once delivery semantics in common configurations.

Meaning:

```text
Message should not be lost
```

but duplicates may occur.

Therefore:

```text
Consumer idempotency
```

is important.

---

# 63. Exactly Once

"Exactly once" is often misunderstood.

Even if a broker provides certain exactly-once guarantees within specific boundaries, the entire business workflow can still have external side effects.

Example:

```text
Kafka
 ↓
Payment Provider
```

Broker-level semantics do not automatically make the external payment exactly once.

---

# 64. Dead Letter Queue

Messages that repeatedly fail can be moved to a dead-letter mechanism.

Conceptually:

```text
Main Queue
 ↓
Processing failure
 ↓
Retry
 ↓
Retry
 ↓
DLQ
```

This prevents one bad message from blocking normal processing indefinitely.

---

# 65. Poison Message

A poison message repeatedly fails processing.

Example:

```text
Malformed event
```

If retried forever:

```text
Consumer
 ↓
fail
 ↓
retry
 ↓
fail
 ↓
retry...
```

Use bounded retries and dead-letter handling.

---

# 66. Message Ordering

Ordering requirements must be explicit.

Example:

```text
OrderCreated
OrderCancelled
```

Processing:

```text
OrderCancelled
before
OrderCreated
```

could be incorrect.

Kafka partitioning can preserve ordering within a partition.

---

# 67. Partition Key

For an order workflow:

```text
key = orderId
```

Events for the same order can be routed to the same partition.

This can help preserve per-order ordering.

---

# 68. Consumer Groups

Kafka consumer groups allow consumers to share work.

Example:

```text
Topic: orders

Partition 0 → Consumer A
Partition 1 → Consumer B
Partition 2 → Consumer C
```

Consumers in the same group divide partitions among themselves.

---

# 69. Independent Consumers

Different consumer groups can independently process the same event stream.

```text
OrderCreated
   |
   +→ Inventory Group
   |
   +→ Analytics Group
   |
   +→ Notification Group
```

This is one reason event streaming is powerful.

---

# 70. Backpressure in Messaging

If consumers are slower than producers:

```text
Producer rate > Consumer rate
```

then:

```text
Lag increases
```

Possible responses:

```text
Scale consumers
Optimize processing
Increase partitions where appropriate
Fix downstream bottleneck
Apply producer controls
```

---

# 71. Deadlines

A request can have an end-to-end deadline.

Example:

```text
Gateway budget = 2 sec
```

Downstream calls should respect the remaining budget.

This prevents:

```text
Nested services
each waiting the full timeout
```

---

# 72. Request Context Propagation

Propagate useful context:

```text
Trace ID
Correlation ID
Tenant ID
Request ID
```

Don't propagate sensitive information blindly.

---

# 73. API Versioning

Services evolve.

Options:

```text
URI versioning
Header versioning
Media-type versioning
Backward-compatible evolution
```

Example:

```text
/api/v1/orders
/api/v2/orders
```

---

# 74. Backward Compatibility

A provider should avoid unexpectedly breaking consumers.

Safer change:

```text
Add optional response field
```

Riskier change:

```text
Rename existing required field
```

---

# 75. Consumer-Driven Contracts

Contract testing verifies that a provider satisfies consumer expectations.

Conceptually:

```text
Consumer expectation
 ↓
Contract
 ↓
Provider test
```

Tools can include:

```text
Pact
Spring Cloud Contract
```

---

# 76. Why Contract Testing?

Without contract tests:

```text
Service A assumes:
field = customerName
```

Service B changes:

```text
field = name
```

Production breaks.

Contract tests detect incompatible changes earlier.

---

# 77. Distributed Tracing

For REST:

```text
Gateway
 ↓
Order
 ↓
Inventory
```

For messaging:

```text
Order
 ↓
Kafka
 ↓
Inventory
```

Trace context can be propagated through supported message headers.

---

# 78. Communication Observability

Monitor:

```text
Request count
Latency
Timeouts
Retries
5xx
4xx
Circuit state
Message lag
Consumer errors
DLQ count
```

---

# 79. REST Communication Failure

```text
Order
 ↓
Inventory
 ↓
Timeout
```

Possible response:

```text
504
```

depending on which component owns the timeout and API semantics.

---

# 80. Messaging Communication Failure

```text
Producer
 ↓
Kafka
 ↓
Consumer fails
```

Producer may still have successfully published the event.

Consumer can:

```text
Retry
Dead-letter
Recover later
```

This is fundamentally different from synchronous request failure.

---

# 81. Synchronous Failure Propagation

```text
A
 ↓
B
 ↓
C
```

If C fails:

```text
B may fail
 ↓
A may fail
```

Failure can propagate directly.

---

# 82. Asynchronous Failure Isolation

```text
A
 ↓
Kafka
```

A can potentially complete its local operation even if:

```text
Consumer B
```

is temporarily unavailable.

The broker can buffer the event depending on retention and capacity.

---

# 83. But Async Doesn't Remove Failure

Eventually:

```text
Consumer must process event
```

If consumer remains down:

```text
Lag increases
```

Therefore asynchronous communication changes failure behavior; it doesn't eliminate failure.

---

# 84. REST Communication Best Practices

```text
Use timeouts
Use connection pooling
Use bounded retries
Use circuit breakers
Use idempotency
Validate responses
Use DTOs
Propagate trace context
Version APIs
Monitor latency/errors
```

---

# 85. Messaging Best Practices

```text
Define event contracts
Use schema evolution
Use idempotent consumers
Use retries carefully
Use DLQ
Monitor lag
Handle poison messages
Consider ordering
Propagate trace context
Design for eventual consistency
```

---

# 86. OpenFeign vs WebClient

| OpenFeign | WebClient |
|---|---|
| Declarative | Programmatic/reactive |
| Easy REST clients | Flexible HTTP client |
| Great for straightforward service calls | Strong for reactive/non-blocking flows |
| Interface-based | Fluent API |
| Common Spring Cloud usage | Spring WebFlux ecosystem |

---

# 87. OpenFeign vs RestClient

| OpenFeign | RestClient |
|---|---|
| Declarative interface | Fluent synchronous API |
| Less boilerplate | More direct control |
| Service-oriented client abstraction | General HTTP client |
| Convenient for many REST contracts | Convenient for straightforward HTTP calls |

---

# 88. WebClient vs RestClient

Use `RestClient` when:

```text
Synchronous application
Simple HTTP calls
Blocking model is appropriate
```

Use `WebClient` when:

```text
Reactive application
Non-blocking I/O
Streaming
Reactive composition
```

---

# 89. Remote Call Is Not Local Call

This is a key interview concept.

Local:

```java
inventoryService.check();
```

Remote:

```text
Network
Serialization
Authentication
Routing
Timeout
Remote processing
Response
```

Therefore remote calls need:

```text
Timeout
Error handling
Observability
Resilience
```

---

# 90. Chatty Communication

Bad:

```text
Order
 ↓
Product
 ↓
Product
 ↓
Product
 ↓
Product
```

Repeated network calls increase:

```text
Latency
Network traffic
Failure probability
```

Prefer:

```text
Batch APIs
Aggregation
Caching
Better service boundaries
```

where appropriate.

---

# 91. N+1 Remote Calls

Example:

```text
Get 100 orders
 ↓
Call Customer Service 100 times
```

This is an N+1 remote-call problem.

Better approaches:

```text
Batch endpoint
Caching
Data composition
Local read model
```

depending on requirements.

---

# 92. API Composition

A backend can combine data:

```text
Order API
 ↓
Order
Customer
Product
```

into:

```json
{
  "orderId": 500,
  "customer": {...},
  "items": [...]
}
```

This reduces client-side orchestration.

---

# 93. Data Ownership

A service should own its domain data.

Example:

```text
Product Service
→ Product data

Order Service
→ Order data

Payment Service
→ Payment data
```

Avoid direct cross-service database access.

---

# 94. Why Direct DB Access Is Bad

Bad:

```text
Order Service
 ↓
Payment DB
```

This creates:

```text
Tight coupling
Security problems
Schema coupling
Ownership violations
Hard deployments
```

Prefer:

```text
Order Service
 ↓
Payment API/event
 ↓
Payment Service
```

---

# 95. Communication Choice

Ask:

```text
Does caller need immediate result?
        |
     YES
        ↓
HTTP/gRPC
        |
     NO
        ↓
Can work be asynchronous?
        |
     YES
        ↓
Messaging/event
```

---

# 96. REST vs gRPC

REST:

```text
HTTP/JSON
Human-friendly
Broad ecosystem
Good public APIs
```

gRPC:

```text
HTTP/2
Protobuf
Strong contracts
Efficient binary serialization
Good internal service communication
```

Choice depends on requirements.

---

# 97. Messaging vs gRPC

gRPC:

```text
Direct request/response
Low latency
Synchronous
```

Messaging:

```text
Asynchronous
Buffered
Decoupled
Event-driven
```

---

# 98. Production Scenario

### "Order creation requires inventory and payment."

If both must complete before confirming order:

```text
Order
 ↓
Inventory
 ↓
Payment
```

Could use synchronous calls for immediate decisions.

But use:

```text
Timeouts
Circuit breakers
Idempotency
Saga/recovery
```

because failures can occur.

---

# 99. Production Scenario

### "After order creation, send email."

Don't necessarily block order creation on email.

Better:

```text
Order created
 ↓
OrderCreated event
 ↓
Kafka
 ↓
Notification Service
 ↓
Email
```

The order doesn't need to wait for the email provider.

---

# 100. Production Scenario

### "Analytics should process every order."

Use:

```text
OrderCreated
 ↓
Kafka
 ↓
Analytics consumer
```

Analytics can process asynchronously.

---

# 101. Production Scenario

### "Inventory service is temporarily down."

Synchronous approach:

```text
Order → Inventory
        ↓
      timeout
```

Use:

```text
Retry if appropriate
Circuit breaker
Return appropriate business status
```

Asynchronous workflows may instead:

```text
OrderCreated
 ↓
Inventory event
 ↓
Pending
 ↓
Inventory processes later
```

depending on business requirements.

---

# 102. Production Scenario

### "Kafka consumer processes the same OrderCreated event twice."

Use:

```text
eventId
 ↓
Idempotency check
 ↓
Already processed?
```

If already processed:

```text
Skip duplicate
```

---

# 103. Production Scenario

### "Event was published but consumer never processed it."

Check:

```text
Consumer health
Consumer lag
Partition assignment
Consumer errors
DLQ
Broker health
Deserialization failures
```

---

# 104. Production Scenario

### "Order API became slow after adding Customer Service."

Check:

```text
Customer call latency
Connection pool
Timeout
Number of calls
N+1 behavior
Trace
```

Maybe:

```text
One request → 100 customer calls
```

caused the problem.

---

# 105. Production Scenario

### "Payment provider times out."

Do not blindly retry:

```text
POST /payment
```

Instead:

```text
Idempotency key
Provider status query
Payment state = PENDING/UNKNOWN
Reconciliation
```

---

# 106. Interview Question

### "How do microservices communicate?"

Answer:

> "They can communicate synchronously using HTTP/REST, OpenFeign, RestClient or WebClient, and asynchronously using messaging platforms such as Kafka or RabbitMQ. I choose based on whether the caller needs an immediate response, the required coupling, reliability and consistency requirements."

---

# 107. Interview Question

### "Why use OpenFeign?"

Answer:

> "OpenFeign provides a declarative HTTP client. I define an interface with the remote API mappings and Spring handles much of the HTTP client plumbing, which reduces boilerplate and makes service contracts easier to read."

---

# 108. Interview Question

### "Feign vs WebClient?"

Answer:

> "Feign is declarative and convenient for straightforward service-to-service REST calls. WebClient is a flexible reactive HTTP client and is better suited to non-blocking/reactive workflows. The choice depends on the application's execution model."

---

# 109. Interview Question

### "Why shouldn't services access each other's databases?"

Answer:

> "It creates tight coupling to another service's schema and violates data ownership. A schema change can break multiple services. Services should normally expose APIs or events instead."

---

# 110. Interview Question

### "REST or Kafka?"

Answer:

> "If the caller needs an immediate response, REST is usually appropriate. If the work can happen asynchronously or multiple consumers need to react to an event, Kafka or another messaging system is often a better fit. Many real systems use both."

---

# 111. Interview Question

### "What is eventual consistency?"

Answer:

> "With asynchronous communication, different services may temporarily have different views of the same business state. They become consistent after events are processed. This is called eventual consistency."

---

# 112. Interview Question

### "What is the Outbox Pattern?"

Answer:

> "The service writes its business data and an event record to an outbox table in the same local database transaction. A separate publisher then sends the event to the broker. This prevents a successful database transaction from losing its corresponding event."

---

# 113. Interview Question

### "What is a Saga?"

Answer:

> "A Saga coordinates a distributed business transaction as a sequence of local transactions. If a later step fails, compensating actions undo or offset the earlier steps. It can be implemented using orchestration or choreography."

---

# 114. Interview Question

### "How do you prevent duplicate event processing?"

Answer:

> "I'd make the consumer idempotent, typically using a unique event ID or business key to detect already-processed messages. The exact implementation depends on the broker and business requirements."

---

# 115. Interview Question

### "Why is retry dangerous for POST?"

Answer:

> "POST isn't inherently idempotent. If the first request succeeds but the response is lost, a retry can create a duplicate operation. For critical operations like payments, I'd use an idempotency key or another business-level deduplication mechanism."

---

# 116. Interview Question

### "How do you trace a request across microservices?"

Answer:

> "I'd propagate a trace or correlation ID through HTTP headers and message headers. With distributed tracing, each service creates a span under the same trace, allowing us to follow the request across the complete workflow."

---

# 117. Interview Question

### "What is a poison message?"

Answer:

> "It's a message that repeatedly fails processing, often because of invalid data or an incompatible schema. I'd use bounded retries and move the message to a dead-letter mechanism so it doesn't block normal processing indefinitely."

---

# 118. Interview Question

### "What is consumer lag?"

Answer:

> "Consumer lag represents how far a consumer is behind the available messages in a Kafka partition or topic. Increasing lag can indicate that producers are outpacing consumers or that consumers are blocked by processing or downstream dependencies."

---

# 119. Interview Question

### "What is the Outbox Pattern solving?"

Answer:

> "It solves the dual-write problem where the application needs to update its database and publish an event. By storing the event in the same database transaction, we make the local state change and event creation atomic."

---

# 120. Final Communication Architecture

```text
                         Client
                           |
                         HTTPS
                           |
                           ↓
                     API Gateway
                           |
                    +------+------+
                    |             |
                    ↓             ↓
                  Order         Product
                  Service       Service
                    |
             +------+------+
             |             |
         REST/Feign      Kafka
             |             |
             ↓             ↓
        Inventory     Notification
        Service        Service
             |
             ↓
          Payment
          Service
```

Use:

```text
REST/Feign
→ Immediate request/response

Kafka
→ Events and asynchronous workflows

Timeouts
→ Bound waiting

Circuit Breakers
→ Protect dependencies

Idempotency
→ Safe retries

Outbox
→ Reliable event publication

DLQ
→ Isolate poison messages

Tracing
→ Follow distributed workflows
```

---

# 121. Final Interview Answer

If asked:

> "How do you decide between synchronous REST and asynchronous messaging in microservices?"

Use:

> "I'd use synchronous REST when the caller needs an immediate response or decision, such as checking inventory during checkout. I'd use asynchronous messaging when the work can happen later, when I want to decouple services, or when multiple consumers need to react to an event, such as publishing OrderCreated for notifications and analytics. In both cases I'd design for failures, using timeouts and resilience for synchronous calls and idempotency, retries, DLQs and observability for asynchronous processing."

---

# 122. Revision Checklist

```text
□ Synchronous communication
□ Asynchronous communication
□ REST
□ HTTP
□ RestClient
□ WebClient
□ WebFlux
□ OpenFeign
□ Feign DTOs
□ API contracts
□ HTTP error handling
□ Timeouts
□ Connection pools
□ Retries
□ Idempotency
□ Circuit breaker
□ Bulkhead
□ Correlation ID
□ Authentication
□ Service-to-service security
□ Distributed transactions
□ Saga
□ Orchestration
□ Choreography
□ Events
□ Commands
□ Kafka
□ RabbitMQ
□ Message broker
□ Serialization
□ JSON
□ Avro
□ Protobuf
□ Eventual consistency
□ Outbox Pattern
□ Duplicate events
□ Idempotent consumer
□ At-least-once delivery
□ Exactly-once caveats
□ Dead Letter Queue
□ Poison messages
□ Message ordering
□ Kafka partitions
□ Partition keys
□ Consumer groups
□ Consumer lag
□ Backpressure
□ Request context propagation
□ API versioning
□ Contract testing
□ Distributed tracing
□ Chatty communication
□ N+1 remote calls
□ Data ownership
□ REST vs gRPC
□ REST vs Kafka
□ Production scenarios
```

---

# 123. The Interviewer's Real Test

If asked:

> "An Order Service calls Inventory and Payment synchronously. Payment becomes slow and now Order is exhausting its threads. What would you change?"

Think:

```text
Order
 |
 +→ Inventory
 |
 +→ Payment
       ↓
    SLOW
       ↓
Strict timeout
       ↓
Bounded retry only if appropriate
       ↓
Circuit breaker
       ↓
Bulkhead
       ↓
Protect Order resources
```

Then ask whether the business flow actually requires synchronous Payment.

If payment can be asynchronous:

```text
Order
 ↓
OrderCreated
 ↓
Kafka
 ↓
Payment
 ↓
PaymentCompleted
```

This can reduce temporal coupling, but introduces:

```text
Pending states
Eventual consistency
Idempotency
Saga/recovery
```

The key interview lesson is:

> **Choose communication style based on business requirements, not because REST or Kafka is "better." Synchronous calls are simple but tightly coupled in time; asynchronous messaging improves decoupling but introduces eventual consistency and messaging complexity.**

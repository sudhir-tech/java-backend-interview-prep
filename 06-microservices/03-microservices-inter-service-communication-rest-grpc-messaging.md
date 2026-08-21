# Microservices — Inter-Service Communication: REST, gRPC & Messaging

This file covers how microservices communicate with each other.

The three major approaches to know for interviews are:

```text
1. REST / HTTP
2. gRPC
3. Asynchronous messaging/events
```

The important part is not memorizing technologies.

You should understand **when to use each approach and what trade-offs it creates**.

---

# 1. Why Communication Matters

In a monolith:

```java
orderService.createOrder();
```

The call is inside the same process.

In microservices:

```text
Order Service
      |
      | network
      ↓
Payment Service
```

Now communication can fail.

Possible problems:

```text
Network failure
Timeout
DNS failure
Connection refused
Service unavailable
Slow response
Serialization failure
Authentication failure
Version mismatch
```

This is why service-to-service communication is a major microservices concern.

---

# 2. Main Communication Styles

Microservices commonly use:

```text
Synchronous
├── REST
└── gRPC

Asynchronous
├── Kafka
├── RabbitMQ
└── Cloud messaging systems
```

---

# 3. Synchronous Communication

The caller waits for the response.

Example:

```text
Order Service
      |
      | HTTP
      ↓
Payment Service
      |
      ↓
Payment Response
```

Order Service cannot continue past the required step until it receives the response.

---

# 4. Advantages of Synchronous Communication

```text
Simple request/response model
Immediate result
Easy to understand
Good for interactive operations
Straightforward error handling
```

Example:

```text
GET /products/100
```

The client needs the product now.

Synchronous communication is natural here.

---

# 5. Disadvantages of Synchronous Communication

```text
Network latency
Timeouts
Dependency availability
Failure propagation
Thread/resource consumption
Tighter runtime coupling
```

Example:

```text
Order
 ↓
Payment
 ↓
Tax
 ↓
Shipping
```

If every call is synchronous, one slow dependency can slow down the entire request.

---

# 6. Asynchronous Communication

The producer sends a message/event and does not need to wait for every consumer.

Example:

```text
Order Service
      |
      | OrderCreated
      ↓
    Kafka
    /   \
   ↓     ↓
Email   Analytics
```

The Order Service can finish its own transaction without waiting for notification processing.

---

# 7. Advantages of Asynchronous Communication

```text
Loose runtime coupling
Better fault isolation
Buffering
Independent processing
Better scalability for event-driven workloads
```

---

# 8. Disadvantages of Asynchronous Communication

```text
Eventual consistency
More complex debugging
Duplicate messages
Ordering concerns
Retry handling
Dead-letter handling
Consumer failures
Harder local development
```

---

# 9. REST

REST commonly uses:

```text
HTTP
JSON
```

Example:

```http
GET /products/101
```

Response:

```json
{
  "id": 101,
  "name": "Wireless Headphones",
  "price": 2999
}
```

REST is widely used for public APIs and service-to-service communication.

---

# 10. REST Characteristics

Common HTTP methods:

```text
GET
POST
PUT
PATCH
DELETE
```

Typical semantics:

```text
GET
→ Retrieve

POST
→ Create/process

PUT
→ Replace

PATCH
→ Partial update

DELETE
→ Delete
```

---

# 11. REST Example: Order Service

Request:

```http
POST /orders
Content-Type: application/json

{
  "productId": 101,
  "quantity": 2
}
```

Response:

```http
201 Created
```

```json
{
  "orderId": 5001,
  "status": "CREATED"
}
```

---

# 12. REST Status Codes

Know these:

```text
200 OK
201 Created
202 Accepted
204 No Content

400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Content

429 Too Many Requests

500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

---

# 13. 401 vs 403

Very common interview question.

### 401

The request lacks valid authentication.

Think:

```text
"Who are you?"
```

### 403

The user is authenticated but doesn't have permission.

Think:

```text
"I know who you are,
but you aren't allowed to do this."
```

---

# 14. 404 vs 204

```text
404
→ Requested resource doesn't exist.

204
→ Request succeeded and there is intentionally no response body.
```

Example:

```http
DELETE /users/10
```

could return:

```http
204 No Content
```

if deletion succeeds.

---

# 15. 409 Conflict

Use when the request conflicts with the current state.

Examples:

```text
Duplicate email
Inventory conflict
Optimistic locking conflict
Invalid state transition
```

Example:

```text
Two users attempt to reserve the final item.
```

---

# 16. 202 Accepted

Useful when work has been accepted but is processed asynchronously.

Example:

```http
POST /reports
```

Response:

```http
202 Accepted
```

The system may return:

```json
{
  "jobId": "abc123",
  "status": "PROCESSING"
}
```

The client can later check status.

---

# 17. REST Idempotency

Common HTTP semantics:

```text
GET
→ Idempotent

PUT
→ Idempotent

DELETE
→ Generally idempotent

POST
→ Not inherently idempotent
```

But actual behavior depends on implementation.

---

# 18. Why POST Can Be Dangerous to Retry

Suppose:

```http
POST /payments
```

Payment succeeds.

But response is lost.

Client retries.

Without idempotency:

```text
Payment #1
Payment #2
```

The customer may be charged twice.

---

# 19. Idempotency Key

Client sends:

```http
Idempotency-Key: abc123
```

Payment Service stores:

```text
abc123 → payment result
```

If the same request arrives again:

```text
abc123
```

the service can return the original result instead of processing another payment.

---

# 20. REST API Design

Good:

```text
GET /users/10
GET /users/10/orders
POST /orders
GET /orders/100
PATCH /orders/100
DELETE /orders/100
```

Avoid overly action-oriented APIs when resource semantics are enough:

```text
POST /doCreateUser
POST /getUserData
```

That said, domain actions can be appropriate when the operation is genuinely an action.

---

# 21. Resource-Oriented API

Example:

```text
/users
/users/{id}

/orders
/orders/{id}

/products
/products/{id}
```

This creates a predictable API structure.

---

# 22. Nested Resources

Example:

```text
GET /users/10/orders
```

Useful when the relationship is meaningful.

But avoid excessive nesting:

```text
/users/10/orders/20/items/3/product/5
```

This can become difficult to maintain.

---

# 23. Pagination

Never assume an endpoint should return every record.

Use:

```http
GET /products?page=0&size=20
```

or cursor pagination:

```http
GET /products?cursor=abc123&limit=20
```

For large datasets, cursor/keyset pagination is often more scalable.

---

# 24. Filtering

Example:

```http
GET /products?category=electronics&minPrice=1000&maxPrice=5000
```

Keep filtering predictable and bounded.

---

# 25. Sorting

Example:

```http
GET /products?sort=price,desc
```

The server should validate allowed sort fields.

Don't directly concatenate arbitrary user input into SQL.

---

# 26. API Versioning

Common:

```text
/api/v1/products
/api/v2/products
```

Alternative approaches:

```text
Headers
Content negotiation
```

The important goal is:

```text
Backward compatibility
```

---

# 27. Backward-Compatible API Changes

Usually safe:

```text
Add an optional response field
Add a new endpoint
Add an optional request field
```

Potentially breaking:

```text
Remove a field
Rename a field
Change field type
Change meaning
Make an optional field mandatory
```

---

# 28. REST Error Contract

Avoid inconsistent errors.

Bad:

```json
"failed"
```

Another endpoint:

```json
{
  "errorMessage": "something went wrong"
}
```

A consistent contract is easier for clients.

Example:

```json
{
  "code": "PRODUCT_NOT_FOUND",
  "message": "Product 101 was not found",
  "timestamp": "2026-08-21T10:30:00Z",
  "traceId": "abc123"
}
```

---

# 29. REST Timeouts

Never rely on infinite waits.

Example:

```text
Order Service
     |
     | timeout = 2s
     ↓
Payment Service
```

If Payment doesn't respond:

```text
Timeout
```

Then apply:

```text
Retry if appropriate
Circuit breaker
Fallback
Error response
```

---

# 30. REST Retries

Retry only when appropriate.

Good candidates:

```text
Temporary network failure
503
Connection reset
Transient infrastructure issue
```

Dangerous:

```text
POST payment
```

without idempotency.

---

# 31. Exponential Backoff

Instead of:

```text
Retry
Retry
Retry
```

use:

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

Add jitter to reduce synchronized retry spikes.

---

# 32. Circuit Breaker + REST

Example:

```text
Order Service
      |
      ↓
Payment Service
```

If Payment repeatedly fails:

```text
CLOSED
 ↓
OPEN
```

Order Service stops sending calls temporarily.

This prevents repeatedly hammering an unhealthy service.

---

# 33. REST and API Gateway

Client:

```text
Mobile App
    |
    ↓
API Gateway
```

Gateway:

```text
/api/users
→ User Service

/api/products
→ Product Service

/api/orders
→ Order Service
```

The client doesn't need to know internal service addresses.

---

# 34. Service-to-Service REST with Spring Boot

Example:

```java
@RestController
@RequestMapping("/payments")
public class PaymentController {

    @PostMapping
    public PaymentResponse process(
            @RequestBody PaymentRequest request) {

        return paymentService.process(request);
    }
}
```

Another service can call it using:

```text
RestClient
WebClient
OpenFeign
```

depending on the Spring architecture.

---

# 35. RestClient

For synchronous HTTP calls in modern Spring applications, `RestClient` provides a convenient blocking HTTP client API.

Conceptually:

```java
PaymentResponse response =
    restClient.post()
        .uri("/payments")
        .body(request)
        .retrieve()
        .body(PaymentResponse.class);
```

---

# 36. WebClient

`WebClient` supports reactive/non-blocking HTTP communication.

Conceptually:

```java
webClient.post()
    .uri("/payments")
    .bodyValue(request)
    .retrieve()
    .bodyToMono(PaymentResponse.class);
```

Don't use reactive programming simply because microservices are involved.

Choose it when the application's workload and architecture benefit from it.

---

# 37. OpenFeign

Feign provides a declarative HTTP client.

Example:

```java
@FeignClient(name = "payment-service")
public interface PaymentClient {

    @PostMapping("/payments")
    PaymentResponse process(
        @RequestBody PaymentRequest request
    );
}
```

Then:

```java
paymentClient.process(request);
```

The interface hides HTTP plumbing.

---

# 38. REST vs OpenFeign

These aren't really competing protocols.

Think:

```text
REST
→ Communication style/API

OpenFeign
→ Client abstraction used to call HTTP APIs
```

Feign can make REST calls easier to write.

---

# 39. REST vs gRPC

### REST

```text
HTTP
Often JSON
Human-readable
Widely supported
Excellent for public APIs
```

### gRPC

```text
HTTP/2
Protocol Buffers
Strongly typed contracts
Efficient binary serialization
Excellent for internal service communication
```

---

# 40. What Is gRPC?

gRPC is an RPC framework commonly used for service-to-service communication.

A service defines its contract using Protocol Buffers.

Example:

```protobuf
service PaymentService {
    rpc ProcessPayment(PaymentRequest)
        returns (PaymentResponse);
}
```

---

# 41. Protocol Buffers

Example:

```protobuf
message PaymentRequest {
    int64 order_id = 1;
    double amount = 2;
}
```

The schema defines the data contract.

Code can then be generated for supported languages.

---

# 42. Why gRPC Can Be Faster

gRPC commonly uses:

```text
HTTP/2
Binary Protocol Buffers
```

Compared with:

```text
HTTP/1.1 + JSON
```

this can reduce serialization size and improve efficiency for suitable workloads.

Actual performance must be measured.

---

# 43. gRPC Benefits

```text
Strong contracts
Efficient serialization
HTTP/2
Streaming support
Code generation
Good internal service communication
```

---

# 44. gRPC Streaming

gRPC supports:

```text
Unary
Server streaming
Client streaming
Bidirectional streaming
```

Example:

```text
Client
  ⇅
Streaming RPC
  ⇅
Server
```

This can be useful for real-time or high-throughput communication.

---

# 45. gRPC Disadvantages

```text
Less browser-friendly
Less human-readable than JSON
Requires tooling
More complex debugging for some teams
Not always ideal for public APIs
```

---

# 46. When Would You Use REST?

Good examples:

```text
Public APIs
Browser/mobile clients
Simple CRUD
Broad ecosystem compatibility
Human-readable APIs
```

---

# 47. When Would You Use gRPC?

Good examples:

```text
Internal service-to-service calls
High-throughput communication
Low-latency requirements
Strongly typed contracts
Streaming
Polyglot internal systems
```

---

# 48. Important Interview Answer

### "REST or gRPC?"

Strong answer:

> "For public APIs and browser/mobile clients, REST is often simpler and more interoperable. For internal service-to-service communication where low latency, efficient serialization and strongly typed contracts matter, gRPC can be a good choice. I would choose based on requirements rather than assuming one is universally better."

---

# 49. Asynchronous Messaging

Instead of:

```text
Order → Notification
```

synchronously:

```text
Order → Kafka → Notification
```

The producer publishes an event.

---

# 50. Event vs Message

An event usually represents:

```text
Something happened.
```

Example:

```text
OrderCreated
```

A command represents:

```text
Please perform this action.
```

Example:

```text
ProcessPayment
```

---

# 51. Kafka Example

```text
Order Service
      |
      | publish
      ↓
Kafka Topic: order-events
      |
      +------------+
      ↓            ↓
Notification   Analytics
Consumer       Consumer
```

Multiple consumers can independently process the same event.

---

# 52. Why Kafka?

Kafka is commonly used for:

```text
High-throughput event streaming
Durable event storage
Multiple consumers
Event-driven architectures
Asynchronous processing
```

Kafka concepts will be covered in much more detail in a later file.

---

# 53. RabbitMQ

RabbitMQ is commonly used as a message broker.

Typical use cases:

```text
Task queues
Work distribution
Asynchronous commands
Routing messages
```

Kafka and RabbitMQ overlap in some use cases but have different architectural characteristics.

---

# 54. Kafka vs RabbitMQ — Interview Level

A simple answer:

> "Kafka is commonly chosen for durable high-throughput event streaming and replayable event logs, while RabbitMQ is often used for traditional message queuing and flexible routing. The right choice depends on the workload and delivery requirements."

Avoid claiming:

```text
Kafka is always better.
```

---

# 55. Message Delivery

Possible concepts:

```text
At-most-once
At-least-once
Exactly-once
```

### At-most-once

Message may be lost.

```text
0 or 1 delivery
```

### At-least-once

Message may be delivered more than once.

```text
1 or more deliveries
```

### Exactly-once

The intended effect occurs once under specific system guarantees.

This is much more nuanced than simply saying "the message is delivered once."

---

# 56. Why At-Least-Once Requires Idempotency

Suppose:

```text
OrderCreated
```

is delivered twice.

Consumer:

```text
send email
```

Without idempotency:

```text
Two emails
```

With idempotency:

```text
eventId = 123
already processed?
→ yes
→ ignore duplicate
```

---

# 57. Dead Letter Queue

If a message repeatedly fails:

```text
Kafka / Queue
     ↓
Consumer
     ↓
failure
     ↓
retry
     ↓
retry
     ↓
DLQ
```

A dead-letter mechanism isolates problematic messages for investigation/reprocessing.

---

# 58. Ordering

Some business workflows require ordering.

Example:

```text
OrderCreated
PaymentCompleted
OrderShipped
```

Processing:

```text
OrderShipped
```

before:

```text
OrderCreated
```

could be invalid.

Messaging systems provide different ordering guarantees.

You need to understand the ordering model of the technology you use.

---

# 59. Message Partitioning

For Kafka, partitions affect:

```text
Parallelism
Ordering
Consumer distribution
```

Messages within a partition have ordering guarantees under Kafka's model.

A common design is to key events by:

```text
orderId
```

so events for one order are routed consistently to the same partition.

---

# 60. Synchronous vs Asynchronous Example

Requirement:

```text
Customer clicks "View Product"
```

Use:

```text
REST
```

because the response is needed immediately.

Requirement:

```text
Order completed → send email
```

Potentially use:

```text
Event
```

because email doesn't need to block order completion.

---

# 61. Communication Decision Matrix

| Requirement | Possible Choice |
|---|---|
| Public API | REST |
| Browser/mobile API | REST |
| Simple internal request | REST |
| Strongly typed internal RPC | gRPC |
| Low-latency internal calls | gRPC |
| Streaming | gRPC |
| Async notification | Kafka/RabbitMQ |
| Analytics events | Kafka |
| Background task | Queue |
| High-throughput event stream | Kafka |

This is guidance, not a rigid rule.

---

# 62. Communication Anti-Pattern

Bad:

```text
Order Service
 ↓
User Service
 ↓
Product Service
 ↓
Inventory Service
 ↓
Payment Service
 ↓
Notification Service
```

all synchronously during one HTTP request.

Potential result:

```text
High latency
Many failure points
Cascading failures
Hard debugging
```

---

# 63. Better Design

Critical synchronous path:

```text
Client
 ↓
Gateway
 ↓
Order
 ↓
Payment
```

Asynchronous:

```text
Order
 ↓
OrderConfirmed event
 ↓
Kafka
 ├── Notification
 ├── Analytics
 └── Audit
```

This separates critical and non-critical work.

---

# 64. Communication Timeout Budget

Suppose API SLA is:

```text
500 ms
```

and the request calls:

```text
Service A
Service B
Service C
```

You cannot give every dependency:

```text
2 seconds timeout
```

because the overall request could exceed its latency budget.

Think about:

```text
End-to-end latency budget
```

when setting dependency timeouts.

---

# 65. Retry Budget

Suppose:

```text
API receives 1,000 requests/sec
```

If every request retries 3 times:

```text
Potential downstream load
≈ 4,000 attempts/sec
```

during failure.

This can make an outage worse.

Retries should be bounded and carefully designed.

---

# 66. Correlation IDs

Example:

```http
X-Correlation-ID: 8f31abc
```

Request flow:

```text
Gateway
  ID: 8f31abc
     ↓
Order
  ID: 8f31abc
     ↓
Payment
  ID: 8f31abc
```

Logs can be correlated across services.

Distributed tracing provides richer context using trace/span information.

---

# 67. Authentication Between Services

Don't assume:

```text
Internal network = trusted
```

Possible approaches:

```text
OAuth2 client credentials
JWT
mTLS
Service identity
Cloud IAM
```

The architecture should protect internal APIs too.

---

# 68. Service Discovery + REST

A service shouldn't necessarily call:

```text
http://192.168.1.45:8080
```

because instances change.

Instead:

```text
payment-service
```

can resolve to available instances through:

```text
Service discovery
DNS
Kubernetes Service
Load balancer
```

---

# 69. Load Balancing

Suppose:

```text
payment-service
├── instance 1
├── instance 2
└── instance 3
```

Traffic can be distributed across instances.

This improves:

```text
Capacity
Availability
```

assuming the service is designed for horizontal scaling.

---

# 70. Health Checks

A load balancer should avoid routing traffic to unhealthy instances.

Common concepts:

```text
Liveness
Readiness
Health endpoint
```

For example:

```text
Instance started
but database unavailable
```

It may not be ready to receive traffic.

---

# 71. REST Contract Evolution

Suppose version 1 returns:

```json
{
  "name": "Phone"
}
```

Adding:

```json
{
  "name": "Phone",
  "brand": "Acme"
}
```

is usually backward-compatible for clients that ignore unknown fields.

But changing:

```text
name → productName
```

can break consumers.

---

# 72. Event Contract Evolution

Events need backward compatibility too.

Bad:

```text
Remove orderId
Rename customerId
```

without considering consumers.

Safer:

```text
Add optional field
```

and allow old consumers to continue working.

---

# 73. Schema Registry

In event-driven systems, especially Kafka ecosystems, schemas may be managed through a schema registry.

Benefits:

```text
Schema validation
Version management
Compatibility checks
```

This helps prevent incompatible event changes.

---

# 74. Consumer-Driven Contracts

Consumer-driven contract testing helps ensure:

```text
Provider
```

doesn't accidentally break:

```text
Consumer
```

Example:

```text
Order Service
expects Payment API:
POST /payments
```

A contract test can validate this expectation.

---

# 75. Service Mesh

At larger scale, service-to-service communication concerns can be handled partly by a service mesh.

Examples include:

```text
Istio
Linkerd
```

A service mesh can provide features such as:

```text
Traffic management
mTLS
Observability
Retries
Load balancing
```

Don't introduce a service mesh unless the operational complexity is justified.

---

# 76. Sidecar Pattern

A service mesh commonly uses a sidecar proxy:

```text
Pod
├── Application
└── Proxy
```

The proxy handles some networking concerns.

Conceptually:

```text
Service A
   |
Proxy A
   |
Network
   |
Proxy B
   |
Service B
```

---

# 77. When Not to Use a Service Mesh

For a small system:

```text
5 services
2 developers
```

a service mesh may create more complexity than value.

For large platforms with many services and strong platform engineering capabilities, it can become useful.

---

# 78. Communication Security

For internal APIs:

```text
TLS
Authentication
Authorization
Certificate management
Secret rotation
```

may be required.

For highly sensitive service-to-service traffic:

```text
mTLS
```

can provide mutual authentication.

---

# 79. API Gateway vs Service Mesh

They solve different problems.

### API Gateway

Primarily handles:

```text
External/client → backend
```

### Service Mesh

Primarily handles:

```text
Service → service
```

They can coexist.

---

# 80. Example Architecture

```text
                  Web/Mobile
                      |
                      ↓
                 API Gateway
                      |
          +-----------+-----------+
          |                       |
          ↓                       ↓
     Order Service          Product Service
          |
          ↓
      Payment Service
          |
          ↓
        Kafka
       /     \
      ↓       ↓
Notification Analytics
```

Possible infrastructure:

```text
Gateway
→ authentication/routing

REST
→ external APIs

gRPC
→ selected internal low-latency calls

Kafka
→ asynchronous events
```

---

# 81. Interview Scenario

### "Payment Service is slow. What happens?"

Good answer:

> "I'd first enforce a timeout so Order Service doesn't wait indefinitely. Depending on whether the operation is retryable and idempotent, I might use bounded retries with exponential backoff. A circuit breaker can prevent repeated calls when Payment remains unhealthy. For non-critical workflows I'd consider asynchronous processing."

---

# 82. Interview Scenario

### "Why not retry every failed request?"

Answer:

> "Retries can amplify load during an outage. They are most useful for transient failures and should be bounded with backoff and jitter. For non-idempotent operations, I also need idempotency protection before retrying."

---

# 83. Interview Scenario

### "REST or Kafka for order creation?"

Answer:

> "The initial order creation request is naturally synchronous because the client needs an immediate result. After the order is created, I could publish an event such as OrderCreated for asynchronous processing like notifications, analytics or fulfillment."

---

# 84. Interview Scenario

### "REST or Kafka for email?"

Answer:

> "Usually asynchronous messaging is a good fit because email doesn't need to block the main business transaction. The order service can publish an event and a notification consumer can process it independently."

---

# 85. Interview Scenario

### "REST or gRPC?"

Answer:

> "I'd use REST when interoperability and public API simplicity are priorities. For internal service-to-service communication where strong contracts, efficient serialization, low latency or streaming are important, I'd consider gRPC."

---

# 86. Interview Scenario

### "How do you prevent duplicate event processing?"

Answer:

> "I'd make the consumer idempotent. For example, I can persist a unique event ID or business idempotency key and ignore events that have already been successfully processed."

---

# 87. Interview Scenario

### "How do you handle poison messages?"

Answer:

> "I'd use bounded retries and then move the message to a dead-letter mechanism. I'd monitor the DLQ and investigate the root cause before reprocessing the message."

---

# 88. Interview Scenario

### "How do you preserve order?"

Answer:

> "I'd first determine whether ordering is actually required. If it is, I'd choose a messaging design that provides the required ordering guarantee, such as partitioning events by a stable key like orderId in Kafka."

---

# 89. Interview Scenario

### "What if a response is lost after payment succeeds?"

Answer:

> "The client or upstream service may retry. Payment processing therefore needs an idempotency mechanism so the retry doesn't create another charge."

---

# 90. Interview Scenario

### "What if Kafka is unavailable?"

Possible answer:

> "I'd first determine whether the event is critical to the business transaction. If the event must be guaranteed, I'd use a transactional outbox so the database transaction records the event reliably and a separate publisher can retry delivery. I wouldn't silently discard a business-critical event."

---

# 91. Interview Scenario

### "Can asynchronous messaging eliminate all failures?"

No.

It changes the failure model.

You still need:

```text
Retries
DLQ
Idempotency
Monitoring
Ordering strategy
Consumer recovery
Schema compatibility
```

---

# 92. Interview Scenario

### "Can asynchronous communication eliminate latency?"

No.

It can remove waiting from the synchronous request path, but processing still takes time.

You trade:

```text
Immediate consistency
```

for potentially:

```text
Eventual consistency
```

---

# 93. Common Communication Mistakes

```text
❌ Infinite HTTP timeouts
❌ Blind retries
❌ Retrying non-idempotent operations
❌ Every dependency synchronous
❌ No API versioning
❌ No error contract
❌ No correlation ID
❌ No event idempotency
❌ No DLQ strategy
❌ Ignoring schema evolution
❌ Treating internal network as trusted
❌ Excessive service-to-service calls
```

---

# 94. Practical Decision Framework

When choosing communication, ask:

```text
Does caller need an immediate response?
        |
       Yes
        ↓
REST / gRPC

       No
        ↓
Can it be event-driven?
        |
       Yes
        ↓
Kafka / Queue
```

Then evaluate:

```text
Latency
Availability
Consistency
Throughput
Ordering
Retry behavior
Idempotency
Security
Observability
```

---

# 95. Quick Comparison

| Feature | REST | gRPC | Messaging |
|---|---|---|---|
| Style | Request/response | RPC | Async |
| Typical format | JSON | Protobuf | Event/message |
| Human readable | Yes | Less | Depends |
| Public API | Excellent | Possible | Usually not direct |
| Internal calls | Good | Excellent for some workloads | Excellent for async |
| Streaming | Limited compared with gRPC | Strong | Event stream |
| Coupling | Runtime | Runtime | Looser |
| Immediate response | Yes | Yes | Usually no |
| Eventual consistency | Not inherent | Not inherent | Common |

---

# 96. Final Mental Model

Think:

```text
REST
→ "I need an answer now."

gRPC
→ "I need an efficient, strongly typed internal RPC."

Messaging
→ "Something happened; process it asynchronously."
```

This is not absolute, but it is a useful interview starting point.

---

# 97. Final Interview Answer

If asked:

> "How do microservices communicate?"

Use:

> "Microservices can communicate synchronously using REST or gRPC, or asynchronously using messaging systems such as Kafka or RabbitMQ. I use synchronous communication when the caller needs an immediate response and asynchronous messaging when the workflow can be decoupled. For synchronous calls I consider timeouts, retries, circuit breakers and idempotency. For messaging I consider delivery semantics, ordering, retries, dead-letter handling and idempotent consumers. The choice depends on latency, consistency, reliability and coupling requirements."

---

# 98. Revision Checklist

Before moving on, make sure you can explain:

```text
□ Synchronous communication
□ Asynchronous communication
□ REST
□ HTTP methods
□ HTTP status codes
□ 401 vs 403
□ 404 vs 204
□ 409
□ 202
□ REST idempotency
□ Idempotency keys
□ API versioning
□ Pagination
□ REST error contracts
□ Timeouts
□ Retries
□ Exponential backoff
□ Jitter
□ Circuit breaker
□ REST + Spring Boot
□ RestClient
□ WebClient
□ OpenFeign
□ gRPC
□ Protocol Buffers
□ gRPC streaming
□ REST vs gRPC
□ Kafka
□ RabbitMQ
□ Events vs commands
□ At-most-once
□ At-least-once
□ Exactly-once nuance
□ Idempotent consumers
□ Dead-letter queues
□ Ordering
□ Kafka partitions
□ Correlation IDs
□ Service discovery
□ Load balancing
□ Health checks
□ API contracts
□ Event contracts
□ Schema evolution
□ Contract testing
□ Service mesh
□ API Gateway vs service mesh
□ Communication failure scenarios
```

---

# 99. The Interviewer's Real Test

When an interviewer asks:

> "Would you use REST or Kafka?"

They usually aren't testing whether you memorized definitions.

They're testing whether you can reason about:

```text
Does the caller need the result immediately?
              ↓
How important is availability?
              ↓
Can the operation be asynchronous?
              ↓
What consistency is required?
              ↓
What happens if the dependency fails?
              ↓
Can retries create duplicate side effects?
              ↓
How will we observe and recover?
```

If you walk through those questions, your answer will sound like a backend engineer rather than someone simply listing technologies.

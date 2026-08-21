# REST API — Versioning, Idempotency & Compatibility

This file covers three important production REST API concepts:

```text
API Versioning
Idempotency
Backward Compatibility
```

These concepts become especially important when APIs are consumed by frontend applications, mobile clients, external partners, or other microservices.

---

# 1. Why API Evolution Matters

An API is a contract between:

```text
Provider
   ↕
Consumer
```

Consumers may include:

```text
Web frontend
Mobile app
Other services
External customers
Third-party integrations
```

Changing an API carelessly can break existing consumers.

---

# 2. API Contract

An API contract defines things such as:

```text
Endpoint
HTTP method
Request format
Response format
Status codes
Authentication
Error structure
Field meanings
```

Example:

```http
GET /api/v1/products/100
```

Response:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 75000
}
```

Changing this response can affect clients.

---

# 3. Breaking vs Non-Breaking Changes

A breaking change can cause existing clients to stop working.

Examples:

```text
Remove a response field clients require
Rename a required request field
Change field type
Change authentication requirements
Change endpoint semantics
Remove an endpoint
```

Usually safer changes include:

```text
Add an optional response field
Add a new endpoint
Add an optional request field
Improve documentation
```

But even additive changes can break poorly implemented clients, so compatibility should always be considered.

---

# 4. API Versioning

API versioning provides a mechanism for evolving an API while maintaining older contracts.

Common strategies:

```text
URI versioning
Header versioning
Media type versioning
Query parameter versioning
```

---

# 5. URI Versioning

Example:

```http
GET /api/v1/products
GET /api/v2/products
```

Advantages:

```text
Easy to understand
Easy to test
Easy to route
Visible in URLs
```

Disadvantages:

```text
Version appears in URLs
Multiple endpoint versions may need maintenance
```

It is a common and practical approach.

---

# 6. Header Versioning

Example:

```http
GET /api/products
X-API-Version: 2
```

Advantages:

```text
URL remains stable
Versioning is expressed through request metadata
```

Disadvantages:

```text
Less visible
Harder to discover manually
Requires clients to set headers correctly
```

---

# 7. Media Type Versioning

Example:

```http
Accept: application/vnd.example.v2+json
```

The requested representation determines the API version.

Advantages:

```text
Explicit content negotiation
URL remains stable
```

Disadvantages:

```text
More complex
Less familiar to many consumers
Harder to test manually
```

---

# 8. Query Parameter Versioning

Example:

```http
GET /api/products?version=2
```

This is simple but often less attractive for long-lived API contracts because the version becomes part of query semantics rather than the resource path or representation.

---

# 9. Which Versioning Strategy?

There is no universal winner.

For a typical Spring Boot backend, URI versioning can be attractive because it is:

```text
Simple
Explicit
Easy to document
Easy to route
Easy for clients to understand
```

The most important requirement is consistency.

---

# 10. Spring Boot URI Versioning

Example:

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductControllerV1 {
}
```

New version:

```java
@RestController
@RequestMapping("/api/v2/products")
public class ProductControllerV2 {
}
```

This allows both versions to coexist during migration.

---

# 11. Versioning at the Gateway

In a microservices architecture:

```text
Client
   ↓
API Gateway
   ↓
/api/v1 → Service
/api/v2 → Service
```

The gateway can route different API versions to different implementations.

However, versioning should not become only a gateway concern if service contracts themselves differ.

---

# 12. Versioning Responses

Suppose V1 returns:

```json
{
  "id": 100,
  "name": "Laptop"
}
```

V2 may return:

```json
{
  "productId": 100,
  "name": "Laptop",
  "pricing": {
    "amount": 75000,
    "currency": "INR"
  }
}
```

Do not silently change V1 to the V2 structure.

---

# 13. Separate DTOs by Version

A clean approach can be:

```text
ProductResponseV1
ProductResponseV2
```

Then:

```text
Controller V1
    ↓
V1 DTO

Controller V2
    ↓
V2 DTO
```

The internal domain model can remain shared when appropriate.

---

# 14. Avoid Versioning the Entire Codebase

Do not automatically create:

```text
ProductServiceV1
ProductServiceV2
ProductRepositoryV1
ProductRepositoryV2
```

for every version.

Often only the API representation differs.

Prefer:

```text
Controller V1 ──┐
                ├──> Shared business logic
Controller V2 ──┘
```

unless the business behavior genuinely differs.

---

# 15. API Deprecation

Older versions should have a migration plan.

Example:

```text
v1
 ↓
Deprecated
 ↓
Migration guidance
 ↓
v2
 ↓
v1 removed
```

Deprecation should communicate:

```text
What is changing
Why it is changing
What clients should use
When old behavior will be removed
```

---

# 16. Deprecation Header

An API can communicate deprecation through documentation and headers.

For example, an implementation may return:

```http
Deprecation: true
```

Some organizations also communicate a planned sunset date through their API lifecycle conventions.

The exact header strategy should be standardized across the organization.

---

# 17. Backward Compatibility

Backward compatibility means existing clients continue to work after a change.

Good principle:

```text
Prefer additive changes.
Avoid unnecessary breaking changes.
```

---

# 18. Adding Response Fields

Old:

```json
{
  "id": 100,
  "name": "Laptop"
}
```

New:

```json
{
  "id": 100,
  "name": "Laptop",
  "category": "Electronics"
}
```

This is often backward compatible because existing clients can ignore the additional field.

But clients that strictly validate schemas should still be considered.

---

# 19. Removing Response Fields

Old:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 75000
}
```

New:

```json
{
  "id": 100,
  "name": "Laptop"
}
```

This can break clients that depend on `price`.

Treat field removal as a potentially breaking change.

---

# 20. Renaming Fields

Changing:

```json
"name"
```

to:

```json
"productName"
```

is potentially breaking.

A safer migration can temporarily provide both fields if appropriate:

```json
{
  "name": "Laptop",
  "productName": "Laptop"
}
```

Then:

```text
Document new field
 ↓
Migrate consumers
 ↓
Deprecate old field
 ↓
Remove in a future breaking version
```

---

# 21. Changing Field Types

Old:

```json
"price": 75000
```

New:

```json
"price": "75000"
```

This can break consumers expecting a number.

Avoid changing types casually.

---

# 22. Enum Compatibility

Suppose:

```text
CREATED
PAID
SHIPPED
```

and a new value is added:

```text
CANCELLED
```

Some clients may fail if they assume the enum is closed.

Therefore, even adding enum values should be evaluated for client compatibility.

---

# 23. Optional Request Fields

Adding an optional request field is generally safer:

```json
{
  "name": "Laptop",
  "description": "Gaming laptop"
}
```

Existing clients that omit `description` can continue working if the server provides a sensible default.

---

# 24. Required Request Fields

Adding a new required request field can break old clients.

Old client:

```json
{
  "name": "Laptop"
}
```

New server requires:

```json
{
  "name": "Laptop",
  "currency": "INR"
}
```

The old client now fails.

Avoid this in a backward-compatible release.

---

# 25. Error Contract Compatibility

Changing:

```json
{
  "message": "Product not found"
}
```

to:

```json
{
  "errorMessage": "Product not found"
}
```

can break clients.

Error responses are part of the API contract too.

---

# 26. HTTP Status Compatibility

Changing:

```text
404 → 200
```

or:

```text
409 → 400
```

can affect client behavior.

Status codes are not merely implementation details.

Document them and treat meaningful changes carefully.

---

# 27. Idempotency

Idempotency means:

> Repeating the same operation produces the same intended business result.

This is especially important when clients retry requests because of:

```text
Network timeout
Connection failure
Client retry
Gateway retry
Mobile network instability
```

---

# 28. Safe vs Idempotent

These concepts are different.

Safe methods:

```text
GET
HEAD
OPTIONS
```

are intended not to modify server state.

Idempotent methods include:

```text
GET
PUT
DELETE
HEAD
OPTIONS
```

by HTTP semantics.

`POST` is generally not inherently idempotent.

---

# 29. GET Idempotency

Example:

```http
GET /api/products/100
```

Repeating it should not create multiple products or change the resource.

Therefore GET is intended to be safe and idempotent.

---

# 30. PUT Idempotency

Example:

```http
PUT /api/products/100
```

Request:

```json
{
  "name": "Laptop",
  "price": 75000
}
```

Sending the same request repeatedly should result in the same final representation.

```text
Request 1 → product updated
Request 2 → same final state
Request 3 → same final state
```

---

# 31. DELETE Idempotency

Example:

```http
DELETE /api/products/100
```

First request:

```text
Product deleted
```

Second request:

```text
Product already absent
```

The server state remains:

```text
Product does not exist
```

The response code may differ depending on API design.

Idempotency is about the resulting intended state, not necessarily identical response bodies for every repeated request.

---

# 32. POST and Idempotency

Consider:

```http
POST /api/orders
```

Without idempotency:

```text
Request
 ↓
Create Order #100
```

Client times out.

Client retries:

```text
Request
 ↓
Create Order #101
```

Now one user action created two orders.

---

# 33. Idempotency Key

A common solution is:

```http
Idempotency-Key: abc123
```

The client sends the same key when retrying the same logical operation.

---

# 34. Idempotency Flow

```text
POST /api/orders
Idempotency-Key: abc123
        ↓
Check key
   |
   +-- exists → return previous result
   |
   +-- absent
        ↓
     Process request
        ↓
     Store result
        ↓
     Return response
```

---

# 35. Idempotency Storage

The server can store:

```text
idempotency key
request fingerprint
operation status
response/result
resource ID
expiration time
```

Example:

```text
idempotency:abc123
```

---

# 36. Redis for Idempotency

Redis can be useful for idempotency state.

Conceptually:

```text
Client
  ↓
Spring Boot
  ↓
Redis
  ↓
Already processed?
```

A TTL can remove old keys after the appropriate retention period.

For critical financial operations, persistence and correctness requirements should be carefully evaluated rather than relying on a cache-like mechanism alone.

---

# 37. Idempotency Key Collision

Suppose:

```text
Request A
Idempotency-Key: abc123
```

Then a different request uses:

```text
Idempotency-Key: abc123
```

This should not silently return the wrong result.

A robust implementation can store a request fingerprint and reject reuse when the payload or important parameters differ.

---

# 38. Concurrent Duplicate Requests

Two identical requests may arrive simultaneously:

```text
Request A ─┐
           ├──> Server
Request B ─┘
```

Both may check:

```text
Key does not exist
```

before either stores it.

Therefore the idempotency mechanism needs atomic coordination.

Redis operations, database unique constraints, or another concurrency-safe design can help.

---

# 39. Database Unique Constraint

A database can enforce uniqueness:

```text
UNIQUE(user_id, idempotency_key)
```

This protects against duplicate creation even when multiple application instances receive the same request.

A strong design may combine:

```text
Application logic
+
Database constraint
```

---

# 40. Idempotent Order Creation

Example:

```text
User 100
Idempotency-Key: order-abc123
```

Database:

```text
user_id = 100
idempotency_key = order-abc123
order_id = 5001
```

Retry:

```text
Same user
Same key
```

Return:

```text
Order 5001
```

instead of creating another order.

---

# 41. Idempotency Status

An idempotent operation may have states such as:

```text
PROCESSING
SUCCEEDED
FAILED
```

The system needs clear rules for what happens when a retry arrives while the original request is still processing.

---

# 42. Idempotency and Transactions

For important operations:

```text
BEGIN
 ↓
Validate request
 ↓
Create business record
 ↓
Store idempotency record
 ↓
COMMIT
```

The business operation and idempotency state should be coordinated carefully.

Otherwise:

```text
Order committed
Idempotency key not stored
```

can allow a retry to create a duplicate.

---

# 43. Outbox and Idempotency

Suppose order creation also publishes an event:

```text
Create Order
   ↓
Publish OrderCreated
```

A robust architecture may use:

```text
Database transaction
   ↓
Order
+
Outbox event
   ↓
Commit
   ↓
Event publisher
   ↓
Kafka
```

Idempotency protects request duplication, while the Outbox Pattern helps protect database-to-event consistency.

---

# 44. Idempotent Consumers

Event consumers may also receive duplicate messages.

Example:

```text
Kafka message
OrderCreated #5001
```

could be delivered again.

The consumer should be designed so repeated processing does not corrupt state.

Possible approaches:

```text
Processed-event table
Unique event ID
Idempotent update
Upsert
Business key constraint
```

---

# 45. Retry Safety

Before adding retries, ask:

```text
Is this operation idempotent?
```

Retrying:

```text
GET
```

is generally safe.

Retrying:

```text
POST /payments
```

without idempotency can be dangerous.

---

# 46. Timeout + Retry Problem

Consider:

```text
Client
 ↓
POST /orders
 ↓
Server processes successfully
 ↓
Network timeout
 ↓
Client sees failure
```

The client does not know whether the server completed the operation.

Retrying without idempotency can create a duplicate.

This is one of the strongest practical reasons for idempotency keys.

---

# 47. Idempotency vs Exactly Once

Idempotency does not mean:

```text
The request physically executes exactly once.
```

It means repeated attempts produce one intended business outcome.

This distinction matters in distributed systems.

---

# 48. Exactly-Once Processing

In distributed systems, true end-to-end exactly-once behavior is difficult.

Instead, systems commonly combine:

```text
At-least-once delivery
+
Idempotent processing
+
Unique constraints
+
Transactional boundaries
```

to achieve effectively-once business behavior.

---

# 49. Compatibility in Microservices

Suppose:

```text
Order Service
     ↓
Payment Service
```

If Payment Service changes:

```text
POST /payments
```

it can break Order Service.

Therefore service-to-service APIs also need:

```text
Versioning
Compatibility
Contract testing
Deprecation
Migration
```

---

# 50. Consumer-Driven Contracts

A consumer-driven contract captures what a consumer expects from a provider.

Conceptually:

```text
Consumer
   ↓
Expected API behavior
   ↓
Contract
   ↓
Provider verifies it
```

This helps detect breaking changes before deployment.

---

# 51. Contract Testing

Contract testing can verify:

```text
Request schema
Response schema
Status codes
Required fields
Headers
Behavior
```

It is particularly useful for microservices.

---

# 52. Schema Evolution

When changing JSON schemas:

```text
Prefer additive changes
```

For example:

```text
Add optional field
```

before:

```text
Remove existing field
```

This allows consumers time to migrate.

---

# 53. Consumer Tolerance

A resilient client should generally:

```text
Ignore unknown response fields
```

when its serialization framework and contract allow this.

This makes additive server changes safer.

But clients should not blindly ignore changes to fields they depend on.

---

# 54. Versioning vs Compatibility

Versioning:

```text
Mechanism for supporting different contracts
```

Compatibility:

```text
Ability to change the API without breaking existing consumers
```

Good API design tries to maximize compatibility so that new versions are needed less frequently.

---

# 55. When Should You Create a New API Version?

Consider a new major version when you need a breaking change such as:

```text
Change response structure substantially
Remove required fields
Change field types
Change authentication model
Change endpoint semantics
Remove important behavior
```

Do not create a new version for every small additive feature.

---

# 56. Versioning Ecommerce API

Suppose V1:

```http
GET /api/v1/products/100
```

returns:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 75000
}
```

V2:

```http
GET /api/v2/products/100
```

returns:

```json
{
  "productId": 100,
  "name": "Laptop",
  "pricing": {
    "amount": 75000,
    "currency": "INR"
  }
}
```

Both can coexist while clients migrate.

---

# 57. Ecommerce Order Idempotency

Request:

```http
POST /api/v1/orders
Idempotency-Key: checkout-123
```

Server:

```text
Check key
 ↓
Not found
 ↓
Create order
 ↓
Store key + result
 ↓
Return 201
```

Retry:

```text
Same key
 ↓
Existing result
 ↓
Return same logical order result
```

---

# 58. Payment Idempotency

Payment operations are highly sensitive to duplicate requests.

Example:

```http
POST /api/v1/payments
Idempotency-Key: payment-987
```

A robust implementation should ensure:

```text
One logical payment
```

even if the client retries because of network failures.

Payment providers often expose idempotency mechanisms for this reason.

---

# 59. Idempotency Key Lifetime

Keys should not necessarily live forever.

Choose retention based on:

```text
Expected retry window
Business risk
Storage cost
Operation type
```

Example:

```text
Short-lived order request → hours
High-risk financial operation → potentially longer
```

There is no universal TTL.

---

# 60. Idempotency Response Replay

When a retry uses the same valid idempotency key, the API can return the previously stored result.

Conceptually:

```text
First:
201 Created
Order #5001

Retry:
201 Created
Order #5001
```

Some APIs may return a different status depending on implementation, but the important property is the same logical outcome.

---

# 61. Invalid Idempotency Key

Reject keys that are:

```text
Missing where required
Malformed
Too long
Unsafe
Reused for a different request
```

Return an appropriate documented error such as:

```text
400 Bad Request
409 Conflict
```

depending on the API contract.

---

# 62. Idempotency and HTTP PUT

PUT is naturally suited to idempotent replacement semantics.

Example:

```http
PUT /api/users/100
```

Repeatedly sending:

```json
{
  "name": "Sudhir"
}
```

should leave the resource in the same intended state.

---

# 63. PATCH and Idempotency

PATCH can be idempotent or non-idempotent depending on the operation.

Example idempotent patch:

```json
{
  "status": "SHIPPED"
}
```

Repeated application leaves the same state.

Potentially non-idempotent operation:

```json
{
  "incrementBalance": 100
}
```

Each execution changes the state again.

Therefore PATCH semantics must be designed carefully.

---

# 64. DELETE and Response Semantics

First request:

```http
DELETE /api/products/100
```

might return:

```http
204 No Content
```

Second request might return:

```http
404 Not Found
```

This can still be consistent with DELETE being idempotent because the intended final state remains:

```text
Product 100 does not exist
```

---

# 65. API Compatibility Checklist

Before changing an API ask:

```text
Will existing clients break?
Did a field disappear?
Did a field type change?
Did a required field get added?
Did status codes change?
Did error schema change?
Did authentication change?
Did endpoint semantics change?
```

---

# 66. API Evolution Checklist

```text
1. Identify consumers
2. Define the change
3. Classify breaking/non-breaking
4. Prefer additive changes
5. Update documentation
6. Add/modify contract tests
7. Communicate deprecation
8. Provide migration path
9. Monitor usage
10. Remove old version only after migration
```

---

# 67. Common Mistakes

Avoid:

```text
Creating a new version for every tiny change
Breaking clients without notice
Removing fields suddenly
Changing status codes casually
Ignoring error contracts
Using POST without idempotency for retry-sensitive operations
Assuming retries are always safe
Using non-atomic idempotency storage
Keeping old versions forever
```

---

# 68. Interview: What Is API Versioning?

> API versioning allows an API to evolve while supporting different contracts for existing and new consumers. Common approaches include URI, header and media-type versioning. I prefer a consistent strategy and use a new version mainly when a breaking change is required.

---

# 69. Interview: What Is Idempotency?

> Idempotency means that repeating the same logical operation produces the same intended business result. GET, PUT and DELETE are designed to be idempotent by HTTP semantics, while POST is generally not. For retry-sensitive POST operations such as order creation, I can use an idempotency key.

---

# 70. Interview: How Does an Idempotency Key Work?

> The client sends a unique idempotency key with the request. The server atomically records the key and the operation result. If the same key is retried, the server returns the existing result instead of creating another business operation. I also validate that the same key is not reused with a different request payload.

---

# 71. Interview: How Would You Implement Idempotency in Spring Boot?

> I would accept an `Idempotency-Key` header for operations that need it, validate it, and use a database unique constraint or an atomic distributed store such as Redis to coordinate requests. The idempotency record needs to be consistent with the business transaction so that a successful order cannot exist without its corresponding idempotency record.

---

# 72. Interview: Why Is Idempotency Important for Payments?

> A client can time out after the payment server successfully processes the request. If the client retries without idempotency, the payment could be charged twice. An idempotency key lets the server recognize the retry as the same logical payment.

---

# 73. Interview: What Is a Breaking API Change?

> A breaking change is a change that can cause an existing consumer to stop working correctly. Examples include removing required response fields, changing field types, adding required request fields, changing authentication behavior, or changing endpoint semantics.

---

# 74. Interview: How Do You Maintain Backward Compatibility?

> I prefer additive changes, avoid removing or changing existing fields unnecessarily, keep error and status-code contracts stable, and use versioning when a genuine breaking change is required. For microservices, contract testing can help detect incompatible provider changes before deployment.

---

# 75. Interview: PUT vs POST for Idempotency?

> PUT is defined to be idempotent, so repeating the same request should result in the same intended resource state. POST is not inherently idempotent, so when POST represents a retry-sensitive operation such as creating an order or payment, I use an idempotency mechanism when appropriate.

---

# 76. Interview: Can DELETE Return 404 on a Retry?

> Yes. Idempotency is about the resulting state, not necessarily identical responses. After the first DELETE, the resource is absent. A subsequent DELETE can return 404 while the final state remains unchanged. The API should document its chosen behavior consistently.

---

# 77. Interview: Is PATCH Idempotent?

> PATCH is not automatically idempotent. It depends on the operation. Setting a field to a fixed value can be idempotent, while an operation such as incrementing a value may not be. The API contract should make the behavior clear.

---

# 78. Interview: Version Every API Change?

> No. I avoid unnecessary versions because maintaining multiple versions increases complexity. I first determine whether the change can be backward compatible. I create a new version when the required change is genuinely breaking or when the API lifecycle strategy calls for it.

---

# 79. Final Mental Model

```text
                    API CONTRACT
                         |
          +--------------+--------------+
          |              |              |
      Versioning     Idempotency   Compatibility
          |              |              |
     API evolution    Safe retry     Client safety
          |              |              |
          +--------------+--------------+
                         |
                         v
                  Reliable REST API
```

For an ecommerce backend:

```text
Product API
    ↓
Backward-compatible changes

Order API
    ↓
Idempotency-Key

Payment API
    ↓
Strong idempotency

Microservice API
    ↓
Versioning + contract testing

Old clients
    ↓
Deprecation + migration
```

> **A production REST API is not finished when the endpoint works. It must also survive retries, client evolution, deployment changes and multiple consumers. Design the contract for compatibility, make retry-sensitive operations idempotent, and introduce new versions only when a breaking change is genuinely necessary.**

# REST API — Design Best Practices and Production Patterns

This file covers practical REST API design decisions that matter in real Spring Boot backend systems and interviews.

---

# 1. What Makes a Good REST API?

A good API should be:

```text
Consistent
Predictable
Secure
Maintainable
Observable
Versionable
Efficient
Easy for clients to consume
```

The goal is not to make every endpoint complicated. Good API design usually comes from applying a small set of consistent rules.

---

# 2. Resource-Oriented URLs

Prefer nouns representing resources.

Good:

```http
GET /api/products
GET /api/products/100
POST /api/products
PUT /api/products/100
DELETE /api/products/100
```

Avoid action-heavy URLs such as:

```http
GET /api/getProducts
POST /api/createProduct
POST /api/deleteProduct
```

HTTP methods already communicate the operation.

---

# 3. HTTP Methods

Common methods:

```text
GET     → read
POST    → create/process
PUT     → replace/update
PATCH   → partial update
DELETE  → delete
```

Example:

```http
GET /products/100
POST /products
PUT /products/100
PATCH /products/100
DELETE /products/100
```

---

# 4. GET Should Not Modify State

A GET request should generally be safe.

Good:

```http
GET /api/products/100
```

Bad:

```http
GET /api/products/100/delete
```

Do not hide state-changing operations behind GET.

---

# 5. POST vs PUT

POST is commonly used when the server creates a resource and determines the resource identifier.

```http
POST /api/products
```

PUT is commonly used when the client knows the target resource URI and sends a complete replacement representation.

```http
PUT /api/products/100
```

The exact semantics should remain consistent across the API.

---

# 6. PATCH

PATCH is useful for partial updates.

Example:

```http
PATCH /api/users/100
```

Request:

```json
{
  "phoneNumber": "9999999999"
}
```

Only the specified field needs to change.

Be explicit about PATCH semantics because different implementations can interpret partial updates differently.

---

# 7. Idempotency

An operation is idempotent if repeating the same request produces the same intended final state.

Commonly:

```text
GET     → idempotent
PUT     → idempotent
DELETE  → idempotent
POST    → generally not inherently idempotent
```

Example:

```http
PUT /products/100
```

Sending the same replacement repeatedly should result in the same final representation.

---

# 8. POST and Idempotency Keys

Some POST operations must safely handle retries.

Example:

```http
POST /api/payments
Idempotency-Key: 7f4e...
```

If the client retries because of a network timeout, the server can recognize the same operation and avoid processing it twice.

This is especially useful for:

```text
Payments
Orders
Reservations
External side effects
```

---

# 9. HTTP Status Codes

Use status codes consistently.

Common examples:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Content
429 Too Many Requests
500 Internal Server Error
503 Service Unavailable
```

The exact choice should match the API contract.

---

# 10. 200 OK

Use when the request succeeds and a response body is returned.

Example:

```http
GET /api/products/100
```

Response:

```http
200 OK
```

---

# 11. 201 Created

Use when a request successfully creates a resource.

Example:

```http
POST /api/products
```

Response:

```http
201 Created
```

It is often useful to return the created representation and/or a `Location` header.

---

# 12. 204 No Content

Use when the request succeeds but there is no response body.

Example:

```http
DELETE /api/products/100
```

Response:

```http
204 No Content
```

Do not return an unnecessary JSON body with a 204 response.

---

# 13. 400 Bad Request

Use for malformed or invalid request syntax/parameters.

Examples:

```text
Invalid JSON
Invalid query parameter format
Malformed request structure
```

For validation errors, some APIs use 400 while others use 422. The important thing is consistency in the API contract.

---

# 14. 401 Unauthorized

Use when the request lacks valid authentication.

Examples:

```text
Missing token
Expired token
Invalid token
```

The user needs valid authentication.

---

# 15. 403 Forbidden

Use when the user is authenticated but does not have sufficient permission.

Example:

```text
USER attempts ADMIN operation
```

---

# 16. 404 Not Found

Use when the requested resource does not exist.

Example:

```http
GET /api/products/999999
```

If no such product exists:

```http
404 Not Found
```

---

# 17. 409 Conflict

Useful when the request conflicts with the current state.

Examples:

```text
Duplicate email
Concurrent update conflict
Order cannot transition from current state
```

---

# 18. 422 Unprocessable Content

Can be used when the request is syntactically valid but semantically invalid.

Example:

```json
{
  "quantity": -5
}
```

Whether to use 400 or 422 should be standardized within the API.

---

# 19. 429 Too Many Requests

Use when a client exceeds a rate limit.

Example:

```http
429 Too Many Requests
```

The response may include:

```text
Retry-After
```

when appropriate.

---

# 20. 500 Internal Server Error

Use for unexpected server-side failures.

Do not expose:

```text
Stack traces
Database credentials
Internal exception details
Implementation paths
```

to API clients.

Log technical details internally.

---

# 21. 503 Service Unavailable

Useful when the service is temporarily unable to handle the request.

Examples:

```text
Dependency unavailable
Maintenance
Temporary overload
```

It can communicate that retrying later may be appropriate.

---

# 22. Consistent Response Structure

A large API should avoid every endpoint returning a completely different structure.

Success:

```json
{
  "data": {
    "id": 100,
    "name": "Laptop"
  }
}
```

Error:

```json
{
  "timestamp": "2026-08-21T10:00:00Z",
  "status": 404,
  "error": "Not Found",
  "message": "Product not found",
  "path": "/api/products/100"
}
```

The exact schema should be standardized.

---

# 23. Error Response Design

A useful error response can contain:

```text
status
code
message
timestamp
path
field errors
trace/correlation ID
```

Avoid exposing internal implementation details.

---

# 24. Validation Error

Example request:

```json
{
  "name": "",
  "price": -10
}
```

Response:

```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "name",
      "message": "Name is required"
    },
    {
      "field": "price",
      "message": "Price must be positive"
    }
  ]
}
```

This makes client-side correction easier.

---

# 25. DTOs

Do not automatically expose JPA entities directly from controllers.

Prefer:

```text
Request DTO
     ↓
Service
     ↓
Entity
     ↓
Repository
     ↓
Entity
     ↓
Response DTO
```

DTOs provide control over:

```text
API contract
Sensitive fields
Validation
Serialization
Versioning
```

---

# 26. Request DTO

Example:

```java
public record CreateProductRequest(
    @NotBlank String name,
    @Positive BigDecimal price
) {}
```

This defines exactly what the client is allowed to submit.

---

# 27. Response DTO

Example:

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {}
```

The API response is deliberately separated from the persistence entity.

---

# 28. Avoid Returning Passwords

Even if an entity contains:

```text
passwordHash
```

the response DTO should not expose it.

Use explicit response models.

---

# 29. Pagination

Large collections should generally be paginated.

Example:

```http
GET /api/products?page=0&size=20
```

Response:

```json
{
  "data": [],
  "page": 0,
  "size": 20,
  "totalElements": 250,
  "totalPages": 13
}
```

For very large or changing datasets, cursor/keyset pagination can be more efficient.

---

# 30. Cursor Pagination

Instead of:

```text
page=10000
```

use a cursor:

```http
GET /api/products?limit=20&cursor=eyJpZCI6...
```

This can avoid expensive large offsets.

Cursor values should be treated as opaque API tokens rather than something clients need to understand.

---

# 31. Sorting

Example:

```http
GET /api/products?sort=price,asc
```

Multiple sort fields may be supported:

```http
GET /api/products?sort=category,asc&sort=price,desc
```

Whitelist allowed sort fields.

Do not blindly pass arbitrary client input into SQL fragments.

---

# 32. Filtering

Example:

```http
GET /api/products?category=phones&minPrice=10000&maxPrice=50000
```

Filtering should be:

```text
Predictable
Validated
Documented
Efficiently indexed
```

---

# 33. Search

Example:

```http
GET /api/products?search=iphone
```

For simple datasets, database search may be enough.

For complex full-text requirements, consider:

```text
Elasticsearch/OpenSearch
Dedicated search indexes
```

Do not introduce a search engine without a real requirement.

---

# 34. API Versioning

Common approaches:

```text
URI versioning
/api/v1/products

Header versioning
Accept: application/vnd.company.v1+json

Query parameter
/api/products?version=1
```

URI versioning is easy for developers to understand, but the organization should choose one consistent strategy.

---

# 35. Backward Compatibility

When evolving APIs, avoid breaking existing clients unnecessarily.

Prefer:

```text
Add optional fields
Add new endpoints
Deprecate old fields gradually
```

Be careful when:

```text
Removing fields
Changing types
Changing meanings
Changing required behavior
```

---

# 36. API Deprecation

A deprecated API should have:

```text
Documentation
Migration path
Timeline
Replacement endpoint
Monitoring
```

Do not silently remove APIs used by clients.

---

# 37. URL Naming

Prefer lowercase, predictable resource paths:

```text
/api/products
/api/products/100
/api/product-categories
```

Avoid inconsistent naming such as:

```text
/api/Product
/api/get_product
/api/productList
```

Consistency matters more than the exact convention.

---

# 38. Nested Resources

Sometimes relationships can be represented in URLs.

Example:

```http
GET /api/users/100/orders
```

This can be useful when the relationship is central to the request.

Avoid deeply nested paths such as:

```text
/users/1/orders/2/items/3/reviews/4
```

when they become difficult to maintain.

---

# 39. Query Parameters vs Path Parameters

Path parameter:

```http
GET /products/100
```

Use when identifying a specific resource.

Query parameters:

```http
GET /products?category=phones&page=0
```

Use for:

```text
Filtering
Sorting
Pagination
Searching
Optional behavior
```

---

# 40. Request Body

Use the request body for structured data being created or updated.

Example:

```http
POST /api/products
Content-Type: application/json
```

```json
{
  "name": "Laptop",
  "price": 75000
}
```

---

# 41. Content Negotiation

HTTP can support different representations using headers.

Example:

```http
Accept: application/json
```

The server can select an appropriate representation.

Most modern JSON APIs standardize on JSON rather than supporting many representations without a clear need.

---

# 42. Content-Type

`Content-Type` describes the request body format.

Example:

```http
Content-Type: application/json
```

The server can reject unsupported content types.

---

# 43. Accept Header

`Accept` tells the server which response representations the client can handle.

Example:

```http
Accept: application/json
```

Do not confuse:

```text
Content-Type → request body format
Accept → desired response format
```

---

# 44. ETags

An ETag identifies a particular representation version.

Example:

```http
ETag: "product-100-v7"
```

A client can later send:

```http
If-None-Match: "product-100-v7"
```

If unchanged:

```http
304 Not Modified
```

This can reduce unnecessary response payloads.

---

# 45. Cache-Control

HTTP caching can be controlled using headers.

Example:

```http
Cache-Control: max-age=300
```

Be careful caching personalized or sensitive responses.

---

# 46. Conditional Requests

Useful headers include:

```text
If-None-Match
If-Match
If-Modified-Since
If-Unmodified-Since
```

They can support:

```text
Efficient caching
Optimistic concurrency
```

---

# 47. Optimistic Concurrency

Suppose two users edit the same product.

Version:

```text
Product version = 7
```

Client sends:

```http
If-Match: "7"
```

If the database is already at version 8:

```http
412 Precondition Failed
```

The client can reload the latest version before updating.

---

# 48. Correlation ID

A correlation/request ID helps trace a request across services.

Example:

```http
X-Correlation-ID: 8f3a...
```

Flow:

```text
Gateway
  ↓
Product Service
  ↓
Order Service
  ↓
Database
```

The same identifier can appear in logs.

---

# 49. Distributed Tracing

For microservices, tools such as:

```text
OpenTelemetry
```

can propagate trace context.

This helps answer:

```text
Which service caused the latency?
Where did the request fail?
How long did each dependency take?
```

---

# 50. API Timeouts

Every external dependency should have sensible timeouts.

Without timeouts:

```text
Slow dependency
      ↓
Request threads wait
      ↓
Thread pool exhaustion
      ↓
Application outage
```

Configure:

```text
Connect timeout
Read/response timeout
Connection pool timeout
```

according to the client and dependency.

---

# 51. Retries

Retries can help with transient failures.

But careless retries can make an outage worse:

```text
Service A
   ↓ retry
Service B
   ↓ retry
Database
```

Use:

```text
Limited attempts
Exponential backoff
Jitter
Retry only transient failures
```

---

# 52. Retry + Idempotency

Retries are safer for idempotent operations.

For non-idempotent operations such as payment creation, use an idempotency mechanism when appropriate.

---

# 53. Circuit Breaker

A circuit breaker prevents repeatedly calling an unhealthy dependency.

States:

```text
CLOSED
   ↓ failures
OPEN
   ↓ wait
HALF_OPEN
   ↓ success
CLOSED
```

This protects system resources during dependency failures.

---

# 54. Bulkhead

Bulkhead isolation prevents one dependency from consuming all application resources.

Example:

```text
Payment calls → pool A
Product calls → pool B
```

If payment becomes slow, product requests can still have available capacity.

---

# 55. API Gateway

A gateway can centralize:

```text
Routing
Authentication
Rate limiting
TLS termination
CORS
Observability
```

But avoid putting all business logic in the gateway.

Business rules generally belong in the appropriate service.

---

# 56. Service Layer

Typical Spring Boot architecture:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

Controller:

```text
HTTP concerns
```

Service:

```text
Business logic
```

Repository:

```text
Data access
```

---

# 57. Controller Responsibility

A controller should mainly:

```text
Receive request
Validate input
Map request
Call service
Build HTTP response
```

Avoid putting complex business logic into controllers.

---

# 58. Service Responsibility

The service layer should handle:

```text
Business rules
Transactions
Workflow orchestration
Authorization checks where appropriate
Coordination between repositories/services
```

---

# 59. Repository Responsibility

Repository code should focus on:

```text
Persistence
Queries
Entity retrieval
Database interaction
```

Do not hide complex business rules inside repository classes.

---

# 60. Transaction Boundary

For a business operation:

```text
Create order
 ↓
Reserve inventory
 ↓
Save order items
```

transaction boundaries should be designed around the consistency requirements.

In Spring:

```java
@Transactional
public OrderResponse createOrder(...) {
    ...
}
```

Do not assume every HTTP request needs a database transaction.

---

# 61. N+1 Queries

A REST endpoint can accidentally trigger:

```text
1 query for products
+
N queries for related data
```

Example:

```text
1 product query
100 category queries
```

This can destroy performance.

Use appropriate:

```text
Fetch joins
Entity graphs
Projections
Batch fetching
Explicit query design
```

---

# 62. Lazy Loading

JPA lazy relationships can cause unexpected database access.

Avoid returning entities directly from controllers because serialization may trigger lazy loading.

Map entities to DTOs inside a controlled transaction/service boundary.

---

# 63. API Performance

Measure:

```text
p50 latency
p95 latency
p99 latency
Throughput
Error rate
Database time
External dependency time
```

Averages alone can hide slow requests.

---

# 64. Request Size Limits

Protect APIs from excessively large requests.

Examples:

```text
Maximum JSON body size
Maximum upload size
Maximum header size
```

This reduces memory and resource exhaustion risks.

---

# 65. File Upload APIs

For uploads:

```http
POST /api/files
Content-Type: multipart/form-data
```

Validate:

```text
File size
File type
File name
Content
Storage destination
```

Do not trust the filename or MIME type alone.

---

# 66. API Documentation

Use:

```text
OpenAPI
Swagger UI
```

Document:

```text
Endpoints
Request schemas
Response schemas
Authentication
Status codes
Errors
Examples
```

Documentation should match the actual API behavior.

---

# 67. OpenAPI

OpenAPI provides a machine-readable API contract.

It can support:

```text
Documentation
Client generation
Testing
Contract review
API discovery
```

---

# 68. Health Endpoints

Production APIs should expose health information carefully.

Spring Boot Actuator can provide:

```text
/actuator/health
```

Avoid exposing sensitive operational details publicly.

---

# 69. Graceful Shutdown

During deployment:

```text
Application receives shutdown
        ↓
Stop accepting new work
        ↓
Finish active requests
        ↓
Close resources
        ↓
Terminate
```

Graceful shutdown reduces dropped requests during deployments.

---

# 70. API Security Basics

A production API should consider:

```text
HTTPS
Authentication
Authorization
Validation
Rate limiting
CORS
CSRF where relevant
Secrets
Security headers
Dependency vulnerabilities
Audit logging
```

Security should be part of API design from the beginning.

---

# 71. API Observability

Three major pillars:

```text
Logs
Metrics
Traces
```

Logs answer:

```text
What happened?
```

Metrics answer:

```text
How often/how much?
```

Traces answer:

```text
Where did the request spend time?
```

---

# 72. Structured Logging

Prefer structured logs such as:

```json
{
  "level": "ERROR",
  "service": "product-service",
  "traceId": "abc123",
  "endpoint": "/api/products/100",
  "message": "Database timeout"
}
```

This is easier to search and analyze than inconsistent free-form messages.

---

# 73. Don't Log Sensitive Data

Never casually log:

```text
Passwords
JWTs
Refresh tokens
Credit card numbers
Secrets
Personal information
```

Mask sensitive values when operational debugging requires some visibility.

---

# 74. API Contract Testing

Contract testing verifies that:

```text
Provider
```

and:

```text
Consumer
```

agree on API behavior.

This is particularly useful in microservice environments.

---

# 75. Integration Testing

Test the actual interaction between:

```text
Controller
Service
Database
Security
External dependencies
```

where appropriate.

Testcontainers can provide realistic infrastructure dependencies during integration testing.

---

# 76. Unit vs Integration Test

Unit test:

```text
Fast
Isolated
Mocks dependencies
```

Integration test:

```text
Tests real component interaction
May use real database/container
Slower
Higher confidence for integration behavior
```

A good project uses both appropriately.

---

# 77. API Testing Tools

Common tools include:

```text
Postman
Insomnia
curl
Swagger UI
JUnit
MockMvc
WebTestClient
```

Automated tests should be part of the build pipeline.

---

# 78. API Contract Example

For:

```http
POST /api/products
```

Document:

```text
Request
Authentication
Validation
201 response
400 validation response
401 authentication response
403 authorization response
409 conflict response
```

A clear contract reduces client-side confusion.

---

# 79. Production API Checklist

```text
□ Resource-oriented URLs
□ Correct HTTP methods
□ Consistent status codes
□ DTOs
□ Validation
□ Pagination
□ Filtering
□ Sorting
□ Versioning strategy
□ Error contract
□ Authentication
□ Authorization
□ Rate limiting
□ Timeouts
□ Retry strategy
□ Idempotency
□ Circuit breakers where needed
□ Observability
□ API documentation
□ Integration tests
□ Graceful shutdown
```

---

# 80. Interview: What Makes a Good REST API?

> I focus on predictable resource-oriented URLs, correct HTTP semantics, consistent status codes and error responses, DTO-based contracts, validation, pagination, security, observability and backward compatibility. For production systems I also consider timeouts, retries, idempotency and resilience.

---

# 81. Interview: PUT vs PATCH?

> PUT is generally used to replace the representation of a resource, while PATCH is used for partial modifications. PUT is commonly idempotent, whereas PATCH idempotency depends on how the operation is designed.

---

# 82. Interview: What Is Idempotency?

> An operation is idempotent when repeating the same request results in the same intended final state. GET, PUT and DELETE are generally designed to be idempotent. For retry-sensitive POST operations such as payments, I can use an idempotency key.

---

# 83. Interview: How Do You Handle API Errors?

> I use a consistent error response structure with an HTTP status, application error code, safe message, timestamp and correlation information where useful. Technical details such as stack traces remain in internal logs rather than being exposed to clients.

---

# 84. Interview: Why Use DTOs?

> DTOs keep the API contract separate from persistence entities. They prevent accidental exposure of sensitive fields, give me explicit request validation and response control, and make API evolution easier.

---

# 85. Interview: How Do You Improve API Performance?

> I first measure where the latency comes from. Then I look at database queries, indexes, N+1 problems, pagination, caching, connection pools and external dependency latency. I also use timeouts and appropriate resilience patterns rather than simply adding more threads.

---

# 86. Interview: How Do You Handle Retries?

> I retry only transient failures, use a small number of attempts with exponential backoff and jitter, and avoid retrying operations that are not safe to repeat unless they have idempotency protection.

---

# 87. Interview: What Is a Circuit Breaker?

> A circuit breaker stops repeatedly calling an unhealthy dependency. It moves from closed to open after failures, rejects or fails fast while the dependency is unhealthy, and then enters half-open to test whether recovery has occurred.

---

# 88. Interview: How Do You Design APIs for Microservices?

> I keep service boundaries around business capabilities, expose stable contracts, use DTOs, avoid sharing database schemas between services, and design for timeouts, retries, observability and eventual consistency where appropriate.

---

# 89. Interview: How Do You Version REST APIs?

> I choose a consistent versioning strategy such as `/api/v1` and introduce breaking changes through a new version. I prefer additive and backward-compatible changes whenever possible and provide a migration path before retiring an older version.

---

# 90. Interview: How Do You Prevent Duplicate Orders?

> For retry-sensitive order creation, I would use an idempotency key tied to the logical client operation. The server stores the result or operation state and returns the same outcome for a repeated key instead of creating another order.

---

# 91. Interview: How Do You Trace a Request Across Microservices?

> I propagate a correlation or trace ID through the request chain and use structured logs, metrics and distributed tracing. With OpenTelemetry, I can inspect the individual spans and identify which service or dependency caused latency or failure.

---

# 92. Interview: What Happens When a Dependency Is Slow?

> I use connection and response timeouts so requests do not wait indefinitely. Depending on the dependency, I may add bounded retries, circuit breaking, bulkheads or a fallback. The important point is to prevent one slow dependency from exhausting the whole application.

---

# 93. Interview: How Would You Design an Ecommerce Product API?

> I would expose resource-oriented endpoints such as `/api/products` and `/api/products/{id}`. Product listing would support pagination, filtering and sorting. Product responses would use DTOs, authentication would protect administrative operations, Redis could cache frequently accessed product details, and database updates would invalidate affected cache entries.

---

# 94. Ecommerce API Example

```text
GET    /api/products
GET    /api/products/{id}
POST   /api/products
PUT    /api/products/{id}
PATCH  /api/products/{id}
DELETE /api/products/{id}

GET    /api/products?category=phones&page=0&size=20
```

Possible order APIs:

```text
POST /api/orders
GET  /api/orders/{id}
GET  /api/users/{id}/orders
```

Authorization must ensure users can only access resources they are permitted to access.

---

# 95. Final Mental Model

```text
                    CLIENT
                       ↓
                    HTTPS
                       ↓
                 API GATEWAY
                       ↓
             Security + Rate Limit
                       ↓
                 CONTROLLER
                       ↓
                   SERVICE
                       ↓
          +------------+------------+
          |                         |
        CACHE                    REPOSITORY
          |                         |
          |                       DATABASE
          |
       Fast reads

Supporting concerns:

Logs + Metrics + Traces
Timeouts + Retries
Validation + Error Handling
Documentation + Testing
```

> **A production REST API is more than a collection of controllers. It is a stable contract backed by correct HTTP semantics, security, validation, persistence, resilience, observability and thoughtful evolution.**

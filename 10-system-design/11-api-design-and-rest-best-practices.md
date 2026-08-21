# System Design — File 11: API Design & REST Best Practices

APIs are the contract between clients and backend services.

A good API should be:

```text
Clear
Consistent
Predictable
Secure
Versionable
Observable
Easy to evolve
```

For backend interviews, don't just memorize HTTP methods.

Be able to explain:

```text
Resource design
HTTP semantics
Status codes
Validation
Pagination
Filtering
Sorting
Idempotency
Authentication
Authorization
Versioning
Rate limiting
Error handling
Caching
Concurrency
```

---

# 1. What Is an API?

An API defines how one system communicates with another.

Example:

```text
Frontend
   ↓
REST API
   ↓
Spring Boot
   ↓
Database
```

The API defines:

```text
Endpoint
Method
Request
Response
Errors
Authentication
```

---

# 2. REST

REST is an architectural style for distributed systems.

Common REST principles include:

```text
Resource-oriented design
Stateless communication
Uniform interface
Cacheability
Layered architecture
```

REST is not simply:

```text
"Use JSON over HTTP."
```

---

# 3. Resource-Oriented API

Think in terms of resources.

Good:

```text
GET /users/42
GET /orders/101
GET /products/500
```

Less RESTful:

```text
GET /getUser
POST /createOrder
POST /deleteProduct
```

The HTTP method already communicates the action.

---

# 4. HTTP Methods

Common methods:

```text
GET
POST
PUT
PATCH
DELETE
```

---

# 5. GET

Used to retrieve a resource.

Example:

```http
GET /products/101
```

Response:

```json
{
  "id": 101,
  "name": "Laptop",
  "price": 65000
}
```

GET should not normally change server state.

---

# 6. POST

Usually used to create a new resource or trigger an operation that doesn't fit another method.

Example:

```http
POST /orders
```

Request:

```json
{
  "productId": 101,
  "quantity": 1
}
```

---

# 7. PUT

Typically represents replacing a resource with the supplied representation.

Example:

```http
PUT /users/42
```

Request:

```json
{
  "name": "Sudhir",
  "email": "sudhir@example.com"
}
```

PUT is generally designed to be idempotent.

---

# 8. PATCH

Used for partial modification.

Example:

```http
PATCH /users/42
```

Request:

```json
{
  "name": "Sudhir Kumar"
}
```

PATCH semantics depend on the API design and patch format.

Do not automatically assume every PATCH operation is idempotent.

---

# 9. DELETE

Deletes a resource.

Example:

```http
DELETE /users/42
```

A successful deletion might return:

```text
204 No Content
```

The exact behavior depends on the API contract.

---

# 10. Idempotency

An operation is idempotent if repeating the same request has the same intended effect as making it once.

Examples:

```text
GET
PUT
DELETE
```

are generally defined as idempotent by HTTP semantics.

POST is generally not.

---

# 11. Why Idempotency Matters

Imagine:

```text
POST /payments
```

Client sends request.

Network times out.

Client doesn't know whether payment succeeded.

If it retries:

```text
POST /payments
```

you could charge twice.

Use:

```text
Idempotency-Key: abc123
```

The server can recognize a retry.

---

# 12. Idempotency Key Flow

```text
Client
 ↓
POST /payments
Idempotency-Key: abc123
 ↓
Payment Service
 ↓
Check key
 ↓
Already processed?
 ├── Yes → return previous result
 └── No  → process + store result
```

The key should be scoped appropriately and stored with enough information to safely identify the operation.

---

# 13. HTTP Status Codes

Know the common ones:

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

# 14. 200 OK

Successful request.

Example:

```http
GET /products/101
```

Response:

```text
200 OK
```

with the requested representation.

---

# 15. 201 Created

Use when a new resource has been created.

Example:

```http
POST /users
```

Response:

```text
201 Created
```

Often include:

```http
Location: /users/42
```

---

# 16. 202 Accepted

Useful when work has been accepted for asynchronous processing.

Example:

```http
POST /reports
```

Response:

```text
202 Accepted
```

Body:

```json
{
  "jobId": "job-123"
}
```

The client can check:

```http
GET /jobs/job-123
```

---

# 17. 204 No Content

Successful request with no response body.

Common example:

```http
DELETE /users/42
```

---

# 18. 400 Bad Request

The request is invalid.

Examples:

```text
Malformed JSON
Invalid query parameter
Invalid request structure
```

---

# 19. 401 Unauthorized

Means the request lacks valid authentication credentials.

Examples:

```text
Missing token
Expired token
Invalid token
```

A useful memory trick:

```text
401 → Authentication problem
```

---

# 20. 403 Forbidden

The server understood who the caller is but refuses access.

Example:

```text
User is authenticated
but not allowed to access admin endpoint
```

Memory trick:

```text
403 → Authorization problem
```

---

# 21. 404 Not Found

The requested resource cannot be found.

Example:

```http
GET /users/999999
```

when that user doesn't exist.

---

# 22. 409 Conflict

Useful when the request conflicts with the current state.

Examples:

```text
Duplicate username
Version conflict
Resource state conflict
```

---

# 23. 422 Unprocessable Content

Can be used when the request is syntactically valid but semantically invalid.

Example:

```json
{
  "startDate": "2026-08-20",
  "endDate": "2026-08-10"
}
```

The JSON is valid, but the business data is invalid.

Use 400 vs 422 consistently according to your API conventions.

---

# 24. 429 Too Many Requests

Client exceeded the configured rate limit.

Response may include:

```http
Retry-After: 30
```

---

# 25. 500 Internal Server Error

Unexpected server-side failure.

Don't expose:

```text
Stack traces
Database credentials
Internal implementation details
```

to clients.

---

# 26. 502 Bad Gateway

A gateway/proxy received an invalid response from an upstream service.

Example:

```text
API Gateway
   ↓
Service
   X
Invalid upstream response
```

---

# 27. 503 Service Unavailable

Service is temporarily unable to handle the request.

Possible reasons:

```text
Overload
Maintenance
Dependency/resource availability
```

---

# 28. 504 Gateway Timeout

A gateway/proxy timed out waiting for an upstream service.

Example:

```text
Gateway
 ↓
Order Service
 ↓
Payment Service
 ↓
Timeout
```

---

# 29. URL Structure

A clean API:

```text
/users
/users/42

/products
/products/101

/orders
/orders/5001
```

Avoid unnecessarily deep paths.

---

# 30. Nested Resources

Example:

```text
GET /users/42/orders
```

This can be useful when:

```text
Orders are naturally scoped to a user.
```

But avoid excessive nesting:

```text
/users/42/orders/101/items/5/reviews/9
```

It becomes hard to use and evolve.

---

# 31. Query Parameters

Use query parameters for filtering, sorting and pagination.

Example:

```http
GET /products?category=electronics&sort=price&page=2
```

---

# 32. Filtering

Example:

```http
GET /orders?status=PAID
```

Multiple filters:

```http
GET /orders?status=PAID&userId=42
```

Define behavior clearly.

---

# 33. Sorting

Example:

```http
GET /products?sort=price
```

Descending:

```http
GET /products?sort=-price
```

Or:

```http
GET /products?sort=price&direction=desc
```

Choose one consistent convention.

---

# 34. Pagination

Never return millions of records in one response.

Instead:

```http
GET /orders?page=1&size=20
```

For large datasets, cursor/keyset pagination is often preferable.

---

# 35. Offset Pagination

Example:

```http
GET /orders?offset=100&limit=20
```

Simple but deep offsets can become expensive.

---

# 36. Cursor Pagination

Example:

```http
GET /orders?limit=20&cursor=eyJpZCI6MTAw...
```

Response:

```json
{
  "data": [],
  "nextCursor": "eyJpZCI6MTIw..."
}
```

Useful for:

```text
Large datasets
Infinite scrolling
Stable traversal
```

---

# 37. Pagination Response

A useful response:

```json
{
  "data": [
    {
      "id": 101,
      "name": "Laptop"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 1500
}
```

For cursor pagination:

```json
{
  "data": [],
  "nextCursor": "abc123",
  "hasMore": true
}
```

Don't return expensive total counts unless they are actually needed.

---

# 38. API Response Shape

Keep responses consistent.

Example:

```json
{
  "id": 101,
  "name": "Laptop",
  "price": 65000
}
```

Avoid one endpoint returning:

```json
{
  "data": ...
}
```

and another returning:

```json
{
  "result": ...
}
```

without a reason.

---

# 39. Error Response

A useful error structure:

```json
{
  "code": "PRODUCT_NOT_FOUND",
  "message": "Product was not found",
  "traceId": "abc123"
}
```

For validation:

```json
{
  "code": "VALIDATION_ERROR",
  "message": "Request contains invalid fields",
  "fields": [
    {
      "field": "email",
      "message": "Invalid email"
    }
  ],
  "traceId": "abc123"
}
```

---

# 40. Don't Leak Internal Errors

Bad:

```json
{
  "error": "NullPointerException at OrderService.java:142"
}
```

Good:

```json
{
  "code": "INTERNAL_ERROR",
  "message": "Something went wrong",
  "traceId": "abc123"
}
```

Log technical details internally.

---

# 41. Validation

Validate at the API boundary.

Examples:

```text
Required fields
String length
Email format
Numeric ranges
Enum values
```

Spring Boot example:

```java
public class CreateUserRequest {

    @NotBlank
    private String name;

    @Email
    @NotBlank
    private String email;
}
```

---

# 42. Business Validation

Not all validation is structural.

Example:

```text
quantity > 0
```

is simple validation.

But:

```text
Cannot cancel an order that has already shipped
```

is business validation.

Keep business rules in the appropriate domain/service layer.

---

# 43. Authentication vs Authorization

### Authentication

```text
Who are you?
```

### Authorization

```text
What are you allowed to do?
```

Example:

```text
JWT proves identity.

Role/permission determines access.
```

---

# 44. Resource-Level Authorization

Don't rely only on:

```text
ROLE_USER
```

Example:

```http
GET /orders/5001
```

The service should verify:

```text
Does this user own order 5001?
```

or:

```text
Does this user have permission to access it?
```

---

# 45. API Security

Important practices:

```text
HTTPS
Authentication
Authorization
Input validation
Rate limiting
Secure headers
Secret management
Audit logging
```

Never send sensitive data unnecessarily.

---

# 46. HTTPS

Always use encrypted transport for sensitive APIs.

```text
HTTP
 ↓ TLS
HTTPS
```

Protects data in transit from many network-level attacks.

---

# 47. CORS

CORS controls which browser origins are allowed to make cross-origin requests.

Example:

```text
Frontend:
https://app.example.com

API:
https://api.example.com
```

Configure allowed origins carefully.

Avoid:

```text
Allow *
```

for sensitive authenticated APIs unless the exact security model permits it.

---

# 48. CSRF

CSRF is primarily a concern for browser-based authentication mechanisms such as cookie-based sessions.

If using:

```text
Cookie authentication
```

CSRF protection may be necessary.

With certain stateless bearer-token architectures, the threat model differs.

Don't claim:

```text
JWT automatically eliminates every CSRF concern.
```

---

# 49. API Rate Limiting

Example:

```text
100 requests/minute/user
```

Protects:

```text
API
Database
Downstream services
```

Can be implemented at:

```text
Gateway
Redis
Application
Infrastructure
```

---

# 50. API Caching

GET responses can sometimes be cached.

Headers may include:

```http
Cache-Control
ETag
Last-Modified
```

Caching strategy depends on:

```text
Freshness
Sensitivity
Consistency
```

---

# 51. ETag

An ETag identifies a representation version.

Example:

```http
ETag: "abc123"
```

Client later sends:

```http
If-None-Match: "abc123"
```

If unchanged:

```text
304 Not Modified
```

This saves bandwidth.

---

# 52. Conditional Requests

Useful for:

```text
Large resources
Frequently requested resources
Low-change data
```

The server can avoid returning the full representation if it hasn't changed.

---

# 53. API Versioning

Common approach:

```text
/api/v1/products
/api/v2/products
```

Another option:

```text
Header/media type versioning
```

There is no single universal strategy.

---

# 54. When to Version

Version when making breaking changes such as:

```text
Removing fields
Changing field meaning
Changing response structure incompatibly
Changing behavior clients depend on
```

Don't create a new version for every small additive field.

---

# 55. API Evolution

Prefer:

```text
Add new optional field
```

over:

```text
Remove existing field immediately
```

For a breaking change:

```text
v1
 ↓
Migration period
 ↓
v2
 ↓
Deprecate v1
 ↓
Remove v1
```

---

# 56. HATEOAS

HATEOAS allows responses to include links describing possible next actions.

Example:

```json
{
  "id": 101,
  "status": "PAID",
  "_links": {
    "self": "/orders/101",
    "cancel": "/orders/101/cancel"
  }
}
```

It's part of the original REST constraints, but many practical APIs don't implement full HATEOAS.

Don't over-focus on it in typical backend interviews.

---

# 57. API Gateway Responsibilities

A gateway can handle:

```text
Routing
Authentication
Rate limiting
TLS termination
Request IDs
Observability
```

Avoid putting:

```text
Order business rules
Payment business rules
Inventory rules
```

inside it.

---

# 58. BFF

BFF:

```text
Backend for Frontend
```

Different clients can have different backend APIs.

Example:

```text
Mobile BFF
Web BFF
```

Each can optimize responses for its client.

---

# 59. BFF Example

Instead of mobile calling:

```text
User Service
Product Service
Recommendation Service
Order Service
```

individually:

```text
Mobile
 ↓
Mobile BFF
 ├── User
 ├── Product
 ├── Recommendation
 └── Order
```

The BFF aggregates data.

---

# 60. API Aggregation

A gateway/BFF can aggregate:

```text
GET /home
```

from:

```text
User
Product
Recommendation
Promotion
```

This reduces client round trips.

But the aggregator can become a dependency bottleneck.

---

# 61. API Timeout Design

If:

```text
Gateway timeout = 5 sec
```

don't let downstream calls independently wait:

```text
10 sec
```

Timeout hierarchy should leave enough time for outer layers to fail cleanly.

---

# 62. Request IDs

Generate or propagate:

```text
X-Request-ID
```

Example:

```text
abc-123
```

Use it in:

```text
Logs
Traces
Error responses
```

This makes debugging much easier.

---

# 63. Correlation ID vs Trace ID

A correlation ID is an application-level identifier for associating related operations.

A trace ID belongs to distributed tracing.

They can be used together.

Modern tracing systems often propagate:

```text
traceparent
```

or equivalent context.

---

# 64. API Observability

For every endpoint monitor:

```text
Request rate
Error rate
Latency
Status codes
Dependency latency
```

Important percentiles:

```text
p50
p95
p99
```

---

# 65. Why p95/p99?

Average latency can hide bad experiences.

Example:

```text
Average = 100 ms
p99 = 3 seconds
```

Most requests are fast, but a significant tail is slow.

For user-facing APIs, tail latency matters.

---

# 66. API SLO

Example:

```text
99.9% successful requests
p95 latency < 300 ms
```

SLOs turn vague requirements into measurable targets.

---

# 67. Request Timeout

Client:

```text
5 second timeout
```

Server:

```text
10 second processing
```

The server may finish work after the client has already abandoned the request.

For expensive operations, asynchronous processing may be better.

---

# 68. Long-Running APIs

Bad:

```text
POST /generate-report
```

waits:

```text
10 minutes
```

Better:

```text
POST /reports
→ 202 Accepted
→ jobId
```

Then:

```text
GET /reports/{jobId}
```

---

# 69. API Contract

Document:

```text
Endpoint
Method
Request
Response
Errors
Authentication
Rate limits
Pagination
```

Common tools:

```text
OpenAPI
Swagger
```

---

# 70. OpenAPI

OpenAPI can describe:

```text
Endpoints
Parameters
Schemas
Responses
Authentication
```

This can generate:

```text
Documentation
Client SDKs
Validation
Testing support
```

---

# 71. API Testing

Test:

```text
Happy path
Validation errors
Authentication failures
Authorization failures
Not found
Conflict
Rate limiting
Timeouts
Dependency failures
Duplicate requests
```

---

# 72. Contract Testing

Consumer-driven contract testing verifies that:

```text
Consumer expectations
```

remain compatible with:

```text
Provider API
```

This is particularly useful in microservices.

---

# 73. Backward Compatibility Testing

When changing an API:

```text
Old client
+
New server
```

should continue working if the change is intended to be backward compatible.

---

# 74. API Pagination Pitfall

Suppose:

```text
Page 1
```

returns:

```text
Orders 1–20
```

while new orders are continuously inserted.

Offset pagination can cause:

```text
Duplicates
Missing records
```

depending on ordering and concurrent changes.

Keyset/cursor pagination can provide more stable traversal.

---

# 75. Stable Ordering

Pagination should use deterministic ordering.

Example:

```sql
ORDER BY created_at DESC, id DESC
```

Using a unique tie-breaker such as:

```text
id
```

helps make ordering deterministic when timestamps are equal.

---

# 76. API Concurrency

Two users update the same resource:

```text
User A → version 5
User B → version 5
```

Both edit it.

Use:

```text
Optimistic locking
```

to prevent silent overwrites.

---

# 77. ETag + Optimistic Concurrency

Example:

```http
GET /products/101
ETag: "v5"
```

Client sends:

```http
PUT /products/101
If-Match: "v5"
```

If resource is now:

```text
v6
```

server can return:

```text
412 Precondition Failed
```

This prevents overwriting newer changes.

---

# 78. 412 Precondition Failed

Useful when a conditional request fails.

Example:

```http
If-Match: "v5"
```

but current resource:

```text
v6
```

Response:

```text
412 Precondition Failed
```

This differs from:

```text
409 Conflict
```

which is often used for broader application-level state conflicts.

---

# 79. Bulk APIs

Sometimes clients need:

```text
Create 100 records
```

Instead of:

```text
100 HTTP requests
```

a bulk endpoint may be appropriate.

Example:

```http
POST /users/bulk
```

But define:

```text
Partial failure behavior
Idempotency
Maximum batch size
Error reporting
```

---

# 80. Partial Failure

Suppose bulk request has:

```text
100 items
```

and:

```text
3 fail
97 succeed
```

API must clearly communicate the result.

Possible response:

```json
{
  "succeeded": 97,
  "failed": 3,
  "errors": [...]
}
```

---

# 81. API Security — Mass Assignment

Don't blindly bind arbitrary request fields directly to domain entities.

Bad:

```text
Request
 ↓
Entity
```

A malicious client may attempt:

```json
{
  "role": "ADMIN"
}
```

Use:

```text
DTO
 ↓
Validation
 ↓
Explicit mapping
 ↓
Domain/entity
```

---

# 82. API Security — Sensitive Data

Avoid returning:

```text
Password hashes
Internal IDs when unnecessary
Payment secrets
Internal service details
```

Return only fields the client actually needs.

---

# 83. API Security — Logging

Don't log:

```text
Passwords
JWT secrets
Credit card data
Sensitive personal information
```

Use:

```text
Redaction
Masking
Structured logging
```

---

# 84. API Security — Authorization

A common bug:

```text
GET /orders/123
```

checks:

```text
User is logged in
```

but doesn't check:

```text
User owns order 123
```

This can become an IDOR/BOLA vulnerability.

Always enforce resource-level authorization.

---

# 85. API Design for E-commerce

Resources:

```text
/users
/products
/carts
/orders
/payments
```

Examples:

```http
GET /products/101
POST /orders
GET /orders/5001
POST /orders/5001/cancel
```

Use sub-actions carefully.

---

# 86. Order Cancellation

Possible API:

```http
POST /orders/5001/cancellation
```

or:

```http
POST /orders/5001/cancel
```

The choice depends on the domain model.

The important part is:

```text
Cancel only if business state allows it.
```

---

# 87. API for Async Payment

Example:

```http
POST /payments
```

Response:

```text
202 Accepted
```

```json
{
  "paymentId": "pay-101",
  "status": "PROCESSING"
}
```

Later:

```http
GET /payments/pay-101
```

---

# 88. API Rate Limit Headers

A service may expose information such as:

```http
Retry-After: 30
```

For custom rate-limit headers, define a consistent contract.

Don't rely on undocumented behavior.

---

# 89. API Caching and Authorization

Be careful caching personalized responses.

Example:

```text
GET /profile
```

Response differs by user.

A shared cache must not accidentally return:

```text
User A's profile
```

to:

```text
User B
```

Use appropriate cache keys and cache-control rules.

---

# 90. API Gateway and Authentication

A gateway can perform initial authentication, but authorization should remain enforced by the service that owns the resource.

Think:

```text
Gateway
→ Is the token valid?

Service
→ Is this user allowed to access this resource?
```

---

# 91. API Documentation

A good API specification should answer:

```text
What endpoint?
What method?
What request?
What response?
What errors?
What authentication?
What limits?
```

This reduces integration problems.

---

# 92. API Design Interview Framework

When asked to design an API:

```text
1. Identify resources
2. Define endpoints
3. Choose HTTP methods
4. Define request/response models
5. Define status codes
6. Define validation
7. Define authentication/authorization
8. Define pagination/filtering
9. Define idempotency
10. Define errors
11. Define versioning
12. Define rate limits
13. Define observability
```

---

# 93. Interview Question

### PUT vs PATCH?

Answer:

> "PUT generally represents replacing the resource representation and is idempotent. PATCH represents a partial modification and its idempotency depends on the operation and patch semantics."

---

# 94. Interview Question

### 401 vs 403?

Answer:

> "401 means the request doesn't have valid authentication credentials. 403 means the server knows the caller but refuses access."

---

# 95. Interview Question

### When would you return 202?

Answer:

> "When the server has accepted the request but the actual work will happen asynchronously. I'd normally return a job or resource identifier so the client can check status."

---

# 96. Interview Question

### How do you prevent duplicate payments?

Answer:

> "I'd use an idempotency key tied to the payment operation and persist the result so retries return the original outcome instead of creating another payment."

---

# 97. Interview Question

### How would you paginate millions of records?

Answer:

> "I'd avoid loading everything at once and would generally prefer keyset or cursor pagination for large datasets, backed by an appropriate index and deterministic ordering."

---

# 98. Interview Question

### How do you handle API versioning?

Answer:

> "I'd prefer backward-compatible additive changes where possible. For breaking changes I'd introduce a versioning strategy, support a migration period and deprecate the old version gradually."

---

# 99. Interview Question

### How do you design error responses?

Answer:

> "I'd use a consistent error structure containing a machine-readable code, safe human-readable message and trace ID. Detailed stack traces stay in internal logs rather than being exposed to clients."

---

# 100. Interview Question

### How do you secure a REST API?

Answer:

> "I'd use HTTPS, authentication, resource-level authorization, input validation, rate limiting, secure secret management and careful handling of sensitive data. I'd also make sure authorization is enforced by the service that owns the resource."

---

# 101. Interview Question

### What is an API Gateway?

Answer:

> "It's a centralized entry point that can handle routing and cross-cutting concerns such as authentication, rate limiting, TLS termination and observability. Business logic should generally remain in the services."

---

# 102. Interview Question

### REST vs gRPC?

Answer:

> "REST is usually convenient for public APIs and browser-facing clients. gRPC is often useful for internal service-to-service calls when strong contracts, efficient serialization and streaming are valuable."

---

# 103. Interview Question

### How do you make an API highly available?

Answer:

> "I'd run multiple stateless instances behind a load balancer, use health checks, externalize state, protect dependencies with timeouts and circuit breakers, and make the database and other critical dependencies highly available."

---

# 104. Practical Scenario

### API suddenly receives 10× traffic.

First:

```text
Rate limiting
Load balancing
Autoscaling
Caching
```

Then investigate:

```text
Database
Connection pools
Downstream dependencies
```

Don't simply increase every timeout.

---

# 105. Practical Scenario

### API latency increases from 100 ms to 2 seconds.

Check:

```text
Application latency
Database latency
Dependency latency
Connection pool waits
CPU
GC
Queueing
Cache hit rate
```

Use tracing to identify where time is spent.

---

# 106. Practical Scenario

### Client retries POST after timeout.

Risk:

```text
Duplicate operation
```

Use:

```text
Idempotency key
```

for operations such as:

```text
Payment
Order creation
Resource provisioning
```

where duplicate execution is dangerous.

---

# 107. Practical Scenario

### Product API returns 10 million records.

Problem:

```text
Memory
Network
Serialization
Database load
Client performance
```

Solution:

```text
Pagination
Filtering
Sorting
```

with appropriate indexes.

---

# 108. Practical Scenario

### One API endpoint is expensive.

Possible improvements:

```text
Caching
Async processing
Database optimization
Read model
Pagination
Precomputation
```

Don't automatically add more application servers if the bottleneck is a database query.

---

# 109. Practical Scenario

### User requests a report that takes 2 minutes.

Don't keep HTTP request open for 2 minutes.

Prefer:

```text
POST /reports
 ↓
202 Accepted
 ↓
jobId
 ↓
Background worker
 ↓
GET /reports/{jobId}
```

---

# 110. Practical Scenario

### API has multiple downstream dependencies.

Use:

```text
Timeouts
Circuit breakers
Bulkheads
Tracing
```

and make non-critical dependencies optional where possible.

---

# 111. Final Checklist

You should be able to explain:

```text
□ REST
□ Resource-oriented design
□ GET/POST/PUT/PATCH/DELETE
□ Idempotency
□ Idempotency keys
□ HTTP status codes
□ 200/201/202/204
□ 400/401/403/404/409/422/429
□ 500/502/503/504
□ Query parameters
□ Filtering
□ Sorting
□ Offset pagination
□ Cursor pagination
□ Stable ordering
□ Error response design
□ Validation
□ Authentication
□ Authorization
□ Resource-level authorization
□ HTTPS
□ CORS
□ CSRF
□ Rate limiting
□ ETag
□ Conditional requests
□ API versioning
□ Backward compatibility
□ API Gateway
□ BFF
□ API aggregation
□ Request IDs
□ Correlation IDs
□ Distributed tracing
□ SLOs
□ OpenAPI
□ Contract testing
□ Optimistic concurrency
□ Bulk APIs
□ Mass assignment
□ Sensitive-data handling
□ API caching
```

---

# 112. One-Minute Interview Answer

### "How would you design a production-ready REST API?"

> "I'd start with resource-oriented endpoints and use HTTP methods and status codes consistently. I'd define clear request and response DTOs with validation and a standard error structure. For large datasets I'd use pagination and filtering with appropriate indexes. I'd secure the API with HTTPS, authentication, resource-level authorization and rate limiting. For operations that can be retried, especially payments, I'd support idempotency. I'd also define backward-compatible API evolution, timeouts for downstream calls, observability with logs, metrics and traces, and asynchronous processing for long-running operations."

---

# 113. Key Takeaway

> **A good API is a stable contract, not just a collection of endpoints. Design around resources, use HTTP semantics correctly, protect against retries and abuse, make responses predictable, and plan for evolution from the beginning.**

**File 11 complete.**

# REST API — Versioning, Compatibility & Evolution

This file covers how REST APIs evolve safely over time, including versioning strategies, backward compatibility, breaking changes, deprecation, API contracts, pagination evolution, database changes, migration strategies, and interview questions.

---

# 1. Why API Evolution Matters

A REST API is often consumed by:

```text
Web frontend
Mobile applications
Other backend services
Third-party clients
Internal teams
External partners
```

Changing an API without considering existing consumers can break production systems.

The goal is:

```text
New requirements
      ↓
API evolves
      ↓
Existing clients continue working
```

---

# 2. What Is API Versioning?

API versioning allows different versions of an API contract to coexist.

Example:

```text
/api/v1/products
/api/v2/products
```

Versioning is useful when a change cannot be introduced without breaking existing clients.

---

# 3. Breaking Change

A breaking change is a change that can cause an existing client to stop working correctly.

Examples:

```text
Removing a field
Renaming a field
Changing a field type
Changing authentication requirements
Removing an endpoint
Changing required request fields
Changing response semantics
Changing status-code behavior unexpectedly
```

---

# 4. Non-Breaking Change

Examples:

```text
Adding an optional request field
Adding a new endpoint
Adding a response field when clients tolerate unknown fields
Improving internal implementation
Adding optional query parameters
```

However, whether a change is truly backward compatible depends on the client implementation and contract.

---

# 5. Versioning Strategies

Common approaches:

```text
URI/path versioning
Query parameter versioning
Header versioning
Media-type/content-negotiation versioning
```

Each has trade-offs.

---

# 6. URI Versioning

Example:

```http
GET /api/v1/products
GET /api/v2/products
```

Advantages:

```text
Easy to understand
Easy to test
Visible in URLs
Easy to route
```

Disadvantage:

```text
Version becomes part of the resource URL
```

It is one of the most commonly encountered strategies.

---

# 7. Query Parameter Versioning

Example:

```http
GET /api/products?version=1
```

or:

```http
GET /api/products?apiVersion=2
```

Advantages:

```text
Same base URL
Simple to implement
```

Disadvantages:

```text
Version is less visible
Can complicate caching and routing
```

---

# 8. Header Versioning

Example:

```http
X-API-Version: 2
```

Advantages:

```text
URL remains stable
Version is represented as request metadata
```

Disadvantages:

```text
Less obvious when manually testing
Caching and documentation need careful handling
```

---

# 9. Media-Type Versioning

Example:

```http
Accept: application/vnd.company.product-v2+json
```

The requested representation determines the API version.

This can be powerful but is more complex for consumers and tooling.

---

# 10. Which Versioning Strategy Should You Use?

There is no universal answer.

For many business APIs:

```text
URI versioning
```

is attractive because it is:

```text
Simple
Visible
Easy to document
Easy to route
```

The most important thing is consistency within the API ecosystem.

---

# 11. Version Only When Necessary

Do not create:

```text
v1
v2
v3
v4
v5
```

for every small improvement.

If a change is backward compatible, you may not need a new API version.

Prefer evolving the existing contract safely where possible.

---

# 12. Example: Adding an Optional Field

Existing response:

```json
{
  "id": 100,
  "name": "Laptop"
}
```

New response:

```json
{
  "id": 100,
  "name": "Laptop",
  "description": "Gaming laptop"
}
```

If clients safely ignore unknown fields, this can be backward compatible.

---

# 13. Example: Removing a Field

Existing:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 50000
}
```

Removing:

```text
price
```

can break clients that depend on it.

This may require:

```text
Deprecation
Migration
New API version
```

---

# 14. Changing a Field Type

Existing:

```json
{
  "price": 50000
}
```

Changing to:

```json
{
  "price": "50000 INR"
}
```

can break clients expecting a number.

Changing types should generally be treated as a breaking change.

---

# 15. Renaming a Field

Changing:

```json
{
  "userName": "Sudhir"
}
```

to:

```json
{
  "username": "Sudhir"
}
```

can break consumers.

Safer migration:

```text
Expose old field
      +
Expose new field
      ↓
Migrate consumers
      ↓
Deprecate old field
      ↓
Remove later
```

---

# 16. Adding Required Request Fields

Existing:

```json
{
  "name": "Laptop"
}
```

New API requires:

```json
{
  "name": "Laptop",
  "category": "Electronics"
}
```

Old clients sending only `name` may fail.

Therefore adding a required request field can be a breaking change.

---

# 17. Optional Request Fields

Safer evolution:

```json
{
  "name": "Laptop"
}
```

New optional field:

```json
{
  "name": "Laptop",
  "description": "Gaming laptop"
}
```

Old clients can continue sending the original request.

---

# 18. Enum Evolution

Suppose clients receive:

```text
PENDING
PAID
CANCELLED
```

Adding:

```text
REFUNDED
```

can cause problems if a client assumes the enum is closed and crashes on unknown values.

Therefore even seemingly additive changes need consumer compatibility testing.

---

# 19. Status Code Changes

Suppose an endpoint historically returns:

```text
200 OK
```

and is changed to:

```text
204 No Content
```

Existing clients may depend on the response body or status code.

Status-code semantics are part of the API contract.

---

# 20. Error Response Compatibility

Existing:

```json
{
  "message": "Product not found"
}
```

New:

```json
{
  "errorCode": "PRODUCT_NOT_FOUND",
  "message": "Product not found"
}
```

Adding a field may be safe.

But changing:

```text
message → object
```

could break clients.

Error schemas should be versioned and evolved carefully.

---

# 21. Stable Error Codes

Prefer machine-readable codes:

```json
{
  "code": "PRODUCT_NOT_FOUND",
  "message": "Product was not found"
}
```

Clients can depend on:

```text
code
```

rather than parsing human-readable messages.

---

# 22. API Contract

An API contract describes:

```text
Endpoints
HTTP methods
Request schemas
Response schemas
Headers
Authentication
Status codes
Error formats
```

A good contract helps frontend and backend teams work independently.

---

# 23. OpenAPI

OpenAPI is commonly used to describe REST APIs.

It can document:

```text
Endpoints
Parameters
Request bodies
Responses
Authentication
Schemas
```

Tools can generate:

```text
Documentation
Client SDKs
Server stubs
Testing artifacts
```

---

# 24. Swagger UI

Swagger UI can render an OpenAPI specification as interactive documentation.

Typical workflow:

```text
Spring Boot
   ↓
OpenAPI specification
   ↓
Swagger UI
   ↓
Developers test endpoints
```

Keep production exposure of interactive API documentation appropriate to your security requirements.

---

# 25. Consumer-Driven Contracts

In a microservices environment:

```text
Consumer
    ↓
Defines expected behavior
    ↓
Provider verifies contract
```

This reduces the risk of silently breaking consumers.

Tools such as Pact are commonly used for consumer-driven contract testing.

---

# 26. Backward Compatibility

A backward-compatible API change allows existing clients to continue working.

Think:

```text
Old client
   ↓
New server
   ↓
Still works
```

This is often a key goal for public and distributed APIs.

---

# 27. Forward Compatibility

Forward compatibility considers whether newer clients can work with older systems or responses.

It is harder to guarantee and depends on the contract.

For example, tolerant readers can ignore fields they do not understand.

---

# 28. Tolerant Reader Pattern

A tolerant client:

```text
Reads fields it understands
Ignores unknown fields
```

Example:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 50000,
  "newField": "value"
}
```

An older client may ignore:

```text
newField
```

This helps API evolution.

---

# 29. Consumer Compatibility

Before changing an API ask:

```text
Who consumes this endpoint?
How many consumers exist?
Which versions are deployed?
Which fields do they depend on?
Can they be migrated?
```

Do not assume an endpoint has only one consumer.

---

# 30. Deprecation

Deprecation means:

```text
This API is still available,
but consumers should migrate away from it.
```

Typical lifecycle:

```text
Active
  ↓
Deprecated
  ↓
Migration period
  ↓
Sunset
  ↓
Removed
```

---

# 31. Deprecation Communication

Tell consumers:

```text
What is changing?
Why?
What is the replacement?
When will old behavior be removed?
How can consumers migrate?
```

For public APIs, publish this information clearly.

---

# 32. Sunset Strategy

Do not remove a widely used endpoint suddenly.

A safer approach:

```text
Announce
   ↓
Deprecate
   ↓
Measure usage
   ↓
Migrate consumers
   ↓
Set sunset date
   ↓
Remove
```

---

# 33. Deprecation Headers

HTTP provides mechanisms such as:

```text
Deprecation
Sunset
```

when supported by the API strategy and tooling.

The exact header format and dates should follow the organization's chosen standard and documentation.

---

# 34. API Usage Monitoring

Before removing an API, measure:

```text
Request volume
Consumers
Client versions
Error rates
Geographic usage
Traffic trends
```

An endpoint that appears unused may still have an important low-frequency consumer.

---

# 35. Database Evolution

API evolution often requires database evolution.

Example:

```text
API v2 introduces product.description
```

Database migration may be:

```sql
ALTER TABLE product
ADD COLUMN description VARCHAR(500);
```

The database and API should evolve in compatible steps.

---

# 36. Expand-and-Contract Pattern

A safe database migration often uses:

```text
Expand
   ↓
Deploy compatible application
   ↓
Migrate data
   ↓
Switch reads/writes
   ↓
Contract
```

---

# 37. Example: Rename Database Column

Unsafe:

```text
Rename column immediately
```

because old application versions may still use the original column.

Safer:

```text
Add new column
      ↓
Application writes both
      ↓
Backfill old data
      ↓
Application reads new column
      ↓
Remove old column later
```

This supports rolling deployments.

---

# 38. Zero-Downtime Deployment

A rolling deployment may temporarily have:

```text
Old application version
        +
New application version
```

Therefore the database contract must support both versions during the transition.

This is why expand-and-contract migrations are useful.

---

# 39. API Gateway Version Routing

A gateway can route:

```text
/api/v1/*
     ↓
v1 service

/api/v2/*
     ↓
v2 service
```

This can help teams migrate consumers gradually.

---

# 40. Strangler Pattern

The Strangler Pattern gradually replaces an old system.

Example:

```text
Old monolith
    ↓
Gateway
   ↙ ↘
Old  New service
```

New functionality moves to the new architecture over time.

Eventually:

```text
Old system
   ↓
Removed
```

---

# 41. API Versioning in Microservices

Not every microservice needs URL versioning.

Instead, teams can evolve contracts through:

```text
Backward-compatible schemas
Consumer-driven contracts
Event versioning
Explicit breaking versions
```

The strategy should match the architecture.

---

# 42. REST API vs Event Versioning

REST:

```text
/api/v1/orders
```

Events:

```text
OrderCreated v1
OrderCreated v2
```

Event consumers may process events long after they were produced, so schema compatibility is especially important.

---

# 43. Event Schema Evolution

For events:

```json
{
  "orderId": 100,
  "customerId": 50
}
```

Adding an optional field:

```json
{
  "orderId": 100,
  "customerId": 50,
  "couponCode": "SAVE10"
}
```

is often easier to evolve than removing or changing existing fields.

---

# 44. Pagination Evolution

Initial API:

```http
GET /products?page=0&size=20
```

As data grows, offset pagination may become expensive.

An API may evolve toward cursor pagination:

```http
GET /products?cursor=eyJpZCI6MTAwfQ==
```

This should be introduced without unexpectedly breaking existing consumers.

---

# 45. Offset vs Cursor

Offset:

```text
page=100
size=20
```

Pros:

```text
Simple
Easy to understand
```

Cons:

```text
Large offsets can become expensive
Results can shift when data changes
```

Cursor:

```text
cursor=abc123
```

Pros:

```text
Efficient for large datasets
Better for continuously changing data
```

Cons:

```text
More complex
Less convenient for arbitrary page jumping
```

---

# 46. API Filtering Evolution

Initial:

```http
GET /products?category=electronics
```

Later:

```http
GET /products?category=electronics&minPrice=1000
```

Adding optional filters is generally backward compatible.

But be careful that new filtering semantics do not unexpectedly change the meaning of existing requests.

---

# 47. API Sorting Evolution

Avoid exposing database implementation details.

Bad:

```http
sort=internal_column_17
```

Prefer stable API fields:

```http
sort=price
sort=createdAt
```

Map API fields to internal database fields.

---

# 48. API Field Mapping

External:

```text
createdAt
```

Internal database:

```text
created_timestamp
```

A mapping layer allows database changes without forcing API consumers to change.

---

# 49. DTO Versioning

You may use separate DTOs:

```text
ProductResponseV1
ProductResponseV2
```

This can make major contract differences explicit.

However, avoid duplicating huge amounts of logic just because the DTO version differs.

Share business logic where appropriate.

---

# 50. Controller Versioning

Conceptually:

```text
ProductControllerV1
ProductControllerV2
```

can expose different representations while sharing services.

Example:

```text
Controller V1
      ↓
Shared Service

Controller V2
      ↓
Shared Service
```

This keeps business logic from being unnecessarily duplicated.

---

# 51. Versioning Business Logic

Avoid:

```text
V1 Service
V2 Service
V3 Service
```

for every small response change.

Prefer separating:

```text
API representation
```

from:

```text
Business logic
```

unless the business behavior itself genuinely differs.

---

# 52. API Compatibility Testing

Before releasing a new version, test:

```text
Old requests
Old responses
New requests
New responses
Authentication
Error behavior
Pagination
Filtering
```

Contract tests are especially useful when many consumers exist.

---

# 53. Golden Tests

A golden/snapshot-style test stores an expected representation.

Example:

```json
{
  "id": 100,
  "name": "Laptop"
}
```

A change in output can then be detected automatically.

Use these carefully because snapshots can become noisy if they capture too much unstable data.

---

# 54. Consumer Testing

If frontend depends on:

```text
GET /products
```

the frontend should test its expectations against the API contract.

This reduces surprises such as:

```text
Backend changes field
Frontend crashes
```

---

# 55. Contract-First Development

Contract-first approach:

```text
Define OpenAPI
      ↓
Review contract
      ↓
Generate/stub code
      ↓
Implement
      ↓
Test contract
```

This can improve collaboration between frontend and backend teams.

---

# 56. Code-First Development

Code-first:

```text
Write Spring controllers
      ↓
Generate OpenAPI documentation
```

This can be convenient for teams that prefer implementation-first development.

Neither approach is universally better.

---

# 57. API Documentation

Every public endpoint should clearly document:

```text
Purpose
HTTP method
Path
Parameters
Request body
Authentication
Response
Errors
Examples
Version
Deprecation status
```

Good documentation reduces integration mistakes.

---

# 58. Semantic Versioning

Semantic versioning is commonly expressed as:

```text
MAJOR.MINOR.PATCH
```

Conceptually:

```text
MAJOR → breaking changes
MINOR → backward-compatible features
PATCH → backward-compatible fixes
```

API URL versioning does not have to exactly follow semantic versioning.

For example:

```text
/api/v1
/api/v2
```

usually represents major contract generations.

---

# 59. API Changelog

Maintain a changelog containing:

```text
New endpoints
Changed fields
Deprecated fields
Breaking changes
Bug fixes
Migration instructions
```

This becomes valuable for consumers and interview discussions.

---

# 60. Compatibility Matrix

For a large platform, maintain something like:

```text
             Server V1   Server V2
Client V1       ✓           ✓
Client V2       -           ✓
```

This makes supported combinations explicit.

---

# 61. Blue-Green Deployment

Blue-green deployment maintains two environments:

```text
Blue → current
Green → new
```

Traffic can switch from:

```text
Blue
 ↓
Green
```

after validation.

API compatibility is important because consumers may experience the switch without changing their client immediately.

---

# 62. Canary Deployment

Canary deployment gradually sends traffic to a new version.

Example:

```text
95% → V1
5%  → V2
```

Then:

```text
80% → V1
20% → V2
```

Eventually:

```text
0% → V1
100% → V2
```

Monitor:

```text
Errors
Latency
Business metrics
Security events
```

---

# 63. Feature Flags

Feature flags can separate deployment from release.

Example:

```text
New checkout flow
```

can be enabled for:

```text
Internal users
5%
25%
50%
100%
```

This reduces deployment risk.

---

# 64. Avoid Breaking Changes in Distributed Systems

A safe principle:

> Be tolerant when reading and conservative when producing.

For example:

```text
Consumers ignore unknown optional fields.
Producers avoid removing fields abruptly.
```

This is particularly useful in event-driven and microservice architectures.

---

# 65. API Evolution Workflow

A practical workflow:

```text
Identify change
      ↓
Classify breaking/non-breaking
      ↓
Identify consumers
      ↓
Design migration
      ↓
Update contract/docs
      ↓
Implement compatibility
      ↓
Add tests
      ↓
Deploy
      ↓
Monitor
      ↓
Deprecate old behavior
      ↓
Remove after migration
```

---

# 66. Example: Product API v1 → v2

V1:

```http
GET /api/v1/products/100
```

Response:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 50000
}
```

V2:

```http
GET /api/v2/products/100
```

Response:

```json
{
  "id": 100,
  "name": "Laptop",
  "pricing": {
    "amount": 50000,
    "currency": "INR"
  }
}
```

Changing:

```text
price
```

into:

```text
pricing.amount
pricing.currency
```

is a substantial contract change and can justify a new version.

---

# 67. Migration from V1 to V2

Possible approach:

```text
V1 remains active
       ↓
V2 released
       ↓
Consumers migrate
       ↓
Monitor V1 traffic
       ↓
Deprecate V1
       ↓
Set sunset date
       ↓
Remove V1
```

---

# 68. What Not to Do

Avoid:

```text
Breaking changes without notice
Removing fields suddenly
Changing status codes unexpectedly
Changing data types silently
Leaving deprecated APIs forever
Ignoring unknown consumers
Relying only on documentation
```

---

# 69. Versioning and Security

A new API version may introduce:

```text
New authentication
New authorization
Different scopes
Different token requirements
New security headers
```

Do not assume that because V1 is secure, V2 automatically is.

Security requirements must be tested for every supported version.

---

# 70. Versioning and Observability

Track metrics by version:

```text
API requests by version
Error rate by version
Latency by version
Consumer/client
Status codes
```

Example:

```text
GET /products
V1 → 20,000 requests/day
V2 → 80,000 requests/day
```

This helps determine when V1 can safely be retired.

---

# 71. Versioning and Caching

If version affects the response:

```text
/api/v1/products
/api/v2/products
```

separate URLs naturally help cache separation.

With header-based versioning, cache configuration may need to vary based on the relevant request header.

---

# 72. Versioning and API Gateway

A gateway can handle:

```text
Routing
Authentication
Rate limiting
Version selection
Deprecation policies
Observability
```

Example:

```text
/api/v1 → old backend
/api/v2 → new backend
```

This can simplify consumer migration.

---

# 73. API Compatibility Checklist

```text
□ Is the change breaking?
□ Which clients consume this API?
□ Can the change be made backward compatible?
□ Is a new version necessary?
□ Are request schemas compatible?
□ Are response schemas compatible?
□ Are status codes unchanged?
□ Are error schemas compatible?
□ Are authentication rules compatible?
□ Are authorization rules compatible?
□ Are pagination/filtering semantics unchanged?
□ Are OpenAPI docs updated?
□ Are contract tests updated?
□ Is migration documented?
□ Is deprecation communicated?
□ Is usage monitored?
□ Is a sunset date defined?
```

---

# 74. Interview: Why Do We Version REST APIs?

> We version APIs when a contract change would break existing consumers. Versioning lets old clients continue using the previous contract while new clients migrate to the new behavior. I try to avoid unnecessary versions by making backward-compatible changes whenever possible.

---

# 75. Interview: What Is a Breaking Change?

> A breaking change is a change that can cause an existing client to fail or behave incorrectly, such as removing a field, changing a field type, making a request field mandatory, removing an endpoint or changing authentication requirements.

---

# 76. Interview: Which API Versioning Strategy Do You Prefer?

> For many business REST APIs, I prefer URI versioning such as `/api/v1` and `/api/v2` because it's simple, visible and easy to document and route. The important thing is to choose a consistent strategy based on the organization's requirements.

---

# 77. Interview: How Do You Evolve an API Without Breaking Clients?

> I first identify consumers and classify the change. If possible, I add optional fields or endpoints instead of removing or changing existing behavior. For a genuinely breaking change, I introduce a new version, support both versions during migration, monitor usage and deprecate the old version gradually.

---

# 78. Interview: How Do You Deprecate an API?

> I announce the replacement and migration timeline, mark the old endpoint as deprecated, monitor its usage and give consumers enough time to migrate. After usage drops and the agreed sunset date arrives, I remove the old API.

---

# 79. Interview: What Is Backward Compatibility?

> Backward compatibility means existing clients can continue working when the server is changed. For example, adding an optional response field can be backward compatible if clients tolerate unknown fields, while removing an existing field generally is not.

---

# 80. Interview: How Would You Migrate a Database Without Downtime?

> I would typically use an expand-and-contract approach. First I add the new schema in a backward-compatible way, deploy code that supports both old and new structures, migrate or backfill the data, switch reads and writes, and only then remove the old schema after all application versions are migrated.

---

# 81. Interview: Why Is API Contract Testing Important?

> It verifies that the provider and consumers agree on the API behavior. This is especially valuable in microservices because one service can change independently and accidentally break another service.

---

# 82. Interview: What Is the Strangler Pattern?

> It is a gradual migration strategy where a new system takes over functionality from an old system piece by piece. A gateway or routing layer directs some functionality to the new services while the old system continues handling the rest until it can eventually be removed.

---

# 83. Interview: How Do You Handle V1 and V2 in Spring Boot?

> I can expose separate controller or request/response representations for V1 and V2 while keeping shared business logic in the service layer where possible. The API layer handles differences in the external contract rather than duplicating the entire business implementation.

---

# 84. Interview: Can Adding a Field Be a Breaking Change?

> Usually adding an optional response field is considered backward compatible, but it depends on the consumer. Some clients incorrectly assume an exact schema or closed enum set, so compatibility should be verified rather than assumed.

---

# 85. Interview: Offset vs Cursor Pagination?

> Offset pagination is simpler and works well for many use cases, but large offsets can become inefficient and results can shift as data changes. Cursor pagination is usually better for large or frequently changing datasets because it provides more stable and efficient traversal.

---

# 86. Final Mental Model

```text
                 API EVOLUTION
                       |
          +------------+------------+
          |                         |
   Non-breaking                 Breaking
          |                         |
  Evolve existing API         New API version
          |                         |
          |                    V1 + V2
          |                         |
          +------------+------------+
                       |
                   Migration
                       |
                 Deprecation
                       |
                    Monitor
                       |
                    Sunset
                       |
                    Remove
```

For database changes:

```text
Expand
  ↓
Compatible deployment
  ↓
Migrate data
  ↓
Switch application
  ↓
Contract
```

For distributed systems:

```text
Contract
  ↓
Compatibility tests
  ↓
Deploy
  ↓
Monitor
  ↓
Migrate consumers
  ↓
Retire old version
```

> **Good API design is not only about creating endpoints. It is about creating contracts that can survive change. The best backend engineers think about compatibility, migration, observability and consumer impact before making breaking changes.**

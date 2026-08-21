# REST API — Fundamentals

This file covers the core REST API concepts expected from a Java/Spring Boot backend developer.

## 1. What Is REST?

REST stands for **Representational State Transfer**. It is an architectural style for designing distributed systems around resources and a uniform interface.

Typical ecommerce resources:

```text
/users
/products
/carts
/orders
```

Clients interact with these resources using HTTP methods.

## 2. REST Constraints

The major REST constraints are:

```text
1. Client-Server
2. Stateless
3. Cacheable
4. Uniform Interface
5. Layered System
6. Code-on-Demand (optional)
```

### Client-Server

Client and server have separate responsibilities.

```text
Client → HTTP → Server
```

The client handles presentation and user interaction while the server handles business logic, persistence, authentication, and authorization.

### Stateless

Each request should contain the information needed to process it. The server should not depend on hidden conversational state from a previous request.

Statelessness makes horizontal scaling easier:

```text
              ┌── Server A
Client → Load Balancer ├── Server B
              └── Server C
```

Stateless does **not** mean the application cannot use databases, caches, or persistent state.

### Cacheable

Responses can be cached when appropriate. HTTP supports headers such as:

```text
Cache-Control
ETag
Last-Modified
Expires
```

Caching must respect freshness and consistency requirements.

### Uniform Interface

A consistent interface lets clients understand how to interact with resources:

```text
GET    /products/100
POST   /products
PUT    /products/100
PATCH  /products/100
DELETE /products/100
```

### Layered System

A client does not need to know whether it communicates directly with the application:

```text
Client
  ↓
CDN
  ↓
Load Balancer
  ↓
API Gateway
  ↓
Spring Boot Service
  ↓
Database
```

### Code-on-Demand

Servers may optionally transfer executable code to clients. This is rarely used in modern Spring Boot APIs.

## 3. Resources and URIs

REST APIs generally model **resources**, not actions.

Prefer:

```text
GET /products/100
POST /orders
```

over:

```text
GET /getProductById/100
POST /createOrder
```

The HTTP method already communicates the operation.

A resource can be:

```text
User
Product
Order
Cart
Payment
Review
```

### Collection vs Individual Resource

```text
/products        → collection
/products/100    → individual resource
```

Another example:

```text
/orders
/orders/5001
```

### Nested Resources

```text
/users/100/orders
```

can represent orders belonging to user `100`.

Avoid excessively deep nesting. A flatter endpoint can sometimes be clearer:

```text
/order-items/10
```

## 4. HTTP Methods

The common REST methods are:

```text
GET     → Read
POST    → Create
PUT     → Replace
PATCH   → Partial update
DELETE  → Delete
```

Detailed method semantics are covered in the next REST file.

## 5. HTTP Status Codes

Status code groups:

```text
2xx → Success
3xx → Redirection
4xx → Client error
5xx → Server error
```

Common codes:

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
500 Internal Server Error
```

### 200 OK

Used when a request succeeds and a response body is returned.

### 201 Created

Commonly returned after successful resource creation:

```http
POST /products
```

A `Location` header may identify the new resource:

```http
201 Created
Location: /products/100
```

### 204 No Content

Used when an operation succeeds but there is intentionally no response body.

### 400 Bad Request

Used for invalid requests such as malformed JSON or invalid request syntax.

### 401 Unauthorized

Generally means the request lacks valid authentication credentials.

```text
401 → Authentication problem
```

### 403 Forbidden

The client is authenticated but does not have permission.

```text
403 → Authorization problem
```

### 404 Not Found

The requested resource does not exist, or the API intentionally chooses not to reveal its existence.

### 409 Conflict

Useful when the request conflicts with the current resource state, such as a duplicate unique value or a concurrency conflict.

### 500 Internal Server Error

Represents an unexpected server-side failure. Do not expose stack traces, credentials, or internal implementation details to clients.

## 6. JSON and HTTP Headers

JSON is commonly used as the representation format:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 75000
}
```

Spring Boot commonly uses Jackson for JSON serialization and deserialization.

### Content-Type

Describes the request body format:

```http
Content-Type: application/json
```

### Accept

Describes the response media types the client prefers:

```http
Accept: application/json
```

Mental model:

```text
Content-Type → what I am sending
Accept       → what I prefer to receive
```

Other useful headers include:

```text
Authorization
If-None-Match
If-Match
User-Agent
Trace/Correlation ID
```

## 7. Request Body

Request bodies are commonly used with:

```text
POST
PUT
PATCH
```

Example:

```json
{
  "name": "Laptop",
  "price": 75000
}
```

GET requests generally should not rely on a request body.

## 8. Path Variables

Path variables identify a specific resource:

```text
GET /products/100
```

Spring Boot:

```java
@GetMapping("/products/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return productService.getProduct(id);
}
```

## 9. Query Parameters

Query parameters are useful for filtering, searching, pagination, and optional parameters:

```text
GET /products?category=phones&page=0&size=20
```

Spring Boot:

```java
@GetMapping("/products")
public Page<ProductResponse> getProducts(
        @RequestParam String category,
        @RequestParam int page,
        @RequestParam int size) {

    return productService.getProducts(
        category, page, size
    );
}
```

Mental model:

```text
Path  → Which resource?
Query → How should I query/filter it?
```

## 10. DTOs

DTO means **Data Transfer Object**.

Example response DTO:

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {
}
```

DTOs separate the API contract from the persistence model.

### Why Not Return JPA Entities Directly?

Returning entities can cause:

```text
Lazy-loading problems
Circular JSON references
Unwanted fields exposed
Tight coupling to the database model
Difficult API evolution
Security issues
```

Prefer explicit request and response DTOs.

Example request DTO:

```java
public record CreateProductRequest(
    String name,
    BigDecimal price
) {
}
```

## 11. Spring Boot REST Controllers

`@RestController` combines:

```java
@Controller
@ResponseBody
```

Example:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
}
```

### @GetMapping

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return productService.getProduct(id);
}
```

### @PostMapping

```java
@PostMapping
public ResponseEntity<ProductResponse> createProduct(
        @RequestBody CreateProductRequest request) {

    ProductResponse response =
        productService.create(request);

    return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(response);
}
```

### @PutMapping

```java
@PutMapping("/{id}")
public ProductResponse updateProduct(
        @PathVariable Long id,
        @RequestBody UpdateProductRequest request) {

    return productService.update(id, request);
}
```

### @PatchMapping

```java
@PatchMapping("/{id}")
public ProductResponse updatePrice(
        @PathVariable Long id,
        @RequestBody UpdatePriceRequest request) {

    return productService.updatePrice(id, request);
}
```

### @DeleteMapping

```java
@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deleteProduct(
        @PathVariable Long id) {

    productService.delete(id);
}
```

## 12. ResponseEntity

`ResponseEntity` gives explicit control over:

```text
HTTP status
Headers
Response body
```

Example:

```java
return ResponseEntity
    .status(HttpStatus.CREATED)
    .body(response);
```

Use it when explicit response control is useful. It is not necessary to wrap every response in `ResponseEntity`.

## 13. Controller → Service → Repository

A clean Spring Boot application commonly follows:

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
HTTP mapping
Request binding
Validation trigger
Response semantics
```

Service:

```text
Business logic
Transactions
Orchestration
```

Repository:

```text
Persistence
Database access
```

Typical structure:

```text
src/main/java/com/example
├── controller
├── service
├── repository
├── entity
├── dto
└── exception
```

## 14. Complete Ecommerce Product API

A typical product API:

```text
GET    /api/products
GET    /api/products/{id}
POST   /api/products
PUT    /api/products/{id}
PATCH  /api/products/{id}
DELETE /api/products/{id}
```

Example:

```text
GET /api/products/100
```

Response:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 75000
}
```

## 15. Validation

Request DTOs can use Jakarta Bean Validation:

```java
public record CreateProductRequest(

    @NotBlank
    String name,

    @Positive
    BigDecimal price

) {
}
```

Controller:

```java
@PostMapping
public ProductResponse create(
        @Valid @RequestBody CreateProductRequest request) {

    return productService.create(request);
}
```

Validation and error handling are covered in more detail later.

## 16. Global Exception Handling

Use `@RestControllerAdvice` for centralized API error handling:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiError> handleNotFound(
            ResourceNotFoundException ex) {

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(new ApiError(ex.getMessage()));
    }
}
```

A predictable error response might look like:

```json
{
  "timestamp": "2026-08-21T10:30:00Z",
  "status": 404,
  "error": "Not Found",
  "message": "Product not found",
  "path": "/api/products/100"
}
```

For larger APIs, RFC 9457 Problem Details is another standardized error representation.

## 17. Authentication and Security

Common API authentication mechanisms include:

```text
JWT
OAuth 2.0
Session-based authentication
API keys
```

For a stateless Spring Boot API, JWT is common:

```http
Authorization: Bearer <token>
```

Production REST APIs should also consider:

```text
HTTPS
Authorization
Input validation
Rate limiting
Secure headers
Secret management
```

Never rely only on frontend validation.

## 18. Idempotency

An operation is idempotent when repeating the same request has the same intended effect as performing it once.

Common HTTP semantics:

```text
GET     → idempotent
PUT     → idempotent
DELETE  → idempotent
POST    → generally not idempotent
```

Idempotency is especially important for payments, orders, and retryable distributed operations.

PATCH idempotency depends on the specific patch operation.

## 19. REST and Database Transactions

REST statelessness and database transactions solve different problems.

Example:

```text
POST /orders
      ↓
Controller
      ↓
Service
      ↓
@Transactional
      ↓
Create order
      ↓
Create order items
      ↓
Commit
```

## 20. REST API Performance

Common techniques include:

```text
Pagination
Caching
Compression
Efficient SQL
Connection pooling
Avoiding N+1 queries
Async processing
CDN
Response optimization
```

Measure bottlenecks before optimizing.

### Pagination

Avoid returning thousands of records at once:

```text
GET /products?page=0&size=20
```

Example response:

```json
{
  "content": [],
  "page": 0,
  "size": 20,
  "totalElements": 1000,
  "totalPages": 50
}
```

Pagination is covered in detail in a later file.

## 21. API Contract and Compatibility

An API contract defines expectations between client and server:

```text
Endpoints
HTTP methods
Request format
Response format
Status codes
Validation
Authentication
Error structure
```

Adding a new optional response field is often backward compatible.

Removing or changing the meaning/type of an existing field can break clients.

API evolution should therefore be deliberate.

## 22. API Versioning

When breaking changes are necessary, an API may be versioned:

```text
/api/v1/products
/api/v2/products
```

Common strategies:

```text
URI versioning
Header versioning
Media-type versioning
```

Detailed versioning is covered in a later file.

## 23. API Documentation and Testing

Common tools:

```text
Postman
curl
Insomnia
Swagger UI
JUnit
MockMvc
WebTestClient
```

Example:

```bash
curl -X GET http://localhost:8080/api/products/100
```

A controller test can use MockMvc:

```java
mockMvc.perform(
    get("/api/products/100")
)
.andExpect(status().isOk());
```

OpenAPI can describe the API and Swagger UI can provide interactive documentation.

## 24. Correlation IDs

A correlation or trace ID helps follow a request across services:

```http
X-Correlation-Id: 8f42a1
```

Example:

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

The identifier can be propagated into logs and tracing systems.

## 25. REST vs SOAP

REST:

```text
Architectural style
Commonly HTTP + JSON
Resource-oriented
Flexible
```

SOAP:

```text
Protocol
XML-based messaging
Strict contract options
WS-* standards
```

Both remain relevant depending on the integration requirements.

## 26. REST vs GraphQL

REST:

```text
Multiple resource endpoints
Server defines response representation
HTTP semantics are central
```

GraphQL:

```text
Typically a single endpoint
Client specifies requested fields
Schema-driven query language
```

Neither is universally better.

## 27. REST vs gRPC

REST:

```text
Human-friendly HTTP APIs
Broad tooling compatibility
Commonly JSON
```

gRPC:

```text
RPC-oriented
HTTP/2
Protocol Buffers
Strongly typed contracts
Efficient service-to-service communication
```

REST is often convenient for public APIs; gRPC can be attractive for internal service-to-service communication.

## 28. Common REST Mistakes

### Verbs in URLs

Bad:

```text
GET /getProducts
POST /createProduct
```

Prefer:

```text
GET /products
POST /products
```

### Returning entities directly

Prefer DTOs.

### Always returning 200

Use meaningful HTTP status codes.

### Exposing stack traces

Return controlled errors and log details internally.

### No validation

Validate incoming data.

### Huge responses

Use pagination and appropriate projections.

### Business logic in controllers

Keep controllers thin and move business logic into services/domain components.

## 29. Ecommerce Order Example

A request:

```text
POST /api/orders
```

might flow through:

```text
Client
  ↓
Load Balancer
  ↓
Spring Security
  ↓
OrderController
  ↓
OrderService
  ↓
Inventory check
  ↓
OrderRepository
  ↓
MySQL
  ↓
Response
```

In a microservice architecture, inventory and payment may be separate services.

## 30. REST API Design Checklist

```text
□ Resource names
□ HTTP method
□ URI structure
□ Request DTO
□ Response DTO
□ Validation
□ Status codes
□ Error model
□ Authentication
□ Authorization
□ Idempotency
□ Pagination
□ Filtering
□ Sorting
□ Versioning
□ Caching
□ Rate limiting
□ Documentation
□ Logging
□ Monitoring
□ Backward compatibility
```

# Interview Questions

## What is REST?

> REST is an architectural style for designing distributed systems around resources and a uniform interface. REST APIs commonly use HTTP methods, status codes, headers, and representations such as JSON. Stateless communication between client and server is a key principle.

## What does stateless mean?

> Stateless means the server doesn't rely on hidden conversational state from previous requests to process the current request. Each request contains the information needed for processing, while persistent data can still exist in databases or shared systems.

## REST vs RESTful API?

> REST is the architectural style and its constraints. A RESTful API is an API designed to follow those principles appropriately. Using HTTP and JSON alone doesn't automatically make an API fully RESTful.

## What is a resource?

> A resource is a business entity or concept exposed through the API, such as a product or order. It is identified by a URI such as `/products/100`.

## PUT vs PATCH?

> PUT is generally used to replace the representation of a resource and is idempotent. PATCH is used for partial modifications, and its idempotency depends on the specific patch operation.

## POST vs PUT?

> POST is commonly used to create a resource under a collection and is generally not idempotent. PUT targets a specific resource URI and is generally idempotent.

## 401 vs 403?

> 401 generally means the request lacks valid authentication credentials. 403 means the client is authenticated but does not have permission to perform the requested operation.

## Why use DTOs?

> DTOs separate the API contract from the persistence model. They prevent unwanted entity fields from being exposed, avoid lazy-loading and serialization problems, and make API evolution easier.

## Why use a service layer?

> The service layer keeps business logic separate from HTTP and persistence concerns. Controllers handle HTTP concerns, repositories handle data access, and services coordinate business operations and transactions.

## How would you design an ecommerce product API?

> I would expose resource-oriented endpoints such as `GET /products`, `GET /products/{id}`, `POST /products`, `PUT /products/{id}`, and `DELETE /products/{id}`. I would use DTOs, validation, appropriate status codes, centralized exception handling, pagination for lists, authentication and authorization where required, and caching for frequently accessed product data.

## How do you handle API errors?

> I use centralized exception handling with `@RestControllerAdvice` and return a consistent error structure with an appropriate HTTP status. Detailed stack traces stay in server logs rather than being exposed to clients.

## How do you make REST APIs scalable?

> I keep the API stateless, use horizontal scaling behind a load balancer, optimize database queries, use pagination, caching and connection pooling where appropriate, and add rate limiting and observability. For long-running operations, I consider asynchronous processing instead of blocking the request.

## How do you secure a REST API?

> I use HTTPS, authentication such as JWT or OAuth 2.0, authorization at the backend, input validation, rate limiting, secure secret management, and centralized security configuration. I also avoid exposing sensitive implementation details in responses and logs.

# Final Mental Model

```text
                    REST API
                       |
            +----------+----------+
            |          |          |
         Resource     HTTP       JSON
            |        Semantics   /DTO
            |          |
         URI +     Methods +
       representation Status
                       |
                       v
                 Spring Boot
                       |
              +--------+--------+
              |                 |
           Service          Repository
              |                 |
              +--------+--------+
                       |
                    Database
```

A clean Spring Boot REST API follows:

```text
Resource-oriented URLs
        ↓
Correct HTTP semantics
        ↓
DTO-based contract
        ↓
Validation
        ↓
Service/business layer
        ↓
Repository/persistence
        ↓
Consistent errors
        ↓
Security + observability
```

# Final Rule

> **Design REST APIs around resources, use HTTP semantics correctly, keep controllers thin, separate API DTOs from persistence entities, return meaningful status codes, validate inputs, handle errors consistently, and design the API contract with scalability and backward compatibility in mind.**

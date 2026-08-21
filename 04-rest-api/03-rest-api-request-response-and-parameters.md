# REST API — Request, Response, Headers, Path Variables and Query Parameters

This file covers how HTTP requests and responses are structured and how Spring Boot maps request data into controller methods.

---

# 1. HTTP Request

A typical HTTP request contains:

```text
Method
URL
Headers
Query Parameters
Path Variables
Request Body
```

Example:

```http
POST /api/products?notify=true HTTP/1.1
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "Laptop",
  "price": 75000
}
```

---

# 2. HTTP Response

A typical response contains:

```text
Status Code
Headers
Response Body
```

Example:

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/products/100

{
  "id": 100,
  "name": "Laptop",
  "price": 75000
}
```

---

# 3. Path Variable

A path variable identifies a specific resource.

Example:

```text
GET /api/products/100
```

Spring Boot:

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return productService.getProduct(id);
}
```

Here:

```text
100 → id
```

---

# 4. Multiple Path Variables

Example:

```text
GET /api/users/10/orders/500
```

Spring Boot:

```java
@GetMapping("/users/{userId}/orders/{orderId}")
public OrderResponse getOrder(
        @PathVariable Long userId,
        @PathVariable Long orderId) {

    return orderService.getOrder(userId, orderId);
}
```

Path variables are best when the value identifies the resource hierarchy.

---

# 5. Query Parameters

Query parameters provide additional information used for:

```text
Filtering
Searching
Sorting
Pagination
Optional behavior
```

Example:

```text
GET /api/products?category=phones&page=0&size=20
```

Spring Boot:

```java
@GetMapping
public List<ProductResponse> getProducts(
        @RequestParam String category,
        @RequestParam int page,
        @RequestParam int size) {

    return productService.getProducts(
        category, page, size
    );
}
```

---

# 6. Optional Query Parameter

Use `required = false` when a parameter is optional.

```java
@GetMapping
public List<ProductResponse> getProducts(
        @RequestParam(required = false)
        String category) {

    return productService.getProducts(category);
}
```

Request with parameter:

```text
GET /api/products?category=phones
```

Request without it:

```text
GET /api/products
```

---

# 7. Default Query Parameter

You can define a default value.

```java
@GetMapping
public List<ProductResponse> getProducts(
        @RequestParam(
            defaultValue = "0"
        ) int page) {

    return productService.getProducts(page);
}
```

If the client doesn't provide `page`:

```text
page = 0
```

---

# 8. Path Variable vs Query Parameter

Use a path variable when the value identifies a resource:

```text
GET /products/100
```

Use query parameters for filtering or modifying how a collection is returned:

```text
GET /products?category=phones
```

Good mental model:

```text
Path → WHICH resource?

Query → HOW should I retrieve it?
```

---

# 9. Request Body

The request body carries data sent to the server.

Example:

```http
POST /api/products
Content-Type: application/json

{
  "name": "Laptop",
  "price": 75000
}
```

Spring Boot:

```java
@PostMapping
public ProductResponse createProduct(
        @RequestBody ProductRequest request) {

    return productService.create(request);
}
```

---

# 10. @RequestBody

`@RequestBody` tells Spring to deserialize the HTTP request body into a Java object.

Example:

```java
public class ProductRequest {

    private String name;
    private BigDecimal price;

    // getters/setters
}
```

JSON:

```json
{
  "name": "Laptop",
  "price": 75000
}
```

Spring converts it into:

```text
ProductRequest
```

---

# 11. JSON Serialization and Deserialization

Two important concepts:

### Serialization

Java object:

```text
ProductResponse
```

becomes:

```text
JSON
```

### Deserialization

JSON:

```text
Request body
```

becomes:

```text
Java object
```

Spring Boot commonly uses Jackson for JSON processing.

---

# 12. Content-Type

`Content-Type` tells the server what format the request body uses.

Example:

```http
Content-Type: application/json
```

Other examples:

```text
application/xml
multipart/form-data
application/x-www-form-urlencoded
```

For JSON REST APIs, the most common value is:

```text
application/json
```

---

# 13. Accept Header

`Accept` tells the server what response media types the client prefers.

Example:

```http
Accept: application/json
```

Conceptually:

```text
Content-Type
    ↓
What am I sending?

Accept
    ↓
What response format do I want?
```

---

# 14. Content Negotiation

A client can request a preferred representation.

Example:

```http
Accept: application/json
```

The server may return:

```http
Content-Type: application/json
```

If the server cannot produce an acceptable representation, the request can result in:

```text
406 Not Acceptable
```

---

# 15. Authorization Header

Authentication credentials are commonly sent using:

```http
Authorization: Bearer <JWT>
```

Example:

```http
Authorization: Bearer eyJhbGciOi...
```

Spring Security can process this token before the controller executes.

---

# 16. Custom Headers

Applications can define custom headers when needed.

Example:

```http
X-Request-ID: 7f82ab
```

In Spring:

```java
@GetMapping
public ProductResponse getProduct(
        @RequestHeader("X-Request-ID")
        String requestId) {

    return productService.getProduct(requestId);
}
```

Use standard HTTP headers when they already represent the concept. Avoid creating unnecessary custom headers.

---

# 17. Request ID / Correlation ID

A request ID helps trace a request through distributed systems.

Example:

```text
Client
  ↓
API Gateway
  ↓ request-id: abc123
Product Service
  ↓
Database
```

Logs can include:

```text
requestId=abc123
```

This makes debugging much easier.

---

# 18. @RequestHeader Map

When multiple headers are needed:

```java
@GetMapping
public ResponseEntity<?> get(
        @RequestHeader Map<String, String> headers) {

    ...
}
```

This can be useful for generic processing, but explicit header parameters are often clearer when only a few known headers are required.

---

# 19. Request Parameters vs Headers

Query parameters:

```text
GET /products?page=0&size=20
```

are generally used for request-specific filtering or control.

Headers:

```http
Authorization: Bearer ...
X-Request-ID: abc123
```

are generally used for metadata, authentication, content negotiation, tracing, and other request context.

---

# 20. Request Mapping

Spring provides:

```java
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

Example:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
}
```

Then:

```java
@GetMapping("/{id}")
```

creates:

```text
GET /api/products/{id}
```

---

# 21. @RestController

`@RestController` combines:

```java
@Controller
@ResponseBody
```

It tells Spring that controller methods generally return data directly as the HTTP response body.

Example:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
}
```

---

# 22. @ResponseBody

With `@ResponseBody`, the returned object is written to the HTTP response body instead of being interpreted as a view.

Example:

```java
@GetMapping("/{id}")
@ResponseBody
public ProductResponse getProduct(
        @PathVariable Long id) {

    return productService.getProduct(id);
}
```

With `@RestController`, this behavior is already provided.

---

# 23. ResponseEntity

`ResponseEntity` gives explicit control over:

```text
Status code
Headers
Body
```

Example:

```java
return ResponseEntity
    .status(HttpStatus.CREATED)
    .body(productResponse);
```

For a normal successful response:

```java
return ResponseEntity.ok(productResponse);
```

For no content:

```java
return ResponseEntity.noContent().build();
```

---

# 24. ResponseEntity with Headers

Example:

```java
HttpHeaders headers = new HttpHeaders();

headers.add(
    "Location",
    "/api/products/100"
);

return ResponseEntity
    .status(HttpStatus.CREATED)
    .headers(headers)
    .body(response);
```

This is useful when the API needs to explicitly control response metadata.

---

# 25. Returning DTOs

Prefer returning DTOs rather than exposing JPA entities directly.

Example:

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return productService.getProduct(id);
}
```

Benefits:

```text
Stable API contract
Security
Avoid leaking internal fields
Avoid persistence concerns
Better control over JSON
```

---

# 26. Request DTO

A request DTO represents data expected from the client.

Example:

```java
public class CreateProductRequest {

    private String name;
    private BigDecimal price;
}
```

Controller:

```java
@PostMapping
public ProductResponse create(
        @RequestBody CreateProductRequest request) {

    return productService.create(request);
}
```

---

# 27. Response DTO

A response DTO represents data returned to the client.

Example:

```java
public class ProductResponse {

    private Long id;
    private String name;
    private BigDecimal price;
}
```

The API can return only the fields that clients need.

---

# 28. Why Not Return Entities Directly?

Returning entities can cause:

```text
Internal fields exposed
Lazy-loading problems
Recursive relationships
Unstable API contracts
Serialization surprises
Tight coupling between DB and API
```

A DTO boundary keeps the API model separate from the persistence model.

---

# 29. Pagination

Never return millions of records in a single API response.

Instead:

```text
GET /api/products?page=0&size=20
```

Typical response:

```json
{
  "content": [
    {
      "id": 100,
      "name": "Laptop"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 250,
  "totalPages": 13
}
```

---

# 30. Sorting

Example:

```text
GET /api/products?sort=price,desc
```

Multiple sorting fields can be supported:

```text
GET /api/products?sort=category,asc&sort=price,desc
```

Always validate client-controlled sort fields to avoid exposing unintended database behavior.

---

# 31. Filtering

Example:

```text
GET /api/products?category=phones&minPrice=10000
```

Spring:

```java
@GetMapping
public List<ProductResponse> search(
        @RequestParam(required = false)
        String category,

        @RequestParam(required = false)
        BigDecimal minPrice) {

    return productService.search(
        category,
        minPrice
    );
}
```

---

# 32. Search

Search can use a query parameter:

```text
GET /api/products?search=laptop
```

For complex search systems, the backend might eventually use:

```text
Database indexes
Full-text search
Elasticsearch
OpenSearch
```

The API contract should remain simple even if the internal implementation becomes more sophisticated.

---

# 33. Boolean Query Parameters

Example:

```text
GET /products?active=true
```

Spring:

```java
@RequestParam boolean active
```

For optional booleans:

```java
@RequestParam(required = false)
Boolean active
```

Using `Boolean` allows:

```text
true
false
null
```

---

# 34. Enum Query Parameters

Example:

```text
GET /orders?status=SHIPPED
```

Java:

```java
public enum OrderStatus {
    CREATED,
    PAID,
    SHIPPED,
    CANCELLED
}
```

Spring can bind the query value to the enum.

```java
@RequestParam OrderStatus status
```

Invalid values should be handled with a clear client error response.

---

# 35. Date Query Parameters

Example:

```text
GET /orders?from=2026-01-01&to=2026-01-31
```

Prefer standardized formats such as ISO-8601.

Example:

```text
2026-01-31
```

For timestamps:

```text
2026-01-31T15:30:00Z
```

Be explicit about timezone semantics.

---

# 36. Matrix Parameters

Spring MVC supports matrix variables, but they are less common in modern REST APIs.

Example:

```text
/products/phones;color=black
```

Most APIs prefer ordinary query parameters:

```text
/products?category=phones&color=black
```

Use the style consistently with your API conventions.

---

# 37. URL Encoding

URLs must encode special characters appropriately.

Example:

```text
GET /products?search=Java%20Spring
```

Here:

```text
%20 → space
```

Clients and frameworks generally handle encoding automatically when URLs are constructed correctly.

Do not manually concatenate untrusted values into URLs without proper encoding.

---

# 38. Request Body vs Query Parameter

Use the request body for structured data used to create or modify resources.

Example:

```http
POST /products

{
  "name": "Laptop",
  "price": 75000
}
```

Use query parameters for retrieval controls:

```text
GET /products?category=phones&page=0
```

A simple rule:

```text
GET → query/path parameters
POST/PUT/PATCH → body for resource data
```

There are exceptions, but this is a useful API design default.

---

# 39. Path Variable Validation

Example:

```java
@GetMapping("/{id}")
public ProductResponse get(
        @PathVariable
        @Positive Long id) {

    return productService.getProduct(id);
}
```

With method validation enabled, invalid values can be rejected.

For example:

```text
GET /products/-1
```

should not reach business logic if positive IDs are required.

---

# 40. Request Body Validation

Example:

```java
public class CreateProductRequest {

    @NotBlank
    private String name;

    @Positive
    private BigDecimal price;
}
```

Controller:

```java
@PostMapping
public ProductResponse create(
        @Valid
        @RequestBody CreateProductRequest request) {

    return productService.create(request);
}
```

Invalid input should be converted into a consistent validation error response.

---

# 41. Validation Error Response

A clean API can return:

```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": {
    "name": "Name must not be blank",
    "price": "Price must be greater than zero"
  }
}
```

This is much more useful than returning a raw framework stack trace.

---

# 42. Error Response DTO

Example:

```java
public record ApiError(
    int status,
    String message,
    Instant timestamp
) {
}
```

A global exception handler can return this structure consistently.

---

# 43. Global Exception Handling

Use:

```java
@RestControllerAdvice
```

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiError> handleNotFound(
            ResourceNotFoundException ex) {

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(new ApiError(
                404,
                ex.getMessage(),
                Instant.now()
            ));
    }
}
```

This keeps error handling out of individual controllers.

---

# 44. API Versioning

Common approaches include:

```text
/api/v1/products
/api/v2/products
```

or headers/media types.

URI versioning is simple and easy to understand.

Example:

```http
GET /api/v1/products/100
```

Version only when the API contract needs a breaking change.

---

# 45. Naming REST Endpoints

Prefer nouns:

```text
/products
/users
/orders
```

Avoid action-heavy URLs when normal HTTP semantics can express the operation.

Prefer:

```http
POST /orders
```

over:

```http
POST /createOrder
```

Prefer:

```http
DELETE /products/100
```

over:

```http
POST /deleteProduct/100
```

---

# 46. Nested Resources

Example:

```text
GET /users/100/orders
```

This expresses:

```text
Orders belonging to user 100
```

But avoid deeply nested URLs such as:

```text
/users/1/orders/2/items/3/products/4
```

when a simpler resource URL is sufficient.

---

# 47. REST URL Example

Good:

```text
GET    /api/products
GET    /api/products/100
POST   /api/products
PUT    /api/products/100
PATCH  /api/products/100
DELETE /api/products/100
```

This gives the endpoint a predictable structure.

---

# 48. Stateless REST APIs

A REST API is commonly designed to be stateless.

Each request should contain the information required to process it.

Example:

```http
Authorization: Bearer <JWT>
```

The server should not need to remember arbitrary client request state between requests just to understand the next request.

Statelessness helps horizontal scaling.

---

# 49. Statelessness and Load Balancing

With stateless APIs:

```text
             Load Balancer
              /    |                 /     |             Instance A B     C
```

Any instance can handle the request.

This is easier to scale than requiring requests to return to one specific server because of local session state.

Distributed sessions can also be implemented when server-side state is required.

---

# 50. Request/Response Logging

Useful information to log:

```text
Request ID
HTTP method
Endpoint
Status code
Duration
User/service identity where appropriate
```

Avoid logging:

```text
Passwords
JWTs
Payment secrets
Sensitive personal data
```

---

# 51. API Performance

Important metrics:

```text
Average latency
p95 latency
p99 latency
Throughput
Error rate
Database time
External service time
```

A successful API should not only return correct data; it should meet its latency and reliability goals.

---

# 52. API Timeout

Every network call should have sensible timeouts.

For example:

```text
Client
  ↓ 2s timeout
Gateway
  ↓ 1.5s timeout
Product Service
  ↓
Database
```

Timeouts should be designed across layers rather than allowing requests to wait indefinitely.

---

# 53. API Retry

Retries can help with transient failures, but careless retries can make incidents worse.

Use retries carefully for:

```text
Transient network failures
Temporary dependency failures
```

Avoid blindly retrying:

```text
Validation errors
Authentication failures
Permanent business failures
Non-idempotent operations without protection
```

Use exponential backoff and jitter when appropriate.

---

# 54. CORS

CORS means:

```text
Cross-Origin Resource Sharing
```

Browsers enforce same-origin restrictions.

Example:

```text
Frontend:
https://shop.example.com

Backend:
https://api.example.com
```

The backend can explicitly allow the frontend origin.

Spring Boot/Spring Security can be configured to handle CORS.

---

# 55. CORS Preflight

For certain cross-origin requests, browsers send:

```http
OPTIONS /api/products
```

before the actual request.

The server responds with appropriate CORS headers.

This is why `OPTIONS` requests often appear in application logs.

---

# 56. API Security Basics

A REST API should consider:

```text
Authentication
Authorization
Input validation
Rate limiting
TLS/HTTPS
CORS
CSRF where applicable
Secure headers
Secret management
Error handling
```

Never trust client-provided data.

---

# 57. HTTPS

Production APIs should normally use:

```text
HTTPS
```

rather than plain HTTP.

HTTPS protects data in transit through TLS encryption.

It is especially important for:

```text
Credentials
JWTs
Personal data
Payment information
Business data
```

---

# 58. Common Interview Questions

## @PathVariable vs @RequestParam?

> `@PathVariable` extracts a value from the URL path and is typically used to identify a resource. `@RequestParam` extracts query parameters and is commonly used for filtering, pagination, sorting, and optional request behavior.

## @RequestBody?

> `@RequestBody` tells Spring to deserialize the HTTP request body into a Java object, commonly using Jackson for JSON.

## @RequestHeader?

> `@RequestHeader` extracts a value from an HTTP request header, such as a request ID or a custom header.

## Why use ResponseEntity?

> `ResponseEntity` gives explicit control over the response status, headers, and body.

## Why use DTOs?

> DTOs provide a stable API contract, prevent internal entity fields from being exposed, and separate the API model from the persistence model.

## What is Content-Type?

> Content-Type describes the media type of the request or response body, such as `application/json`.

## What is Accept?

> Accept tells the server which response media types the client can or prefers to receive.

## Why pagination?

> Pagination prevents large datasets from being returned in one response, reducing memory usage, network traffic, database load, and response latency.

## Why stateless APIs?

> Stateless APIs make horizontal scaling easier because any application instance can process a request without depending on local request state.

---

# 59. Practical Ecommerce API

A product API might look like:

```text
GET    /api/v1/products
GET    /api/v1/products/{id}
POST   /api/v1/products
PUT    /api/v1/products/{id}
PATCH  /api/v1/products/{id}
DELETE /api/v1/products/{id}
```

Filtering:

```text
GET /api/v1/products?category=phones
```

Pagination:

```text
GET /api/v1/products?page=0&size=20
```

Sorting:

```text
GET /api/v1/products?sort=price,asc
```

Search:

```text
GET /api/v1/products?search=iphone
```

---

# 60. Complete Request Flow

```text
Client
  ↓
HTTP Request
  |
  +-- Method
  +-- URL
  +-- Path Variables
  +-- Query Parameters
  +-- Headers
  +-- Body
  ↓
Spring Security
  ↓
Controller
  ↓
Validation
  ↓
Service
  ↓
Repository / External Service
  ↓
DTO
  ↓
HTTP Response
  |
  +-- Status Code
  +-- Headers
  +-- Body
```

---

# 61. Final Mental Model

Remember these five questions:

```text
1. What resource am I working with?
2. What HTTP method expresses the operation?
3. Where should each piece of request data go?
4. What status code accurately describes the result?
5. What headers and body should the client receive?
```

For example:

```text
Create product
    ↓
POST /api/products
    ↓
JSON request body
    ↓
Validate
    ↓
Save
    ↓
201 Created
    ↓
Location + JSON response
```

> **Good REST API design is mostly about clear contracts: meaningful URLs, correct HTTP semantics, predictable request/response structures, appropriate status codes, validation, security, and consistent error handling.**

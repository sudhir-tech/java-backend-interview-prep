# Spring Boot — REST API Development

REST API development is one of the most important practical areas for a Java/Spring Boot backend developer.

A typical Spring Boot REST API follows:

```text
Client
  ↓
HTTP Request
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Database
  ↓
Repository
  ↓
Service
  ↓
Controller
  ↓
HTTP Response
```

This file covers REST fundamentals, controllers, request/response handling, DTOs, validation, status codes, exception handling, pagination, filtering, API design, and common interview topics.

---

# 1. What Is REST?

REST stands for:

```text
Representational State Transfer
```

It is an architectural style for designing networked applications.

A REST API commonly uses:

```text
HTTP
JSON
Resources
HTTP methods
HTTP status codes
Stateless communication
```

Example resource:

```text
Product
```

API:

```text
GET    /api/products
GET    /api/products/10
POST   /api/products
PUT    /api/products/10
PATCH  /api/products/10
DELETE /api/products/10
```

---

# 2. What Is a Resource?

A resource represents a business entity.

Examples:

```text
Product
Customer
Order
Payment
Category
Cart
```

A REST API identifies resources through URLs.

Example:

```text
/api/products
```

represents the product collection.

```text
/api/products/101
```

represents one product.

---

# 3. RESTful URL Design

Prefer nouns:

```text
/api/products
/api/orders
/api/customers
```

Avoid verbs in resource URLs:

```text
/api/getProducts
/api/createProduct
/api/deleteProduct
```

The HTTP method already communicates the action.

---

# 4. HTTP Methods

Common HTTP methods:

```text
GET
POST
PUT
PATCH
DELETE
```

Typical usage:

| Method | Purpose |
|---|---|
| GET | Read |
| POST | Create |
| PUT | Replace/update |
| PATCH | Partial update |
| DELETE | Delete |

---

# 5. GET

Example:

```http
GET /api/products/101
```

Controller:

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return productService.getProduct(id);
}
```

GET should normally be safe and should not be used to modify server state.

---

# 6. POST

Used to create a resource.

```http
POST /api/products
```

Request:

```json
{
  "name": "Laptop",
  "price": 75000
}
```

Controller:

```java
@PostMapping
public ResponseEntity<ProductResponse> create(
        @Valid
        @RequestBody ProductRequest request) {

    ProductResponse response =
        productService.create(request);

    return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(response);
}
```

---

# 7. PUT

PUT commonly represents replacing the representation of an existing resource.

```http
PUT /api/products/101
```

Example:

```java
@PutMapping("/{id}")
public ProductResponse update(
        @PathVariable Long id,
        @Valid
        @RequestBody ProductRequest request) {

    return productService.update(
        id,
        request
    );
}
```

---

# 8. PATCH

PATCH is generally used for partial updates.

```http
PATCH /api/products/101
```

Example:

```json
{
  "price": 70000
}
```

This can update only the price instead of requiring the entire resource representation.

---

# 9. DELETE

Example:

```http
DELETE /api/products/101
```

Controller:

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> delete(
        @PathVariable Long id) {

    productService.delete(id);

    return ResponseEntity.noContent().build();
}
```

A common successful response is:

```text
204 No Content
```

---

# 10. @RestController

A REST controller:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

}
```

`@RestController` combines:

```java
@Controller
@ResponseBody
```

---

# 11. @RequestMapping

Class-level mapping:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

}
```

Method-level mapping:

```java
@GetMapping("/{id}")
```

Result:

```text
GET /api/products/{id}
```

---

# 12. @GetMapping

```java
@GetMapping
public List<ProductResponse> getProducts() {

    return productService.getProducts();
}
```

Maps:

```text
GET /api/products
```

---

# 13. @PostMapping

```java
@PostMapping
public ProductResponse create(
        @RequestBody ProductRequest request) {

    return productService.create(request);
}
```

Maps:

```text
POST /api/products
```

---

# 14. @PutMapping

```java
@PutMapping("/{id}")
public ProductResponse update(
        @PathVariable Long id,
        @RequestBody ProductRequest request) {

    return productService.update(
        id,
        request
    );
}
```

---

# 15. @PatchMapping

```java
@PatchMapping("/{id}")
public ProductResponse patch(
        @PathVariable Long id,
        @RequestBody ProductPatchRequest request) {

    return productService.patch(
        id,
        request
    );
}
```

---

# 16. @DeleteMapping

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> delete(
        @PathVariable Long id) {

    productService.delete(id);

    return ResponseEntity.noContent()
        .build();
}
```

---

# 17. @PathVariable

Reads values from the URL.

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return productService.getProduct(id);
}
```

Request:

```text
GET /api/products/101
```

Result:

```text
id = 101
```

---

# 18. Named Path Variable

```java
@GetMapping("/{productId}")
public ProductResponse getProduct(
        @PathVariable("productId")
        Long id) {

    return productService.getProduct(id);
}
```

This is useful when the Java parameter name differs from the URL variable.

---

# 19. Multiple Path Variables

```java
@GetMapping(
    "/categories/{categoryId}/products/{productId}"
)
public ProductResponse getProduct(
        @PathVariable Long categoryId,
        @PathVariable Long productId) {

    return productService.getProduct(
        categoryId,
        productId
    );
}
```

---

# 20. @RequestParam

Reads query parameters.

```java
@GetMapping
public List<ProductResponse> search(
        @RequestParam String category) {

    return productService.search(category);
}
```

Request:

```text
GET /api/products?category=electronics
```

---

# 21. Optional Request Parameter

```java
@GetMapping
public List<ProductResponse> search(
        @RequestParam(
            required = false
        )
        String category) {

    return productService.search(category);
}
```

---

# 22. Default Request Parameter

```java
@GetMapping
public List<ProductResponse> search(
        @RequestParam(
            defaultValue = "all"
        )
        String category) {

    return productService.search(category);
}
```

---

# 23. Multiple Request Parameters

```java
@GetMapping
public List<ProductResponse> search(
        @RequestParam String category,
        @RequestParam BigDecimal minPrice,
        @RequestParam BigDecimal maxPrice) {

    return productService.search(
        category,
        minPrice,
        maxPrice
    );
}
```

Request:

```text
/api/products
?category=electronics
&minPrice=1000
&maxPrice=50000
```

---

# 24. @RequestBody

Reads the HTTP request body and converts it into a Java object.

```java
@PostMapping
public ProductResponse create(
        @RequestBody ProductRequest request) {

    return productService.create(request);
}
```

JSON:

```json
{
  "name": "Laptop",
  "price": 75000
}
```

Spring typically uses Jackson for JSON serialization/deserialization.

---

# 25. Request DTO

Create a DTO for incoming API data:

```java
public record ProductRequest(
    String name,
    BigDecimal price
) {
}
```

Controller:

```java
@PostMapping
public ProductResponse create(
        @RequestBody ProductRequest request) {

    return productService.create(request);
}
```

---

# 26. Why Use DTOs?

Avoid exposing JPA entities directly from REST APIs.

DTOs provide:

```text
API stability
Security
Validation
Clear contracts
Controlled fields
Separation from persistence model
```

---

# 27. Entity vs DTO

Entity:

```java
@Entity
public class Product {

    @Id
    private Long id;

    private String name;

    private BigDecimal price;
}
```

Response DTO:

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {
}
```

The API does not need to expose every entity field.

---

# 28. Request and Response DTOs

A good API often separates:

```text
ProductRequest
ProductResponse
```

Example:

```java
public record ProductRequest(
    String name,
    BigDecimal price
) {
}
```

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {
}
```

---

# 29. Validation

Use Bean Validation:

```java
public record ProductRequest(

    @NotBlank
    String name,

    @NotNull
    @Positive
    BigDecimal price

) {
}
```

Controller:

```java
@PostMapping
public ProductResponse create(
        @Valid
        @RequestBody ProductRequest request) {

    return productService.create(request);
}
```

---

# 30. Common Validation Annotations

```text
@NotNull
@NotBlank
@NotEmpty
@Size
@Min
@Max
@Positive
@PositiveOrZero
@Negative
@Email
@Pattern
@Past
@Future
```

Choose constraints based on the field's semantics.

---

# 31. Validation Error

Invalid request:

```json
{
  "name": "",
  "price": -10
}
```

Validation can reject it before business logic executes.

A global exception handler can convert validation failures into a consistent API response.

---

# 32. Global Exception Handling

Use:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

}
```

Then:

```java
@ExceptionHandler(
    ProductNotFoundException.class
)
public ResponseEntity<ErrorResponse>
handleProductNotFound(
        ProductNotFoundException ex) {

    return ResponseEntity
        .status(HttpStatus.NOT_FOUND)
        .body(
            new ErrorResponse(
                "PRODUCT_NOT_FOUND",
                ex.getMessage()
            )
        );
}
```

---

# 33. Custom Exception

```java
public class ProductNotFoundException
        extends RuntimeException {

    public ProductNotFoundException(
            Long id) {

        super(
            "Product not found: " + id
        );
    }
}
```

Service:

```java
public ProductResponse getProduct(
        Long id) {

    Product product =
        repository.findById(id)
            .orElseThrow(
                () -> new ProductNotFoundException(id)
            );

    return mapper.toResponse(product);
}
```

---

# 34. Error Response DTO

A simple response:

```java
public record ErrorResponse(
    String code,
    String message
) {
}
```

More complete APIs may include:

```text
timestamp
status
code
message
path
validation errors
trace/correlation ID
```

Avoid exposing internal stack traces to clients.

---

# 35. Validation Error Response

Example:

```json
{
  "code": "VALIDATION_ERROR",
  "message": "Request validation failed",
  "errors": [
    {
      "field": "name",
      "message": "must not be blank"
    },
    {
      "field": "price",
      "message": "must be greater than 0"
    }
  ]
}
```

A consistent error format makes frontend and client integration easier.

---

# 36. HTTP Status Codes

Important status codes:

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
```

---

# 37. 200 OK

Used for successful operations that return a response.

Example:

```text
GET /api/products/101
```

Response:

```text
200 OK
```

---

# 38. 201 Created

Used when a resource is successfully created.

Example:

```text
POST /api/products
```

Response:

```text
201 Created
```

Often include a `Location` header pointing to the new resource.

---

# 39. Location Header

Example:

```java
URI location =
    URI.create(
        "/api/products/" + response.id()
    );

return ResponseEntity
    .created(location)
    .body(response);
```

Response:

```text
201 Created
Location: /api/products/101
```

---

# 40. 204 No Content

Used when the operation succeeds but there is no response body.

Example:

```text
DELETE /api/products/101
```

Response:

```text
204 No Content
```

---

# 41. 400 Bad Request

Usually indicates invalid request data.

Examples:

```text
Malformed JSON
Invalid parameter format
Validation failure
```

---

# 42. 401 Unauthorized

Means the request lacks valid authentication credentials.

Important distinction:

```text
401 → authentication problem
403 → authorization problem
```

---

# 43. 403 Forbidden

The user is authenticated but does not have permission.

Example:

```text
User is authenticated
but
ADMIN role is required
```

---

# 44. 404 Not Found

Resource does not exist.

Example:

```text
GET /api/products/999999
```

when product 999999 does not exist.

---

# 45. 409 Conflict

Useful when the request conflicts with the current resource state.

Examples:

```text
Duplicate email
Duplicate SKU
Concurrent update conflict
Invalid state transition
```

---

# 46. 422 Unprocessable Content

Can be used when the request is syntactically valid but semantically unacceptable.

Example:

```text
Order cannot be cancelled
because it has already shipped.
```

The exact choice between:

```text
400
409
422
```

depends on the API contract and team conventions.

---

# 47. 500 Internal Server Error

Represents an unexpected server-side failure.

Do not return:

```text
Database stack trace
Java exception class
Internal SQL
Secret information
```

to the client.

Log the details internally and return a safe error response.

---

# 48. ResponseEntity

Example:

```java
@GetMapping("/{id}")
public ResponseEntity<ProductResponse> get(
        @PathVariable Long id) {

    ProductResponse response =
        service.getProduct(id);

    return ResponseEntity.ok(response);
}
```

For created:

```java
return ResponseEntity
    .created(location)
    .body(response);
```

For no content:

```java
return ResponseEntity
    .noContent()
    .build();
```

---

# 49. Service Layer

Controller should not contain business logic.

Bad:

```java
@PostMapping
public ProductResponse create(
        @RequestBody ProductRequest request) {

    // validation
    // database access
    // calculations
    // business rules
}
```

Better:

```text
Controller
    ↓
Service
    ↓
Repository
```

---

# 50. Controller Responsibility

Controller should mainly handle:

```text
HTTP request
Input binding
Validation trigger
Authentication/authorization integration
Calling service
HTTP response
```

Avoid putting complex business rules here.

---

# 51. Service Responsibility

Service handles:

```text
Business logic
Business validation
Transactions
Orchestration
Calling repositories
Calling external services
```

Example:

```java
@Service
public class ProductService {

    private final ProductRepository repository;

    public ProductService(
            ProductRepository repository) {

        this.repository = repository;
    }
}
```

---

# 52. Repository Responsibility

Repository handles data access.

Example:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long> {

}
```

The service should normally not know:

```text
HTTP request details
HTTP status codes
JSON structure
```

---

# 53. Complete REST Flow

```text
POST /api/products
        ↓
ProductController
        ↓
ProductRequest
        ↓
@Valid
        ↓
ProductService
        ↓
Business logic
        ↓
ProductRepository
        ↓
Database
        ↓
Product entity
        ↓
ProductResponse
        ↓
201 Created
```

---

# 54. Complete Example

Controller:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService service;

    public ProductController(
            ProductService service) {

        this.service = service;
    }

    @PostMapping
    public ResponseEntity<ProductResponse> create(
            @Valid
            @RequestBody ProductRequest request) {

        ProductResponse response =
            service.create(request);

        URI location =
            URI.create(
                "/api/products/" + response.id()
            );

        return ResponseEntity
            .created(location)
            .body(response);
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> get(
            @PathVariable Long id) {

        return ResponseEntity.ok(
            service.getProduct(id)
        );
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(
            @PathVariable Long id) {

        service.delete(id);

        return ResponseEntity
            .noContent()
            .build();
    }
}
```

---

# 55. Idempotency

An operation is idempotent if repeating the same request has the same intended effect as making it once.

Typically:

```text
GET     → idempotent
PUT     → idempotent
DELETE  → idempotent
POST    → generally not idempotent
PATCH   → depends on implementation
```

This is about the intended server state, not necessarily identical response metadata.

---

# 56. POST and Idempotency

Suppose:

```text
POST /orders
```

creates an order.

Calling it twice may create:

```text
Order 101
Order 102
```

Therefore POST is generally not idempotent.

For payment/order APIs, an idempotency key can be used to prevent accidental duplicate processing.

---

# 57. Idempotency Key

Example:

```http
POST /api/payments
Idempotency-Key: 7f3c-1234
```

The server can associate the key with the operation.

If the client retries the same request:

```text
Same key
↓
Same logical operation
↓
Avoid duplicate processing
```

This is especially important in distributed systems.

---

# 58. Pagination

Do not return millions of records in one response.

Instead:

```text
GET /api/products?page=0&size=20
```

Spring Data commonly uses:

```java
Pageable
```

Example:

```java
@GetMapping
public Page<ProductResponse> getProducts(
        Pageable pageable) {

    return service.getProducts(pageable);
}
```

---

# 59. Page vs Slice

Spring Data provides:

```text
Page<T>
Slice<T>
```

`Page` can provide total-count information.

`Slice` focuses on whether another slice exists and can avoid some count-query overhead.

Choose based on API requirements.

---

# 60. Pagination Response

Example:

```json
{
  "content": [
    {
      "id": 101,
      "name": "Laptop"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 150,
  "totalPages": 8
}
```

For public APIs, define the response contract explicitly rather than exposing internal framework types blindly.

---

# 61. Sorting

Example:

```text
GET /api/products
    ?sort=price,desc
```

Spring Data can bind sorting through:

```java
Sort
```

or:

```java
Pageable
```

---

# 62. Filtering

Example:

```text
GET /api/products
    ?category=electronics
    &minPrice=1000
    &maxPrice=50000
```

The service can translate these filters into repository queries.

Avoid putting complex filtering logic directly inside controllers.

---

# 63. Search

Example:

```text
GET /api/products?search=laptop
```

The API contract should clearly define:

```text
Case sensitivity
Partial matching
Pagination
Sorting
Maximum page size
```

---

# 64. Maximum Page Size

Never blindly trust:

```text
size=1000000
```

A backend should enforce a reasonable maximum.

Example:

```text
Default = 20
Maximum = 100
```

The exact values depend on the API.

---

# 65. API Versioning

Common approaches:

```text
/api/v1/products
/api/v2/products
```

Another approach uses headers/media types.

For a simple backend:

```text
/api/v1
```

is easy to understand and maintain.

Version only when there is a meaningful contract change.

---

# 66. API Naming

Prefer:

```text
/api/v1/products
/api/v1/products/{id}
/api/v1/orders
/api/v1/customers
```

Avoid unnecessary verbs:

```text
/api/v1/getProducts
/api/v1/createOrder
```

---

# 67. Nested Resources

Example:

```text
GET /api/customers/10/orders
```

can represent:

```text
Orders belonging to customer 10
```

But avoid excessively deep URLs such as:

```text
/api/customers/10/orders/20/items/30/product/40
```

Keep resource paths practical.

---

# 68. API Contract

A REST endpoint should define:

```text
HTTP method
URL
Request headers
Path parameters
Query parameters
Request body
Success status
Success response
Error statuses
Error response
```

Example:

```text
POST /api/v1/products

201 Created
400 Bad Request
409 Conflict
```

---

# 69. Content-Type

For JSON requests:

```http
Content-Type: application/json
```

Example:

```http
POST /api/products
Content-Type: application/json
```

Body:

```json
{
  "name": "Laptop",
  "price": 75000
}
```

---

# 70. Accept Header

The client can indicate what response representation it accepts:

```http
Accept: application/json
```

Spring MVC uses message converters to handle request and response representations.

---

# 71. JSON Serialization

Example:

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {
}
```

Spring Boot can serialize it into JSON:

```json
{
  "id": 101,
  "name": "Laptop",
  "price": 75000
}
```

Jackson is commonly used for this in Spring Boot.

---

# 72. JSON Deserialization

Incoming JSON:

```json
{
  "name": "Laptop",
  "price": 75000
}
```

can be deserialized into:

```java
ProductRequest
```

using Spring's HTTP message conversion infrastructure.

---

# 73. DTO Mapping

You can map manually:

```java
ProductResponse toResponse(
        Product product) {

    return new ProductResponse(
        product.getId(),
        product.getName(),
        product.getPrice()
    );
}
```

For larger applications, a mapper library such as MapStruct can reduce repetitive mapping code.

---

# 74. Entity Exposure Problem

Avoid:

```java
@GetMapping("/{id}")
public Product getProduct(...) {

    return repository.findById(id)
        .orElseThrow();
}
```

Potential problems:

```text
Internal fields exposed
Lazy-loading surprises
Coupling API to database model
Sensitive data leakage
Serialization recursion
Unstable API contract
```

Prefer:

```text
Entity → DTO
```

---

# 75. DTO Mapping Flow

```text
Request JSON
    ↓
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
    ↓
JSON
```

---

# 76. API Error Handling

Centralize common errors:

```text
ValidationException
ResourceNotFoundException
ConflictException
AuthenticationException
AccessDeniedException
UnexpectedException
```

Then map them consistently.

---

# 77. Global Error Handler

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(
        ProductNotFoundException.class
    )
    ResponseEntity<ErrorResponse>
    handleNotFound(
            ProductNotFoundException ex) {

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(
                new ErrorResponse(
                    "PRODUCT_NOT_FOUND",
                    ex.getMessage()
                )
            );
    }
}
```

---

# 78. Handling Validation Errors

Depending on Spring version and validation setup, request validation can result in exceptions such as:

```text
MethodArgumentNotValidException
HandlerMethodValidationException
```

A global handler can translate these into the API's standard validation-error format.

Do not assume every validation failure uses exactly the same exception type.

---

# 79. Exception Handling Strategy

A good structure:

```text
Controller
    ↓
Service
    ↓
Throw meaningful business exception
    ↓
@RestControllerAdvice
    ↓
HTTP response
```

Avoid:

```java
try {
    // everything
}
catch (Exception e) {
    return ResponseEntity
        .badRequest()
        .build();
}
```

This hides real server errors and produces inaccurate status codes.

---

# 80. Logging Errors

When handling an unexpected exception:

```text
Log internal details
Return safe client message
Include correlation/request ID when available
```

Do not expose:

```text
Stack trace
Database password
SQL credentials
Internal hostnames
Secret keys
```

---

# 81. Correlation ID

Distributed systems often use a correlation/request ID:

```text
X-Correlation-Id
```

Flow:

```text
Client
 ↓
API Gateway
 ↓
Service A
 ↓
Service B
 ↓
Database
```

The same identifier helps connect logs across services.

---

# 82. REST API Security

REST APIs commonly need:

```text
Authentication
Authorization
Input validation
HTTPS
Rate limiting
CORS configuration
Secure headers
Sensitive-data protection
```

Spring Security can handle many of these concerns.

---

# 83. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

Example:

```text
Authentication
→ JWT is valid

Authorization
→ user has ADMIN role
```

---

# 84. Stateless REST APIs

A common REST architecture is stateless.

Each request carries the information required for authentication/processing.

Example:

```http
Authorization: Bearer <token>
```

The server does not need to maintain conversational state for every client request in the traditional HTTP-session sense.

---

# 85. CORS

CORS controls whether browsers allow frontend applications from one origin to call another origin.

Example:

```text
Angular:
http://localhost:4200

Spring Boot:
http://localhost:8080
```

These are different origins.

The backend must be configured appropriately for browser-based cross-origin requests.

---

# 86. CORS Best Practice

Avoid:

```java
@CrossOrigin("*")
```

in production without understanding the security implications.

Prefer explicit allowed origins and centralized configuration where appropriate.

---

# 87. REST API and Database Transactions

A service method may define the transaction:

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(
            OrderRequest request) {

        saveOrder();
        updateInventory();
    }
}
```

The controller should normally not manage database transactions.

---

# 88. REST API and Validation

Recommended flow:

```text
HTTP Request
    ↓
DTO Binding
    ↓
@Valid
    ↓
Validation
    ↓
Service
    ↓
Business Rules
```

Do not rely only on database constraints for user-facing request validation.

Both application validation and database constraints can be useful.

---

# 89. Business Validation

Example:

```text
Request:
quantity = 10
```

Basic validation:

```text
quantity > 0
```

Business validation:

```text
Requested quantity <= available inventory
```

The second rule belongs to business/service logic, not merely DTO validation.

---

# 90. API Performance

Important backend considerations:

```text
Pagination
Database indexing
Efficient queries
Caching
Avoiding N+1 queries
Connection pooling
Response size
Compression
Async processing where appropriate
```

REST controller code itself is usually not the primary performance bottleneck.

---

# 91. Avoid N+1 Queries

Example:

```text
Load 100 orders
    ↓
For each order
    ↓
Load customer
```

Could produce:

```text
1 query
+
100 additional queries
=
101 queries
```

Use appropriate:

```text
Fetch strategy
Entity graphs
JOIN FETCH
DTO projections
Batching
Query design
```

based on the use case.

---

# 92. API Caching

For read-heavy endpoints:

```java
@Cacheable("products")
public ProductResponse getProduct(
        Long id) {

    ...
}
```

Caching can reduce database load.

But always consider:

```text
Cache invalidation
TTL
Stale data
Cache key
Memory
Distributed cache consistency
```

---

# 93. API Documentation

A REST API should ideally have machine-readable documentation.

A common approach in Spring Boot is:

```text
OpenAPI
Swagger UI
```

Documentation can describe:

```text
Endpoints
Request parameters
Request bodies
Responses
Status codes
Authentication
Schemas
```

---

# 94. OpenAPI Example

A controller method can be documented with OpenAPI annotations when the project uses an OpenAPI library.

Conceptually:

```java
@Operation(
    summary = "Get product by ID"
)
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return service.getProduct(id);
}
```

Keep API documentation synchronized with the actual contract.

---

# 95. REST API Testing

Tools commonly used:

```text
Postman
Insomnia
curl
Swagger UI
JUnit
MockMvc
```

Example curl:

```bash
curl \
  -X GET \
  http://localhost:8080/api/products/101
```

---

# 96. Controller Test

Example:

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {

}
```

Mock the service dependency and test:

```text
HTTP status
JSON response
Validation
Request mapping
Error handling
```

---

# 97. Integration Test

Example:

```java
@SpringBootTest
@AutoConfigureMockMvc
class ProductApiIntegrationTest {

}
```

This can verify broader application wiring.

Use the lightest test type that provides the confidence you need.

---

# 98. REST API Checklist

```text
□ REST resources
□ HTTP methods
□ URL design
□ @RestController
□ @RequestMapping
□ @GetMapping
□ @PostMapping
□ @PutMapping
□ @PatchMapping
□ @DeleteMapping
□ @PathVariable
□ @RequestParam
□ @RequestBody
□ @RequestHeader
□ DTOs
□ Validation
□ @Valid
□ HTTP status codes
□ ResponseEntity
□ Global exception handling
□ Pagination
□ Sorting
□ Filtering
□ API versioning
□ Idempotency
□ CORS
□ Authentication
□ Authorization
□ Transactions
□ API documentation
□ Controller testing
□ Integration testing
```

---

# 99. Quick REST Architecture

```text
                    Client
                      │
                      ▼
                REST Controller
                      │
                Request DTO
                      │
                 Validation
                      │
                      ▼
                  Service
                      │
             Business Rules
                      │
              Transaction
                      │
                      ▼
                 Repository
                      │
                      ▼
                  Database
                      │
                      ▼
                 Repository
                      │
                      ▼
                  Service
                      │
                Response DTO
                      │
                      ▼
                REST Controller
                      │
                 HTTP Response
```

---

# 100. Final Interview Rules

> **Keep controllers thin, put business logic in services, use DTOs instead of exposing entities directly, validate incoming requests, return meaningful HTTP status codes, centralize exception handling, and design URLs around resources rather than actions.**

A strong Spring Boot REST API should be:

```text
Clear
Consistent
Stateless where appropriate
Validated
Secure
Testable
Documented
Performant
```

Next:

```text
01 Fundamentals
      ↓
02 Project Structure
      ↓
03 Dependency Injection & IoC
      ↓
04 Spring Beans & Configuration
      ↓
05 Spring Boot Annotations
      ↓
06 Configuration Properties & Profiles
      ↓
07 REST API Development
      ↓
08 Spring Data JPA
```

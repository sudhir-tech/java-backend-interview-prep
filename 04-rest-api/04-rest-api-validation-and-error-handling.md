# REST API — Validation and Error Handling

This file covers request validation, exception handling, consistent error responses, and practical Spring Boot patterns for production REST APIs.

---

# 1. Why Validate Requests?

Client input cannot be trusted.

Examples:

```text
Missing required field
Invalid email
Negative price
Invalid ID
String too long
Unsupported enum value
```

Validation protects the application from invalid data reaching business logic.

---

# 2. Bean Validation

Spring Boot commonly uses Jakarta Bean Validation annotations.

Common annotations:

```text
@NotNull
@NotBlank
@NotEmpty
@Size
@Min
@Max
@Positive
@PositiveOrZero
@Email
@Pattern
@Past
@Future
```

Example:

```java
public class CreateProductRequest {

    @NotBlank
    private String name;

    @Positive
    private BigDecimal price;
}
```

---

# 3. @NotNull vs @NotBlank vs @NotEmpty

### @NotNull

Checks that the value is not `null`.

### @NotBlank

Used mainly for strings. Rejects null, empty, and whitespace-only values.

### @NotEmpty

Rejects null and empty strings or collections, but does not necessarily reject whitespace-only strings.

---

# 4. Numeric Validation

Example:

```java
@Positive
private BigDecimal price;
```

This rejects zero and negative values.

For values that can be zero:

```java
@PositiveOrZero
private BigDecimal quantity;
```

Other constraints include:

```java
@Min(1)
private int quantity;

@Max(100)
private int discount;
```

---

# 5. String Validation

Example:

```java
@NotBlank
@Size(min = 3, max = 100)
private String name;
```

This prevents missing values and excessively long input.

---

# 6. Email Validation

Example:

```java
@Email
@NotBlank
private String email;
```

This performs basic email-format validation.

It does not prove that the email address actually exists.

---

# 7. Pattern Validation

Use `@Pattern` when a specific format is required.

Example:

```java
@Pattern(
    regexp = "^[A-Z]{2}[0-9]{4}$"
)
private String code;
```

The regular expression should match the actual business requirement.

---

# 8. @Valid

Use `@Valid` to trigger validation of a request body.

```java
@PostMapping
public ProductResponse createProduct(
        @Valid
        @RequestBody CreateProductRequest request) {

    return productService.create(request);
}
```

Spring validates the DTO before normal business processing.

---

# 9. @Validated

`@Validated` is useful for method-level validation and validation groups.

Example:

```java
@Validated
@RestController
public class ProductController {
}
```

A parameter can then be validated:

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable
        @Positive Long id) {

    return productService.getProduct(id);
}
```

---

# 10. Nested Validation

Example:

```java
public class CreateOrderRequest {

    @NotNull
    private Long userId;

    @Valid
    @NotNull
    private AddressRequest address;
}
```

`@Valid` on the nested object enables cascading validation.

---

# 11. Collection Validation

Example:

```java
public class CreateOrderRequest {

    @Valid
    @NotEmpty
    private List<OrderItemRequest> items;
}
```

Each `OrderItemRequest` can contain its own validation annotations.

---

# 12. DTO Validation Example

```java
public class CreateProductRequest {

    @NotBlank(
        message = "Product name is required"
    )
    @Size(
        max = 100,
        message = "Product name cannot exceed 100 characters"
    )
    private String name;

    @NotNull(
        message = "Price is required"
    )
    @Positive(
        message = "Price must be greater than zero"
    )
    private BigDecimal price;
}
```

---

# 13. Validation Failure

Suppose the client sends:

```json
{
  "name": "",
  "price": -10
}
```

Validation detects:

```text
name → blank
price → not positive
```

The request should be rejected before invalid data reaches the business layer.

---

# 14. Validation Error Response

A consistent response could be:

```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": {
    "name": "Product name is required",
    "price": "Price must be greater than zero"
  }
}
```

The exact JSON structure is an API design choice.

Consistency is the important part.

---

# 15. Why Consistent Errors Matter

Clients should not need different error-handling logic for every endpoint.

Bad:

```text
Endpoint A → {"error":"invalid"}
Endpoint B → {"message":"failed"}
Endpoint C → HTML error page
```

Better:

```text
All REST endpoints
       ↓
Consistent error contract
```

---

# 16. Error Response DTO

Example:

```java
public record ApiError(
    int status,
    String message,
    Instant timestamp
) {
}
```

For validation errors:

```java
public record ValidationErrorResponse(
    int status,
    String message,
    Map<String, String> errors,
    Instant timestamp
) {
}
```

---

# 17. Custom Exception

Create meaningful application exceptions.

```java
public class ResourceNotFoundException
        extends RuntimeException {

    public ResourceNotFoundException(
            String message) {

        super(message);
    }
}
```

Usage:

```java
Product product =
    repository.findById(id)
        .orElseThrow(() ->
            new ResourceNotFoundException(
                "Product not found: " + id
            )
        );
```

---

# 18. @ExceptionHandler

An exception handler maps an exception to an HTTP response.

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(
        ResourceNotFoundException.class
    )
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

---

# 19. @RestControllerAdvice

`@RestControllerAdvice` provides centralized exception handling for REST controllers.

Conceptually:

```text
Controller
   ↓
Exception
   ↓
RestControllerAdvice
   ↓
HTTP response
```

This avoids repetitive `try/catch` blocks in controllers.

---

# 20. Why Avoid try/catch in Every Controller?

Repeated controller-level exception handling causes:

```text
Duplicate code
Inconsistent responses
Harder maintenance
Large controllers
```

Centralized handling is generally cleaner.

---

# 21. Resource Not Found

Example:

```http
GET /api/products/999
```

If the product does not exist:

```http
404 Not Found
```

Response:

```json
{
  "status": 404,
  "message": "Product not found",
  "timestamp": "2026-08-21T10:00:00Z"
}
```

---

# 22. Duplicate Resource

Suppose a user registers with an email that already exists.

The service can throw:

```java
throw new DuplicateResourceException(
    "Email already registered"
);
```

A common response is:

```http
409 Conflict
```

The database should also enforce the unique constraint.

---

# 23. Business Exceptions

Business failures are different from programming errors.

Examples:

```text
Insufficient inventory
Order already cancelled
Payment cannot be refunded
Account is suspended
```

These can be represented by domain-specific exceptions.

---

# 24. Business Exception Example

```java
public class InsufficientStockException
        extends RuntimeException {

    public InsufficientStockException(
            String message) {

        super(message);
    }
}
```

The global handler can map it to an appropriate status such as:

```http
409 Conflict
```

---

# 25. 400 vs 409

Example:

```text
quantity = -5
```

Invalid request:

```text
400 Bad Request
```

Example:

```text
quantity = 5
available stock = 2
```

Business state conflict:

```text
409 Conflict
```

The exact API contract should be consistent.

---

# 26. Authentication and Authorization

Common mental model:

```text
Missing/invalid authentication → 401
Authenticated but insufficient permission → 403
```

Spring Security can handle many security failures before the request reaches the controller.

---

# 27. Validation Exceptions

Invalid request bodies can result in framework exceptions such as:

```text
MethodArgumentNotValidException
```

A global handler can convert them into the application's standard validation response.

---

# 28. ConstraintViolationException

Method-level or parameter-level validation can result in:

```text
ConstraintViolationException
```

Handle it centrally and return a consistent client error.

---

# 29. Malformed JSON

Suppose the client sends:

```json
{
  "price": 1000
```

The JSON is malformed.

The API should return a controlled client error instead of exposing a framework stack trace.

---

# 30. Invalid Enum

Suppose:

```java
enum OrderStatus {
    CREATED,
    PAID,
    SHIPPED
}
```

The client sends:

```text
status=UNKNOWN
```

Spring may reject the value during request binding.

Return a clear client error.

---

# 31. Type Conversion Error

Example:

```text
GET /products/abc
```

when the controller expects:

```java
@PathVariable Long id
```

The framework cannot convert `abc` to `Long`.

This should become a controlled 400-style response.

---

# 32. Missing Required Parameter

Example:

```java
@RequestParam String category
```

Client sends:

```text
GET /products
```

without the required parameter.

Spring can reject the request.

Return a consistent client error.

---

# 33. Unsupported HTTP Method

If an endpoint supports GET and POST but the client sends PATCH:

```http
405 Method Not Allowed
```

The response can indicate supported methods with the `Allow` header.

---

# 34. Unsupported Media Type

If the endpoint expects JSON:

```http
Content-Type: application/json
```

but receives an unsupported content type, the server can return:

```http
415 Unsupported Media Type
```

---

# 35. Not Acceptable

If the client requests a response format the API cannot produce:

```http
Accept: application/xml
```

while only JSON is supported, the API may return:

```http
406 Not Acceptable
```

---

# 36. Typical Exception Mapping

A practical mental model:

```text
Validation failure        → 400
Malformed request         → 400
Resource not found        → 404
Duplicate resource        → 409
Business conflict        → 409
Authentication failure   → 401
Authorization failure    → 403
Unsupported method        → 405
Unsupported media type    → 415
Rate limit exceeded      → 429
Unexpected failure       → 500
```

The exact mapping depends on the API contract.

---

# 37. Unexpected Exceptions

Example:

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<ApiError> handleUnexpected(
        Exception ex) {

    log.error(
        "Unexpected application error",
        ex
    );

    return ResponseEntity
        .status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body(new ApiError(
            500,
            "An unexpected error occurred",
            Instant.now()
        ));
}
```

The client receives a safe message while detailed information stays in server logs.

---

# 38. Never Leak Internal Errors

Do not expose:

```text
SQL queries
Database hostnames
Stack traces
Filesystem paths
JWT secrets
Passwords
Internal class names
```

Bad:

```json
{
  "error": "NullPointerException at ProductService.java:87"
}
```

Better:

```json
{
  "status": 500,
  "message": "An unexpected error occurred"
}
```

---

# 39. Error Logging

Useful server-side information includes:

```text
Request ID
Exception type
HTTP method
Endpoint
Relevant user/service identity
Stack trace
```

Never log passwords, tokens, or other secrets.

---

# 40. Correlation ID

An API can return a request ID:

```json
{
  "status": 500,
  "message": "An unexpected error occurred",
  "requestId": "abc123"
}
```

Support teams can search logs using:

```text
abc123
```

This is especially useful in distributed systems.

---

# 41. Validation vs Business Rules

Validation:

```text
price > 0
name not blank
email has valid format
```

Business rules:

```text
User cannot cancel shipped order
Product cannot be purchased when unavailable
Refund cannot exceed captured amount
```

Keep these concepts separate.

---

# 42. Controller Responsibility

Controllers should mainly handle:

```text
HTTP request
Request validation
Mapping
Calling service
HTTP response
```

Avoid complex business logic inside controllers.

Preferred:

```text
Controller
    ↓
Service
    ↓
Repository / dependencies
```

---

# 43. Service Responsibility

The service layer handles:

```text
Business rules
Transactions
Orchestration
Domain operations
```

Example:

```java
@Transactional
public OrderResponse createOrder(
        CreateOrderRequest request) {

    validateStock(request);

    Order order = createOrderEntity(request);

    return map(
        orderRepository.save(order)
    );
}
```

---

# 44. Repository Responsibility

Repositories should focus on persistence.

Example:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long> {
}
```

Avoid putting HTTP concerns into repositories.

---

# 45. Complete Error Flow

```text
HTTP Request
     ↓
Controller
     ↓
Validation
     |
     +-- invalid → Global Handler → 400
     |
     ↓
Service
     |
     +-- not found → Global Handler → 404
     |
     +-- business conflict → Global Handler → 409
     |
     +-- unexpected error → Global Handler → 500
     |
     ↓
Response
```

---

# 46. Problem Details

Modern Spring applications can use the standardized HTTP Problem Details format.

Conceptually:

```json
{
  "type": "https://example.com/problems/product-not-found",
  "title": "Product Not Found",
  "status": 404,
  "detail": "Product 100 was not found",
  "instance": "/api/products/100"
}
```

This provides a standardized structure for HTTP API errors.

---

# 47. Retryable vs Non-Retryable Errors

Transient failures may be retryable:

```text
429
502
503
504
```

Usually do not blindly retry:

```text
400
401
403
404
```

For state-changing requests, retries should also consider idempotency.

---

# 48. Rate Limiting Errors

Example:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

The client should wait before retrying.

Repeated immediate retries can make an overload worse.

---

# 49. Testing Error Responses

Test cases should include:

```text
Valid request
Missing required field
Invalid field
Resource not found
Duplicate resource
Unauthorized request
Forbidden request
Malformed JSON
Invalid path variable
Unsupported method
Unexpected server failure
```

Do not test only successful requests.

---

# 50. MockMvc Example

Example:

```java
mockMvc.perform(
    post("/api/products")
        .contentType(MediaType.APPLICATION_JSON)
        .content("""
            {
              "name": "",
              "price": -10
            }
        """)
)
.andExpect(status().isBadRequest());
```

This tests the API behavior at the HTTP layer.

---

# 51. Ecommerce Error Contract

For an ecommerce backend:

```text
Invalid product request
        → 400

Product not found
        → 404

Duplicate SKU
        → 409

Insufficient stock
        → 409

Invalid/expired JWT
        → 401

User lacks admin role
        → 403

Too many requests
        → 429

Unexpected failure
        → 500
```

---

# 52. Interview: How Do You Handle Exceptions in Spring Boot?

> I use centralized exception handling with `@RestControllerAdvice` and `@ExceptionHandler`. Business-specific exceptions are mapped to appropriate HTTP status codes, validation errors are converted into a consistent response structure, and unexpected exceptions return a safe 500 response while detailed information is logged internally.

---

# 53. Interview: How Do You Validate Request Data?

> I use Jakarta Bean Validation annotations such as `@NotBlank`, `@Positive`, `@Size`, and `@Email` on request DTOs and trigger validation with `@Valid`. I handle validation failures centrally so clients receive a consistent error response.

---

# 54. Interview: Why Use Global Exception Handling?

> It keeps controllers clean and gives the API a consistent error contract. Instead of writing try/catch logic in every controller, exceptions are handled centrally through `@RestControllerAdvice`.

---

# 55. Interview: 400 vs 404 vs 409?

> 400 is generally for an invalid request, 404 means the requested resource does not exist, and 409 indicates a conflict with the current state of a resource, such as a duplicate unique value or an invalid state transition.

---

# 56. Interview: How Do You Handle Unexpected Exceptions?

> I log the full exception internally with a request or correlation ID and return a generic 500 response to the client. I avoid exposing stack traces, database details, credentials, or other internal information.

---

# 57. Interview Scenario — Product Not Found

Question:

```text
GET /api/products/100
```

Product 100 does not exist. What happens?

Answer:

> The repository returns an empty result. The service throws a `ResourceNotFoundException`, and the global exception handler converts it into a 404 response with a consistent error body.

---

# 58. Interview Scenario — Duplicate Email

Question:

A user registers with an existing email. What status would you return?

Answer:

> I would normally return 409 Conflict because the request conflicts with the current state of the system. I would also enforce the uniqueness rule at the database level.

---

# 59. Interview Scenario — Invalid Product Price

Question:

The client sends:

```json
{
  "name": "Laptop",
  "price": -100
}
```

Answer:

> I would validate the request DTO using `@Positive` or `@PositiveOrZero` depending on the business rule. The invalid request should be rejected before business processing and returned as a consistent 400 validation response.

---

# 60. Interview Scenario — Malformed JSON

Question:

What happens if the client sends malformed JSON?

Answer:

> Jackson cannot deserialize the request body, so Spring raises a message-conversion-related exception. I handle that centrally and return a controlled 400 response rather than exposing the framework exception.

---

# 61. Interview Scenario — Unexpected Exception

Question:

A `NullPointerException` occurs in production. What should the API return?

Answer:

> The API should return a generic 500 response without exposing internal details. I would log the full exception with the request or correlation ID so the issue can be diagnosed from server-side logs.

---

# 62. Final Checklist

```text
□ Validate request DTOs
□ Use @Valid appropriately
□ Validate path/query parameters
□ Validate nested objects
□ Separate validation from business rules
□ Create meaningful custom exceptions
□ Use @RestControllerAdvice
□ Map exceptions to correct status codes
□ Return consistent error bodies
□ Never expose stack traces
□ Log detailed server-side errors
□ Include request/correlation IDs where useful
□ Handle malformed JSON
□ Handle type conversion errors
□ Handle authentication/authorization failures
□ Handle rate limiting
□ Test negative scenarios
□ Consider retryability
□ Protect sensitive information
```

---

# Final Mental Model

```text
Client Input
     ↓
Validate
     ↓
Can the request be processed?
     |
     +-- No → 4xx
     |
     ↓
Business Logic
     |
     +-- Resource missing → 404
     |
     +-- Business conflict → 409
     |
     +-- Unexpected failure → 500
     |
     ↓
Successful Response
```

> **A production REST API should fail predictably. Validate early, keep business rules in the appropriate layer, centralize exception handling, return meaningful status codes, expose useful client-safe errors, and keep sensitive implementation details inside server-side logs.**

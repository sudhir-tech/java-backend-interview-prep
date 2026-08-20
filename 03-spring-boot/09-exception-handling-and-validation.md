# Spring Boot — Exception Handling and Validation

Exception handling and validation are essential for building reliable Spring Boot REST APIs.

A good backend should:

```text
Reject invalid input
Return meaningful HTTP status codes
Provide consistent error responses
Keep stack traces away from clients
Log unexpected failures
Separate validation from business rules
Handle exceptions centrally
```

Typical flow:

```text
HTTP Request
     ↓
Request DTO
     ↓
Bean Validation
     ↓
Controller
     ↓
Service
     ↓
Business Validation
     ↓
Repository
     ↓
Exception if necessary
     ↓
Global Exception Handler
     ↓
Consistent Error Response
```

---

# 1. What Is an Exception?

An exception represents an abnormal condition during program execution.

Example:

```java
Product product =
    repository.findById(id)
        .orElseThrow(
            () -> new ProductNotFoundException(id)
        );
```

If the product does not exist, the service throws an exception instead of returning an invalid result.

---

# 2. Checked vs Unchecked Exceptions

Java exceptions are broadly divided into:

```text
Checked Exceptions
Unchecked Exceptions
```

Checked exceptions extend:

```java
Exception
```

but not:

```java
RuntimeException
```

Unchecked exceptions extend:

```java
RuntimeException
```

---

# 3. Checked Exception

Example:

```java
public void readFile()
        throws IOException {

}
```

The caller must handle or declare it.

Checked exceptions are commonly used when the caller can reasonably recover from the condition.

---

# 4. Unchecked Exception

Example:

```java
throw new IllegalArgumentException(
    "Invalid product ID"
);
```

Unchecked exceptions do not require explicit declaration.

Spring applications commonly use unchecked exceptions for business and programming errors.

---

# 5. Custom Business Exception

Example:

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
                () ->
                    new ProductNotFoundException(id)
            );

    return mapper.toResponse(product);
}
```

---

# 6. Why Custom Exceptions?

Instead of:

```java
throw new RuntimeException(
    "Something went wrong"
);
```

use meaningful exceptions:

```text
ProductNotFoundException
OrderNotFoundException
DuplicateEmailException
InsufficientStockException
InvalidOrderStateException
```

This makes the code and error handling easier to understand.

---

# 7. Exception Handling in REST APIs

A REST API should not return raw exceptions.

Bad response:

```text
java.lang.NullPointerException
at com.example...
```

Better:

```json
{
  "code": "PRODUCT_NOT_FOUND",
  "message": "Product not found: 101"
}
```

---

# 8. @ExceptionHandler

Spring MVC supports:

```java
@ExceptionHandler
```

Example:

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

# 9. Local Exception Handler

You can place an exception handler inside a controller:

```java
@RestController
public class ProductController {

    @ExceptionHandler(
        ProductNotFoundException.class
    )
    public ResponseEntity<ErrorResponse>
    handleNotFound(
            ProductNotFoundException ex) {

        return ResponseEntity
            .notFound()
            .build();
    }
}
```

This is useful for controller-specific behavior.

For application-wide handling, prefer:

```text
@RestControllerAdvice
```

---

# 10. @RestControllerAdvice

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

}
```

It allows exception handling across REST controllers.

Typical architecture:

```text
Controller
Controller
Controller
    ↓
GlobalExceptionHandler
```

---

# 11. @ControllerAdvice vs @RestControllerAdvice

`@ControllerAdvice` is a specialization of `@Component` used for global MVC exception handling and other cross-cutting controller concerns.

`@RestControllerAdvice` is effectively:

```text
@ControllerAdvice
+
@ResponseBody
```

It is convenient for REST APIs because handler responses are written directly to the HTTP response body.

---

# 12. Standard Error DTO

Example:

```java
public record ErrorResponse(
    String code,
    String message
) {
}
```

Response:

```json
{
  "code": "PRODUCT_NOT_FOUND",
  "message": "Product not found: 101"
}
```

---

# 13. Better Error Response

A production API may include:

```java
public record ErrorResponse(
    Instant timestamp,
    int status,
    String code,
    String message,
    String path
) {
}
```

Example:

```json
{
  "timestamp": "2026-08-20T12:30:00Z",
  "status": 404,
  "code": "PRODUCT_NOT_FOUND",
  "message": "Product not found: 101",
  "path": "/api/products/101"
}
```

---

# 14. Validation

Validation checks whether incoming data satisfies defined constraints.

Example:

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

    return service.create(request);
}
```

---

# 15. @Valid

`@Valid` triggers Bean Validation for the object.

Example:

```java
@Valid
@RequestBody ProductRequest request
```

If validation fails, Spring can reject the request before the controller method proceeds normally.

---

# 16. @Validated

Spring's:

```java
@Validated
```

supports validation scenarios such as method-level validation and validation groups.

Example:

```java
@Validated
@RestController
public class ProductController {

}
```

For simple request-body validation:

```text
@Valid
```

is often sufficient.

---

# 17. Common Validation Annotations

Important constraints:

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
@NegativeOrZero
@DecimalMin
@DecimalMax
@Email
@Pattern
@Past
@PastOrPresent
@Future
@FutureOrPresent
```

---

# 18. @NotNull

```java
@NotNull
private BigDecimal price;
```

Rejects:

```text
null
```

but allows:

```text
""
```

for a String.

---

# 19. @NotEmpty

```java
@NotEmpty
private List<String> tags;
```

Rejects:

```text
null
empty collection
```

but whitespace-only strings are not necessarily rejected.

---

# 20. @NotBlank

```java
@NotBlank
private String name;
```

Rejects:

```text
null
""
"   "
```

This is commonly appropriate for required text fields.

---

# 21. @Size

```java
@Size(
    min = 3,
    max = 100
)
private String name;
```

Useful for:

```text
String
Collection
Map
Array
```

depending on the validation provider.

---

# 22. @Positive

```java
@Positive
private BigDecimal price;
```

Requires a positive value.

For an ecommerce product:

```text
price > 0
```

---

# 23. @PositiveOrZero

```java
@PositiveOrZero
private int quantity;
```

Allows:

```text
0
1
2
...
```

but rejects negative values.

---

# 24. @Min and @Max

Example:

```java
@Min(1)
@Max(100)
private int quantity;
```

These are useful for numeric validation.

For decimal values, use appropriate decimal constraints instead of assuming `@Min` is suitable for every numeric type.

---

# 25. @Email

```java
@Email
private String email;
```

Checks whether the value follows a valid email-like format according to the validation provider.

Do not treat it as complete verification that an email address actually exists.

---

# 26. @Pattern

```java
@Pattern(
    regexp = "^[A-Z0-9-]+$"
)
private String sku;
```

Useful when a field must follow a specific format.

Avoid extremely complicated regular expressions when a simpler validation approach is clearer.

---

# 27. Date Validation

Example:

```java
@Future
private LocalDate deliveryDate;
```

Requires the date to be in the future.

Other useful constraints:

```text
@Past
@PastOrPresent
@Future
@FutureOrPresent
```

---

# 28. Validation Example

Request:

```java
public record ProductRequest(

    @NotBlank(
        message = "Product name is required"
    )
    @Size(
        max = 100,
        message = "Product name cannot exceed 100 characters"
    )
    String name,

    @NotNull(
        message = "Price is required"
    )
    @Positive(
        message = "Price must be greater than zero"
    )
    BigDecimal price

) {
}
```

---

# 29. Validation Error

Invalid request:

```json
{
  "name": "",
  "price": -10
}
```

Possible API response:

```json
{
  "code": "VALIDATION_ERROR",
  "message": "Request validation failed",
  "errors": [
    {
      "field": "name",
      "message": "Product name is required"
    },
    {
      "field": "price",
      "message": "Price must be greater than zero"
    }
  ]
}
```

---

# 30. Handling Validation Exceptions

Depending on the type of validation being performed and Spring Framework version, common exceptions include:

```text
MethodArgumentNotValidException
HandlerMethodValidationException
```

A global exception handler should map the relevant exceptions to the API's standard error structure.

---

# 31. MethodArgumentNotValidException

For invalid `@RequestBody` validation, a handler can process:

```java
@ExceptionHandler(
    MethodArgumentNotValidException.class
)
public ResponseEntity<ErrorResponse>
handleValidation(
        MethodArgumentNotValidException ex) {

    // collect field errors

    return ResponseEntity
        .badRequest()
        .body(...);
}
```

---

# 32. Field Errors

Spring provides validation errors such as:

```java
ex.getBindingResult()
    .getFieldErrors();
```

Each field error can provide:

```text
Field name
Rejected value
Validation message
Constraint information
```

Avoid returning sensitive rejected values in API responses.

---

# 33. Validation Error DTO

Example:

```java
public record FieldErrorResponse(
    String field,
    String message
) {
}
```

Then:

```java
public record ValidationErrorResponse(
    String code,
    String message,
    List<FieldErrorResponse> errors
) {
}
```

---

# 34. Global Validation Handler

Example structure:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(
        MethodArgumentNotValidException.class
    )
    public ResponseEntity<ValidationErrorResponse>
    handleValidation(
            MethodArgumentNotValidException ex) {

        List<FieldErrorResponse> errors =
            ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(error ->
                    new FieldErrorResponse(
                        error.getField(),
                        error.getDefaultMessage()
                    )
                )
                .toList();

        return ResponseEntity
            .badRequest()
            .body(
                new ValidationErrorResponse(
                    "VALIDATION_ERROR",
                    "Request validation failed",
                    errors
                )
            );
    }
}
```

---

# 35. Business Validation

Not every validation rule belongs on the DTO.

Simple structural validation:

```text
price > 0
name not blank
email valid
```

belongs naturally to request validation.

Business rules:

```text
Product must be in stock
Order cannot be cancelled after shipping
Coupon cannot be used after expiry
User cannot buy more than available inventory
```

belong in the service/domain layer.

---

# 36. Example Business Validation

```java
public void placeOrder(
        OrderRequest request) {

    Product product =
        productRepository
            .findById(request.productId())
            .orElseThrow(
                () -> new ProductNotFoundException(
                    request.productId()
                )
            );

    if (product.getStock()
            < request.quantity()) {

        throw new InsufficientStockException(
            product.getId()
        );
    }

    // continue order processing
}
```

---

# 37. Why Not Put Business Validation in DTO?

DTO validation should answer:

```text
Is this request structurally valid?
```

Business validation answers:

```text
Is this operation allowed in the current business state?
```

Keeping these separate makes the application easier to maintain.

---

# 38. Custom Validation Annotation

Sometimes standard constraints are not enough.

Example requirement:

```text
Password and confirmPassword must match.
```

You can create a custom constraint:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(
    validatedBy = PasswordMatchValidator.class
)
public @interface PasswordMatches {

    String message()
        default = "Passwords do not match";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload()
        default {};
}
```

---

# 39. Cross-Field Validation

A field-level annotation validates one field:

```text
@NotBlank
```

A class-level custom constraint can validate multiple fields:

```text
password
confirmPassword
```

This is called:

```text
Cross-field validation
```

---

# 40. Validation Groups

Sometimes different operations require different constraints.

Example:

```text
Create Product
Update Product
```

A field might be:

```text
Required on create
Optional on update
```

Validation groups can model this.

Use them only when they add real value; separate request DTOs are often simpler.

---

# 41. Separate Create and Update DTOs

Instead of complex validation groups:

```java
public record CreateProductRequest(
    @NotBlank String name,
    @NotNull @Positive BigDecimal price
) {
}
```

Update:

```java
public record UpdateProductRequest(
    @Size(max = 100) String name,
    @Positive BigDecimal price
) {
}
```

This is often easier to understand.

---

# 42. Exception Hierarchy

A clean application might define:

```text
ApplicationException
    ↓
BusinessException
    ├── ProductNotFoundException
    ├── OrderNotFoundException
    ├── DuplicateResourceException
    └── InsufficientStockException
```

Example:

```java
public class BusinessException
        extends RuntimeException {

    public BusinessException(
            String message) {

        super(message);
    }
}
```

---

# 43. Error Codes

Prefer stable error codes:

```text
PRODUCT_NOT_FOUND
DUPLICATE_SKU
INSUFFICIENT_STOCK
ORDER_ALREADY_SHIPPED
VALIDATION_ERROR
```

The client can use:

```text
code
```

for programmatic handling instead of parsing human-readable messages.

---

# 44. Error Messages

Good:

```text
Product with ID 101 was not found.
```

Bad:

```text
Error 123
```

Messages should be:

```text
Clear
Safe
Useful
Non-sensitive
```

---

# 45. Do Not Expose Internal Exceptions

Bad:

```json
{
  "message": "org.hibernate.exception.SQLGrammarException..."
}
```

Better:

```json
{
  "code": "INTERNAL_ERROR",
  "message": "An unexpected error occurred."
}
```

Log the technical details internally.

---

# 46. Handling Unexpected Exceptions

A fallback handler can catch unexpected failures:

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<ErrorResponse>
handleUnexpected(
        Exception ex) {

    log.error(
        "Unexpected error",
        ex
    );

    return ResponseEntity
        .status(
            HttpStatus.INTERNAL_SERVER_ERROR
        )
        .body(
            new ErrorResponse(
                "INTERNAL_ERROR",
                "An unexpected error occurred"
            )
        );
}
```

Do not swallow the exception silently.

---

# 47. Logging

For unexpected exceptions:

```java
log.error(
    "Unexpected error while processing order",
    ex
);
```

Logging should provide enough context to diagnose the problem without leaking secrets.

Avoid logging:

```text
Passwords
JWT secrets
Credit card numbers
API keys
Sensitive personal data
```

---

# 48. HTTP Status Mapping

A common mapping:

```text
Validation failure
→ 400 Bad Request

Authentication failure
→ 401 Unauthorized

Authorization failure
→ 403 Forbidden

Resource missing
→ 404 Not Found

Business state conflict
→ 409 Conflict

Unexpected server failure
→ 500 Internal Server Error
```

The exact API contract may vary.

---

# 49. 400 Bad Request

Use when the request is invalid.

Examples:

```text
Malformed JSON
Invalid field format
Validation failure
Invalid query parameter
```

---

# 50. 401 Unauthorized

Use when authentication is missing or invalid.

Example:

```text
Missing/invalid bearer token
```

Important:

```text
401 ≠ permission denied
```

---

# 51. 403 Forbidden

Use when the client is authenticated but does not have permission.

Example:

```text
USER tries to access ADMIN endpoint
```

---

# 52. 404 Not Found

Example:

```java
throw new ProductNotFoundException(id);
```

Handler:

```java
@ExceptionHandler(
    ProductNotFoundException.class
)
public ResponseEntity<ErrorResponse>
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
```

---

# 53. 409 Conflict

Example:

```java
throw new DuplicateSkuException(
    request.sku()
);
```

Response:

```text
409 Conflict
```

Useful for conflicts with current resource state.

---

# 54. 422 Unprocessable Content

Can be used when:

```text
JSON is syntactically valid
but
business semantics are unacceptable
```

Example:

```text
Cancel an order that has already shipped.
```

Whether to use 400, 409, or 422 should follow the project's API contract.

---

# 55. Exception Handling Flow

```text
Controller
    ↓
Service
    ↓
Exception
    ↓
@RestControllerAdvice
    ↓
@ExceptionHandler
    ↓
Error DTO
    ↓
HTTP Status
    ↓
Client
```

---

# 56. Exception Handling Example

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

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

    @ExceptionHandler(
        DuplicateSkuException.class
    )
    public ResponseEntity<ErrorResponse>
    handleDuplicateSku(
            DuplicateSkuException ex) {

        return ResponseEntity
            .status(HttpStatus.CONFLICT)
            .body(
                new ErrorResponse(
                    "DUPLICATE_SKU",
                    ex.getMessage()
                )
            );
    }
}
```

---

# 57. Spring's ProblemDetail

Modern Spring Framework versions support:

```java
ProblemDetail
```

for standardized HTTP API error responses.

Example:

```java
@ExceptionHandler(
    ProductNotFoundException.class
)
public ProblemDetail handleNotFound(
        ProductNotFoundException ex) {

    ProblemDetail problem =
        ProblemDetail.forStatus(
            HttpStatus.NOT_FOUND
        );

    problem.setTitle("Product not found");
    problem.setDetail(ex.getMessage());

    return problem;
}
```

This aligns with the standard HTTP problem-details approach.

---

# 58. ProblemDetail

A response can contain information such as:

```text
type
title
status
detail
instance
```

and application-specific properties.

Example conceptually:

```json
{
  "type": "...",
  "title": "Product not found",
  "status": 404,
  "detail": "Product 101 was not found"
}
```

Use either `ProblemDetail` or a project-specific error DTO consistently rather than mixing formats unnecessarily.

---

# 59. Exception Handler Ordering

If you have:

```java
@ExceptionHandler(Exception.class)
```

and:

```java
@ExceptionHandler(
    ProductNotFoundException.class
)
```

Spring should select the more specific applicable handler.

Still, keep exception mappings clear rather than relying on complicated overlapping handlers.

---

# 60. Validation and JSON Parsing

Not every bad request is Bean Validation failure.

For example:

```json
{
  "price":
}
```

is malformed JSON.

This is a request parsing problem, not simply:

```text
@NotNull
```

The global exception strategy should account for both:

```text
Validation errors
Parsing/binding errors
```

---

# 61. Type Mismatch

Request:

```text
GET /api/products/abc
```

Controller:

```java
@GetMapping("/{id}")
public ProductResponse get(
        @PathVariable Long id) {
}
```

`abc` cannot be converted to:

```text
Long
```

This should be mapped to a clear client-facing error rather than exposing framework internals.

---

# 62. Missing Request Parameter

Example:

```java
@GetMapping
public ProductResponse get(
        @RequestParam String category) {
}
```

Request:

```text
GET /api/products
```

may fail because:

```text
category
```

is required.

The API should return a clear `400 Bad Request` style response according to its contract.

---

# 63. Invalid Enum Parameter

Example:

```java
@GetMapping
public List<OrderResponse> get(
        @RequestParam OrderStatus status) {
}
```

Request:

```text
?status=INVALID
```

can cause parameter conversion failure.

A global handler can translate this into a meaningful API error.

---

# 64. Validation vs Database Constraint

Validation:

```java
@NotBlank
String sku;
```

Database:

```text
UNIQUE(sku)
```

Validation improves client feedback.

Database constraints guarantee integrity under concurrency.

Use both where appropriate.

---

# 65. Duplicate Resource Race Condition

Bad assumption:

```text
Check SKU exists
    ↓
No
    ↓
Insert
```

Two requests can execute:

```text
Request A → check → no
Request B → check → no
Request A → insert
Request B → insert
```

The database unique constraint should prevent the duplicate.

The application should translate the resulting constraint violation into a meaningful response such as:

```text
409 Conflict
```

---

# 66. Exception Translation

A persistence exception such as:

```text
DataIntegrityViolationException
```

may be translated into:

```text
DUPLICATE_SKU
```

when the application knows the violated constraint corresponds to a business conflict.

Do not expose raw database exception messages.

---

# 67. Global Handler Example

```java
@ExceptionHandler(
    DataIntegrityViolationException.class
)
public ResponseEntity<ErrorResponse>
handleDataIntegrityViolation(
        DataIntegrityViolationException ex) {

    log.error(
        "Database constraint violation",
        ex
    );

    return ResponseEntity
        .status(HttpStatus.CONFLICT)
        .body(
            new ErrorResponse(
                "RESOURCE_CONFLICT",
                "The requested resource conflicts with existing data"
            )
        );
}
```

For more precise error codes, inspect the violated constraint safely rather than exposing raw database details.

---

# 68. Exception Handling Best Practices

```text
Use meaningful custom exceptions
Centralize REST exception handling
Return consistent error structures
Use appropriate HTTP status codes
Validate requests early
Keep business validation in services/domain logic
Log unexpected failures
Do not expose stack traces
Do not expose secrets
Use stable error codes
Preserve useful debugging context
```

---

# 69. What Not To Do

Bad:

```java
try {
    service.create(request);
} catch (Exception e) {
    return ResponseEntity
        .badRequest()
        .build();
}
```

Why?

```text
All exceptions become 400
Server failures are hidden
Debugging becomes difficult
Clients receive incorrect status codes
```

---

# 70. Better Approach

```text
Service
 ↓
Meaningful exception
 ↓
Global exception handler
 ↓
Correct HTTP status
```

Example:

```text
ProductNotFoundException
→ 404

DuplicateSkuException
→ 409

Validation failure
→ 400

Unexpected exception
→ 500
```

---

# 71. Validation Architecture

```text
Request JSON
     ↓
DTO
     ↓
Bean Validation
     ↓
Structural validation
     ↓
Controller
     ↓
Service
     ↓
Business validation
     ↓
Repository
```

This separation is important for maintainability.

---

# 72. Ecommerce Example

Create product:

```java
public record CreateProductRequest(

    @NotBlank
    @Size(max = 100)
    String name,

    @NotBlank
    @Size(max = 50)
    String sku,

    @NotNull
    @Positive
    BigDecimal price,

    @NotNull
    @PositiveOrZero
    Integer stock

) {
}
```

Business rule:

```text
SKU must be unique
```

Database:

```text
UNIQUE(sku)
```

Service:

```text
Create request
↓
Validate fields
↓
Create entity
↓
Save
↓
Constraint violation?
↓
409 Conflict
```

---

# 73. Order Example

Request validation:

```text
quantity > 0
```

Business validation:

```text
product exists
stock is sufficient
order state allows operation
payment is valid
```

Possible exceptions:

```text
ProductNotFoundException
InsufficientStockException
InvalidOrderStateException
PaymentFailedException
```

---

# 74. Error Response Standard

A mature API should have one predictable error style.

Example:

```json
{
  "timestamp": "2026-08-20T12:30:00Z",
  "status": 409,
  "code": "DUPLICATE_SKU",
  "message": "SKU already exists",
  "path": "/api/v1/products"
}
```

Validation:

```json
{
  "timestamp": "2026-08-20T12:31:00Z",
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Request validation failed",
  "path": "/api/v1/products",
  "errors": [
    {
      "field": "price",
      "message": "must be greater than 0"
    }
  ]
}
```

---

# 75. Correlation ID

For production systems, include a correlation/request identifier.

Example:

```text
X-Correlation-Id: abc-123
```

Logs:

```text
abc-123 Product request received
abc-123 Database query started
abc-123 Product not found
```

This makes debugging distributed requests easier.

---

# 76. Error Logging Levels

Typical approach:

```text
Expected business error
→ INFO/WARN depending on context

Unexpected application error
→ ERROR

Debugging details
→ DEBUG
```

Avoid logging every normal client validation error as an ERROR.

---

# 77. Monitoring

Exception handling should work with:

```text
Application logs
Metrics
Tracing
APM
Alerting
```

For example:

```text
500 errors increase
      ↓
Monitoring alert
      ↓
Check correlation ID
      ↓
Inspect logs/traces
      ↓
Identify root cause
```

---

# 78. Exception Handling and Transactions

Suppose:

```java
@Transactional
public void placeOrder() {

    createOrder();
    reduceInventory();
    processPayment();
}
```

If a relevant exception causes rollback:

```text
createOrder
     ↓
reduceInventory
     ↓
payment failure
     ↓
rollback
```

The exact rollback behavior depends on exception type and transaction configuration.

---

# 79. Important Rollback Interview Point

Human-written answer:

> By default, Spring's declarative transaction management generally rolls back for unchecked exceptions. If I need rollback for a checked exception, I can configure `rollbackFor` explicitly.

Example:

```java
@Transactional(
    rollbackFor = IOException.class
)
```

---

# 80. Don't Catch and Hide Transaction Exceptions

Bad:

```java
@Transactional
public void placeOrder() {

    try {
        saveOrder();
        processPayment();
    } catch (Exception e) {
        log.error("failed", e);
    }
}
```

If the exception is swallowed:

```text
Transaction may not roll back as intended
```

Handle or rethrow according to the application's transaction strategy.

---

# 81. Validation in Service Methods

For complex applications, services may also use:

```java
@Validated
```

and method-level constraints.

Example:

```java
@Validated
@Service
public class ProductService {

    public ProductResponse get(
            @Positive Long id) {

        ...
    }
}
```

The exact validation setup depends on the Spring Framework version and method-validation configuration.

---

# 82. Controller vs Service Validation

Controller/request validation:

```text
Input format
Required fields
Length
Range
Pattern
```

Service/business validation:

```text
Resource exists
User can perform operation
Inventory available
Order state valid
Business limits
```

This separation is a common interview topic.

---

# 83. Exception Handling Interview Question

### How do you handle exceptions globally in Spring Boot?

Human-written answer:

> I use `@RestControllerAdvice` with specific `@ExceptionHandler` methods. Business exceptions are mapped to appropriate HTTP statuses such as 404 or 409, validation errors are converted into a consistent 400 response, and unexpected exceptions are logged internally and returned as a safe 500 response.

---

# 84. Why Use @RestControllerAdvice?

Human-written answer:

> It centralizes REST exception handling so individual controllers don't need repetitive try-catch blocks. It also helps keep error responses consistent across the application.

---

# 85. How Do You Validate Request Bodies?

Human-written answer:

> I create request DTOs with Bean Validation annotations such as `@NotBlank`, `@NotNull`, and `@Positive`, then use `@Valid` on the controller's `@RequestBody`.

---

# 86. What Is the Difference Between @Valid and @Validated?

Human-written answer:

> `@Valid` is the standard Bean Validation trigger and is commonly used for request-body validation. `@Validated` is Spring's variant and is useful for method validation and validation groups.

---

# 87. How Do You Handle Validation Errors?

Human-written answer:

> I handle the validation exception globally, extract the field errors, and return a consistent response containing the field name and validation message with a 400 status.

---

# 88. How Do You Handle Unexpected Exceptions?

Human-written answer:

> I catch them at the global exception-handling layer, log the full technical exception internally, and return a generic safe error response to the client. I don't expose stack traces or internal implementation details.

---

# 89. How Do You Handle Duplicate Data?

Human-written answer:

> I enforce uniqueness at the database level and translate the resulting constraint violation into a meaningful business response, usually 409 Conflict. An application-level existence check alone isn't enough because of race conditions.

---

# 90. What Status Code Would You Return?

### Invalid request

```text
400 Bad Request
```

### Missing authentication

```text
401 Unauthorized
```

### Insufficient permissions

```text
403 Forbidden
```

### Resource doesn't exist

```text
404 Not Found
```

### Duplicate/conflicting state

```text
409 Conflict
```

### Unexpected server error

```text
500 Internal Server Error
```

---

# 91. Common Interview Trap

Question:

> Is 401 the same as 403?

Answer:

> No. 401 means the request is not successfully authenticated, while 403 means the client is authenticated but doesn't have permission to access the resource.

---

# 92. Common Interview Trap

Question:

> Should every exception return 400?

Answer:

> No. 400 represents a client-side bad request. A missing resource may be 404, a state conflict may be 409, and an unexpected server failure should normally be 500.

---

# 93. Common Interview Trap

Question:

> Should we return the exception message directly?

Answer:

> Not always. For expected business errors, a safe message can be useful. For unexpected exceptions, I return a generic message and log the technical details internally to avoid exposing sensitive implementation information.

---

# 94. Common Interview Trap

Question:

> Is validation enough to guarantee database integrity?

Answer:

> No. Validation improves request handling, but database constraints are still important for integrity and concurrency. For example, a unique database constraint is necessary to reliably prevent duplicate values under concurrent requests.

---

# 95. Production Exception Flow

```text
                 Client
                   │
                   ▼
             REST Controller
                   │
             @Valid Request
                   │
          ┌────────┴────────┐
          │                 │
       Invalid            Valid
          │                 │
          ▼                 ▼
       400                 Service
                            │
                    Business Validation
                            │
                     ┌──────┴──────┐
                     │             │
                  Success       Exception
                     │             │
                     ▼             ▼
                  Response    Custom Exception
                                   │
                                   ▼
                         @RestControllerAdvice
                                   │
                                   ▼
                            Error Response
```

---

# 96. Best-Practice Checklist

```text
□ Use request DTOs
□ Use @Valid for request validation
□ Use @Validated when appropriate
□ Separate structural and business validation
□ Create meaningful business exceptions
□ Use @RestControllerAdvice
□ Use specific @ExceptionHandler methods
□ Return correct HTTP statuses
□ Use stable error codes
□ Return consistent error structures
□ Log unexpected exceptions
□ Never expose stack traces
□ Never expose secrets
□ Enforce database constraints
□ Handle validation/binding errors
□ Handle malformed JSON
□ Handle type conversion errors
□ Consider correlation IDs
□ Keep transaction rollback behavior in mind
□ Test error scenarios
```

---

# 97. Testing Exception Handling

Controller tests should verify:

```text
Valid request → expected success
Invalid request → 400
Missing resource → 404
Conflict → 409
Unauthorized → 401
Forbidden → 403
Unexpected failure → 500
```

Example:

```java
mockMvc.perform(
        post("/api/products")
            .contentType(
                MediaType.APPLICATION_JSON
            )
            .content("""
                {
                    "name": "",
                    "price": -10
                }
            """)
    )
    .andExpect(
        status().isBadRequest()
    );
```

---

# 98. Testing Business Exceptions

Example:

```java
when(service.getProduct(101L))
    .thenThrow(
        new ProductNotFoundException(101L)
    );
```

Then verify:

```text
HTTP 404
Error code
Message
Response structure
```

---

# 99. Final Mental Model

```text
Validation
    ↓
Reject malformed/invalid input
    ↓
Business Logic
    ↓
Throw meaningful exceptions
    ↓
Global Handler
    ↓
Map exception → HTTP response
    ↓
Log unexpected failures
```

---

# 100. Final Interview Rule

> **Validate early, keep business validation in the service/domain layer, throw meaningful exceptions, handle them centrally with `@RestControllerAdvice`, return accurate HTTP status codes, and never expose internal exception details to clients.**

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
      ↓
09 Exception Handling & Validation
      ↓
10 Spring Security
```

# REST API — Documentation with OpenAPI and Swagger

This file covers how to document Spring Boot REST APIs using OpenAPI and Swagger UI, including API descriptions, request/response schemas, security documentation, examples, versioning, and interview-focused concepts.

---

# 1. Why API Documentation Matters

A REST API is used by clients such as:

```text
Frontend applications
Mobile applications
Other backend services
External partners
QA automation
Developers
```

Without documentation, consumers have to guess:

```text
Available endpoints
HTTP methods
Request formats
Required fields
Authentication
Response structures
Error responses
```

Good API documentation makes the contract explicit.

---

# 2. OpenAPI

OpenAPI is a specification for describing HTTP APIs.

It can describe:

```text
Endpoints
HTTP methods
Parameters
Request bodies
Responses
Schemas
Authentication
Error responses
Examples
```

The OpenAPI document is commonly represented as:

```text
JSON
```

or:

```text
YAML
```

---

# 3. Swagger

Swagger is a collection of tools built around API documentation and the OpenAPI ecosystem.

Commonly encountered tools include:

```text
Swagger UI
Swagger Editor
Swagger Codegen
```

In modern Spring Boot projects, a common approach is:

```text
Spring Boot
     ↓
springdoc-openapi
     ↓
OpenAPI specification
     ↓
Swagger UI
```

OpenAPI is the specification; Swagger UI is one tool that presents the specification interactively.

---

# 4. Swagger UI

Swagger UI provides a browser-based interface where developers can inspect and interact with API endpoints.

Conceptually:

```text
Browser
   ↓
Swagger UI
   ↓
OpenAPI document
   ↓
REST API
```

Developers can see:

```text
GET /products/{id}
POST /products
PUT /products/{id}
DELETE /products/{id}
```

and inspect the expected inputs and outputs.

---

# 5. Springdoc OpenAPI

A popular library for Spring Boot applications is:

```text
springdoc-openapi
```

It can generate OpenAPI documentation from Spring MVC/WebFlux application metadata and annotations.

The exact dependency and version should match the Spring Boot version used by the project.

---

# 6. Typical Maven Dependency

A common Spring MVC setup uses:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
</dependency>
```

The version should normally be managed explicitly or through the project's dependency management according to the Springdoc documentation.

---

# 7. Automatic Documentation

Springdoc can discover:

```text
@RestController
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
```

and generate OpenAPI information from the application.

This reduces the need to manually write every endpoint definition.

---

# 8. Basic Controller

Example:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping("/{id}")
    public ProductResponse getProduct(
            @PathVariable Long id) {

        return productService.getProduct(id);
    }
}
```

Springdoc can expose information about this endpoint.

---

# 9. OpenAPI Document

The generated specification conceptually looks like:

```yaml
paths:
  /api/products/{id}:
    get:
      parameters:
        - name: id
          in: path
      responses:
        "200":
          description: Product found
```

The actual generated document contains considerably more metadata.

---

# 10. OpenAPI Metadata

You can define API-level information such as:

```text
Title
Description
Version
Contact
License
```

Example:

```java
@OpenAPIDefinition(
    info = @Info(
        title = "E-commerce API",
        version = "1.0",
        description = "REST API for an e-commerce backend"
    )
)
@SpringBootApplication
public class EcommerceApplication {
}
```

---

# 11. API Title and Version

Good metadata helps consumers understand what they are using.

Example:

```text
E-commerce API
Version: 1.0
```

For larger systems, versioning should be planned as part of API lifecycle management.

---

# 12. @Operation

`@Operation` describes an individual endpoint.

Example:

```java
@Operation(
    summary = "Get product by ID",
    description = "Returns a product using its unique identifier"
)
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {
    ...
}
```

This makes generated documentation more useful than relying only on method names.

---

# 13. @Parameter

Parameters can be documented explicitly.

Example:

```java
@Parameter(
    description = "Unique product identifier",
    example = "100"
)
@PathVariable Long id
```

This helps API consumers understand what values are expected.

---

# 14. Request Body Documentation

For POST requests:

```java
@PostMapping
public ProductResponse createProduct(
        @RequestBody ProductRequest request) {
    ...
}
```

The request DTO can be documented with schema metadata.

Example:

```java
@Schema(
    description = "Product creation request"
)
public class ProductRequest {
}
```

---

# 15. @Schema

`@Schema` documents models and fields.

Example:

```java
@Schema(
    description = "Product name",
    example = "Laptop"
)
private String name;
```

For price:

```java
@Schema(
    description = "Product price",
    example = "74999.00"
)
private BigDecimal price;
```

---

# 16. Required Fields

If a request field is mandatory, the API documentation should make that clear.

Example:

```java
@NotBlank
@Schema(
    description = "Product name",
    example = "Laptop",
    requiredMode = Schema.RequiredMode.REQUIRED
)
private String name;
```

The validation annotation and documentation should agree.

---

# 17. Example Request

Swagger UI can show an example such as:

```json
{
  "name": "Laptop",
  "price": 74999.00,
  "category": "Electronics"
}
```

Examples are especially useful for complex request bodies.

---

# 18. Response Documentation

Example:

```java
@ApiResponse(
    responseCode = "200",
    description = "Product found"
)
```

You can also specify the returned schema.

Conceptually:

```java
@ApiResponse(
    responseCode = "200",
    content = @Content(
        schema = @Schema(
            implementation = ProductResponse.class
        )
    )
)
```

---

# 19. Multiple Responses

A production API often has several possible responses.

Example:

```text
200 → Product found
400 → Invalid request
401 → Unauthenticated
403 → Forbidden
404 → Product not found
500 → Unexpected server error
```

Document the important contract responses.

---

# 20. @ApiResponses

Example:

```java
@ApiResponses({
    @ApiResponse(
        responseCode = "200",
        description = "Product found"
    ),
    @ApiResponse(
        responseCode = "404",
        description = "Product not found"
    )
})
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {
    ...
}
```

---

# 21. Error Response Schema

Suppose your API returns:

```json
{
  "status": 404,
  "code": "PRODUCT_NOT_FOUND",
  "message": "Product not found",
  "timestamp": "2026-08-21T10:30:00Z"
}
```

Document this as a reusable error model.

Example:

```java
@Schema(description = "Standard API error")
public class ErrorResponse {
}
```

---

# 22. Why Standard Error Responses Matter

A consistent error format makes client development easier.

Instead of:

```text
Endpoint A → {"message": "..."}
Endpoint B → {"error": "..."}
Endpoint C → plain text
```

prefer a consistent contract.

Example:

```json
{
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Invalid request"
}
```

---

# 23. Documenting Security

A secured API should tell consumers:

```text
Authentication required
Authentication scheme
Authorization requirements
```

For JWT APIs, OpenAPI can describe:

```text
Bearer authentication
```

---

# 24. HTTP Bearer Authentication

Conceptually:

```text
Authorization: Bearer <JWT>
```

OpenAPI can define a security scheme:

```java
@Bean
public OpenAPI customOpenAPI() {

    return new OpenAPI()
        .components(
            new Components()
                .addSecuritySchemes(
                    "bearerAuth",
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")
                )
        );
}
```

---

# 25. @SecurityRequirement

An endpoint can indicate that authentication is required.

Example:

```java
@SecurityRequirement(
    name = "bearerAuth"
)
@GetMapping("/profile")
public UserResponse profile() {
    ...
}
```

Swagger UI can then provide an authorization mechanism for testing protected APIs.

---

# 26. Authorizing Swagger UI

For JWT-protected APIs:

```text
Swagger UI
    ↓
Authorize
    ↓
Enter JWT
    ↓
Try endpoint
    ↓
Authorization header
```

This is convenient for development and QA.

---

# 27. Do Not Expose Sensitive Information

API documentation should never expose:

```text
Passwords
Production secrets
Private keys
Database credentials
Internal tokens
```

Examples should use safe dummy data.

---

# 28. Hiding Internal Endpoints

Not every endpoint should necessarily be public.

Examples:

```text
Internal admin APIs
Internal health operations
Debug endpoints
Private service-to-service endpoints
```

Documentation visibility should follow the API's intended audience.

---

# 29. Swagger UI in Production

Swagger UI can be useful in production, but exposing interactive API documentation publicly may increase attack surface and reveal internal information.

Possible approaches:

```text
Development only
Internal network
Authenticated access
Separate documentation deployment
Public documentation for intentionally public APIs
```

Make the decision based on the application's security requirements.

---

# 30. API Groups

Large applications may have many endpoints.

You can organize documentation into groups such as:

```text
Public APIs
Admin APIs
User APIs
Order APIs
Product APIs
```

This makes Swagger UI easier to navigate.

---

# 31. Tagging APIs

Example:

```java
@Tag(
    name = "Products",
    description = "Product management APIs"
)
@RestController
@RequestMapping("/api/products")
public class ProductController {
}
```

Endpoints can then appear under the appropriate category.

---

# 32. Operation Tags

You can also assign tags to operations.

Example:

```java
@Operation(
    summary = "Create product",
    tags = {"Products"}
)
```

Keep tag names consistent across the API.

---

# 33. Documenting Pagination

Example endpoint:

```http
GET /api/products?page=0&size=20
```

Document:

```text
page
size
sort
```

Explain:

```text
Minimum page
Maximum page size
Default page size
Allowed sorting fields
```

This prevents client confusion.

---

# 34. Documenting Filtering

Example:

```http
GET /api/products?category=phones
```

Document:

```text
Parameter name
Allowed values
Optional/required
Example
Behavior
```

---

# 35. Documenting Sorting

Example:

```http
GET /api/products?sort=price,asc
```

Document supported fields:

```text
price
name
createdAt
```

Avoid allowing arbitrary database field names.

---

# 36. Documenting Path Parameters

Example:

```http
GET /api/products/{id}
```

Document:

```text
id
Type: Long
Required: yes
Example: 100
```

---

# 37. Documenting Query Parameters

Example:

```java
@GetMapping
public Page<ProductResponse> getProducts(
        @RequestParam(defaultValue = "0")
        int page,

        @RequestParam(defaultValue = "20")
        int size) {
    ...
}
```

Documentation should explain defaults and constraints.

---

# 38. HTTP Status Codes

Good API documentation should make status codes clear.

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
```

Use status codes consistently.

---

# 39. 201 Created

For resource creation:

```http
POST /api/products
```

A typical successful response:

```http
201 Created
```

You may also return a `Location` header pointing to the new resource.

---

# 40. 204 No Content

Useful when the operation succeeds but there is no response body.

Example:

```http
DELETE /api/products/100
```

Response:

```http
204 No Content
```

Do not return an unnecessary JSON body for a true 204 response.

---

# 41. 409 Conflict

Useful when the request conflicts with current application state.

Examples:

```text
Duplicate username
Duplicate resource
Concurrent update conflict
Business state conflict
```

Document these cases when they are part of the API contract.

---

# 42. Content Types

Document request and response media types.

Typical:

```http
Content-Type: application/json
```

Response:

```http
Accept: application/json
```

If an API supports multiple formats, document each one explicitly.

---

# 43. File Upload Documentation

For file upload:

```http
POST /api/files
Content-Type: multipart/form-data
```

The documentation should explain:

```text
Field name
File type
Maximum size
Required/optional
Other form fields
```

---

# 44. Enum Documentation

For:

```java
enum OrderStatus {
    CREATED,
    PAID,
    SHIPPED,
    CANCELLED
}
```

documentation should show the allowed values.

This is particularly useful for frontend and external API consumers.

---

# 45. Date and Time Documentation

Be explicit about:

```text
Format
Timezone
Precision
```

Example:

```text
2026-08-21T10:30:00Z
```

Use a consistent representation across the API.

---

# 46. API Versioning

Common approaches include:

```text
URI versioning
/api/v1/products

Header versioning
X-API-Version: 1

Media type versioning
Accept: application/vnd.company.v1+json
```

The important part is having a clear compatibility strategy.

---

# 47. Backward Compatibility

When changing an API, consider:

```text
Existing clients
Optional fields
Required fields
Response changes
Enum additions
Deprecation
Migration timeline
```

Avoid breaking clients unnecessarily.

---

# 48. Deprecating an Endpoint

An old endpoint can be marked as deprecated.

Conceptually:

```text
GET /api/v1/products
```

↓

```text
Deprecated
```

Provide migration guidance:

```text
Use /api/v2/products
```

A deprecation should include a reasonable migration period.

---

# 49. API Documentation as a Contract

Think of OpenAPI as:

```text
Consumer
   ↕
API Contract
   ↕
Provider
```

The documentation should represent actual application behavior.

Incorrect documentation can be worse than no documentation because clients build against false assumptions.

---

# 50. Documentation Drift

Documentation drift occurs when:

```text
Code changes
      ↓
Documentation not updated
```

Result:

```text
Actual API ≠ documented API
```

Prevent this through:

```text
Generated documentation
API reviews
Contract tests
CI validation
```

---

# 51. OpenAPI Validation in CI

A CI pipeline can validate the generated OpenAPI specification.

Conceptually:

```text
Build
 ↓
Generate OpenAPI
 ↓
Validate specification
 ↓
Check breaking changes
 ↓
Deploy
```

This helps prevent accidental contract changes.

---

# 52. API-First vs Code-First

Two common approaches:

## Code-First

```text
Java controller
      ↓
OpenAPI generated
```

Advantages:

```text
Fast development
Documentation close to code
Less duplication
```

## API-First

```text
OpenAPI specification
      ↓
Implementation
```

Advantages:

```text
Explicit contract
Parallel frontend/backend development
Contract review before implementation
```

Neither is universally better.

---

# 53. API-First Example

Specification:

```yaml
paths:
  /products/{id}:
    get:
      summary: Get product
```

Then:

```text
Frontend
Backend
QA
```

can agree on the contract before implementation is complete.

---

# 54. Generated Client Code

OpenAPI can be used to generate client SDKs.

Conceptually:

```text
OpenAPI
   ↓
Client Generator
   ↓
JavaScript client
Java client
Python client
```

Generated clients can reduce repetitive API integration code.

But generated code should still be reviewed and managed carefully.

---

# 55. Swagger UI vs Postman

Swagger UI:

```text
API documentation
Interactive API exploration
OpenAPI-based
```

Postman:

```text
API requests
Collections
Environment variables
Automated tests
Team workflows
```

They solve overlapping but different problems.

Many teams use both.

---

# 56. Swagger UI vs OpenAPI

Important distinction:

```text
OpenAPI = specification
Swagger UI = visualization/interface
```

OpenAPI describes the API.

Swagger UI renders that description interactively.

---

# 57. REST API Documentation Workflow

A practical workflow:

```text
Create controller
      ↓
Add meaningful API metadata
      ↓
Document request/response schemas
      ↓
Document security
      ↓
Generate OpenAPI
      ↓
Review Swagger UI
      ↓
Test endpoints
      ↓
Validate in CI
```

---

# 58. Ecommerce API Documentation

Example:

```text
Products
 ├── GET /api/products
 ├── GET /api/products/{id}
 ├── POST /api/products
 ├── PUT /api/products/{id}
 └── DELETE /api/products/{id}

Cart
 ├── GET /api/cart
 ├── POST /api/cart/items
 └── DELETE /api/cart/items/{id}

Orders
 ├── POST /api/orders
 └── GET /api/orders/{id}
```

Swagger UI can organize these using tags.

---

# 59. Product API Example

```java
@Operation(
    summary = "Get product",
    description = "Returns a product by ID"
)
@ApiResponses({
    @ApiResponse(
        responseCode = "200",
        description = "Product found"
    ),
    @ApiResponse(
        responseCode = "404",
        description = "Product not found"
    )
})
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return productService.getProduct(id);
}
```

---

# 60. API Documentation Checklist

```text
□ API title
□ API version
□ Description
□ Endpoint summaries
□ Request parameters
□ Request body schemas
□ Required fields
□ Examples
□ Response schemas
□ Success status codes
□ Error responses
□ Authentication
□ Authorization
□ Pagination
□ Filtering
□ Sorting
□ Enum values
□ Date/time formats
□ Deprecations
□ API versioning
□ Security considerations
```

---

# 61. Interview: What Is OpenAPI?

> OpenAPI is a standard specification for describing REST APIs. It defines endpoints, parameters, request and response schemas, authentication and other API metadata. Tools such as Swagger UI can render that specification into interactive documentation.

---

# 62. Interview: Swagger vs OpenAPI?

> OpenAPI is the API description specification, while Swagger is a set of tools built around the OpenAPI ecosystem. For example, Swagger UI can render an OpenAPI specification as interactive API documentation.

---

# 63. Interview: How Do You Document Spring Boot APIs?

> I typically use Springdoc OpenAPI to generate an OpenAPI specification from my Spring Boot controllers and DTOs. I add annotations such as `@Operation`, `@ApiResponse`, `@Parameter` and `@Schema` where automatic documentation isn't sufficient, and I use Swagger UI to review and test the contract.

---

# 64. Interview: How Do You Document JWT Authentication?

> I define an HTTP bearer security scheme with JWT as the bearer format and associate it with protected operations. Swagger UI can then provide an Authorize option so developers can supply a token while testing secured endpoints.

---

# 65. Interview: Why Are Error Responses Important in API Documentation?

> Clients need to know not only the successful response but also how failures are represented. I document important status codes and maintain a consistent error response schema so frontend and service consumers can handle errors predictably.

---

# 66. Interview: What Is API-First?

> API-first means defining and agreeing on the API contract before implementing the backend. An OpenAPI specification can be reviewed by backend, frontend and QA teams first, allowing them to develop against a shared contract.

---

# 67. Interview: How Do You Prevent Documentation Drift?

> I prefer generated documentation where possible, keep API metadata close to the implementation, and validate the generated OpenAPI specification in CI. For important APIs, contract testing and backward-compatibility checks can catch unintended changes.

---

# 68. Interview: Would You Expose Swagger UI in Production?

> It depends on the API. For internal systems I may restrict Swagger UI to authenticated or internal access. For intentionally public APIs, public documentation can be appropriate. I would avoid exposing sensitive internal endpoints or operational details unnecessarily.

---

# 69. Final Mental Model

```text
                    SPRING BOOT
                         |
                    Controllers
                         |
                         v
                     OpenAPI
                         |
              +----------+----------+
              |                     |
          Specification         Swagger UI
              |                     |
              v                     v
        API Contract          Interactive Docs
```

The important distinction is:

```text
OpenAPI
    =
machine-readable API contract

Swagger UI
    =
human-friendly interactive view
```

> **Good API documentation is part of the API contract. It should tell consumers what the endpoint accepts, what it returns, how authentication works, what errors can occur, and how the API evolves without forcing them to read the implementation code.**

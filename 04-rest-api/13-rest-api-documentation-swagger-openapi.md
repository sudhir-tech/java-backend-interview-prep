# REST API — Documentation with Swagger/OpenAPI

This file covers how to document Spring Boot REST APIs using OpenAPI and Swagger UI, including endpoint documentation, schemas, parameters, responses, security, validation, versioning, and interview questions.

---

# 1. Why API Documentation Matters

A REST API is a contract between the backend and its consumers.

Consumers may include:

```text
Frontend applications
Mobile applications
Other backend services
External clients
QA teams
Developers
```

Good documentation explains:

```text
What endpoint exists
What request it accepts
What response it returns
Which status codes are possible
What authentication is required
What validation rules apply
```

---

# 2. OpenAPI

OpenAPI is a specification for describing HTTP APIs.

An OpenAPI document can describe:

```text
Endpoints
HTTP methods
Parameters
Request bodies
Response bodies
Schemas
Authentication
Status codes
```

The specification is machine-readable and can be used by tools for documentation, testing and code generation.

---

# 3. Swagger vs OpenAPI

These terms are often confused.

```text
OpenAPI
    ↓
API specification

Swagger
    ↓
Tools built around API documentation
```

Historically, Swagger was the name of the specification. The specification is now called OpenAPI.

Common Swagger-related tools include:

```text
Swagger UI
Swagger Editor
Swagger Codegen
```

---

# 4. Swagger UI

Swagger UI provides an interactive browser interface for an OpenAPI specification.

It can show:

```text
GET endpoints
POST endpoints
Request parameters
Request body
Response schemas
Authentication
Try it out
```

A developer can inspect and call API endpoints directly from the browser.

---

# 5. Typical Spring Boot Flow

```text
Spring Boot Controllers
        ↓
OpenAPI metadata
        ↓
OpenAPI document
        ↓
Swagger UI
        ↓
Developer
```

The documentation can be generated from the application's API definitions and annotations.

---

# 6. Springdoc OpenAPI

For Spring Boot applications, `springdoc-openapi` is a common library for integrating OpenAPI documentation.

A typical Maven dependency is:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
</dependency>
```

The exact version should match the Spring Boot version and current springdoc compatibility guidance.

---

# 7. Swagger UI URL

After configuring the application, Swagger UI is commonly available at:

```text
/swagger-ui.html
```

or:

```text
/swagger-ui/index.html
```

The exact path can be customized.

The generated OpenAPI document is commonly exposed at:

```text
/v3/api-docs
```

---

# 8. OpenAPI JSON

An OpenAPI document may contain information similar to:

```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "Product API",
    "version": "1.0.0"
  },
  "paths": {}
}
```

The document describes the API in a structured format.

---

# 9. Basic Controller

Example:

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {

    @GetMapping("/{id}")
    public ProductResponse getProduct(
            @PathVariable Long id) {

        return productService.getProduct(id);
    }
}
```

Without additional metadata, springdoc can infer much of the API structure.

---

# 10. Why Add Explicit Documentation?

Automatic generation is useful, but explicit documentation improves:

```text
Descriptions
Business meaning
Expected responses
Validation rules
Examples
Authentication requirements
Deprecation information
```

Good API documentation should help another developer use the API without reading the implementation.

---

# 11. @Operation

`@Operation` describes an API operation.

Example:

```java
@Operation(
    summary = "Get product by ID",
    description = "Returns product details for the supplied product ID"
)
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {
    ...
}
```

This improves Swagger UI documentation.

---

# 12. Summary vs Description

`summary`:

```text
Short description
```

Example:

```text
Get product by ID
```

`description`:

```text
Detailed explanation
```

Example:

```text
Returns the product details including name,
price and category.
```

Keep summaries concise.

---

# 13. @ApiResponse

Document possible responses.

Example:

```java
@ApiResponse(
    responseCode = "200",
    description = "Product found"
)
```

Multiple responses:

```java
@Operation(summary = "Get product")
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
```

---

# 14. Response Schema

You can describe the response model:

```java
@ApiResponse(
    responseCode = "200",
    description = "Product found",
    content = @Content(
        schema = @Schema(
            implementation = ProductResponse.class
        )
    )
)
```

This tells Swagger/OpenAPI which model represents the response.

---

# 15. @Schema

`@Schema` describes models and fields.

Example:

```java
@Schema(description = "Product returned by the API")
public class ProductResponse {

    @Schema(
        description = "Unique product identifier",
        example = "100"
    )
    private Long id;

    @Schema(
        description = "Product name",
        example = "Laptop"
    )
    private String name;
}
```

---

# 16. Request Body Documentation

Example:

```java
@PostMapping
@Operation(summary = "Create product")
public ProductResponse createProduct(
        @RequestBody ProductRequest request) {
    ...
}
```

OpenAPI can infer the request schema from:

```text
ProductRequest
```

Additional annotations can provide examples and descriptions.

---

# 17. RequestBody Annotation

For more explicit documentation:

```java
@RequestBody(
    description = "Product creation request",
    required = true
)
```

This can be combined with OpenAPI `@io.swagger.v3.oas.annotations.parameters.RequestBody`.

---

# 18. Path Parameters

Example:

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @Parameter(
            description = "Product ID",
            example = "100"
        )
        @PathVariable Long id) {
    ...
}
```

The documentation now tells consumers what the path variable represents.

---

# 19. Query Parameters

Example:

```java
@GetMapping
public Page<ProductResponse> getProducts(
        @Parameter(
            description = "Page number",
            example = "0"
        )
        @RequestParam(defaultValue = "0")
        int page) {
    ...
}
```

Document:

```text
Meaning
Default value
Allowed range
Example
Required/optional status
```

---

# 20. Request Parameter Validation

Suppose:

```java
@RequestParam
@Min(1)
int size
```

The API documentation should communicate that:

```text
size must be at least 1
```

Validation and documentation should agree.

---

# 21. Enum Documentation

Suppose:

```java
public enum OrderStatus {
    CREATED,
    PAID,
    SHIPPED,
    CANCELLED
}
```

OpenAPI can expose these values in the schema.

This makes client integration easier.

---

# 22. Example Request

Good API documentation often provides examples.

Example:

```json
{
  "name": "Gaming Laptop",
  "price": 85000,
  "category": "ELECTRONICS"
}
```

Examples are especially useful for:

```text
Complex JSON
Authentication
Nested objects
Search requests
Error responses
```

---

# 23. Example Response

Example:

```json
{
  "id": 100,
  "name": "Gaming Laptop",
  "price": 85000,
  "category": "ELECTRONICS"
}
```

Examples make API behavior easier to understand than descriptions alone.

---

# 24. Error Response Documentation

Do not document only successful responses.

Example:

```json
{
  "status": 404,
  "error": "NOT_FOUND",
  "message": "Product not found",
  "path": "/api/v1/products/100"
}
```

Document common errors such as:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
429 Too Many Requests
500 Internal Server Error
```

Only document responses that the endpoint can meaningfully produce.

---

# 25. Standard Error Model

A consistent error model makes APIs easier to consume.

Example:

```java
public class ApiError {

    private int status;
    private String error;
    private String message;
    private String path;
    private Instant timestamp;
}
```

Then controllers can document this response model.

---

# 26. Global Exception Handler

If the application uses:

```java
@RestControllerAdvice
```

for centralized exception handling, the API documentation should still describe the resulting error responses.

Documentation should represent actual API behavior, not just controller success paths.

---

# 27. Authentication Documentation

Suppose the API uses:

```text
JWT Bearer authentication
```

OpenAPI can define a security scheme.

Conceptually:

```text
Authorization: Bearer <JWT>
```

Swagger UI can then provide an authorization control for testing secured endpoints.

---

# 28. Bearer Authentication Scheme

Example:

```java
@SecurityScheme(
    name = "bearerAuth",
    type = SecuritySchemeType.HTTP,
    bearerFormat = "JWT",
    scheme = "bearer"
)
```

This describes the JWT bearer scheme to OpenAPI.

---

# 29. Secured Endpoint

An endpoint can indicate that authentication is required.

Conceptually:

```java
@SecurityRequirement(name = "bearerAuth")
```

Then Swagger UI knows that the endpoint expects the configured authentication scheme.

---

# 30. Swagger UI Authorization

Typical flow:

```text
Open Swagger UI
      ↓
Authorize
      ↓
Enter JWT
      ↓
Swagger sends:
Authorization: Bearer <token>
      ↓
Call secured endpoint
```

This is convenient for development and testing.

---

# 31. Don't Expose Production Secrets

Swagger UI can be useful in production, but security must be considered.

Do not expose:

```text
Database credentials
JWT signing secrets
Internal infrastructure
Sensitive endpoint details
```

Swagger documentation itself is not a security boundary.

---

# 32. Restrict Swagger in Production

Depending on the environment, an organization may:

```text
Disable Swagger UI
Protect Swagger with authentication
Expose it only internally
Expose documentation separately
```

A public API may intentionally expose documentation, but internal APIs often should not be publicly discoverable.

---

# 33. API Groups

Large applications may have multiple API groups:

```text
Public API
Admin API
Internal API
```

OpenAPI grouping can make documentation easier to navigate.

---

# 34. API Tags

Tags group endpoints.

Example:

```text
Products
Orders
Users
Authentication
Payments
```

A controller can be associated with a tag.

This creates a cleaner Swagger UI.

---

# 35. @Tag

Example:

```java
@Tag(
    name = "Products",
    description = "Product management APIs"
)
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {
}
```

---

# 36. API Information

OpenAPI metadata can define:

```text
Title
Description
Version
Contact
License
```

Example:

```java
@Bean
public OpenAPI customOpenAPI() {
    return new OpenAPI()
        .info(new Info()
            .title("E-commerce API")
            .version("1.0.0")
            .description(
                "REST API for ecommerce operations"
            ));
}
```

---

# 37. API Version vs OpenAPI Document Version

These are different concepts.

API version:

```text
v1
v2
```

represents the API contract exposed to consumers.

OpenAPI specification version:

```text
3.0.x
3.1.x
```

describes the format of the API specification.

Do not confuse them.

---

# 38. Documenting Pagination

For a paginated endpoint, document:

```text
page
size
sort
cursor
nextCursor
```

Example:

```http
GET /api/v1/products?page=0&size=20&sort=price,asc
```

Response:

```json
{
  "content": [],
  "page": 0,
  "size": 20,
  "totalPages": 10,
  "totalElements": 200
}
```

---

# 39. Documenting Filtering

Example:

```http
GET /api/v1/products
    ?category=electronics
    &minPrice=10000
    &maxPrice=50000
```

Documentation should explain:

```text
Allowed filters
Data types
Defaults
Valid ranges
Combination rules
```

---

# 40. Documenting Idempotency

For retry-sensitive POST endpoints:

```http
POST /api/v1/orders
Idempotency-Key: checkout-123
```

Document:

```text
Header name
Required/optional status
Format
Lifetime
Duplicate behavior
Conflict behavior
```

This is important for payment and order APIs.

---

# 41. Documenting Rate Limits

API documentation can explain:

```text
Requests per minute
Rate-limit scope
429 response
Retry-After behavior
```

Example:

```text
100 requests/minute/user
```

Do not assume clients know your rate-limit policy.

---

# 42. Documenting Deprecation

If an endpoint is deprecated:

```text
Deprecated: true
```

Documentation should explain:

```text
Replacement endpoint
Migration guidance
Planned removal timeline
```

Do not leave consumers guessing.

---

# 43. OpenAPI and Client Generation

OpenAPI can be used to generate client code.

Conceptually:

```text
OpenAPI spec
     ↓
Code generator
     ↓
Java client
TypeScript client
Python client
```

This can reduce manual client implementation work.

Generated code still needs review and integration testing.

---

# 44. OpenAPI and Testing

The OpenAPI contract can also support:

```text
Contract testing
Schema validation
Mock servers
API testing
Client testing
```

The documentation can become part of the API development workflow rather than an afterthought.

---

# 45. Contract-First vs Code-First

Two common approaches exist.

Code-first:

```text
Write Spring controllers
       ↓
Generate OpenAPI documentation
```

Contract-first:

```text
Write OpenAPI specification
       ↓
Design implementation around contract
```

---

# 46. Code-First

Advantages:

```text
Fast for small/medium projects
Implementation drives documentation
Easy to start
```

Risks:

```text
Documentation may become incomplete
Annotations can become cluttered
API contract may not be designed carefully first
```

---

# 47. Contract-First

Advantages:

```text
API contract designed before implementation
Frontend and backend can work in parallel
Clear consumer expectations
Useful for large teams
```

Risks:

```text
More upfront work
Specification must stay synchronized with implementation
```

---

# 48. Which Approach?

For a small Spring Boot project:

```text
Code-first + OpenAPI
```

is often practical.

For large organizations or public APIs:

```text
Contract-first
```

can provide stronger governance.

The best choice depends on team size and workflow.

---

# 49. Keep Documentation Accurate

Bad:

```text
Swagger says 200
Actual API returns 201
```

Bad:

```text
Swagger says field required
Actual API allows null
```

Bad:

```text
Swagger says JWT required
Endpoint actually permits anonymous access
```

Documentation must match runtime behavior.

---

# 50. CI Validation

A mature API workflow can validate that:

```text
OpenAPI specification
        ↕
Actual API behavior
```

remains compatible.

CI can catch:

```text
Breaking schema changes
Missing documentation
Incorrect response contracts
```

before deployment.

---

# 51. Swagger UI vs Postman

Swagger UI:

```text
API documentation
Interactive exploration
Schema visibility
Quick endpoint testing
```

Postman:

```text
Collections
Environment variables
Advanced request workflows
Test scripts
Automation
```

They complement each other.

---

# 52. Swagger UI vs REST Client

IDE REST clients are useful for:

```text
Local development
Quick requests
Debugging
```

Swagger UI is useful for:

```text
Discovering the API
Understanding contracts
Sharing API documentation
```

---

# 53. OpenAPI in Ecommerce

Example structure:

```text
E-commerce API
|
+-- Authentication
|     +-- Login
|     +-- Register
|
+-- Products
|     +-- List products
|     +-- Get product
|     +-- Create product
|     +-- Update product
|
+-- Cart
|     +-- Get cart
|     +-- Add item
|     +-- Remove item
|
+-- Orders
      +-- Create order
      +-- Get order
      +-- Cancel order
```

Swagger UI can make this entire API discoverable.

---

# 54. Example Product Endpoint

```java
@Operation(
    summary = "Get product",
    description = "Returns a product by its ID"
)
@ApiResponses({
    @ApiResponse(
        responseCode = "200",
        description = "Product found",
        content = @Content(
            schema = @Schema(
                implementation = ProductResponse.class
            )
        )
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

# 55. Example Create Product Endpoint

```java
@Operation(
    summary = "Create product"
)
@ApiResponses({
    @ApiResponse(
        responseCode = "201",
        description = "Product created"
    ),
    @ApiResponse(
        responseCode = "400",
        description = "Invalid request"
    ),
    @ApiResponse(
        responseCode = "401",
        description = "Authentication required"
    ),
    @ApiResponse(
        responseCode = "403",
        description = "Access denied"
    )
})
@PostMapping
public ResponseEntity<ProductResponse> createProduct(
        @Valid @RequestBody ProductRequest request) {
    ...
}
```

---

# 56. Documentation for Roles

If an endpoint requires:

```text
ADMIN
```

the documentation should make that clear.

For example:

```text
POST /api/v1/products
Requires:
Bearer JWT
Role:
ADMIN
```

Authentication and authorization are different concepts.

---

# 57. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

Swagger can document authentication schemes, while endpoint descriptions can explain authorization requirements.

---

# 58. OpenAPI Security in a JWT Application

Typical architecture:

```text
Login
 ↓
JWT
 ↓
Swagger Authorize
 ↓
Bearer token
 ↓
Protected endpoint
 ↓
Spring Security
```

This makes local API exploration easier.

---

# 59. Documentation for Headers

Some APIs require custom headers:

```http
X-Tenant-ID: tenant-100
```

or:

```http
Idempotency-Key: abc123
```

Document these explicitly.

Consumers should not need to discover required headers by trial and error.

---

# 60. Documentation for Content Types

Document:

```text
Content-Type
Accept
```

Example:

```http
Content-Type: application/json
Accept: application/json
```

For file uploads:

```text
multipart/form-data
```

may be required.

---

# 61. Documentation for Status Codes

A good endpoint should clearly communicate:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
429 Too Many Requests
500 Internal Server Error
```

Only include responses relevant to the endpoint.

---

# 62. Avoid Over-Documentation

Documentation should clarify the API, not reproduce the entire source code.

Avoid:

```text
Huge implementation explanations
Internal variable details
Every private method
Unnecessary technical jargon
```

Focus on consumer-facing behavior.

---

# 63. Documentation as a Contract

Think of OpenAPI as:

```text
Source of truth for API behavior
```

It should answer:

```text
How do I call this?
What do I send?
What do I receive?
What can go wrong?
What authentication do I need?
```

---

# 64. Production API Documentation

A mature API documentation strategy includes:

```text
OpenAPI specification
Swagger UI or equivalent
Authentication documentation
Error models
Examples
Versioning
Deprecation
Rate limits
Pagination
Idempotency
Changelog
Migration guides
```

---

# 65. Security Considerations

Swagger itself does not secure your API.

Even if Swagger UI is protected:

```text
API endpoints still need Spring Security
```

Never assume:

```text
Not visible in Swagger
=
Secure
```

Security must be enforced by the application or infrastructure.

---

# 66. Common Mistakes

Avoid:

```text
Missing response codes
Missing authentication requirements
Outdated examples
Incorrect schemas
Undocumented required headers
Wrong field types
No error documentation
Exposing sensitive internal information
Leaving deprecated endpoints undocumented
```

---

# 67. Interview: What Is OpenAPI?

> OpenAPI is a standard specification for describing REST APIs. It defines endpoints, parameters, request and response schemas, status codes and security requirements. Tools such as Swagger UI can use that specification to provide interactive API documentation.

---

# 68. Interview: Swagger vs OpenAPI?

> OpenAPI is the API specification, while Swagger is a set of tools and the historical name associated with that specification. Swagger UI is commonly used to display an interactive OpenAPI document.

---

# 69. Interview: How Do You Add Swagger to Spring Boot?

> In a Spring Boot application, I can add the appropriate springdoc OpenAPI dependency. It generates an OpenAPI specification from the application's controllers and models, and Swagger UI provides an interactive interface for exploring and testing the endpoints.

---

# 70. Interview: Why Use Swagger?

> Swagger UI makes APIs easier to discover, test and integrate. Developers can see request parameters, request bodies, response schemas, authentication requirements and possible status codes without manually reading controller code.

---

# 71. Interview: How Do You Document JWT Authentication?

> I define a bearer HTTP security scheme in OpenAPI and associate secured endpoints with that scheme. Swagger UI can then provide an Authorize option where I enter the JWT, after which it sends the bearer token with protected requests.

---

# 72. Interview: Code-First vs Contract-First?

> Code-first generates the API contract from the implementation and is convenient for many Spring Boot projects. Contract-first starts with an OpenAPI specification and is useful when multiple teams need to agree on the API before implementation. I would choose based on project size, team workflow and governance requirements.

---

# 73. Interview: Should Swagger Be Enabled in Production?

> It depends on the API. For public APIs, documentation may intentionally be exposed. For internal APIs, I may restrict or disable Swagger UI in production. Regardless, Swagger visibility is not a security control; Spring Security and infrastructure must protect the actual endpoints.

---

# 74. Interview: How Do You Keep API Documentation Accurate?

> I keep documentation close to the implementation, use generated OpenAPI schemas where appropriate, add explicit response and parameter metadata, and validate the contract through tests or CI. The important thing is that the documented behavior matches the actual API.

---

# 75. Interview: What Should API Documentation Include?

> At minimum, I document endpoints, methods, parameters, request bodies, response schemas, status codes, authentication, validation rules and examples. For production APIs I also document pagination, rate limits, idempotency and deprecation when those features apply.

---

# 76. Final Mental Model

```text
                 SPRING BOOT API
                       |
                  Controllers
                       |
                  OpenAPI Metadata
                       |
                OpenAPI Specification
                       |
              +--------+--------+
              |                 |
         Swagger UI        API Clients
              |                 |
          Developers       Frontend / Services
```

For your ecommerce backend:

```text
Spring Boot
     ↓
ProductController
     ↓
OpenAPI
     ↓
Swagger UI
     |
     +── GET /products
     +── GET /products/{id}
     +── POST /products
     +── PUT /products/{id}
     +── DELETE /products/{id}
     |
     +── JWT Authentication
     +── Request validation
     +── Error responses
     +── Pagination
     +── Examples
```

> **Good API documentation turns a backend implementation into a usable contract. OpenAPI describes the contract, Swagger UI makes it interactive, and accurate documentation reduces integration mistakes between backend developers, frontend teams and external consumers.**

# REST API — Testing

This file covers REST API testing in Spring Boot, including unit tests, integration tests, MockMvc, WebTestClient, request validation, exception handling, security testing, test data, Testcontainers, and interview scenarios.

---

# 1. Why Test REST APIs?

REST API tests verify that the API behaves correctly from a consumer's perspective.

They can validate:

```text
HTTP method
URL
Request body
Headers
Authentication
Validation
Status code
Response body
Database behavior
Exception handling
```

A good test suite gives confidence that changes do not break existing API behavior.

---

# 2. Testing Layers

A Spring Boot backend can have several test levels:

```text
Unit tests
    ↓
Controller/API tests
    ↓
Integration tests
    ↓
End-to-end tests
```

Each level has a different purpose.

---

# 3. Unit Testing

A unit test focuses on a small piece of logic in isolation.

Example:

```text
ProductService
     ↓
Mock ProductRepository
```

The test verifies service behavior without starting the entire application.

Advantages:

```text
Fast
Focused
Easy to debug
```

---

# 4. Controller Testing

Controller tests focus on HTTP behavior.

Example:

```text
GET /api/v1/products/100
        ↓
ProductController
        ↓
Response
```

They can verify:

```text
Status code
JSON response
Validation
Request mapping
Exception mapping
Security behavior
```

---

# 5. Integration Testing

Integration tests verify that multiple application components work together.

Example:

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

These tests catch problems that isolated unit tests may miss.

---

# 6. Spring Boot Test

A common annotation is:

```java
@SpringBootTest
```

It loads the Spring application context.

Example:

```java
@SpringBootTest
class ProductServiceIntegrationTest {
}
```

This provides a realistic Spring environment but is usually slower than a focused unit test.

---

# 7. MockMvc

`MockMvc` allows Spring MVC endpoints to be tested without starting a real external HTTP server.

Example:

```java
mockMvc.perform(
    get("/api/v1/products/100")
)
.andExpect(status().isOk());
```

It is very useful for controller/API tests.

---

# 8. MockMvc Request

A more complete request:

```java
mockMvc.perform(
    get("/api/v1/products/{id}", 100)
        .contentType(MediaType.APPLICATION_JSON)
)
.andExpect(status().isOk());
```

The test can inspect the complete HTTP response.

---

# 9. Testing Response JSON

Example:

```java
mockMvc.perform(
    get("/api/v1/products/{id}", 100)
)
.andExpect(status().isOk())
.andExpect(jsonPath("$.id").value(100))
.andExpect(jsonPath("$.name").value("Laptop"));
```

This verifies the actual JSON response.

---

# 10. POST Request

Example:

```java
mockMvc.perform(
    post("/api/v1/products")
        .contentType(MediaType.APPLICATION_JSON)
        .content("""
            {
              "name": "Laptop",
              "price": 50000
            }
        """)
)
.andExpect(status().isCreated());
```

This tests:

```text
HTTP method
URL
Content-Type
JSON body
Response status
```

---

# 11. PUT Request

Example:

```java
mockMvc.perform(
    put("/api/v1/products/{id}", 100)
        .contentType(MediaType.APPLICATION_JSON)
        .content("""
            {
              "name": "Updated Laptop",
              "price": 55000
            }
        """)
)
.andExpect(status().isOk());
```

---

# 12. DELETE Request

Example:

```java
mockMvc.perform(
    delete("/api/v1/products/{id}", 100)
)
.andExpect(status().isNoContent());
```

The expected status depends on the API contract.

---

# 13. Query Parameters

Example:

```java
mockMvc.perform(
    get("/api/v1/products")
        .param("page", "0")
        .param("size", "20")
        .param("category", "electronics")
)
.andExpect(status().isOk());
```

This verifies that query parameters are correctly handled.

---

# 14. Path Variables

Example:

```java
mockMvc.perform(
    get("/api/v1/products/{id}", 100)
)
.andExpect(status().isOk());
```

Using parameterized paths keeps tests readable.

---

# 15. Headers

Example:

```java
mockMvc.perform(
    get("/api/v1/products")
        .header("X-Tenant-ID", "tenant-100")
)
.andExpect(status().isOk());
```

Headers are important when APIs use:

```text
Tenant IDs
Correlation IDs
Idempotency keys
Custom client metadata
Authorization
```

---

# 16. Content Type

A JSON API commonly expects:

```text
Content-Type: application/json
```

Test it explicitly when content negotiation matters.

Example:

```java
.contentType(MediaType.APPLICATION_JSON)
```

---

# 17. Accept Header

Example:

```java
.accept(MediaType.APPLICATION_JSON)
```

This tells the server which response representation the client prefers.

---

# 18. Testing Validation

Suppose:

```java
public class ProductRequest {

    @NotBlank
    private String name;

    @Positive
    private BigDecimal price;
}
```

An invalid request should be tested.

Example:

```json
{
  "name": "",
  "price": -10
}
```

Expected:

```text
400 Bad Request
```

---

# 19. Validation Test

Example:

```java
mockMvc.perform(
    post("/api/v1/products")
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

Also verify the error response if your API has a standard error format.

---

# 20. Testing 404

Example:

```java
mockMvc.perform(
    get("/api/v1/products/{id}", 999999)
)
.andExpect(status().isNotFound());
```

This verifies the API's not-found behavior.

---

# 21. Testing 400

Typical causes:

```text
Invalid JSON
Validation failure
Invalid parameter
Malformed request
```

Example:

```java
mockMvc.perform(
    post("/api/v1/products")
        .contentType(MediaType.APPLICATION_JSON)
        .content("""
            {
              "price": -1
            }
        """)
)
.andExpect(status().isBadRequest());
```

---

# 22. Testing 401

For protected endpoints:

```text
No authentication
        ↓
401 Unauthorized
```

Example:

```java
mockMvc.perform(
    get("/api/v1/orders")
)
.andExpect(status().isUnauthorized());
```

The exact result depends on the Spring Security configuration.

---

# 23. Testing 403

Authenticated user:

```text
ROLE_USER
```

tries:

```text
ADMIN endpoint
```

Expected:

```text
403 Forbidden
```

This verifies authorization rather than authentication.

---

# 24. Authentication vs Authorization Tests

Test both:

```text
Unauthenticated request → 401
Authenticated but insufficient role → 403
Authenticated with required role → success
```

These tests are important for security-sensitive APIs.

---

# 25. Spring Security Test

Spring Security provides testing support.

A common dependency is:

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

This enables convenient security test helpers.

---

# 26. @WithMockUser

Example:

```java
@Test
@WithMockUser(
    username = "admin",
    roles = "ADMIN"
)
void adminCanCreateProduct() throws Exception {
    ...
}
```

This creates a mock authenticated security context for the test.

---

# 27. Testing Roles

Example:

```java
@Test
@WithMockUser(
    roles = "USER"
)
void userCannotCreateProduct() throws Exception {
    mockMvc.perform(
        post("/api/v1/products")
    )
    .andExpect(status().isForbidden());
}
```

The exact endpoint and security rules depend on the application.

---

# 28. Testing JWT

For a real JWT filter, tests may need to verify:

```text
Valid JWT
Expired JWT
Invalid signature
Missing token
Malformed token
Insufficient authority
```

For controller-focused tests, Spring Security test utilities can often simulate authentication without generating real tokens.

---

# 29. When to Use a Real JWT

Use a real JWT in integration/security tests when you specifically want to verify:

```text
JWT parsing
Signature validation
Claims
Expiration
Authorities
Security filter behavior
```

Do not generate real tokens in every unit test unnecessarily.

---

# 30. Mocking Dependencies

For a controller unit-style test:

```text
Controller
   ↓
Mock Service
```

Example:

```java
@Mock
private ProductService productService;

@InjectMocks
private ProductController controller;
```

The exact setup depends on whether you use Mockito directly or Spring test slices.

---

# 31. Mockito

Mockito is commonly used to mock dependencies.

Example:

```java
when(productService.getProduct(100L))
    .thenReturn(response);
```

Then:

```java
verify(productService)
    .getProduct(100L);
```

This verifies interaction with the mocked dependency.

---

# 32. Mocking vs Real Database

Unit/controller test:

```text
Mock repository/service
```

Integration test:

```text
Real application components
Real or containerized database
```

Use mocks when testing behavior in isolation.

Use real infrastructure when integration behavior matters.

---

# 33. @WebMvcTest

`@WebMvcTest` is useful for MVC/controller-focused tests.

Example:

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {
}
```

It loads a focused MVC test slice rather than the entire application.

---

# 34. @WebMvcTest vs @SpringBootTest

`@WebMvcTest`:

```text
Controller layer
MVC infrastructure
Fast
Focused
```

`@SpringBootTest`:

```text
Full application context
More realistic
Slower
Useful for integration tests
```

Choose based on what behavior you need to verify.

---

# 35. @DataJpaTest

For repository/JPA tests:

```java
@DataJpaTest
class ProductRepositoryTest {
}
```

This focuses on JPA-related components.

It is useful for testing:

```text
Repository queries
Entity mappings
Persistence behavior
```

---

# 36. Repository Query Test

Suppose:

```java
List<Product> findByCategory(String category);
```

A repository test can verify:

```text
Correct records returned
Correct filtering
Entity mapping
Database constraints
```

---

# 37. Test Database

Integration tests can use:

```text
H2
Testcontainers
Dedicated test database
```

Each has tradeoffs.

---

# 38. H2

H2 is an in-memory database often used for tests.

Advantages:

```text
Fast
Easy setup
No external database
```

Potential problem:

```text
H2 behavior may differ from MySQL/PostgreSQL
```

SQL syntax, indexes, constraints and database-specific behavior can differ.

---

# 39. Testcontainers

Testcontainers allows tests to run real services in containers.

For example:

```text
JUnit
  ↓
Testcontainers
  ↓
MySQL container
```

This gives much more realistic database integration testing.

---

# 40. Why Testcontainers?

If production uses:

```text
MySQL
```

testing against a real MySQL container can catch issues that H2 might not.

Useful for:

```text
SQL compatibility
Database behavior
Indexes
Constraints
JPA mappings
Transactions
```

---

# 41. Integration Test Architecture

A realistic test can look like:

```text
MockMvc
   ↓
Spring Security
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
MySQL Testcontainer
```

This is much closer to production behavior than a mocked unit test.

---

# 42. Transaction Rollback in Tests

Spring tests may use transactions and rollback behavior to keep test data isolated.

For example:

```java
@Transactional
```

can be used in appropriate test scenarios.

Understand exactly which transaction is being managed before relying on rollback.

---

# 43. Test Data

Avoid random test data that makes failures difficult to reproduce.

Prefer clear fixtures:

```text
productId = 100
name = Laptop
price = 50000
```

For larger systems, use:

```text
Test data builders
Factories
Fixtures
```

---

# 44. Test Data Builder

Example concept:

```java
ProductRequestBuilder
    .aProduct()
    .withName("Laptop")
    .withPrice(50000)
    .build();
```

Builders make tests easier to read and modify.

---

# 45. Arrange-Act-Assert

A common testing structure:

```text
Arrange
   ↓
Set up data and mocks

Act
   ↓
Execute operation

Assert
   ↓
Verify result
```

Example:

```java
// Arrange
when(service.getProduct(100L))
    .thenReturn(response);

// Act
Result result = controller.getProduct(100L);

// Assert
assertEquals(100L, result.id());
```

---

# 46. Given-When-Then

BDD-style wording:

```text
Given
When
Then
```

Example:

```text
Given a product exists
When GET /products/100 is called
Then the API returns 200
```

This is especially useful for readable acceptance and behavior tests.

---

# 47. Happy Path

Every important endpoint should have a successful scenario.

Example:

```text
GET existing product
→ 200
```

or:

```text
POST valid product
→ 201
```

---

# 48. Negative Tests

Also test:

```text
Invalid input
Missing data
Unauthorized request
Forbidden request
Resource not found
Conflict
Unexpected dependency failure
```

Negative tests often expose more bugs than happy-path tests alone.

---

# 49. Boundary Testing

Test boundary values.

For:

```text
size >= 1
```

test:

```text
size = 0
size = 1
size = maximum
size = maximum + 1
```

Boundary tests are especially useful for:

```text
Pagination
Validation
Amounts
String lengths
Dates
Limits
```

---

# 50. Parameterized Tests

JUnit supports parameterized tests.

Conceptually:

```java
@ParameterizedTest
@ValueSource(ints = {0, -1, -100})
void invalidPageSize(int size) {
    ...
}
```

This avoids duplicating nearly identical tests.

---

# 51. Testing Exception Handling

Suppose:

```java
ProductNotFoundException
```

is converted by:

```java
@RestControllerAdvice
```

to:

```text
404 Not Found
```

Test the complete API behavior:

```text
Service exception
      ↓
Exception handler
      ↓
HTTP 404
      ↓
Standard JSON error
```

---

# 52. Testing GlobalExceptionHandler

Verify:

```text
Exception type
Status code
Error body
Message
Timestamp/path if applicable
```

Do not test only that an exception was thrown if the API contract is about the HTTP response.

---

# 53. Testing Content Negotiation

If the API supports multiple media types:

```text
application/json
application/xml
```

test the relevant:

```text
Accept
Content-Type
```

behavior.

---

# 54. Testing CORS

If browser clients depend on CORS, tests can verify relevant response headers and configuration.

Example concepts:

```text
Access-Control-Allow-Origin
Access-Control-Allow-Methods
```

Be careful not to make CORS more permissive than required.

---

# 55. Testing Pagination

Verify:

```text
Page number
Page size
Total elements
Total pages
Sorting
Empty pages
Large page requests
```

Example:

```text
GET /products?page=0&size=20
```

Expected:

```text
200
20 or fewer records
Correct metadata
```

---

# 56. Testing Sorting

Example:

```text
GET /products?sort=price,asc
```

Verify:

```text
Prices are ascending
```

Also test invalid sort fields if the API validates them.

---

# 57. Testing Filtering

Example:

```text
GET /products?category=electronics
```

Verify:

```text
Every returned product belongs to electronics.
```

Test combinations when the endpoint supports multiple filters.

---

# 58. Testing Idempotency

For an idempotent operation:

```text
Same request
same idempotency key
```

should not create duplicate business operations.

For example:

```text
POST /orders
Idempotency-Key: abc123
```

Repeated request:

```text
Same order result
```

The exact contract must be defined by the API.

---

# 59. Testing Rate Limiting

A rate-limit test can simulate:

```text
Request 1 → allowed
Request 2 → allowed
...
Request N → allowed
Request N+1 → 429
```

Also verify:

```text
Retry-After
rate-limit headers
reset behavior
```

when those are part of the contract.

---

# 60. Testing Caching

For a cached endpoint, test behavior such as:

```text
First request → database
Second request → cache
Update → cache invalidated
Next request → database
```

Do not make every test depend on Redis if caching behavior itself is not what you are testing.

---

# 61. Testing Redis

When Redis integration matters, use an actual Redis instance or Testcontainers rather than mocking every Redis operation.

This can catch:

```text
Serialization issues
TTL configuration
Key formatting
Connection problems
Redis command behavior
```

---

# 62. Testing Kafka/Event APIs

For event-driven behavior, integration tests may verify:

```text
API request
   ↓
Database transaction
   ↓
Event publication
   ↓
Consumer
   ↓
Expected side effect
```

Test event behavior separately from simple controller unit tests.

---

# 63. Test Naming

Good:

```java
shouldReturn404WhenProductDoesNotExist()
```

Good:

```java
shouldRejectProductWhenPriceIsNegative()
```

Good:

```java
shouldAllowAdminToCreateProduct()
```

Avoid vague names:

```java
test1()
testProduct()
works()
```

---

# 64. Test Independence

Each test should ideally be independent.

Avoid:

```text
Test B depends on Test A creating data
```

Tests should be able to run:

```text
Individually
In random order
In parallel where supported
```

without changing their meaning.

---

# 65. Avoid Over-Mocking

Bad:

```text
Mock controller
Mock service
Mock repository
Mock database
```

and then only verify that mocks were called.

The test may pass while the real application is broken.

Mock only what should be isolated.

---

# 66. Testing Pyramid

A useful mental model:

```text
             E2E
            /   \
       Integration
         /       \
      Unit Tests
```

Typically:

```text
Many fast unit tests
Some integration tests
Fewer expensive E2E tests
```

The exact balance depends on the system.

---

# 67. API Contract Testing

Contract testing verifies that consumers and providers agree on an API contract.

Example:

```text
Consumer expects:
GET /products/100
response.id = number
response.name = string
```

Provider tests ensure that contract remains valid.

---

# 68. Consumer-Driven Contracts

In consumer-driven contract testing:

```text
Consumer
   ↓
Defines expectations
   ↓
Provider verifies contract
```

Tools such as Pact are commonly associated with this style.

---

# 69. REST API Test with MockMvc

Complete conceptual example:

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {

    @Autowired
    MockMvc mockMvc;

    @MockBean
    ProductService productService;

    @Test
    void shouldReturnProduct() throws Exception {

        ProductResponse response =
            new ProductResponse(100L, "Laptop");

        when(productService.getProduct(100L))
            .thenReturn(response);

        mockMvc.perform(
            get("/api/v1/products/100")
        )
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.id").value(100))
        .andExpect(jsonPath("$.name").value("Laptop"));
    }
}
```

The exact mocking annotation depends on the Spring Boot/Spring Framework version and test setup.

---

# 70. Integration Test with SpringBootTest

Conceptually:

```java
@SpringBootTest
@AutoConfigureMockMvc
class ProductApiIntegrationTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    void shouldGetProduct() throws Exception {

        mockMvc.perform(
            get("/api/v1/products/100")
        )
        .andExpect(status().isOk());
    }
}
```

This can load the full application context.

---

# 71. API Test Through Real HTTP

Another approach is to start the application on a random port.

Conceptually:

```java
@SpringBootTest(
    webEnvironment =
        SpringBootTest.WebEnvironment.RANDOM_PORT
)
```

Then use an HTTP client such as:

```text
TestRestTemplate
WebTestClient
```

depending on the application stack.

---

# 72. TestRestTemplate

Useful for integration testing traditional Spring MVC applications.

Example concept:

```java
ResponseEntity<ProductResponse> response =
    restTemplate.getForEntity(
        "/api/v1/products/100",
        ProductResponse.class
    );
```

This exercises the application through HTTP.

---

# 73. WebTestClient

`WebTestClient` is commonly associated with reactive applications, but can also be useful in some Spring testing scenarios.

Example concept:

```java
webTestClient.get()
    .uri("/api/v1/products/100")
    .exchange()
    .expectStatus().isOk();
```

Choose the test client appropriate for your application architecture.

---

# 74. REST Assured

REST Assured is another library commonly used for API testing.

It provides a fluent syntax for HTTP tests.

Conceptually:

```java
given()
    .contentType("application/json")
.when()
    .get("/api/v1/products/100")
.then()
    .statusCode(200);
```

It can be useful for higher-level API integration tests.

---

# 75. MockMvc vs REST Assured

MockMvc:

```text
Spring MVC focused
No real external server required
Fast
Good controller/API tests
```

REST Assured:

```text
HTTP-oriented
Readable API tests
Useful for integration/system-level testing
```

They solve overlapping but different testing needs.

---

# 76. Testing External Services

Suppose:

```text
Order Service
    ↓
Payment Service
```

Don't make every test call a real payment provider.

Use:

```text
Mock server
WireMock
Stub
Test environment
```

for controlled integration testing.

---

# 77. External Service Failure

Test:

```text
Timeout
500 response
Connection failure
Malformed response
Slow response
Rate limit
```

The goal is to verify that your API behaves safely when dependencies fail.

---

# 78. Timeout Test

Example scenario:

```text
Order API
   ↓
Payment Service
   ↓
Timeout
```

Expected behavior might be:

```text
Controlled error
No infinite wait
Correct order state
Useful logs
```

The exact response depends on the business contract.

---

# 79. Retry Testing

If a service uses retries:

```text
Attempt 1 → failure
Attempt 2 → failure
Attempt 3 → success
```

test that:

```text
Retry count is bounded
Backoff works
Non-retryable errors are not retried
Duplicate side effects are prevented
```

---

# 80. Circuit Breaker Testing

For a circuit breaker:

```text
Repeated failures
      ↓
Circuit opens
      ↓
Fast failure/fallback
      ↓
Recovery period
      ↓
Half-open
      ↓
Success
      ↓
Closed
```

Tests should verify state transitions appropriate to the configured resilience library.

---

# 81. API Performance Testing

Functional tests answer:

```text
Does it work?
```

Performance tests answer:

```text
How does it behave under load?
```

Tools may include:

```text
JMeter
Gatling
k6
```

Performance testing should use realistic workloads.

---

# 82. Load Test

Example:

```text
100 concurrent users
GET /products
```

Measure:

```text
Latency
Throughput
Error rate
CPU
Memory
Database load
```

---

# 83. Stress Test

Increase load until the system approaches or exceeds capacity.

Goal:

```text
Find breaking point
```

Not simply:

```text
Make requests as fast as possible
```

---

# 84. Smoke Test

A smoke test checks that the most important functionality is working.

Example:

```text
Application starts
Login works
GET product works
Create order works
```

It is useful after deployment.

---

# 85. Regression Testing

Regression tests verify that previously working functionality remains working after changes.

Example:

```text
Add Redis caching
     ↓
Run product API regression tests
```

The goal is to ensure caching did not break:

```text
Authentication
Product responses
Updates
Deletes
Pagination
```

---

# 86. CI/CD API Testing

A typical pipeline:

```text
Developer pushes code
        ↓
Build
        ↓
Unit tests
        ↓
Integration tests
        ↓
Static analysis
        ↓
Package
        ↓
Deploy
        ↓
Smoke tests
```

Tests should provide fast feedback before production deployment.

---

# 87. Test Coverage

Coverage measures which code paths are exercised by tests.

Examples:

```text
Line coverage
Branch coverage
Method coverage
```

High coverage does not automatically mean high-quality tests.

A poorly designed test suite can have high coverage and still miss important behavior.

---

# 88. What Good API Tests Verify

Focus on behavior:

```text
Input
→ validation
→ business behavior
→ response
```

Not just implementation details.

For example:

```text
Good:
GET product returns 404 when product doesn't exist

Less useful:
verify exact private helper method was called
```

---

# 89. Test the Contract

For every important endpoint ask:

```text
What input is valid?
What input is invalid?
What authentication is required?
What status code is returned?
What response schema is returned?
What errors are possible?
```

This produces stronger API tests.

---

# 90. Ecommerce API Test Matrix

For:

```text
POST /products
```

test:

```text
Valid admin → 201
Invalid request → 400
Unauthenticated → 401
Normal user → 403
Duplicate product → 409 if applicable
Database failure → controlled error
```

For:

```text
GET /products/{id}
```

test:

```text
Existing product → 200
Missing product → 404
Invalid ID → 400 if validation applies
```

---

# 91. Product API Integration Flow

```text
Test
 ↓
MockMvc
 ↓
Spring Security
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
Test database
```

This verifies much more of the actual application than a controller-only unit test.

---

# 92. Test Checklist

```text
□ Happy path
□ Invalid request
□ Validation
□ 400 response
□ 401 response
□ 403 response
□ 404 response
□ 409 response where applicable
□ Exception handling
□ JSON response
□ Headers
□ Query parameters
□ Pagination
□ Sorting
□ Filtering
□ Authentication
□ Authorization
□ Database integration
□ External service failures
□ Timeout/retry behavior
□ Cache behavior
□ Rate limiting
□ Regression coverage
□ Smoke tests
```

---

# 93. Interview: What Is MockMvc?

> MockMvc is a Spring testing utility for testing Spring MVC endpoints without requiring a real external HTTP server. I use it to verify request mappings, status codes, JSON responses, validation and security behavior.

---

# 94. Interview: @WebMvcTest vs @SpringBootTest?

> `@WebMvcTest` loads a focused MVC test slice and is useful for controller testing. `@SpringBootTest` loads the full Spring application context and is better suited for broader integration testing. I use the smallest test scope that can verify the behavior I need.

---

# 95. Interview: Unit Test vs Integration Test?

> A unit test isolates a small component and usually mocks its dependencies. An integration test verifies that multiple real components work together, such as controller, service, repository and database. Unit tests are faster, while integration tests provide more realistic coverage.

---

# 96. Interview: How Do You Test REST APIs?

> I test the happy path as well as validation, authentication, authorization, error responses and edge cases. For controller-focused testing I can use MockMvc, while integration tests can use a real application context and a test database or Testcontainers.

---

# 97. Interview: How Do You Test JWT Authentication?

> I test missing, invalid and expired tokens, plus valid tokens with different authorities. For focused controller tests I can use Spring Security test support, while integration tests can verify the actual JWT filter and token validation flow.

---

# 98. Interview: How Do You Test Exception Handling?

> I trigger the expected application exception and verify that the global exception handler converts it into the correct HTTP status and standardized error response. I test both the status and important fields in the response body.

---

# 99. Interview: H2 vs Testcontainers?

> H2 is fast and convenient but can behave differently from the production database. If production uses MySQL or PostgreSQL and database compatibility is important, I prefer Testcontainers so integration tests run against the actual database technology.

---

# 100. Interview: Should Every Test Use a Real Database?

> No. Unit and focused controller tests should usually avoid external infrastructure because they need to be fast. I use real database integration tests where persistence, SQL, transactions or mappings need verification.

---

# 101. Interview: How Do You Test External APIs?

> I normally isolate external dependencies using mocks or tools such as WireMock for controlled integration tests. I cover success, timeout, 4xx/5xx responses, malformed responses and retry behavior without making the test suite depend on a live third-party service.

---

# 102. Interview: What Makes a Good REST API Test?

> A good test verifies observable behavior rather than implementation details. It should clearly define the request, expected response, status code and important side effects, and it should be independent and repeatable.

---

# 103. Final Mental Model

```text
                 REST API TESTING
                       |
       +---------------+---------------+
       |               |               |
      Unit          API/Controller   Integration
       |               |               |
     Mockito         MockMvc        Full Context
       |               |               |
       +---------------+---------------+
                       |
                 Test Database
                       |
                Testcontainers
                       |
              External Services
```

The practical strategy is:

```text
Fast tests
   ↓
Unit tests

API contract behavior
   ↓
MockMvc / focused API tests

Real application behavior
   ↓
Integration tests

Real infrastructure compatibility
   ↓
Testcontainers

Production confidence
   ↓
Smoke + regression + performance tests
```

> **REST API testing is not just checking whether an endpoint returns 200. A strong test strategy verifies the API contract, validation, security, error handling, persistence, external dependencies and important failure scenarios at the appropriate testing level.**

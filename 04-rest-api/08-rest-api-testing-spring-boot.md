# REST API — Testing with Spring Boot

This file covers how to test REST APIs in Spring Boot, from controller unit tests to full integration tests, security testing, database testing, and production-oriented testing strategies.

---

# 1. Why Test REST APIs?

REST API tests verify:

```text
Request handling
Validation
Authentication
Authorization
Business behavior
Database interaction
Error responses
HTTP status codes
JSON contracts
```

Good API tests give confidence that changes do not silently break existing clients.

---

# 2. Testing Pyramid

A typical backend test strategy contains:

```text
        E2E Tests
       /         \
 Integration Tests
     /             \
   Unit Tests
```

Unit tests are usually:

```text
Fast
Focused
Numerous
```

Integration tests are:

```text
Slower
Broader
Closer to production behavior
```

End-to-end tests are usually fewer because they are more expensive.

---

# 3. Unit Test

A unit test tests one component in isolation.

Example:

```text
ProductService
    ↓
Mock ProductRepository
```

The test verifies service behavior without requiring a real database.

---

# 4. Integration Test

An integration test verifies that multiple components work together.

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

It provides higher confidence than a pure unit test for integration behavior.

---

# 5. Spring Boot Testing Support

Spring Boot provides testing support through:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

It commonly brings tools such as:

```text
JUnit
Mockito
Spring Test
AssertJ
JSONPath
```

The exact versions are managed by the Spring Boot dependency management.

---

# 6. JUnit 5

Modern Spring Boot projects commonly use JUnit 5.

Example:

```java
@Test
void shouldCalculateTotal() {

    int result = service.calculateTotal();

    assertEquals(100, result);
}
```

---

# 7. Arrange — Act — Assert

A clean test often follows:

```text
Arrange
Act
Assert
```

Example:

```java
// Arrange
Product product = new Product();
product.setPrice(BigDecimal.valueOf(100));

// Act
BigDecimal result = service.calculatePrice(product);

// Assert
assertEquals(
    BigDecimal.valueOf(100),
    result
);
```

This makes tests easier to read.

---

# 8. Mockito

Mockito is commonly used to mock dependencies.

Example:

```java
@Mock
private ProductRepository repository;
```

Then:

```java
when(repository.findById(100L))
    .thenReturn(Optional.of(product));
```

The service can be tested without a real database.

---

# 9. @InjectMocks

Example:

```java
@Mock
ProductRepository repository;

@InjectMocks
ProductService service;
```

Mockito injects the mock dependency into the service under test.

---

# 10. Service Unit Test

Example:

```java
@Test
void shouldReturnProduct() {

    Product product = new Product();
    product.setId(100L);

    when(repository.findById(100L))
        .thenReturn(Optional.of(product));

    ProductResponse result =
        service.getProduct(100L);

    assertEquals(100L, result.id());

    verify(repository)
        .findById(100L);
}
```

The test verifies both behavior and repository interaction.

---

# 11. Testing Not Found

Example:

```java
@Test
void shouldThrowWhenProductDoesNotExist() {

    when(repository.findById(100L))
        .thenReturn(Optional.empty());

    assertThrows(
        ResourceNotFoundException.class,
        () -> service.getProduct(100L)
    );
}
```

Negative paths are just as important as successful paths.

---

# 12. Verify Interactions

Mockito can verify calls:

```java
verify(repository)
    .findById(100L);
```

You can also verify that something did not happen:

```java
verify(repository, never())
    .save(any());
```

Avoid verifying every internal implementation detail. Focus on meaningful behavior.

---

# 13. Mock vs Spy

Mock:

```text
Fully controlled test double
```

Spy:

```text
Wraps a real object
```

Use mocks for external dependencies.

Use spies carefully because they can make tests tightly coupled to implementation details.

---

# 14. Controller Testing

For MVC controllers, Spring provides:

```text
MockMvc
```

A controller test can verify:

```text
HTTP method
URL
Request body
Validation
Status code
Response JSON
```

without starting a real HTTP server.

---

# 15. @WebMvcTest

Example:

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {
}
```

This loads the MVC-related application context rather than the entire application.

It is useful for focused controller tests.

---

# 16. MockMvc Example

Example:

```java
mockMvc.perform(
        get("/api/products/100")
    )
    .andExpect(status().isOk());
```

---

# 17. Testing JSON Response

Example:

```java
mockMvc.perform(
        get("/api/products/100")
    )
    .andExpect(status().isOk())
    .andExpect(
        jsonPath("$.id")
            .value(100)
    )
    .andExpect(
        jsonPath("$.name")
            .value("Laptop")
    );
```

This verifies the external API contract.

---

# 18. Mocking the Service

A controller test should normally mock the service.

Example:

```java
@MockBean
private ProductService productService;
```

Depending on the Spring Boot version, newer Spring testing setups may use Spring Framework's `@MockitoBean` instead.

The important idea is that the controller test should not require the real service and database.

---

# 19. POST Request Test

Example:

```java
mockMvc.perform(
        post("/api/products")
            .contentType(
                MediaType.APPLICATION_JSON
            )
            .content("""
                {
                  "name": "Laptop",
                  "price": 75000
                }
            """)
    )
    .andExpect(status().isCreated());
```

---

# 20. Testing Request Validation

Suppose:

```java
public record ProductRequest(
    @NotBlank String name,
    @Positive BigDecimal price
) {}
```

Test:

```json
{
  "name": "",
  "price": -10
}
```

Expected:

```http
400 Bad Request
```

The test should also verify the validation error structure if it is part of the API contract.

---

# 21. Testing 404

Example:

```java
when(productService.getProduct(100L))
    .thenThrow(
        new ResourceNotFoundException(
            "Product not found"
        )
    );
```

Then:

```java
mockMvc.perform(
        get("/api/products/100")
    )
    .andExpect(status().isNotFound());
```

This verifies the exception-to-HTTP mapping.

---

# 22. Testing 401

A protected endpoint without authentication should normally return:

```http
401 Unauthorized
```

Example:

```java
mockMvc.perform(
        get("/api/orders/100")
    )
    .andExpect(status().isUnauthorized());
```

The exact result depends on the application's Spring Security configuration.

---

# 23. Testing 403

An authenticated user without sufficient permission should receive:

```http
403 Forbidden
```

Example concept:

```text
USER
 ↓
DELETE /api/products/100
 ↓
403 Forbidden
```

This is an important authorization test.

---

# 24. Spring Security Test

Spring Security provides testing support.

Common utilities include:

```text
@WithMockUser
SecurityMockMvcRequestPostProcessors
```

Example:

```java
@WithMockUser(
    username = "admin",
    roles = "ADMIN"
)
@Test
void adminCanDeleteProduct() {
    ...
}
```

---

# 25. Testing Roles

Example:

```java
@WithMockUser(
    username = "user",
    roles = "USER"
)
@Test
void normalUserCannotDeleteProduct() {

    mockMvc.perform(
        delete("/api/products/100")
    )
    .andExpect(status().isForbidden());
}
```

This verifies authorization rules.

---

# 26. Testing JWT APIs

For JWT-protected APIs, tests should verify:

```text
Missing token
Invalid token
Expired token
Valid token
Wrong audience/issuer where applicable
Insufficient role
```

Spring Security test support can simulate authenticated requests without requiring a real external identity provider for every test.

---

# 27. Integration Testing with @SpringBootTest

Example:

```java
@SpringBootTest
class ProductIntegrationTest {
}
```

This loads a broad Spring application context.

It is useful when you need to test interactions across multiple application layers.

---

# 28. @SpringBootTest Web Environment

You can choose the web environment.

Example:

```java
@SpringBootTest(
    webEnvironment =
        SpringBootTest.WebEnvironment.RANDOM_PORT
)
```

This starts the application on a random available port.

---

# 29. TestRestTemplate

For integration tests, Spring Boot provides:

```text
TestRestTemplate
```

Example:

```java
ResponseEntity<ProductResponse> response =
    restTemplate.getForEntity(
        "/api/products/100",
        ProductResponse.class
    );
```

This exercises a more realistic HTTP path than directly calling the controller.

---

# 30. WebTestClient

`WebTestClient` can test reactive applications and can also be useful for certain HTTP-level testing scenarios.

Example:

```java
webTestClient
    .get()
    .uri("/api/products/100")
    .exchange()
    .expectStatus()
    .isOk();
```

Choose the test client appropriate to the application's web stack.

---

# 31. Testing the Database

A realistic integration test may use:

```text
Spring Boot
   ↓
JPA
   ↓
Test database
```

Options include:

```text
H2
Testcontainers
Dedicated test database
```

---

# 32. H2

H2 is an in-memory database.

Advantages:

```text
Fast
Easy setup
Good for simple tests
```

Limitations:

```text
May behave differently from MySQL/PostgreSQL
SQL dialect differences
Index behavior differences
Transaction behavior differences
```

For database-heavy applications, do not assume H2 proves production database compatibility.

---

# 33. Testcontainers

Testcontainers runs real infrastructure in containers.

Example:

```text
Test
 ↓
Docker
 ↓
MySQL container
```

This is often a stronger choice when application behavior depends heavily on a specific database.

---

# 34. Testcontainers Example

Conceptually:

```java
@Testcontainers
@SpringBootTest
class ProductRepositoryTest {

    @Container
    static MySQLContainer<?> mysql =
        new MySQLContainer<>("mysql:8");
}
```

The exact configuration depends on the project and Testcontainers version.

---

# 35. Repository Test

Spring Data JPA provides:

```text
@DataJpaTest
```

Example:

```java
@DataJpaTest
class ProductRepositoryTest {
}
```

This focuses on JPA repository behavior.

---

# 36. Repository Test Example

```java
@Test
void shouldFindProductByName() {

    Product product = new Product();
    product.setName("Laptop");

    repository.save(product);

    Optional<Product> result =
        repository.findByName("Laptop");

    assertTrue(result.isPresent());
}
```

---

# 37. Transaction Rollback in Tests

Many Spring test configurations run tests inside transactions and roll back changes after the test.

This helps keep tests isolated.

However, always understand the actual test configuration rather than assuming every integration test automatically rolls back.

---

# 38. Testing Pagination

For:

```http
GET /api/products?page=0&size=20
```

verify:

```text
Page number
Page size
Total elements
Total pages
Returned items
```

Also test:

```text
Empty page
Last page
Invalid page
Maximum page size
```

---

# 39. Testing Filtering

Example:

```http
GET /api/products?category=phones
```

Verify:

```text
Only phones returned
Unknown category behavior
Case handling
Invalid filters
Combination of filters
```

---

# 40. Testing Sorting

Example:

```http
GET /api/products?sort=price,asc
```

Verify:

```text
Ascending order
Descending order
Multiple sort fields
Invalid sort field
```

Whitelist sort fields in the application.

---

# 41. Testing Exception Handling

If using:

```java
@RestControllerAdvice
```

test:

```text
ResourceNotFoundException
ValidationException
AccessDeniedException
Unexpected exception
```

Verify:

```text
HTTP status
Error code
Message
Field errors
Correlation ID
```

---

# 42. Global Exception Handler Test

Example:

```java
mockMvc.perform(
        get("/api/products/999")
    )
    .andExpect(status().isNotFound())
    .andExpect(
        jsonPath("$.message")
            .value("Product not found")
    );
```

This ensures the public error contract stays stable.

---

# 43. Testing Service Transactions

For transactional services, integration tests should verify important transaction behavior.

Example:

```text
Create order
 ↓
Save order
 ↓
Save order items
 ↓
Failure
 ↓
Rollback
```

The test should confirm that partial data is not committed when the operation is expected to be atomic.

---

# 44. Testing Concurrency

Some APIs require concurrency tests.

Examples:

```text
Inventory reservation
Order creation
Payment processing
Unique resource creation
```

You may need multiple threads or concurrent HTTP requests.

The goal is to verify:

```text
No overselling
No duplicate processing
Correct locking/versioning
```

---

# 45. Testing Idempotency

For:

```http
POST /api/orders
Idempotency-Key: abc123
```

send the same request twice.

Expected:

```text
First request → creates order
Second request → returns same logical result
```

The test should verify that only one order is created.

---

# 46. Testing Rate Limiting

Example:

```text
Limit = 5 requests/minute
```

Test:

```text
Requests 1–5 → allowed
Request 6 → 429
```

Also verify that the limit works correctly across multiple application instances when Redis is used as the distributed state store.

---

# 47. Testing Caching

For a cached method:

```text
First call
 ↓
Database
 ↓
Cache

Second call
 ↓
Cache
```

Tests should verify that the second call does not unnecessarily hit the database.

Be careful not to make unit tests overly dependent on the cache implementation if the behavior can be tested at a higher level.

---

# 48. Testing External APIs

External dependencies should usually be isolated in unit tests.

For integration tests, use:

```text
WireMock
MockWebServer
Testcontainers
Dedicated sandbox environment
```

depending on the dependency.

---

# 49. Contract Testing External APIs

If your service depends on another service, contract testing can verify that the expected request/response contract remains compatible.

This reduces surprises caused by independent service deployments.

---

# 50. Test Data

Good test data should be:

```text
Deterministic
Readable
Minimal
Independent
```

Avoid giant fixtures when a small setup is enough.

---

# 51. Test Builders

For complex objects, builders can make tests readable.

Example:

```java
Product product =
    ProductTestBuilder.aProduct()
        .withName("Laptop")
        .withPrice(75000)
        .build();
```

This reduces repetitive setup code.

---

# 52. Test Naming

Prefer names that explain behavior.

Good:

```text
shouldReturn404WhenProductDoesNotExist
shouldRejectRequestWhenUserIsNotAdmin
shouldCreateOrderOnlyOnceForSameIdempotencyKey
```

Poor:

```text
test1
testProduct
checkMethod
```

---

# 53. One Behavior Per Test

A test should generally focus on one meaningful behavior.

Good:

```text
shouldRejectNegativePrice
```

instead of one giant test that checks:

```text
validation
database
security
caching
pagination
```

all at once.

---

# 54. Avoid Brittle Tests

Avoid asserting unnecessary implementation details.

For example, if the API contract only guarantees:

```text
status = 200
```

don't require an exact internal method call unless that interaction itself is important.

Tests should survive harmless refactoring.

---

# 55. Test the API Contract

Good API tests verify:

```text
HTTP method
Path
Authentication
Request schema
Validation
Status code
Response schema
Business behavior
```

This is more valuable than testing getters and setters mechanically.

---

# 56. Mocking Too Much

If every dependency is mocked:

```text
Controller mocked
Service mocked
Repository mocked
Mapper mocked
Database mocked
```

the test may pass while the actual application is broken.

Balance:

```text
Unit tests → mocks
Integration tests → real component interactions
```

---

# 57. Test Coverage

Coverage can show which code executed during tests.

Examples:

```text
Line coverage
Branch coverage
Method coverage
```

But:

```text
90% coverage
```

does not automatically mean:

```text
90% correctness
```

Quality of assertions matters.

---

# 58. Mutation Testing

Mutation testing changes code intentionally and checks whether tests detect the changes.

If a mutation survives:

```text
Tests may be too weak
```

Tools such as PIT can be used in Java projects.

This is optional but useful for evaluating test quality.

---

# 59. Testing in CI/CD

Typical pipeline:

```text
Developer push
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
```

A failed test should normally prevent unsafe deployment.

---

# 60. Test Profile

Use a dedicated configuration:

```text
application-test.properties
```

or:

```text
application-test.yml
```

Activate it with:

```java
@ActiveProfiles("test")
```

Do not point tests at production databases.

---

# 61. Test Secrets

Never commit:

```text
Production passwords
Production JWT keys
Cloud credentials
Real API secrets
```

Use:

```text
Test-specific values
Environment variables
CI secret stores
Temporary credentials
```

---

# 62. API Test Layers

A practical Spring Boot project might use:

```text
Controller tests
      ↓
Service unit tests
      ↓
Repository tests
      ↓
Integration tests
      ↓
End-to-end tests
```

Each layer catches a different category of failure.

---

# 63. Example Project Test Structure

```text
src/test/java/
└── com/example/ecommerce/
    ├── controller/
    │   ├── ProductControllerTest.java
    │   └── OrderControllerTest.java
    ├── service/
    │   ├── ProductServiceTest.java
    │   └── OrderServiceTest.java
    ├── repository/
    │   └── ProductRepositoryTest.java
    └── integration/
        └── ProductApiIntegrationTest.java
```

Keep the test structure understandable.

---

# 64. Ecommerce API Test Matrix

For:

```http
POST /api/orders
```

test at least:

```text
Valid order
Unauthenticated request
Unauthorized user
Empty cart
Invalid product
Insufficient inventory
Invalid quantity
Duplicate idempotency key
Database failure
```

---

# 65. Ecommerce Product API

For:

```http
GET /api/products/{id}
```

test:

```text
Existing product → 200
Missing product → 404
Invalid ID → 400 where applicable
Cached response
Database fallback
```

---

# 66. Ecommerce Authorization

For:

```http
DELETE /api/products/{id}
```

test:

```text
No token → 401
USER → 403
ADMIN → 204
Product missing → 404
```

This gives strong confidence in both authentication and authorization.

---

# 67. Test Sequence

A useful development workflow:

```text
1. Write unit test
2. Implement logic
3. Run unit tests
4. Add controller test
5. Add repository/integration test
6. Run complete test suite
7. Commit
```

Tests should be part of normal development rather than something added only before release.

---

# 68. Running Tests

Maven:

```bash
./mvnw test
```

or:

```bash
mvn test
```

Build with tests:

```bash
./mvnw clean verify
```

---

# 69. Test Reports

Maven Surefire commonly generates reports under:

```text
target/surefire-reports/
```

These can help investigate failed tests in local and CI environments.

---

# 70. Debugging a Failed API Test

Check:

```text
Request
Response status
Response body
Application logs
Mock configuration
Security configuration
Database state
Test profile
```

For integration tests also check:

```text
Container status
Database connection
External dependency
Port configuration
```

---

# 71. REST API Test Checklist

```text
□ Happy path
□ Validation failures
□ 400
□ 401
□ 403
□ 404
□ 409
□ 429
□ 500/expected failure mapping
□ Authentication
□ Authorization
□ Pagination
□ Filtering
□ Sorting
□ Error response schema
□ Database behavior
□ Transactions
□ Caching
□ Idempotency
□ Concurrency where needed
□ External dependency failures
□ Timeouts
□ Integration behavior
```

---

# 72. Interview: Unit Test vs Integration Test

> A unit test isolates one component and usually mocks its dependencies, so it is fast and focused. An integration test verifies that multiple components work together, such as a controller, service, repository and database. I use both because they catch different classes of problems.

---

# 73. Interview: How Do You Test a Spring Boot REST API?

> I test the controller layer with MockMvc for request, validation, status and response behavior. I unit-test services with Mockito, test repositories with `@DataJpaTest`, and use `@SpringBootTest` or Testcontainers for broader integration tests. Security, error handling and important business scenarios are included in the test suite.

---

# 74. Interview: How Do You Test Secured APIs?

> I test missing authentication, invalid authentication and insufficient permissions separately. With Spring Security test support I can simulate authenticated users with different roles and verify that protected endpoints return the expected 401 or 403 responses.

---

# 75. Interview: Why Use Testcontainers?

> Testcontainers lets integration tests run against real infrastructure such as MySQL or Redis in containers. It reduces differences between the test environment and production compared with relying only on an in-memory replacement like H2.

---

# 76. Interview: MockMvc vs TestRestTemplate?

> MockMvc is useful for focused MVC tests without requiring a real server. TestRestTemplate is useful in a Spring Boot integration test where I want to exercise the application over HTTP. I choose based on the test scope rather than treating one as universally better.

---

# 77. Interview: How Do You Test Transactions?

> I create an integration scenario where a multi-step business operation fails partway through and verify that the expected database changes are rolled back. This is more reliable than only unit-testing the presence of `@Transactional`.

---

# 78. Interview: How Do You Test Idempotency?

> I send the same request twice with the same idempotency key and verify that only one logical operation is created and both requests receive the appropriate result. I also test concurrent duplicate requests because retries can happen at the same time.

---

# 79. Interview: How Much Test Coverage Is Enough?

> I don't target a percentage blindly. I prioritize critical business logic, security, error paths and integration boundaries. Coverage is useful as a signal, but meaningful assertions and representative scenarios matter more than achieving a specific number.

---

# 80. Interview: How Do You Test an Ecommerce Order API?

> I cover successful order creation, authentication and authorization, empty carts, invalid products, inventory failures, transaction rollback and duplicate requests. I also test the HTTP contract and use integration tests for the database behavior because order creation usually involves multiple persistent operations.

---

# 81. Final Mental Model

```text
                 REST API
                    |
          +---------+---------+
          |                   |
      Unit Tests        Integration Tests
          |                   |
      Mockito/JUnit      Spring Boot
                              |
                    +---------+---------+
                    |         |         |
                 HTTP       JPA       Database
                    |
                 Security
```

The goal is not simply:

```text
"All tests pass"
```

The goal is:

```text
The API contract is protected
The business rules are verified
Security behavior is tested
Database behavior is trusted
Failures are handled correctly
```

> **A strong Spring Boot API test strategy combines fast unit tests with focused controller tests and realistic integration tests. Test the behavior clients depend on, not just the implementation details inside your classes.**

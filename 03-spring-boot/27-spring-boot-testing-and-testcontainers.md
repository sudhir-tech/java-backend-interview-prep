# Spring Boot — Testing and Testcontainers

Testing is a core part of building reliable Spring Boot applications.

A practical backend test strategy should cover:

```text
Unit tests
Integration tests
Controller/API tests
Repository tests
Security tests
Database tests
External-service tests
End-to-end tests
```

The goal is not maximum test count.

The goal is confidence in important behavior.

---

# 1. Test Pyramid

A common model:

```text
             E2E
            /   \
       Integration
          /       \
       Unit Tests
```

Usually:

```text
Many unit tests
Some integration tests
Fewer expensive E2E tests
```

Unit tests are fast.

Integration and E2E tests provide broader confidence but are more expensive.

---

# 2. Spring Boot Test Starter

Spring Boot commonly provides:

```xml
<dependency>
    <groupId>
        org.springframework.boot
    </groupId>

    <artifactId>
        spring-boot-starter-test
    </artifactId>

    <scope>test</scope>
</dependency>
```

It brings together common testing libraries used in Spring applications.

---

# 3. JUnit

JUnit is commonly used for Java tests.

Example:

```java
class CalculatorTest {

    @Test
    void shouldAddNumbers() {

        int result = 2 + 3;

        assertEquals(5, result);
    }
}
```

---

# 4. Arrange Act Assert

A clean test often follows:

```text
Arrange
   ↓
Act
   ↓
Assert
```

Example:

```java
@Test
void shouldCalculateTotal() {

    // Arrange
    Order order = createOrder();

    // Act
    BigDecimal total =
        service.calculateTotal(order);

    // Assert
    assertEquals(
        new BigDecimal("100"),
        total
    );
}
```

---

# 5. Unit Test

A unit test verifies one unit of behavior in isolation.

Example:

```text
OrderService
    ↓
Mock PaymentGateway
Mock OrderRepository
```

The test focuses on:

```text
OrderService business logic
```

rather than the database or network.

---

# 6. Mockito

Mockito can create test doubles.

Example:

```java
@Mock
private ProductRepository repository;
```

Then:

```java
when(repository.findById(1L))
    .thenReturn(Optional.of(product));
```

---

# 7. @InjectMocks

Example:

```java
@Mock
private ProductRepository repository;

@InjectMocks
private ProductService service;
```

Mockito creates the service and injects the mock dependency.

---

# 8. MockitoExtension

For a pure Mockito/JUnit test:

```java
@ExtendWith(MockitoExtension.class)
class ProductServiceTest {
}
```

This initializes Mockito annotations.

---

# 9. When to Mock

Mock dependencies when testing behavior in isolation.

Good candidates:

```text
External API
Repository
Message publisher
Payment gateway
Clock
File system
```

Example:

```text
OrderService
   |
   +-- Mock PaymentGateway
   +-- Mock OrderRepository
```

---

# 10. Don't Mock Everything

If you mock:

```text
Controller
Service
Repository
Database
```

you may end up testing your mocks rather than your application.

Use real integration tests for behavior that depends on:

```text
Spring configuration
JPA
Database
Security
Serialization
Transactions
```

---

# 11. Mockito when()

Example:

```java
when(repository.findBySku("ABC"))
    .thenReturn(
        Optional.of(product)
    );
```

Then:

```java
Product result =
    service.findBySku("ABC");
```

---

# 12. Mockito verify()

Verify an interaction:

```java
verify(repository)
    .findBySku("ABC");
```

Useful when the interaction itself is part of the behavior.

Don't overuse interaction verification when asserting the final result is more meaningful.

---

# 13. verifyNoInteractions()

Example:

```java
verifyNoInteractions(
    paymentGateway
);
```

Useful when a certain branch should not call a dependency.

---

# 14. ArgumentCaptor

Capture an argument passed to a mock.

Example:

```java
ArgumentCaptor<Order> captor =
    ArgumentCaptor.forClass(Order.class);

verify(repository).save(captor.capture());

Order saved =
    captor.getValue();
```

Useful when you want to verify what was sent to a dependency.

---

# 15. Exception Testing

Example:

```java
assertThrows(
    ProductNotFoundException.class,
    () -> service.getProduct(999L)
);
```

Test:

```text
Exception type
Business condition
Important message/details
```

Don't test implementation details unnecessarily.

---

# 16. Service Unit Test Example

```java
@ExtendWith(MockitoExtension.class)
class ProductServiceTest {

    @Mock
    private ProductRepository repository;

    @InjectMocks
    private ProductService service;

    @Test
    void shouldReturnProduct() {

        Product product =
            new Product(1L, "Phone");

        when(repository.findById(1L))
            .thenReturn(
                Optional.of(product)
            );

        ProductResponse response =
            service.getProduct(1L);

        assertEquals(
            "Phone",
            response.name()
        );
    }
}
```

---

# 17. Testing Missing Product

```java
@Test
void shouldThrowWhenProductMissing() {

    when(repository.findById(999L))
        .thenReturn(Optional.empty());

    assertThrows(
        ProductNotFoundException.class,
        () -> service.getProduct(999L)
    );
}
```

This tests an important business branch.

---

# 18. Testing Validation

Suppose:

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

Test invalid inputs through the API/controller integration layer.

Examples:

```text
Blank name
Null price
Negative price
Missing required field
```

---

# 19. Controller Tests

For MVC controllers, `MockMvc` can test HTTP behavior without starting a full external server.

Example:

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {
}
```

---

# 20. @WebMvcTest

`@WebMvcTest` is focused on MVC components.

It is useful for testing:

```text
Controller mapping
JSON serialization
Validation
HTTP status
Controller advice
Security behavior
```

Usually service dependencies are mocked.

---

# 21. MockMvc Example

```java
mockMvc.perform(
        get("/api/products/1")
    )
    .andExpect(
        status().isOk()
    )
    .andExpect(
        jsonPath("$.name")
            .value("Phone")
    );
```

This verifies the HTTP contract.

---

# 22. Controller Error Test

Example:

```java
mockMvc.perform(
        get("/api/products/999")
    )
    .andExpect(
        status().isNotFound()
    );
```

You can also assert:

```text
Error code
Message
Timestamp
Path
```

depending on the API error contract.

---

# 23. @SpringBootTest

`@SpringBootTest` loads the Spring Boot application context.

Example:

```java
@SpringBootTest
class EcommerceApplicationTest {
}
```

Use it when you need broader application integration.

It is heavier than a focused slice test.

---

# 24. SpringBootTest Web Environment

Possible modes include:

```java
@SpringBootTest(
    webEnvironment =
        SpringBootTest.WebEnvironment.MOCK
)
```

or:

```java
@SpringBootTest(
    webEnvironment =
        SpringBootTest.WebEnvironment.RANDOM_PORT
)
```

`RANDOM_PORT` can start an embedded server on a random port for HTTP-level integration testing.

---

# 25. Integration Test

An integration test verifies that multiple real components work together.

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

This can catch problems that unit tests with mocks cannot.

---

# 26. Why Integration Tests Matter

A unit test may pass:

```text
repository.findBySku()
```

because the repository is mocked.

But the real database query might fail due to:

```text
Wrong column
Wrong SQL
Missing index
Constraint
Mapping issue
Database-specific behavior
```

Integration tests expose these problems.

---

# 27. @DataJpaTest

`@DataJpaTest` is useful for testing JPA repositories.

Example:

```java
@DataJpaTest
class ProductRepositoryTest {
}
```

It focuses on:

```text
JPA
Repositories
Entity mappings
Database interaction
```

---

# 28. Repository Test

Example:

```java
@DataJpaTest
class ProductRepositoryTest {

    @Autowired
    ProductRepository repository;

    @Test
    void shouldFindBySku() {

        Product product =
            new Product("ABC", "Phone");

        repository.save(product);

        Optional<Product> result =
            repository.findBySku("ABC");

        assertTrue(result.isPresent());
    }
}
```

---

# 29. Why Repository Tests?

They verify:

```text
Entity mapping
Query methods
JPQL
Native SQL
Constraints
Relationships
```

These are difficult to validate with mocked repositories.

---

# 30. H2 vs Real Database

H2 can be useful for simple tests.

But it may behave differently from:

```text
MySQL
PostgreSQL
Oracle
```

Differences can involve:

```text
SQL syntax
Data types
Indexes
Constraints
Transactions
Functions
```

For database-heavy applications, a real database test is often more reliable.

---

# 31. Testcontainers

Testcontainers allows tests to run real dependencies in containers.

Example:

```text
JUnit
  ↓
Testcontainers
  ↓
MySQL Container
  ↓
Spring Boot
  ↓
Repository
```

This gives realistic database behavior without requiring developers to manually install the database.

---

# 32. Testcontainers Dependency

A project commonly needs:

```xml
<dependency>
    <groupId>
        org.testcontainers
    </groupId>

    <artifactId>
        testcontainers
    </artifactId>

    <scope>test</scope>
</dependency>
```

For database-specific integrations, use the appropriate Testcontainers module for the database and the project's current Testcontainers version.

---

# 33. MySQL Container

Conceptual example:

```java
@Testcontainers
class ProductRepositoryTest {

    @Container
    static MySQLContainer<?> mysql =
        new MySQLContainer<>(
            "mysql:8.4"
        );
}
```

Use a database version compatible with your application and CI environment.

---

# 34. Dynamic Database Properties

The test needs to tell Spring:

```text
JDBC URL
Username
Password
```

Modern Spring Boot/Testcontainers setups can use service connection support where supported.

Example concept:

```java
@Testcontainers
@SpringBootTest
class ProductIntegrationTest {

    @Container
    @ServiceConnection
    static MySQLContainer<?> mysql =
        new MySQLContainer<>(
            "mysql:8.4"
        );
}
```

The exact annotation support depends on the Spring Boot and Testcontainers versions in the project.

---

# 35. @ServiceConnection

Spring Boot can automatically create connection details from supported Testcontainers.

Conceptually:

```text
MySQLContainer
     ↓
@ServiceConnection
     ↓
Spring Boot
     ↓
DataSource
```

This reduces manual property wiring.

---

# 36. Testcontainers Lifecycle

A container can be:

```text
Started
↓
Used by tests
↓
Stopped
```

With:

```java
@Testcontainers
```

and:

```java
@Container
```

the lifecycle can be managed automatically.

---

# 37. Static vs Instance Container

Static:

```java
@Container
static MySQLContainer<?> mysql;
```

can be reused across test methods in the test class.

Instance:

```java
@Container
MySQLContainer<?> mysql;
```

has a different lifecycle.

Choose based on:

```text
Isolation
Startup cost
Test suite size
Parallelism
```

---

# 38. Testcontainers and CI

Testcontainers can run in CI environments if the CI runner provides the required container runtime.

Typical flow:

```text
CI
 ↓
Build
 ↓
Test
 ↓
Start MySQL container
 ↓
Integration tests
 ↓
Stop container
```

This reduces differences between developer and CI environments.

---

# 39. Integration Test With MySQL

A useful ecommerce test:

```text
Start MySQL
↓
Start Spring context
↓
Run migration
↓
Insert product
↓
Call repository/service
↓
Verify result
```

This tests real persistence behavior.

---

# 40. Testing Transactions

A transaction test can verify:

```text
Operation A succeeds
Operation B fails
↓
Transaction rolls back
```

Example business scenario:

```text
Create Order
+
Decrease Stock
```

If the required operation fails:

```text
Order not persisted
Stock update rolled back
```

provided both operations are inside the same appropriate local transaction.

---

# 41. Testing Security

Security tests should verify:

```text
Unauthenticated request
Authenticated USER
Authenticated ADMIN
Forbidden access
Expired token
Invalid token
```

Example:

```java
mockMvc.perform(
    get("/api/admin/products")
)
.andExpect(
    status().isUnauthorized()
);
```

Exact behavior depends on the configured security mechanism.

---

# 42. @WithMockUser

Spring Security test support can create an authenticated test user.

Example:

```java
@WithMockUser(
    username = "sudhir",
    roles = "USER"
)
@Test
void shouldAllowUser() {
    ...
}
```

Then test:

```text
USER access
ADMIN-only access
```

---

# 43. Security Test Example

```java
@WithMockUser(
    roles = "USER"
)
@Test
void userCannotDeleteProduct() {

    mockMvc.perform(
        delete("/api/products/1")
    )
    .andExpect(
        status().isForbidden()
    );
}
```

---

# 44. Testing JWT

For JWT-based APIs, tests should verify:

```text
Valid token
Expired token
Invalid signature
Missing token
Insufficient authority
```

Spring Security provides testing support for simulating authenticated requests depending on the security setup.

---

# 45. Test API Validation

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

This verifies the actual HTTP validation behavior.

---

# 46. Testing JSON

Use:

```java
jsonPath("$.name")
```

or deserialize the response into a DTO.

Example:

```java
.andExpect(
    jsonPath("$.price")
        .value(999)
);
```

---

# 47. Testing Exception Handler

Test that:

```text
Exception
↓
@RestControllerAdvice
↓
Expected status
↓
Expected error response
```

This ensures error handling is part of the API contract.

---

# 48. Testing External APIs

Suppose:

```text
Order Service
    ↓
Payment API
```

Don't call the real payment provider in normal automated tests.

Use:

```text
Mock server
Stub server
WireMock
MockWebServer
```

or another suitable test double.

---

# 49. WireMock Concept

Architecture:

```text
Order Service
     |
     v
WireMock
     |
Fake Payment API
```

You can simulate:

```text
200
400
401
404
500
503
Timeout
Slow response
```

---

# 50. Testing Timeout

Configure the mock server to delay the response.

Then verify:

```text
Timeout
↓
Circuit breaker/retry behavior
↓
Expected application response
```

This is much better than waiting for a real external service.

---

# 51. Testing Retry

Simulate:

```text
Attempt 1 → 503
Attempt 2 → 503
Attempt 3 → 200
```

Then verify:

```text
Request eventually succeeds
```

Also verify that the number of retries is bounded.

---

# 52. Testing Circuit Breaker

Simulate repeated failures:

```text
Failure
Failure
Failure
Failure
```

Then verify:

```text
Circuit opens
↓
Calls fail fast
```

Then simulate recovery:

```text
HALF_OPEN
↓
Success
↓
CLOSED
```

---

# 53. Testing Kafka

For Kafka-based applications, integration tests can use a real Kafka container.

Conceptually:

```text
Test
 ↓
Kafka Container
 ↓
Producer
 ↓
Topic
 ↓
Consumer
 ↓
Database
```

This verifies actual serialization, topics, partitions, and consumer behavior more realistically than mocks.

---

# 54. Testing Kafka Consumer Idempotency

Send:

```text
Event ID = 123
Event ID = 123
```

Verify:

```text
Business operation executed once
```

This is an important real-world test.

---

# 55. Testing Outbox

Test:

```text
Create order
+
Create outbox event
```

inside the same transaction.

Then simulate:

```text
Transaction rollback
```

and verify:

```text
Order absent
Outbox event absent
```

---

# 56. Testing Outbox Publisher

Insert an outbox record:

```text
PENDING
```

Run publisher:

```text
PENDING
↓
Publish
↓
PROCESSED
```

If publishing fails:

```text
PENDING
```

remains available for retry.

---

# 57. Testing Scheduled Jobs

Avoid tests that depend on waiting for a real clock.

Instead:

```text
Call job method directly
```

or inject a controllable clock/scheduler where appropriate.

Verify:

```text
Expected records processed
Failures handled
Duplicate execution safe
```

---

# 58. Testing Time

Production code:

```java
Instant.now()
```

can make tests difficult.

Prefer injecting:

```java
Clock
```

Example:

```java
private final Clock clock;
```

Then tests can use:

```java
Clock.fixed(...)
```

This makes time-dependent behavior deterministic.

---

# 59. Test Naming

Good:

```text
shouldCreateOrderWhenStockIsAvailable
shouldRejectOrderWhenStockIsInsufficient
shouldReturn404WhenProductDoesNotExist
shouldRejectNonAdminDeleteRequest
```

Bad:

```text
test1
testOrder
testSomething
```

Names should describe behavior.

---

# 60. One Behavior Per Test

Prefer:

```text
shouldRejectNegativePrice
```

rather than one test containing:

```text
20 unrelated assertions
```

Focused tests are easier to diagnose.

---

# 61. Avoid Test Interdependence

Bad:

```text
testA creates database state
↓
testB expects testA to have run
```

Tests should be independently executable.

---

# 62. Test Data Builders

Complex test objects can become noisy.

Instead of:

```java
new Order(
    1L,
    customer,
    items,
    address,
    payment,
    ...
);
```

use a builder/factory:

```java
Order order =
    OrderTestData.validOrder();
```

This improves readability.

---

# 63. Test Fixtures

A fixture is reusable test data.

Example:

```java
class ProductFixtures {

    static Product phone() {
        return new Product(
            1L,
            "Phone",
            new BigDecimal("999")
        );
    }
}
```

Keep fixtures understandable and avoid hiding important test setup.

---

# 64. Parameterized Tests

Useful when the same behavior applies to many inputs.

Example:

```java
@ParameterizedTest
@ValueSource(
    ints = { -1, -10, -100 }
)
void shouldRejectNegativeValues(
        int value) {

    ...
}
```

This avoids repetitive test methods.

---

# 65. Boundary Testing

Test boundaries:

```text
0
1
Maximum
Maximum + 1
Empty
Null
Very large input
```

Example stock:

```text
stock = 0
stock = 1
```

These often reveal bugs.

---

# 66. Negative Testing

Don't test only:

```text
Valid request
```

Also test:

```text
Invalid request
Unauthorized
Forbidden
Missing resource
Duplicate resource
Conflict
Dependency failure
Database failure
```

---

# 67. Contract Testing

Verify that:

```text
Consumer expectation
=
Provider behavior
```

Useful for microservices.

Example:

```text
Order Service
      ↓
Payment API contract
      ↓
Payment Service
```

This helps detect breaking changes.

---

# 68. API Contract Test

Verify:

```text
Endpoint
Request
Response
Status code
Required fields
Error schema
```

Contract tests are especially useful when teams deploy independently.

---

# 69. Test Coverage

Coverage measures which code was executed.

Examples:

```text
Line coverage
Branch coverage
Method coverage
```

High coverage does not automatically mean high-quality tests.

A test can execute a line without actually verifying meaningful behavior.

---

# 70. What Good Tests Cover

Prioritize:

```text
Business rules
Failure paths
Security boundaries
Transactions
Concurrency-sensitive behavior
Persistence queries
API contracts
```

---

# 71. Mutation Testing

Mutation testing modifies code intentionally to see whether tests detect the change.

Example:

```text
price > 0
```

mutated to:

```text
price >= 0
```

If tests still pass, coverage may not be testing the business rule strongly enough.

---

# 72. Unit vs Integration Test

Interview answer:

> A unit test isolates one component and usually mocks dependencies, so it is fast and focused. An integration test verifies multiple real components together, such as Spring, JPA, and a real database. I use both because they catch different classes of problems.

---

# 73. Why Testcontainers?

Interview answer:

> Testcontainers lets me run real dependencies such as MySQL, PostgreSQL, or Kafka in containers during tests. It gives more realistic integration testing than mocking or relying entirely on an in-memory database.

---

# 74. Why Not Use H2 for Everything?

Interview answer:

> H2 is fast and useful for some tests, but it may behave differently from the production database. If database-specific SQL, indexes, constraints, or transaction behavior matter, I prefer testing against the same database technology using Testcontainers.

---

# 75. @WebMvcTest vs @SpringBootTest

Interview answer:

> `@WebMvcTest` focuses on the MVC layer and is useful for controller tests. `@SpringBootTest` loads the broader application context and is better when I need to test integration between multiple application components.

---

# 76. @DataJpaTest

Interview answer:

> `@DataJpaTest` focuses on JPA-related components. I use it for repository and entity mapping tests because it gives me a focused test environment for persistence behavior.

---

# 77. When Would You Use MockMvc?

> I use MockMvc when I want to test the HTTP/controller layer without requiring a separately running external server. It lets me verify routing, validation, serialization, security, and HTTP responses.

---

# 78. When Would You Use Testcontainers?

> I use Testcontainers when the behavior depends on a real infrastructure component, such as MySQL, PostgreSQL, Redis, or Kafka, and I want the test environment to behave more like production.

---

# 79. How Do You Test an External Payment API?

> I would normally use a mock server such as WireMock for automated tests. I can simulate success, validation errors, server errors, timeouts, and slow responses without calling the real payment provider.

---

# 80. How Do You Test Concurrent Inventory Updates?

> I would create a concurrency test with multiple simultaneous purchase attempts against a limited stock value. Then I would verify that the number of successful orders never exceeds available stock and that the database state remains valid.

---

# 81. How Do You Test a Transaction Rollback?

> I would execute a business operation where the first database change succeeds and a later required operation fails. Then I would verify that the entire local transaction was rolled back and no partial state remains.

---

# 82. How Do You Test Security?

> I test unauthenticated, authenticated, and unauthorized requests separately. I verify valid credentials, invalid or expired tokens, role-based permissions, and resource-level authorization.

---

# 83. How Do You Test a Kafka Consumer?

> I prefer an integration test with a real Kafka container for important consumer behavior. I publish an event, wait for processing, then verify the resulting business state. I also test duplicate events and failure/retry behavior.

---

# 84. How Do You Test Idempotency?

Example:

```text
Send request with key ABC
↓
Process successfully

Send same request with key ABC
↓
Do not create duplicate effect
```

Verify:

```text
One order
One payment
One business effect
```

---

# 85. How Do You Test a Circuit Breaker?

> I simulate repeated downstream failures and verify that the circuit opens and subsequent calls fail fast. Then I simulate recovery and verify that the circuit eventually returns to the closed state.

---

# 86. How Do You Test a Retry?

> I configure the mock dependency to fail temporarily and then succeed. I verify that the client retries only the configured number of times and eventually succeeds or returns the expected failure.

---

# 87. Testing Production Incidents

If a production bug occurs:

```text
Reproduce
↓
Write regression test
↓
Fix
↓
Run test
↓
Deploy
```

A good regression test ensures the same bug is less likely to return.

---

# 88. Test Database Cleanup

Tests should not depend on leftover state.

Options include:

```text
Transactional test rollback
Database cleanup
Fresh container
Test-specific schema
```

Choose based on the test type.

---

# 89. Testcontainers vs Shared Test Database

Shared database:

```text
Fast setup
But state can leak
Environment can differ
Parallel tests can conflict
```

Testcontainers:

```text
Isolated
Reproducible
Production-like
But slower/resource-intensive
```

For important integration tests, reproducibility is often worth the overhead.

---

# 90. Parallel Tests

Parallel execution can improve speed but introduces concerns:

```text
Shared database
Shared files
Shared ports
Static state
Testcontainers resources
```

Make tests safe for parallel execution before enabling it broadly.

---

# 91. Test Isolation

A good test should have:

```text
Independent setup
Independent data
Independent execution
Predictable cleanup
```

If a test only passes when run after another test, the suite has a design problem.

---

# 92. CI Testing Pipeline

A practical pipeline:

```text
Git push
   ↓
Compile
   ↓
Unit tests
   ↓
Static analysis
   ↓
Integration tests
   ↓
Testcontainers
   ↓
Package
   ↓
Security checks
   ↓
Deploy
```

---

# 93. Test Execution Strategy

Fast feedback:

```text
Every commit
→ Unit tests
```

Pull request:

```text
Unit
+
Integration
+
Static analysis
```

Deployment pipeline:

```text
Full integration
+
Security
+
Smoke tests
```

The exact strategy depends on project size and CI capacity.

---

# 94. Smoke Test

A smoke test verifies that the deployed application is basically functional.

Example:

```text
Application starts
↓
Health check
↓
Authentication
↓
GET product
↓
Basic API response
```

It should be fast.

---

# 95. Regression Test

A regression test ensures a previously fixed problem remains fixed.

Example:

```text
Bug:
Duplicate order

Fix:
Idempotency

Regression test:
Repeated request creates one order
```

---

# 96. Test Pyramid for Ecommerce

For your ecommerce backend:

```text
                    E2E
                 Checkout Flow
                      |
              Integration Tests
       MySQL + Security + APIs + Kafka
                      |
                  Unit Tests
      Services + Business Rules + Utilities
```

High-value scenarios:

```text
Create user
Login
Create product
Get product
Add to cart
Update cart
Place order
Insufficient stock
Duplicate order
Unauthorized access
Payment failure
Transaction rollback
```

---

# 97. Ecommerce Integration Test

Example flow:

```text
POST /api/orders
        ↓
Controller
        ↓
Order Service
        ↓
Product Repository
        ↓
Inventory
        ↓
Order Repository
        ↓
MySQL
```

Verify:

```text
HTTP response
Database state
Stock change
Order record
```

---

# 98. Ecommerce Security Test

Example:

```text
USER
 ↓
DELETE /api/products/10
```

Expected:

```text
403 Forbidden
```

Then:

```text
ADMIN
 ↓
DELETE /api/products/10
```

Expected:

```text
204 No Content
```

---

# 99. Ecommerce Inventory Test

Initial:

```text
Stock = 2
```

Requests:

```text
User A → quantity 2
User B → quantity 1
```

Expected:

```text
A succeeds
B fails
Stock = 0
```

This verifies a core business invariant.

---

# 100. Ecommerce Duplicate Request Test

Request:

```text
POST /api/orders
Idempotency-Key: ABC123
```

Send twice.

Expected:

```text
One order
Same result or documented response
No duplicate payment
No duplicate inventory reservation
```

---

# 101. Testing Service Failure

Example:

```text
Order
 ↓
Payment
 X
```

Test:

```text
Payment timeout
```

Verify:

```text
Timeout handled
Circuit breaker/retry behavior
Order state remains consistent
No duplicate payment
```

---

# 102. Testing Cache

Test:

```text
First request
↓
Cache miss
↓
Database

Second request
↓
Cache hit
```

Verify:

```text
Database called once
Cache used on second request
```

Also test:

```text
Cache expiration
Cache invalidation
Cache unavailable
```

where these behaviors matter.

---

# 103. Testing Logging

Avoid asserting every log message in normal tests.

Test logs when:

```text
Audit event is required
Security event is required
Operational behavior depends on it
```

Otherwise logging implementation can become unnecessarily coupled to tests.

---

# 104. Testing Metrics

Metrics tests may verify that an important metric changes after an operation.

For example:

```text
order.created
```

But don't make every unit test depend on monitoring implementation details.

---

# 105. Testing Timeouts

A timeout test should be deterministic.

```text
Mock dependency
↓
Delay response
↓
Client timeout
↓
Expected fallback/error
```

Avoid real long sleeps whenever possible.

---

# 106. Testing Retry Backoff

Don't make tests wait for long production backoff values.

Use a test-specific configuration:

```text
Production:
1s → 2s → 4s

Test:
10ms → 20ms → 40ms
```

This keeps tests fast.

---

# 107. Test Configuration

Separate test configuration:

```text
application-test.yml
```

Example:

```yaml
app:
  payment:
    timeout: 100ms
```

Do not accidentally use production credentials during tests.

---

# 108. Test Secrets

Never commit:

```text
Real passwords
Production API keys
Private certificates
Cloud credentials
```

Use:

```text
Environment variables
CI secrets
Test-specific credentials
Temporary credentials
```

---

# 109. Test Data Security

Use fake:

```text
Names
Emails
Phone numbers
Addresses
Payment IDs
Tokens
```

Don't copy production customer data into a test repository.

---

# 110. Flaky Tests

A flaky test sometimes:

```text
Passes
↓
Fails
↓
Passes
```

without code changes.

Common causes:

```text
Timing
Thread races
Shared state
External dependency
Random data
Test order
Time zones
Database cleanup
```

Fix the root cause rather than rerunning until green.

---

# 111. Random Test Data

Random data can find edge cases but can make failures hard to reproduce.

If using randomness:

```text
Record the seed
```

so the failure can be reproduced.

---

# 112. Time Zone Tests

If application logic uses dates/times, test:

```text
UTC
Different offsets
Day boundary
Month boundary
DST where applicable
```

Use `Clock` and explicit time zones where appropriate.

---

# 113. Contract Testing vs Integration Testing

Integration testing:

```text
Test actual components together
```

Contract testing:

```text
Verify service interface expectations
```

Both can be useful in microservice environments.

---

# 114. Unit Test Smell

Bad test:

```java
verify(repository, times(1))
    .save(any());
```

with no assertion about the actual business result.

Better:

```text
Assert business outcome
+
Verify important interaction if needed
```

---

# 115. Test the Behavior, Not Implementation

Bad:

```text
Verify private method called
```

Better:

```text
Verify observable behavior
```

For example:

```text
Given insufficient stock
→ order rejected
→ stock unchanged
```

---

# 116. Test Naming Formula

Use:

```text
should + expected behavior + condition
```

Examples:

```text
shouldCreateOrderWhenStockIsAvailable

shouldRejectOrderWhenStockIsInsufficient

shouldReturnNotFoundWhenProductDoesNotExist

shouldRejectUserWhenAdminRoleIsRequired
```

---

# 117. Definition of Done

A backend feature should ideally include:

```text
Code
+
Unit tests
+
Integration tests where needed
+
Validation
+
Error handling
+
Security
+
Observability
+
Documentation
```

Not every feature needs every item, but the decision should be deliberate.

---

# 118. Interview: What Is Your Testing Strategy?

> I use a combination of unit and integration tests. Unit tests cover business logic quickly with mocks, while integration tests verify real Spring behavior, persistence, security, and external infrastructure. For database-heavy functionality I prefer Testcontainers with the same database technology used in production.

---

# 119. Interview: How Do You Test Repository Queries?

> I use repository-focused tests such as `@DataJpaTest`. For important production behavior, I prefer running against the same database technology using Testcontainers because an in-memory database can behave differently.

---

# 120. Interview: How Do You Test Controllers?

> I commonly use `@WebMvcTest` and MockMvc to verify request mapping, validation, serialization, HTTP status codes, exception handling, and security behavior.

---

# 121. Interview: How Do You Test External APIs?

> I use a mock HTTP server such as WireMock to simulate success, errors, timeouts, and slow responses. That makes tests deterministic and avoids depending on third-party services.

---

# 122. Interview: Why Should We Not Mock Everything?

> Because mocks only verify our assumptions about a dependency. They don't prove that the real database, serialization, Spring configuration, or external integration actually works. That's why I combine unit tests with integration tests.

---

# 123. Interview: What Is Testcontainers?

> Testcontainers is a library that allows tests to run real infrastructure dependencies in containers. For example, I can start a MySQL container, connect Spring Boot to it, run repository tests, and verify real database behavior.

---

# 124. Interview: How Do You Test a Production Bug?

> After reproducing and fixing the bug, I add a regression test that fails with the old behavior and passes with the fix. That prevents the same issue from silently returning later.

---

# 125. Interview: How Much Code Coverage Is Enough?

> I don't treat a single percentage as the goal. I prioritize coverage of important business rules, failure paths, security boundaries, persistence behavior, and concurrency-sensitive logic. High coverage without meaningful assertions doesn't provide much confidence.

---

# 126. Interview: Unit Test vs Integration Test

> A unit test isolates one component and is usually fast. An integration test verifies multiple real components together, such as Spring, JPA, and MySQL. I use unit tests for fast feedback and integration tests for confidence in real interactions.

---

# 127. Interview: How Would You Test Your Ecommerce Backend?

> I would unit test the service-layer business rules, controller-test the REST endpoints, repository-test the JPA queries, and use Testcontainers with MySQL for important integration flows. I would also test authentication, authorization, inventory concurrency, duplicate order requests, transaction rollback, and external payment failures.

---

# 128. Final Testing Checklist

```text
□ Unit tests
□ Service business rules
□ Controller/API tests
□ Validation tests
□ Exception tests
□ Repository tests
□ Real database tests
□ Testcontainers
□ Security tests
□ External API tests
□ Retry tests
□ Timeout tests
□ Circuit breaker tests
□ Kafka tests
□ Idempotency tests
□ Transaction tests
□ Concurrency tests
□ Regression tests
□ Smoke tests
□ CI integration
□ No production secrets
□ No flaky dependencies
```

---

# 129. Final Mental Model

```text
                 SPRING BOOT TESTING
                        |
          +-------------+-------------+
          |             |             |
        Unit        Integration       E2E
          |             |             |
       Mockito       Testcontainers  Full flow
       JUnit         MySQL/Kafka     Browser/API
          |
       Fast feedback
                        |
                 Production confidence
```

---

# 130. Final Interview Rule

> **I don't try to test everything at the same level. I use fast unit tests for business logic, focused Spring slice tests for controllers and repositories, and integration tests with real infrastructure such as MySQL or Kafka when the behavior depends on it. For important production bugs, I add regression tests so the same issue is caught automatically in the future.**

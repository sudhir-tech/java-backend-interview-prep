# Spring Boot — Testing

Testing is a core part of production Spring Boot development.

A good test strategy should verify:

```text
Business logic
REST endpoints
Validation
Exception handling
Database interaction
Security
Integration between components
```

A common testing pyramid is:

```text
             E2E Tests
          /-------------\
         / Integration   \
        /     Tests       \
       /-------------------\
      /    Unit Tests       \
     /-----------------------\
```

Unit tests are usually fast and numerous.

Integration tests are broader and verify that multiple components work together.

---

# 1. Why Testing Matters

Testing helps catch:

```text
Regression bugs
Incorrect business logic
API contract problems
Database issues
Security problems
Configuration mistakes
```

For a backend developer, tests also make refactoring safer.

---

# 2. Spring Boot Test Dependencies

A common dependency is:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

It provides commonly used testing libraries and Spring Boot testing support.

Typical tools include:

```text
JUnit
Mockito
Spring Test
Spring Boot Test
AssertJ
JSON testing support
```

---

# 3. JUnit

JUnit is the primary Java testing framework commonly used with Spring Boot.

Example:

```java
class CalculatorTest {

    @Test
    void shouldAddTwoNumbers() {

        int result =
            10 + 20;

        assertEquals(
            30,
            result
        );
    }
}
```

---

# 4. @Test

`@Test` marks a method as a test.

Example:

```java
@Test
void shouldCalculateTotal() {

    ...
}
```

JUnit executes the test method.

---

# 5. Assertions

JUnit provides assertions such as:

```java
assertEquals(
    expected,
    actual
);
```

Other common assertions:

```text
assertTrue
assertFalse
assertNull
assertNotNull
assertThrows
assertDoesNotThrow
```

---

# 6. AssertJ

Spring Boot projects commonly use AssertJ for fluent assertions.

Example:

```java
assertThat(result)
    .isEqualTo(30);
```

Collection:

```java
assertThat(products)
    .hasSize(3)
    .extracting(Product::getName)
    .contains("Laptop");
```

---

# 7. Unit Test

A unit test tests one unit of logic in isolation.

Example:

```text
ProductService
```

while mocking:

```text
ProductRepository
ProductMapper
External APIs
```

Flow:

```text
ProductService
     ↓
Mock Repository
```

---

# 8. Mockito

Mockito creates test doubles such as mocks.

Example:

```java
@Mock
private ProductRepository repository;
```

Then:

```java
when(repository.findById(1L))
    .thenReturn(
        Optional.of(product)
    );
```

---

# 9. @ExtendWith(MockitoExtension.class)

For a pure Mockito unit test:

```java
@ExtendWith(MockitoExtension.class)
class ProductServiceTest {

}
```

This initializes Mockito annotations such as:

```text
@Mock
@InjectMocks
```

without starting the Spring application context.

---

# 10. @Mock

Example:

```java
@Mock
private ProductRepository repository;
```

Mockito creates a mock object.

The mock does not execute the real repository implementation.

---

# 11. @InjectMocks

Example:

```java
@InjectMocks
private ProductService service;
```

Mockito creates the service and injects the mocks into it.

Prefer constructor injection in production code.

---

# 12. Basic Service Unit Test

Example:

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
            new Product();

        product.setId(1L);
        product.setName("Laptop");

        when(repository.findById(1L))
            .thenReturn(
                Optional.of(product)
            );

        Product result =
            service.getProduct(1L);

        assertThat(result.getName())
            .isEqualTo("Laptop");
    }
}
```

---

# 13. Given / When / Then

A readable test can follow:

```text
Given
When
Then
```

Example:

```java
@Test
void shouldReturnProduct() {

    // Given
    Product product = createProduct();

    when(repository.findById(1L))
        .thenReturn(Optional.of(product));

    // When
    Product result =
        service.getProduct(1L);

    // Then
    assertThat(result)
        .isEqualTo(product);
}
```

This structure makes tests easier to understand.

---

# 14. Testing Not Found

Example:

```java
@Test
void shouldThrowWhenProductNotFound() {

    when(repository.findById(99L))
        .thenReturn(Optional.empty());

    assertThatThrownBy(
        () -> service.getProduct(99L)
    )
    .isInstanceOf(
        ProductNotFoundException.class
    );
}
```

---

# 15. assertThrows

JUnit alternative:

```java
assertThrows(
    ProductNotFoundException.class,
    () -> service.getProduct(99L)
);
```

Use AssertJ or JUnit consistently within a project.

---

# 16. Mockito verify

Use:

```java
verify(repository)
    .findById(1L);
```

This checks that the dependency was called.

Example:

```java
verify(repository, times(1))
    .findById(1L);
```

---

# 17. verifyNoInteractions

Example:

```java
verifyNoInteractions(
    paymentService
);
```

Useful when a failure should prevent a downstream operation.

---

# 18. never

Example:

```java
verify(
    paymentService,
    never()
).charge(any());
```

This verifies that the method was not called.

---

# 19. Argument Matchers

Mockito supports:

```java
any()
anyLong()
anyString()
eq(...)
```

Example:

```java
when(repository.findById(anyLong()))
    .thenReturn(
        Optional.of(product)
    );
```

Use matchers carefully.

If you use a matcher for one argument in a multi-argument Mockito call, use matchers consistently for the other arguments.

---

# 20. ArgumentCaptor

Use `ArgumentCaptor` when you want to inspect an object passed to a mock.

Example:

```java
ArgumentCaptor<Product> captor =
    ArgumentCaptor.forClass(
        Product.class
    );

verify(repository)
    .save(captor.capture());

Product saved =
    captor.getValue();

assertThat(saved.getName())
    .isEqualTo("Laptop");
```

---

# 21. When to Mock

Mock dependencies that are outside the unit being tested.

For:

```text
ProductService
```

mock:

```text
ProductRepository
PaymentClient
EmailService
```

when the goal is a focused unit test.

Do not mock the class being tested.

---

# 22. Don't Mock Everything

Bad:

```text
Controller → mock
Service → mock
Repository → mock
Mapper → mock
```

This can produce tests that verify almost nothing useful.

A unit test should isolate the unit while keeping its actual logic real.

---

# 23. Unit Test vs Integration Test

### Unit test

```text
Service
 +
Mocks
```

Fast.

### Integration test

```text
Controller
 +
Service
 +
Repository
 +
Database/test infrastructure
```

Broader and slower.

Both are valuable.

---

# 24. @SpringBootTest

Example:

```java
@SpringBootTest
class ProductServiceIntegrationTest {

}
```

It loads the Spring application context.

This is useful when you need to verify actual Spring configuration and multiple components working together.

---

# 25. @SpringBootTest Cost

`@SpringBootTest` can be relatively expensive because it may load a large portion of the application context.

Do not use it for every small unit test.

Use focused tests when possible.

---

# 26. Integration Test Example

```java
@SpringBootTest
class ProductServiceIntegrationTest {

    @Autowired
    private ProductService service;

    @Test
    void shouldCreateProduct() {

        ProductResponse result =
            service.create(
                new CreateProductRequest(
                    "Laptop",
                    "LAP-001",
                    BigDecimal.valueOf(1000),
                    10
                )
            );

        assertThat(result.name())
            .isEqualTo("Laptop");
    }
}
```

The actual setup depends on the application's database and test configuration.

---

# 27. Web Layer Test

For MVC controllers, use:

```java
@WebMvcTest
```

Example:

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {

}
```

It focuses on the MVC/web layer rather than loading the complete application.

---

# 28. MockMvc

`MockMvc` allows testing Spring MVC endpoints without requiring a real external server.

Example:

```java
mockMvc.perform(
        get("/api/products/1")
    )
    .andExpect(
        status().isOk()
    );
```

---

# 29. Controller Test with Mock Service

Example:

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProductService service;

}
```

Depending on Spring Boot/Spring Framework version, newer projects may use Spring's bean override testing support rather than older `@MockBean` APIs.

---

# 30. MockMvc POST Test

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
                    "price": 1000
                }
            """)
    )
    .andExpect(
        status().isCreated()
    );
```

---

# 31. Testing JSON Response

Example:

```java
mockMvc.perform(
        get("/api/products/1")
    )
    .andExpect(
        status().isOk()
    )
    .andExpect(
        jsonPath("$.name")
            .value("Laptop")
    );
```

---

# 32. Testing Validation

Invalid request:

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

You can also assert the returned error structure.

---

# 33. Testing Exception Handling

If a service throws:

```java
ProductNotFoundException
```

the controller test can verify:

```text
404
error code
error message
JSON response
```

Example:

```java
when(service.getProduct(99L))
    .thenThrow(
        new ProductNotFoundException(99L)
    );

mockMvc.perform(
        get("/api/products/99")
    )
    .andExpect(
        status().isNotFound()
    )
    .andExpect(
        jsonPath("$.code")
            .value("PRODUCT_NOT_FOUND")
    );
```

---

# 34. @DataJpaTest

For repository/JPA tests:

```java
@DataJpaTest
class ProductRepositoryTest {

}
```

This focuses on the JPA layer.

It is useful for testing:

```text
Repository queries
Entity mappings
Relationships
Persistence behavior
```

---

# 35. Repository Test

Example:

```java
@DataJpaTest
class ProductRepositoryTest {

    @Autowired
    private ProductRepository repository;

    @Test
    void shouldFindProductBySku() {

        Product product =
            new Product();

        product.setName("Laptop");
        product.setSku("LAP-001");
        product.setPrice(
            BigDecimal.valueOf(1000)
        );

        repository.save(product);

        Optional<Product> result =
            repository.findBySku("LAP-001");

        assertThat(result)
            .isPresent();
    }
}
```

---

# 36. Test Database

Repository tests often use an isolated test database.

Possible approaches:

```text
H2
Testcontainers
Dedicated test database
```

For realistic database behavior, Testcontainers is often useful.

---

# 37. H2 vs Testcontainers

H2:

```text
Fast
Easy
Lightweight
```

But:

```text
May behave differently from MySQL/PostgreSQL
```

Testcontainers:

```text
Runs the actual database engine in a container
```

This can provide more realistic integration testing.

---

# 38. Testcontainers Concept

Example:

```text
Test
 ↓
Docker container
 ↓
MySQL/PostgreSQL
 ↓
Spring Boot
 ↓
Repository
```

This helps catch database-specific behavior that an in-memory database may not reproduce.

---

# 39. Transactional Tests

A test can use:

```java
@Transactional
```

to run database operations within a transaction.

Depending on the test configuration, the transaction can be rolled back after the test.

Do not assume every integration test automatically rolls back simply because `@Transactional` appears somewhere else in the application.

---

# 40. Test Profiles

Create:

```text
application-test.yml
```

and use:

```java
@ActiveProfiles("test")
```

Example:

```java
@SpringBootTest
@ActiveProfiles("test")
class ProductIntegrationTest {

}
```

This allows test-specific configuration.

---

# 41. Test Configuration

Example:

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: create-drop
```

Configuration should match the chosen test database strategy.

---

# 42. Testing Configuration

Spring Boot can test configuration using:

```java
@SpringBootTest
```

For focused configuration testing, use the appropriate Spring test slice or configuration test approach rather than loading the entire application unnecessarily.

---

# 43. Test Naming

Good:

```java
shouldReturnProductWhenIdExists()
```

```java
shouldThrowProductNotFoundWhenIdDoesNotExist()
```

```java
shouldRejectNegativePrice()
```

Bad:

```java
test1()
testProduct()
check()
```

Names should describe behavior.

---

# 44. Arrange / Act / Assert

Another common pattern:

```text
Arrange
Act
Assert
```

Example:

```java
@Test
void shouldCalculateTotal() {

    // Arrange
    BigDecimal price =
        BigDecimal.valueOf(100);

    // Act
    BigDecimal result =
        service.calculateTotal(price);

    // Assert
    assertThat(result)
        .isEqualByComparingTo("100");
}
```

---

# 45. Test One Behavior

A test should ideally answer one main question.

Good:

```text
shouldReturn404WhenProductDoesNotExist
```

Avoid a giant test that verifies:

```text
create
update
delete
validation
authentication
database
```

all at once.

---

# 46. Avoid Implementation-Coupled Tests

Bad test:

```text
Verify every private implementation detail
```

Better:

```text
Verify observable behavior
```

For example:

```text
Input
→ service
→ expected result
```

rather than verifying every internal variable.

---

# 47. Test Edge Cases

For quantity:

```text
0
1
maximum allowed
negative
null
```

For price:

```text
0
positive
negative
very large
null
```

For strings:

```text
null
empty
blank
minimum length
maximum length
too long
```

---

# 48. Parameterized Tests

JUnit supports:

```java
@ParameterizedTest
```

Example:

```java
@ParameterizedTest
@ValueSource(
    ints = {1, 2, 5, 10}
)
void shouldAcceptPositiveQuantity(
        int quantity) {

    assertThat(quantity)
        .isPositive();
}
```

This avoids repeating similar test methods.

---

# 49. @CsvSource

Example:

```java
@ParameterizedTest
@CsvSource({
    "10, 20, 30",
    "5, 5, 10",
    "0, 10, 10"
})
void shouldAddNumbers(
        int a,
        int b,
        int expected) {

    assertThat(a + b)
        .isEqualTo(expected);
}
```

---

# 50. @MethodSource

Use:

```java
@MethodSource
```

when test data is more complex.

Example:

```java
@ParameterizedTest
@MethodSource("products")
void shouldValidateProducts(
        Product product) {

    ...
}
```

---

# 51. Lifecycle Annotations

JUnit provides:

```text
@BeforeEach
@AfterEach
@BeforeAll
@AfterAll
```

Example:

```java
@BeforeEach
void setUp() {

    service = new ProductService(...);
}
```

Use setup methods carefully; too much hidden setup can make tests difficult to understand.

---

# 52. @BeforeAll

Runs once before all tests in the class.

Example:

```java
@BeforeAll
static void setup() {

}
```

Useful for expensive shared setup when appropriate.

---

# 53. @AfterEach

Runs after each test.

Useful for cleanup:

```java
@AfterEach
void cleanup() {

}
```

Avoid unnecessary cleanup code when the test framework already isolates the resources.

---

# 54. Mock vs Spy

Mock:

```text
Entire object behavior is controlled by Mockito
```

Spy:

```text
Real object behavior by default
+
selected methods can be stubbed
```

Example:

```java
@Spy
private ProductMapper mapper;
```

Use spies sparingly because they can create tests tightly coupled to implementation.

---

# 55. Mocking Void Methods

Example:

```java
doNothing()
    .when(emailService)
    .sendEmail(any());
```

Or for exceptions:

```java
doThrow(
    new EmailException()
)
.when(emailService)
.sendEmail(any());
```

---

# 56. Testing Exceptions with Mockito

Example:

```java
when(paymentService.charge(any()))
    .thenThrow(
        new PaymentException()
    );
```

Then verify that the service handles the failure correctly.

---

# 57. Testing Interaction

Example:

```java
verify(repository)
    .save(any(Product.class));
```

But don't verify every interaction automatically.

Ask:

```text
Does this interaction represent important behavior?
```

---

# 58. Testing REST Status Codes

Important assertions:

```java
status().isOk()
status().isCreated()
status().isNoContent()
status().isBadRequest()
status().isUnauthorized()
status().isForbidden()
status().isNotFound()
status().isConflict()
status().isInternalServerError()
```

---

# 59. Testing Response Headers

Example:

```java
.andExpect(
    header().string(
        "Content-Type",
        "application/json"
    )
);
```

Use header assertions when they are part of the API contract.

---

# 60. Testing Security

Security tests should verify:

```text
Unauthenticated → 401
Wrong role → 403
Correct role → success
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

---

# 61. @WithMockUser

Spring Security tests can use:

```java
@WithMockUser(
    username = "admin",
    roles = "ADMIN"
)
```

Example:

```java
@Test
@WithMockUser(
    username = "admin",
    roles = "ADMIN"
)
void adminCanDeleteProduct() {

    ...
}
```

This is useful for testing authorization behavior without implementing the complete login flow in every test.

---

# 62. Testing Roles

Example:

```java
@Test
@WithMockUser(
    roles = "USER"
)
void userCannotDeleteProduct() {

    mockMvc.perform(
        delete("/api/admin/products/1")
    )
    .andExpect(
        status().isForbidden()
    );
}
```

---

# 63. Testing Authentication

For JWT-based applications, test both:

```text
Valid token
Invalid token
Expired token
Missing token
Insufficient permissions
```

Unit-test the token service separately and integration-test the security filter chain where appropriate.

---

# 64. Integration Testing Security

A realistic integration test can verify:

```text
HTTP request
 ↓
SecurityFilterChain
 ↓
JWT validation
 ↓
Authorization
 ↓
Controller
 ↓
Service
```

This catches configuration problems that a pure unit test cannot.

---

# 65. Test Coverage

Code coverage measures how much code is executed by tests.

Examples:

```text
Line coverage
Branch coverage
Method coverage
```

High coverage does not automatically mean high-quality tests.

A test suite can execute code without actually checking meaningful behavior.

---

# 66. Good Coverage

Prioritize:

```text
Business-critical logic
Authentication
Authorization
Payment logic
Order processing
Inventory updates
Validation
Exception paths
```

---

# 67. Regression Testing

When a production bug is found:

```text
Bug
 ↓
Understand root cause
 ↓
Fix code
 ↓
Add regression test
 ↓
Prevent recurrence
```

This is an important engineering practice.

---

# 68. Testing Service Logic

For an ecommerce backend:

```text
Create Product
Get Product
Update Product
Delete Product
Place Order
Cancel Order
Check Stock
Calculate Price
```

Each important business behavior should have tests.

---

# 69. Testing Order Placement

Example scenarios:

```text
Valid order → success
Product missing → 404
Insufficient stock → business error
Quantity <= 0 → validation error
Payment failure → transaction handled correctly
Duplicate request → appropriate protection
```

---

# 70. Testing Transactions

Transactions should be tested for important failure scenarios.

Example:

```text
Create order
 ↓
Reduce inventory
 ↓
Payment fails
 ↓
Expected rollback behavior
```

Verify the final database state rather than only verifying that an exception was thrown.

---

# 71. Testing Repository Queries

Custom queries should be tested for:

```text
Correct result
No result
Multiple results
Filtering
Sorting
Pagination
Relationships
```

This is especially important for complex JPQL or native SQL.

---

# 72. Testing Pagination

Example:

```java
Page<Product> result =
    repository.findAll(
        PageRequest.of(0, 20)
    );
```

Verify:

```text
Content
Page size
Page number
Total elements
Total pages
```

---

# 73. Testing Mappers

If mapping logic contains business-relevant transformations, test it.

Example:

```text
Entity
 ↓
DTO
```

Verify:

```text
ID
Name
Price
Status
Nested data
```

Simple generated mappings may not need extensive standalone tests if integration tests already cover them.

---

# 74. Mocking External APIs

Suppose:

```text
OrderService
 ↓
PaymentClient
```

Unit test:

```java
when(paymentClient.pay(any()))
    .thenReturn(
        PaymentResult.success()
    );
```

Then test:

```text
Successful payment
Payment failure
Timeout
Unexpected response
```

For integration testing, tools such as WireMock can simulate HTTP dependencies.

---

# 75. Test External Service Failure

Example:

```java
when(paymentClient.pay(any()))
    .thenThrow(
        new PaymentServiceException()
    );
```

Verify:

```text
Correct exception
Correct transaction behavior
Correct API response
```

---

# 76. Contract Testing

Contract testing verifies that services agree on an API contract.

Useful in microservices:

```text
Order Service
     ↕
Payment Service
```

The contract defines:

```text
Request
Response
Headers
Status codes
Schema
```

Tools such as Spring Cloud Contract or Pact can be used depending on the architecture.

---

# 77. End-to-End Testing

E2E testing verifies a complete user flow.

Example:

```text
Register
 ↓
Login
 ↓
Browse product
 ↓
Add to cart
 ↓
Place order
 ↓
Payment
 ↓
Order confirmation
```

E2E tests are valuable but typically slower and more expensive than unit tests.

---

# 78. Test Pyramid

A healthy test suite generally has:

```text
        Few E2E
       /       \
      / Integration \
     /---------------\
    /   Many Unit     \
   /-------------------\
```

The exact ratio varies by project.

---

# 79. Unit Test Characteristics

Good unit tests are:

```text
Fast
Isolated
Deterministic
Readable
Focused
```

They should not require:

```text
Real database
Real payment gateway
Real email provider
```

---

# 80. Integration Test Characteristics

Good integration tests verify:

```text
Real Spring configuration
Real component interaction
Real persistence behavior
Real security configuration
```

They can use infrastructure such as:

```text
Testcontainers
```

for realistic dependencies.

---

# 81. Test Isolation

One test should not depend on another test.

Bad:

```text
testCreateUser
   ↓
testLoginUser depends on created user
```

Better:

```text
Each test creates its own required state.
```

---

# 82. Deterministic Tests

Avoid tests that depend on:

```text
Current time without control
Random values without seeding
External APIs
Network availability
Execution order
Machine-specific state
```

Use controllable clocks and test doubles when appropriate.

---

# 83. Testing Time

Instead of:

```java
LocalDateTime.now()
```

everywhere, production code can depend on:

```java
Clock
```

Then tests can provide:

```java
Clock.fixed(...)
```

This makes time-dependent behavior deterministic.

---

# 84. Testing Randomness

If production code generates random values:

```text
UUID
random numbers
tokens
```

design the code so tests can control or mock the source where appropriate.

Do not assert against unpredictable values unless the test only verifies valid properties.

---

# 85. Test Data Builders

For complex objects, builders can improve readability.

Example:

```java
Product product =
    ProductTestDataBuilder
        .aProduct()
        .withName("Laptop")
        .withPrice(
            BigDecimal.valueOf(1000)
        )
        .build();
```

This avoids repeating large object construction in every test.

---

# 86. Object Mother

Another pattern is:

```text
TestDataFactory
```

Example:

```java
ProductTestData.defaultProduct();
```

Use it carefully.

Overly generic test data factories can hide important test conditions.

---

# 87. Test Fixtures

A fixture is the data/setup required for a test.

Example:

```text
User fixture
Product fixture
Order fixture
```

Good fixtures should be:

```text
Readable
Predictable
Easy to customize
```

---

# 88. Testing Null Handling

Test meaningful null scenarios:

```text
null request
null optional field
missing entity
missing database value
```

But don't add meaningless tests simply to increase coverage.

---

# 89. Testing Boundary Conditions

Important examples:

```text
minimum value
maximum value
just below minimum
just above maximum
empty collection
single item
large collection
```

Boundary tests often catch bugs that normal happy-path tests miss.

---

# 90. Happy Path vs Failure Path

Every important feature should consider:

```text
Happy path
Validation failure
Business failure
Infrastructure failure
Security failure
```

Example:

```text
Create Order
```

Tests:

```text
Valid order
Invalid quantity
Product missing
Stock insufficient
Payment unavailable
Unauthorized user
```

---

# 91. Mutation Testing

Mutation testing intentionally changes code to see whether tests detect the change.

Conceptually:

```text
Original:
price > 0

Mutation:
price >= 0
```

If tests still pass, the test suite may not be strong enough.

Tools such as PIT can be used for Java mutation testing.

---

# 92. Test Quality

Good tests should fail when the behavior is wrong.

A test that always passes is useless.

Example of weak assertion:

```java
assertThat(result)
    .isNotNull();
```

when the actual requirement is:

```text
product name must be Laptop
price must be 1000
```

Prefer meaningful assertions.

---

# 93. Multiple Assertions

Multiple assertions are fine when they verify one coherent behavior.

Example:

```java
assertThat(result.id())
    .isEqualTo(1L);

assertThat(result.name())
    .isEqualTo("Laptop");

assertThat(result.price())
    .isEqualByComparingTo("1000");
```

---

# 94. Testing Collections

Example:

```java
assertThat(products)
    .hasSize(2)
    .extracting(Product::getName)
    .containsExactly(
        "Laptop",
        "Phone"
    );
```

This is often more expressive than manually checking indexes.

---

# 95. Testing Optional

Example:

```java
Optional<Product> result =
    repository.findBySku("LAP-001");

assertThat(result)
    .isPresent()
    .get()
    .extracting(Product::getSku)
    .isEqualTo("LAP-001");
```

---

# 96. Testing Exception Message

Only assert the exact message when it is part of the contract.

Example:

```java
assertThatThrownBy(
    () -> service.getProduct(99L)
)
.isInstanceOf(
    ProductNotFoundException.class
)
.hasMessage(
    "Product not found: 99"
);
```

Avoid making tests fragile by asserting implementation details that don't matter.

---

# 97. Testing Logs

Usually don't make ordinary unit tests depend heavily on log output.

Instead test:

```text
Behavior
Response
Exception
State
```

Use log assertions only when logging itself is a meaningful requirement.

---

# 98. Testing Database Constraints

For unique SKU:

```text
Insert SKU LAP-001
 ↓
Insert SKU LAP-001
 ↓
Constraint violation
```

Verify that the application translates the violation appropriately.

This tests both:

```text
Database integrity
Exception handling
```

---

# 99. Testing API Validation

Test:

```text
Missing required field
Blank string
Negative price
Invalid email
Invalid enum
Too-long field
Malformed JSON
```

A production API should respond consistently for each category.

---

# 100. Testing API Security

Test:

```text
No token
Invalid token
Expired token
USER role
ADMIN role
Missing authority
Correct authority
```

Security bugs can be more serious than ordinary functional bugs, so authorization paths deserve strong test coverage.

---

# 101. Testing Spring Boot Application Startup

An integration test can verify that the application context starts successfully:

```java
@SpringBootTest
class ApplicationContextTest {

    @Test
    void contextLoads() {
    }
}
```

This can catch:

```text
Bean creation failures
Configuration problems
Dependency issues
```

Do not rely on this test alone; it does not prove business behavior is correct.

---

# 102. Slice Tests

Spring Boot provides focused test slices such as:

```text
@WebMvcTest
@DataJpaTest
```

The idea is:

```text
Test only the layer you need
```

This generally makes tests faster and more focused than always using:

```java
@SpringBootTest
```

---

# 103. Testing REST + Database

For broader integration:

```text
HTTP request
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
Test database
```

Use:

```java
@SpringBootTest
```

with an appropriate test database strategy.

---

# 104. Testing with Testcontainers

Conceptually:

```java
@Testcontainers
@SpringBootTest
class ProductIntegrationTest {

    @Container
    static MySQLContainer<?> mysql =
        new MySQLContainer<>(
            "mysql:8.4"
        );
}
```

The exact version and configuration should match the application's supported database.

---

# 105. Why Testcontainers?

Because:

```text
H2 ≠ MySQL
H2 ≠ PostgreSQL
```

Database-specific behavior can include:

```text
SQL syntax
Indexes
Constraints
Data types
Transactions
Isolation
Functions
```

Testcontainers helps test against the actual database engine.

---

# 106. Testing Migrations

If the application uses:

```text
Flyway
Liquibase
```

integration tests should verify that migrations work against the target database.

Example flow:

```text
Clean database
 ↓
Run migrations
 ↓
Start application
 ↓
Run repository tests
```

---

# 107. Test Environment

A good test environment should be:

```text
Isolated
Repeatable
Automatable
Close enough to production where important
```

Avoid depending on a developer's local database.

---

# 108. CI/CD Testing

Typical pipeline:

```text
Git Push
   ↓
Build
   ↓
Unit Tests
   ↓
Integration Tests
   ↓
Static Analysis
   ↓
Package
   ↓
Deploy
```

Tools can include:

```text
GitHub Actions
Jenkins
GitLab CI
SonarQube
```

---

# 109. Maven Test

Run:

```bash
mvn test
```

This normally runs the project's unit tests according to the Maven test lifecycle and configured plugins.

---

# 110. Maven Verify

For broader verification:

```bash
mvn verify
```

Depending on the project configuration, this can execute additional integration/verification steps.

---

# 111. Test Failure

When a test fails:

```text
Read assertion
 ↓
Understand expected vs actual
 ↓
Identify whether test or production code is wrong
 ↓
Reproduce
 ↓
Fix root cause
 ↓
Run relevant tests
 ↓
Run full suite
```

Do not simply change the expected value to make a test pass.

---

# 112. Flaky Tests

A flaky test sometimes passes and sometimes fails without code changes.

Common causes:

```text
Timing
Concurrency
Randomness
External dependencies
Shared state
Test order
Uncontrolled clock
```

Flaky tests reduce trust in CI.

---

# 113. Avoid Thread.sleep

Bad:

```java
Thread.sleep(5000);
```

This makes tests slow and unreliable.

Prefer:

```text
Awaitility
Explicit synchronization
Deterministic test doubles
```

when asynchronous behavior must be tested.

---

# 114. Testing Asynchronous Code

For async workflows:

```text
Trigger event
 ↓
Background processing
 ↓
Wait for expected condition
```

Tools such as Awaitility can help:

```java
await()
    .atMost(
        Duration.ofSeconds(5)
    )
    .untilAsserted(() ->
        assertThat(repository.count())
            .isEqualTo(1)
    );
```

---

# 115. Testing Event-Driven Systems

For Kafka/message-based applications:

```text
Publish message
 ↓
Consumer
 ↓
Business processing
 ↓
Database
```

Test:

```text
Valid message
Invalid message
Duplicate message
Processing failure
Retry
Dead-letter behavior
```

---

# 116. Idempotency Testing

For an endpoint such as:

```text
POST /orders
```

if the application supports idempotency keys:

```text
Request A
Idempotency-Key: ABC

Request A repeated
Idempotency-Key: ABC
```

Both should produce the expected idempotent behavior rather than creating duplicate orders.

---

# 117. Testing Concurrency

Concurrency bugs can require specialized tests.

Examples:

```text
Two users buy last item
Two requests update same product
Two payments process same order
```

Use appropriate concurrency testing rather than assuming sequential unit tests cover these scenarios.

---

# 118. Test Naming Example

Prefer:

```java
shouldRejectOrderWhenStockIsInsufficient()
```

over:

```java
testOrder()
```

The name should describe:

```text
Condition
Behavior
Expected result
```

---

# 119. Test Organization

Typical project:

```text
src/
 ├── main/
 │    └── java/
 │
 └── test/
      └── java/
```

Mirror the production package structure where practical:

```text
src/main/java/com/example/product/ProductService.java

src/test/java/com/example/product/ProductServiceTest.java
```

---

# 120. Testing Strategy for Your Ecommerce Backend

For your Spring Boot ecommerce project, prioritize:

```text
1. ProductService unit tests
2. ProductController tests
3. ProductRepository tests
4. Validation tests
5. Exception handler tests
6. Security tests
7. OrderService tests
8. Inventory/stock tests
9. Database integration tests
10. End-to-end critical flows
```

---

# 121. Minimum Test Set

Before calling an ecommerce backend production-ready, test at least:

```text
Product creation
Product retrieval
Product update
Product deletion
Product validation
Product not found
Duplicate SKU
User registration
Login
Authentication
Authorization
Cart operations
Order creation
Insufficient stock
Order cancellation
Global exception handling
Database persistence
```

---

# 122. Interview Question

### What is the difference between unit and integration testing?

Human-written answer:

> A unit test verifies one component in isolation, usually by mocking its dependencies. An integration test verifies that multiple real components work together, such as a service, repository, Spring context, and database.

---

# 123. Interview Question

### What is @SpringBootTest?

Human-written answer:

> `@SpringBootTest` loads the Spring Boot application context and is useful for integration testing. I don't use it for every unit test because loading the full context is slower.

---

# 124. Interview Question

### What is @WebMvcTest?

Human-written answer:

> `@WebMvcTest` focuses on the MVC layer. I use it to test controllers, request validation, JSON serialization, and HTTP behavior without loading the entire application.

---

# 125. Interview Question

### What is @DataJpaTest?

Human-written answer:

> `@DataJpaTest` focuses on JPA components and is useful for testing repositories, entity mappings, and database queries.

---

# 126. Interview Question

### What is Mockito?

Human-written answer:

> Mockito is a mocking framework that lets me create test doubles for dependencies. For example, when unit testing a service, I can mock the repository and control what it returns.

---

# 127. Interview Question

### Why use mocks?

Human-written answer:

> Mocks isolate the component I'm testing from external dependencies. This makes unit tests faster, deterministic, and focused on the component's behavior.

---

# 128. Interview Question

### Should we mock the database in integration tests?

Human-written answer:

> For a true repository or database integration test, I prefer testing against a real database engine or a realistic isolated test database rather than mocking the repository. Mocking is more appropriate for service-level unit tests.

---

# 129. Interview Question

### What is Testcontainers?

Human-written answer:

> Testcontainers allows tests to run real infrastructure, such as MySQL or PostgreSQL, in containers. It is useful when database-specific behavior matters and an in-memory database isn't sufficiently realistic.

---

# 130. Interview Question

### What is test coverage?

Human-written answer:

> Test coverage measures which parts of the code are executed by tests. I use it as a useful signal, but high coverage alone doesn't guarantee good tests; the assertions and scenarios still need to verify meaningful behavior.

---

# 131. Interview Question

### How do you test exceptions?

Human-written answer:

> For service unit tests, I use `assertThrows` or AssertJ's `assertThatThrownBy` and verify the exception type and important details. For REST tests, I verify the HTTP status and error response returned by the global exception handler.

---

# 132. Interview Question

### How do you test security?

Human-written answer:

> I test unauthenticated requests, invalid credentials or tokens, role-based access, and permission failures. For controller security I can use Spring Security test support such as `@WithMockUser`, and for broader tests I verify the actual security filter chain.

---

# 133. Interview Question

### How do you prevent flaky tests?

Human-written answer:

> I avoid shared mutable state, uncontrolled time, random dependencies, external services, and arbitrary sleeps. For asynchronous behavior I use deterministic synchronization or tools such as Awaitility.

---

# 134. Interview Question

### What makes a good unit test?

Human-written answer:

> A good unit test is fast, isolated, deterministic, readable, and focused on one behavior. It should fail when that behavior is broken without depending on unrelated implementation details.

---

# 135. Production Testing Mindset

Think:

```text
What can fail?
       ↓
How would the user experience it?
       ↓
What should the API return?
       ↓
How do I reproduce it?
       ↓
How do I prevent regression?
```

This mindset is more valuable than simply chasing coverage numbers.

---

# 136. Testing Checklist

```text
□ Unit tests
□ Service tests
□ Controller tests
□ Repository tests
□ Validation tests
□ Exception tests
□ Security tests
□ Integration tests
□ Database tests
□ Transaction tests
□ External-service failure tests
□ Edge-case tests
□ Boundary tests
□ Regression tests
□ CI test execution
□ Flaky-test monitoring
```

---

# 137. Final Mental Model

```text
                 Test Pyramid
                      │
          ┌───────────┴───────────┐
          │       E2E Tests       │
          │   Critical user flow  │
          └───────────┬───────────┘
                      │
          ┌───────────┴───────────┐
          │  Integration Tests    │
          │ Spring + DB + APIs    │
          └───────────┬───────────┘
                      │
          ┌───────────┴───────────┐
          │      Unit Tests       │
          │ Fast + isolated logic │
          └───────────────────────┘
```

---

# 138. Final Interview Rule

> **I use unit tests for fast isolated business logic, focused Spring test slices for controllers and repositories, and integration tests for real component interactions such as database and security configuration. For critical flows, I also use end-to-end tests. The goal isn't just high coverage; it's reliable tests that catch real regressions.**

Next:

```text
10 Spring Security
      ↓
11 Spring Boot Testing
      ↓
12 Spring Boot Actuator & Production Monitoring
```

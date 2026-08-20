# Spring Boot — Project Architecture and Best Practices

A good Spring Boot project should be easy to:

```text
Understand
Test
Maintain
Debug
Extend
Deploy
```

For a typical backend application, keep responsibilities separated instead of putting everything inside controllers.

A practical structure:

```text
src/main/java/com/example/ecommerce
│
├── controller
├── service
├── repository
├── entity
├── dto
├── mapper
├── exception
├── config
├── security
└── EcommerceApplication.java
```

---

# 1. Layered Architecture

A common Spring Boot architecture is:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

Each layer has a clear responsibility.

```text
Controller  → HTTP/API
Service     → Business logic
Repository  → Data access
Entity      → Persistence model
DTO         → API data model
```

---

# 2. Controller Layer

The controller handles HTTP concerns.

Example:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService service;

    public ProductController(
            ProductService service) {
        this.service = service;
    }

    @GetMapping("/{id}")
    public ProductResponse getProduct(
            @PathVariable Long id) {

        return service.getProduct(id);
    }
}
```

The controller should generally:

```text
Receive request
Validate request
Call service
Return response
```

Avoid putting complex business logic here.

---

# 3. Service Layer

The service owns business logic.

```java
@Service
public class ProductService {

    private final ProductRepository repository;

    public ProductService(
            ProductRepository repository) {
        this.repository = repository;
    }

    public ProductResponse getProduct(
            Long id) {

        Product product =
            repository.findById(id)
                .orElseThrow(
                    () ->
                        new ProductNotFoundException(id)
                );

        return ProductResponse.from(product);
    }
}
```

---

# 4. Repository Layer

The repository handles persistence.

```java
@Repository
public interface ProductRepository
        extends JpaRepository<Product, Long> {

    Optional<Product> findBySku(String sku);
}
```

The repository should not contain business workflows.

---

# 5. Entity

An entity represents persistent data.

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private BigDecimal price;

    private Integer stock;
}
```

Entities belong to the persistence model.

---

# 6. DTO

A DTO represents data transferred between the API and application.

Request:

```java
public record CreateProductRequest(

    @NotBlank
    String name,

    @NotNull
    @Positive
    BigDecimal price

) {
}
```

Response:

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {
}
```

---

# 7. Why Use DTOs?

Avoid exposing entities directly from REST APIs.

Bad:

```java
@GetMapping("/{id}")
public Product getProduct(...) {
    return repository.findById(...);
}
```

Better:

```java
@GetMapping("/{id}")
public ProductResponse getProduct(...) {
    return service.getProduct(...);
}
```

DTOs provide:

```text
API stability
Security
Separation of concerns
Controlled fields
Independent API evolution
```

---

# 8. Entity vs DTO

Entity:

```text
Database model
```

DTO:

```text
API model
```

They may look similar, but they serve different purposes.

Example:

```text
Product Entity

id
name
price
costPrice
internalStatus
createdAt
updatedAt
```

API response:

```text
ProductResponse

id
name
price
```

Internal fields stay private.

---

# 9. Mapping

Mapping converts:

```text
Entity → DTO
DTO → Entity
```

Simple mapping:

```java
public static ProductResponse from(
        Product product) {

    return new ProductResponse(
        product.getId(),
        product.getName(),
        product.getPrice()
    );
}
```

For larger applications, a dedicated mapper can be useful.

---

# 10. Mapper Layer

Example:

```text
mapper
└── ProductMapper.java
```

```java
@Component
public class ProductMapper {

    public ProductResponse toResponse(
            Product product) {

        return new ProductResponse(
            product.getId(),
            product.getName(),
            product.getPrice()
        );
    }
}
```

This keeps conversion logic out of controllers and services.

---

# 11. Package Structure

Feature-based structure can also work well.

Instead of:

```text
controller/
service/
repository/
entity/
```

you can organize by business capability:

```text
product/
    ProductController
    ProductService
    ProductRepository
    Product
    ProductMapper

order/
    OrderController
    OrderService
    OrderRepository
    Order
    OrderMapper
```

For larger applications, feature-based packaging can make ownership and navigation easier.

---

# 12. Layered vs Feature-Based

Layered:

```text
controller/
service/
repository/
```

Good for:

```text
Small/medium applications
Simple domain
Learning projects
```

Feature-based:

```text
product/
order/
payment/
```

Good for:

```text
Larger applications
Domain-oriented teams
Clear business boundaries
```

Neither is universally correct.

---

# 13. Dependency Direction

A common dependency direction is:

```text
Controller
    ↓
Service
    ↓
Repository
```

Avoid:

```text
Repository → Controller
```

or:

```text
Entity → Controller
```

Keep dependencies flowing toward lower-level infrastructure where appropriate.

---

# 14. Constructor Injection

Prefer constructor injection.

```java
@Service
public class OrderService {

    private final OrderRepository repository;

    public OrderService(
            OrderRepository repository) {
        this.repository = repository;
    }
}
```

Benefits:

```text
Dependencies are explicit
Fields can be final
Easy unit testing
No hidden dependencies
```

---

# 15. Avoid Field Injection

Less preferable:

```java
@Autowired
private OrderRepository repository;
```

Constructor injection is generally cleaner:

```java
public OrderService(
        OrderRepository repository) {

    this.repository = repository;
}
```

---

# 16. Single Responsibility

A class should have one clear responsibility.

Bad:

```text
OrderController
    ↓
Validation
Payment
Database
Email
Logging
Business rules
```

Better:

```text
OrderController
    ↓
OrderService
    ↓
PaymentService
    ↓
OrderRepository
```

Each component has a focused role.

---

# 17. Keep Controllers Thin

Good:

```java
@PostMapping
public OrderResponse create(
        @Valid @RequestBody CreateOrderRequest request) {

    return service.create(request);
}
```

Avoid:

```java
@PostMapping
public OrderResponse create(...) {

    // 100 lines of business logic
    // database calls
    // payment logic
    // inventory logic

}
```

---

# 18. Service Orchestration

A service can coordinate multiple dependencies.

Example:

```java
@Transactional
public OrderResponse placeOrder(
        CreateOrderRequest request) {

    Product product =
        productService.getProduct(
            request.productId()
        );

    inventoryService.reserve(
        product.getId(),
        request.quantity()
    );

    Order order =
        orderRepository.save(
            createOrder(request, product)
        );

    return mapper.toResponse(order);
}
```

Business workflow belongs in the appropriate service/domain layer.

---

# 19. Transaction Boundary

Transactions should normally be placed around business operations.

Example:

```java
@Transactional
public void placeOrder(...) {
    ...
}
```

This defines the unit of work.

Avoid randomly adding:

```java
@Transactional
```

to every method.

---

# 20. Read-Only Transactions

For read-only operations:

```java
@Transactional(readOnly = true)
public ProductResponse getProduct(
        Long id) {
    ...
}
```

This communicates intent and may provide optimization benefits depending on the persistence setup.

---

# 21. Avoid Long Transactions

Avoid keeping transactions open while performing slow external operations:

```text
Start DB transaction
    ↓
Call Payment API
    ↓
Wait 5 seconds
    ↓
Call another service
    ↓
Commit
```

This can hold database resources unnecessarily.

Design transaction boundaries carefully.

---

# 22. External API Calls

Do not mix external calls casually with database transactions.

Bad pattern:

```text
@Transactional
    ↓
DB update
    ↓
External API
    ↓
Wait
    ↓
DB update
```

Consider:

```text
Local transaction
+
Asynchronous workflow / Saga
```

for distributed operations where appropriate.

---

# 23. Configuration

Keep configuration outside business logic.

Example:

```yaml
app:
  payment:
    timeout: 3000
```

Then bind it to configuration properties.

---

# 24. @ConfigurationProperties

Example:

```java
@ConfigurationProperties(
    prefix = "app.payment"
)
public record PaymentProperties(
    Duration timeout
) {
}
```

This is cleaner than scattering:

```java
@Value("${...}")
```

throughout the codebase for related configuration.

---

# 25. Profiles

Typical environments:

```text
dev
test
staging
prod
```

Example:

```text
application.yml
application-dev.yml
application-prod.yml
```

Use profiles for environment-specific behavior carefully.

---

# 26. Environment Variables

Sensitive or deployment-specific values can come from environment variables.

Example:

```text
DB_URL
DB_USERNAME
DB_PASSWORD
```

Do not commit production secrets to source control.

---

# 27. Secrets

Use:

```text
Vault
AWS Secrets Manager
Azure Key Vault
Kubernetes Secrets
```

depending on your infrastructure.

---

# 28. Logging

Use structured, meaningful logs.

Good:

```java
log.info(
    "Creating order for customerId={}",
    customerId
);
```

Avoid:

```java
System.out.println(
    "something happened"
);
```

Use a logging framework such as SLF4J through Spring Boot's logging setup.

---

# 29. Log Levels

Common levels:

```text
TRACE
DEBUG
INFO
WARN
ERROR
```

Typical production usage:

```text
INFO  → important application events
WARN  → unusual but handled situations
ERROR → unexpected failures
DEBUG → detailed troubleshooting
```

---

# 30. Don't Log Secrets

Never log:

```text
Passwords
JWT secrets
API keys
Credit card numbers
Private credentials
```

Also be careful with personally sensitive information.

---

# 31. API Versioning

For evolving APIs:

```text
/api/v1/products
/api/v2/products
```

Versioning helps maintain compatibility.

However, don't create a new version for every tiny change.

Prefer backward-compatible additive changes where possible.

---

# 32. HTTP Status Codes

Use status codes consistently:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
500 Internal Server Error
```

---

# 33. POST

Creating a resource:

```http
POST /api/products
```

Typical response:

```text
201 Created
```

---

# 34. GET

Reading a resource:

```http
GET /api/products/101
```

Typical response:

```text
200 OK
```

Missing resource:

```text
404 Not Found
```

---

# 35. PUT

Full replacement semantics are commonly represented with:

```http
PUT /api/products/101
```

Use PUT when the API contract treats the request as a replacement/update of the resource representation.

---

# 36. PATCH

Partial update:

```http
PATCH /api/products/101
```

Example:

```json
{
  "price": 999
}
```

PATCH semantics depend on the API contract.

---

# 37. DELETE

Delete:

```http
DELETE /api/products/101
```

A successful deletion may return:

```text
204 No Content
```

---

# 38. API Naming

Prefer resource-oriented URLs:

```text
/api/products
/api/products/{id}
/api/orders
/api/orders/{id}
```

Avoid unnecessarily action-heavy URLs:

```text
/api/getProducts
/api/createProduct
/api/deleteProduct
```

Business actions may still justify action-oriented endpoints when they do not map naturally to CRUD.

---

# 39. Pagination

Avoid returning thousands of records:

```text
GET /api/products
```

Use pagination:

```text
GET /api/products?page=0&size=20
```

A response may include:

```text
content
page
size
totalElements
totalPages
```

---

# 40. Sorting

Example:

```text
GET /api/products
    ?page=0
    &size=20
    &sort=price,desc
```

Validate allowed sort fields rather than blindly accepting arbitrary database expressions.

---

# 41. Filtering

Example:

```text
GET /api/products
    ?category=electronics
    &minPrice=500
    &maxPrice=5000
```

Keep filtering logic out of controllers when it becomes complex.

---

# 42. Search

For simple searches:

```text
GET /api/products?name=phone
```

For complex search requirements, consider a dedicated search strategy rather than forcing every query through one giant repository method.

---

# 43. Repository Best Practices

Good:

```java
Optional<Product> findBySku(
    String sku
);
```

Avoid repository methods containing business workflows.

Bad:

```java
placeOrderAndSendEmailAndChargePayment(...)
```

The repository should primarily handle persistence concerns.

---

# 44. Optional

Good:

```java
Optional<Product> findById(Long id);
```

Then:

```java
.orElseThrow(
    () -> new ProductNotFoundException(id)
);
```

Avoid returning:

```text
null
```

when absence is a normal part of the repository contract.

---

# 45. N+1 Query Problem

Example:

```text
Query orders
   ↓
For each order
   ↓
Query customer
```

If there are 100 orders:

```text
1 + 100 queries
```

This is the N+1 query problem.

---

# 46. Solving N+1

Possible approaches:

```text
Fetch joins
Entity graphs
DTO projections
Batch fetching
Careful query design
```

Do not blindly use eager fetching everywhere.

---

# 47. Lazy vs Eager

Lazy:

```text
Load relationship when accessed
```

Eager:

```text
Load relationship immediately
```

For many associations, careless eager loading can produce large queries and performance problems.

Prefer intentional fetching strategies.

---

# 48. DTO Projections

Instead of loading a huge entity graph:

```text
Product
  + Category
  + Reviews
  + Supplier
  + Inventory
```

query only what the API needs:

```text
ProductSummary
```

This can reduce:

```text
Memory
Database work
Serialization cost
```

---

# 49. Database Indexes

Indexes can improve lookup performance.

Example:

```sql
CREATE INDEX idx_product_sku
ON products(sku);
```

For unique values:

```sql
CREATE UNIQUE INDEX ...
```

Indexes also add:

```text
Storage
Write overhead
Maintenance cost
```

Create them based on actual query patterns.

---

# 50. Connection Pooling

Spring Boot applications commonly use HikariCP.

Conceptually:

```text
Application
    |
HikariCP
    |
+---+---+---+
|   |   |   |
DB connections
```

A pool avoids creating a new database connection for every request.

---

# 51. Connection Pool Problems

If the pool is exhausted:

```text
Requests wait
    ↓
Latency increases
    ↓
Timeouts
```

Monitor:

```text
Active connections
Idle connections
Pending threads
Connection acquisition time
```

---

# 52. Database Transactions

Use transactions around atomic business operations.

Example:

```java
@Transactional
public void transferStock(...) {
    ...
}
```

If an unchecked exception causes rollback:

```text
Operation A
Operation B
Operation C
   ↓
Failure
   ↓
Rollback
```

---

# 53. Optimistic Locking

Useful when multiple transactions may update the same record.

Example:

```java
@Version
private Long version;
```

Conceptually:

```text
Record version = 5

Transaction A reads 5
Transaction B reads 5

A updates → version 6
B updates → conflict
```

This prevents silent lost updates.

---

# 54. Pessimistic Locking

A pessimistic lock can lock a database row while the transaction operates.

Example concept:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
```

Use carefully because locking can reduce concurrency and increase contention.

---

# 55. Caching

Caching can reduce database load.

Example:

```text
Request
  ↓
Redis
  ↓ miss
Database
  ↓
Cache
```

Good candidates:

```text
Frequently read data
Slow calculations
Reference data
Product catalog
```

---

# 56. Cache Invalidation

The classic problem:

```text
Cache has old value
Database has new value
```

Strategies include:

```text
TTL
Explicit eviction
Write-through
Cache-aside
Event-based invalidation
```

Choose based on consistency requirements.

---

# 57. Cache-Aside

Common pattern:

```text
Read
 ↓
Cache?
 ├── Yes → return
 └── No
      ↓
   Database
      ↓
   Cache
      ↓
   return
```

---

# 58. Security

Typical Spring Boot security architecture:

```text
Client
  ↓
Authentication
  ↓
Authorization
  ↓
Controller
  ↓
Service
```

Use:

```text
Spring Security
JWT/OAuth2 where appropriate
Method-level authorization
Password hashing
Secure configuration
```

---

# 59. Password Storage

Never store:

```text
password = "hello123"
```

Use a password hashing algorithm through Spring Security.

Example:

```java
PasswordEncoder encoder =
    new BCryptPasswordEncoder();
```

Then:

```java
String hash =
    encoder.encode(password);
```

Store the hash, not the plaintext password.

---

# 60. Security at Service Layer

Do not rely only on the UI.

Bad:

```text
Frontend hides ADMIN button
```

A malicious client can still call:

```text
POST /api/admin/products
```

Authorization must be enforced server-side.

---

# 61. Method-Level Security

Example:

```java
@PreAuthorize(
    "hasRole('ADMIN')"
)
public void deleteProduct(Long id) {
    ...
}
```

This can protect business operations directly.

---

# 62. CORS

CORS controls which browser origins can make cross-origin requests.

Example:

```text
Frontend:
https://example.com

Backend:
https://api.example.com
```

Configure CORS deliberately.

Avoid:

```text
allow all origins
```

in production unless there is a specific, understood reason.

---

# 63. CSRF

CSRF protection is particularly relevant to browser-based authentication using cookies/session mechanisms.

For stateless APIs using bearer tokens in the Authorization header, the CSRF threat model is different.

Do not disable CSRF blindly; understand how the application authenticates users.

---

# 64. Dependency Management

Use a dependency management strategy such as Spring Boot's dependency management/BOM.

Avoid manually specifying versions for every Spring dependency when Spring Boot already manages compatible versions.

---

# 65. Dependency Hygiene

Regularly check:

```text
Unused dependencies
Vulnerable dependencies
Outdated libraries
Duplicate dependencies
```

Tools can include:

```text
Maven
OWASP Dependency-Check
Snyk
GitHub Dependabot
```

---

# 66. Maven Build Lifecycle

Common commands:

```bash
mvn clean
mvn test
mvn package
mvn verify
```

Typical flow:

```text
Compile
  ↓
Test
  ↓
Package
  ↓
Verify
```

---

# 67. Unit Testing

Test individual components.

Example:

```java
@ExtendWith(MockitoExtension.class)
class ProductServiceTest {

    @Mock
    ProductRepository repository;

    @InjectMocks
    ProductService service;
}
```

Test:

```text
Product exists
Product missing
Invalid business state
```

---

# 68. Integration Testing

Integration tests verify multiple components together.

Example:

```java
@SpringBootTest
class ProductIntegrationTest {
}
```

Useful for:

```text
Database integration
Spring configuration
Repository behavior
Security configuration
HTTP integration
```

---

# 69. Controller Testing

`MockMvc` can test MVC endpoints.

Example:

```java
mockMvc.perform(
    get("/api/products/101")
)
.andExpect(
    status().isOk()
);
```

This is useful for testing:

```text
HTTP status
JSON response
Validation
Security
Controller mapping
```

---

# 70. Testcontainers

Testcontainers can run real dependencies in containers during tests.

Example:

```text
JUnit
  ↓
Testcontainers
  ↓
MySQL/PostgreSQL
```

This can make integration tests closer to production behavior than relying entirely on mocks.

---

# 71. Mock vs Real Dependency

Use mocks when testing:

```text
Business logic in isolation
```

Use real dependencies when testing:

```text
Database integration
Repository queries
Infrastructure behavior
```

Do not mock everything.

---

# 72. Test Pyramid

A healthy test suite often contains:

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

---

# 73. API Documentation

OpenAPI can document:

```text
Endpoints
Request schemas
Response schemas
Status codes
Authentication
```

Spring projects commonly use an OpenAPI integration such as springdoc-openapi.

---

# 74. Actuator

Spring Boot Actuator exposes operational information.

Common endpoint:

```text
/actuator/health
```

Other endpoints may include:

```text
/actuator/metrics
/actuator/info
```

Expose only what is appropriate for the environment.

---

# 75. Production Monitoring

Monitor:

```text
CPU
Memory
JVM
GC
HTTP latency
Error rate
Database connections
Database latency
External API latency
Queue depth
```

---

# 76. CI/CD

A typical pipeline:

```text
Git push
   ↓
Build
   ↓
Unit tests
   ↓
Static analysis
   ↓
Integration tests
   ↓
Package
   ↓
Security checks
   ↓
Deploy
```

Tools may include:

```text
GitHub Actions
Jenkins
GitLab CI
SonarQube
Docker
Kubernetes
```

---

# 77. Code Quality

Use tools such as:

```text
Checkstyle
SpotBugs
SonarQube
PMD
```

Focus on:

```text
Maintainability
Security
Duplication
Complexity
Code smells
```

---

# 78. Git Best Practices

Good commits:

```text
Add product validation
Implement order creation
Fix duplicate SKU handling
Add product repository tests
```

Avoid:

```text
changes
update
final
test
stuff
```

Keep commits focused.

---

# 79. Environment Separation

Do not use production configuration locally.

Example:

```text
Local
→ local database

Test
→ test database

Production
→ managed production database
```

Use externalized configuration and secrets.

---

# 80. Graceful Shutdown

Production applications should shut down cleanly.

Conceptually:

```text
Shutdown signal
      ↓
Stop accepting new work
      ↓
Finish active work
      ↓
Close resources
      ↓
Exit
```

Spring Boot supports graceful shutdown configuration.

---

# 81. API Timeout Strategy

For external dependencies:

```text
Connect timeout
Read timeout
Retry policy
Circuit breaker
```

Do not allow:

```text
Infinite wait
```

---

# 82. Performance Optimization

Do not optimize blindly.

Measure first:

```text
Latency
CPU
Memory
Database queries
GC
Thread pools
Connection pools
External dependencies
```

Then optimize the actual bottleneck.

---

# 83. Common Performance Problems

Watch for:

```text
N+1 queries
Missing indexes
Large result sets
Unbounded pagination
Connection pool exhaustion
Slow external APIs
Excessive logging
Large object graphs
Memory leaks
```

---

# 84. Clean Code Principle

Prefer readable code.

Bad:

```java
if(a!=null&&a.getB()!=null&&a.getB().getC()!=null){
    ...
}
```

Better:

```java
Optional<C> value =
    findC(a);
```

or redesign the model/API to avoid excessive nested null checks.

Do not use `Optional` everywhere; use it where absence is meaningful.

---

# 85. Avoid Overengineering

Do not add:

```text
Kafka
Redis
Gateway
Service discovery
10 microservices
```

to a simple CRUD application without a real requirement.

A good engineer chooses the simplest architecture that satisfies the requirements.

---

# 86. YAGNI

YAGNI means:

```text
You Aren't Gonna Need It
```

Do not build features solely because they might be useful someday.

Build for:

```text
Known requirements
Expected scale
Actual business needs
```

---

# 87. DRY

DRY:

```text
Don't Repeat Yourself
```

Extract shared logic when duplication is meaningful.

But don't create an abstraction merely because two lines happen to look similar.

---

# 88. KISS

KISS:

```text
Keep It Simple
```

Prefer:

```text
Simple design
Clear naming
Small methods
Focused classes
```

over unnecessary abstraction.

---

# 89. SOLID

Important principles:

```text
S → Single Responsibility
O → Open/Closed
L → Liskov Substitution
I → Interface Segregation
D → Dependency Inversion
```

Spring's dependency injection naturally supports many of these principles.

---

# 90. Dependency Inversion

Instead of tightly coupling business logic to concrete implementations:

```text
OrderService
    ↓
ConcretePaymentService
```

depend on abstractions where appropriate:

```text
OrderService
    ↓
PaymentService interface
    ↑
StripePaymentService
MockPaymentService
```

This improves testing and flexibility.

---

# 91. Interface Usage

Don't create interfaces mechanically for every class.

Useful:

```text
PaymentProcessor
    ├── StripePaymentProcessor
    └── RazorpayPaymentProcessor
```

Less useful:

```text
ProductService
    └── ProductServiceImpl
```

when there is no meaningful need for multiple implementations or abstraction.

---

# 92. Dependency Injection

Spring manages dependencies:

```text
ProductController
       |
       v
ProductService
       |
       v
ProductRepository
```

This reduces manual object construction and improves testability.

---

# 93. Avoid Circular Dependencies

Bad:

```text
Service A
   ↓
Service B
   ↓
Service A
```

Circular dependencies usually indicate poor design.

Refactor by:

```text
Extracting shared logic
Changing responsibilities
Introducing a domain service
Using events where appropriate
```

---

# 94. Package Visibility

Keep implementation details as private as practical.

Expose:

```text
Public API
```

Keep internal classes and methods package-private/private where possible.

This reduces accidental coupling.

---

# 95. Naming

Good:

```text
createOrder()
calculateTotal()
reserveInventory()
findProductBySku()
```

Bad:

```text
doStuff()
process()
handle()
execute()
```

unless the generic name is genuinely appropriate.

---

# 96. Error Handling

Good:

```text
Specific exception
Specific code
Specific HTTP status
Useful message
```

Bad:

```text
catch(Exception e)
return 400
```

for every failure.

---

# 97. API Consistency

Keep conventions consistent:

```text
Naming
Pagination
Error responses
Authentication
HTTP statuses
Date formats
JSON structure
```

Consistency reduces client complexity.

---

# 98. Date and Time

Prefer modern Java time types:

```java
Instant
LocalDate
LocalDateTime
OffsetDateTime
ZonedDateTime
```

Choose based on semantics.

For timestamps representing an absolute point in time:

```text
Instant
```

is often appropriate.

---

# 99. Database Migrations

Use a migration tool such as:

```text
Flyway
Liquibase
```

instead of manually changing production schemas.

Migration flow:

```text
Version 1
   ↓
Version 2
   ↓
Version 3
   ↓
Production
```

---

# 100. Migration Best Practices

Migrations should be:

```text
Versioned
Repeatable where appropriate
Reviewed
Tested
Backward-compatible when needed
```

Avoid destructive schema changes without a migration strategy.

---

# 101. Zero-Downtime Schema Changes

For a deployed application:

```text
Old application
      ↓
Database change
      ↓
New application
```

A safer approach can be:

```text
Add new column
      ↓
Deploy compatible application
      ↓
Backfill data
      ↓
Switch reads/writes
      ↓
Remove old column later
```

This is often called an expand-and-contract approach.

---

# 102. Production Readiness

Before production, check:

```text
Configuration
Secrets
Database migrations
Health checks
Logging
Metrics
Tracing
Error handling
Security
Timeouts
Resource limits
Backups
Rollback strategy
```

---

# 103. Code Review Checklist

When reviewing a Spring Boot change:

```text
□ Is responsibility clear?
□ Is business logic in the correct layer?
□ Are DTOs used appropriately?
□ Are exceptions handled consistently?
□ Are transactions correctly scoped?
□ Are queries efficient?
□ Are security checks present?
□ Are tests included?
□ Are logs useful?
□ Are secrets protected?
□ Is the API backward compatible?
□ Is the change unnecessarily complex?
```

---

# 104. Ecommerce Project Structure

For your ecommerce backend, a practical structure is:

```text
src/main/java/com/example/ecommerce
│
├── EcommerceApplication.java
│
├── controller
│   ├── AuthController.java
│   ├── ProductController.java
│   ├── CartController.java
│   └── OrderController.java
│
├── service
│   ├── AuthService.java
│   ├── ProductService.java
│   ├── CartService.java
│   └── OrderService.java
│
├── repository
│   ├── UserRepository.java
│   ├── ProductRepository.java
│   ├── CartRepository.java
│   └── OrderRepository.java
│
├── entity
│   ├── User.java
│   ├── Product.java
│   ├── Cart.java
│   └── Order.java
│
├── dto
│   ├── request
│   └── response
│
├── mapper
│
├── exception
│
├── security
│
└── config
```

---

# 105. Ecommerce Request Flow

Example:

```text
POST /api/orders
        ↓
OrderController
        ↓
CreateOrderRequest
        ↓
@Valid
        ↓
OrderService
        ↓
ProductRepository
        ↓
Inventory validation
        ↓
OrderRepository
        ↓
OrderResponse
        ↓
201 Created
```

---

# 106. What Should the Controller Know?

Controller knows:

```text
HTTP
Request DTO
Response DTO
Path variables
Query parameters
HTTP status
```

Controller should not know:

```text
Database internals
Complex business calculations
Payment workflows
Inventory algorithms
```

---

# 107. What Should the Service Know?

Service knows:

```text
Business rules
Workflow
Transactions
Repository coordination
Domain operations
External service coordination
```

---

# 108. What Should the Repository Know?

Repository knows:

```text
Persistence
Queries
Entity loading
Database-specific operations
```

It should not know:

```text
HTTP
JWT
Email workflows
Controller logic
```

---

# 109. Final Architecture

```text
                         Client
                           |
                           v
                    REST Controller
                           |
                       Request DTO
                           |
                      Validation
                           |
                           v
                     Service Layer
                           |
              +------------+------------+
              |                         |
              v                         v
        Repository Layer          External Services
              |                         |
              v                         v
          Database                APIs / Events
                           |
                           v
                      Response DTO
                           |
                           v
                         Client
```

---

# 110. Final Interview Answer: Explain Your Architecture

> My Spring Boot application follows a layered architecture. Controllers handle HTTP requests and validation, services contain business logic and transaction boundaries, repositories handle persistence, and DTOs separate the API model from database entities. I use centralized exception handling, constructor injection, proper validation, database constraints, and tests around the important business flows. For production, I also focus on observability, security, configuration management, and performance.

---

# 111. Final Interview Answer: Why DTOs?

> I use DTOs to keep the API contract separate from the persistence model. It prevents exposing internal entity fields, gives me control over request and response structures, and allows the database model to evolve without unnecessarily breaking the API.

---

# 112. Final Interview Answer: Why Constructor Injection?

> I prefer constructor injection because dependencies are explicit, fields can be final, and the class becomes easier to unit test without relying on Spring reflection or field injection.

---

# 113. Final Interview Answer: Where Should Business Logic Go?

> Business logic should primarily live in the service or domain layer, not in controllers or repositories. Controllers should handle HTTP concerns, while repositories should focus on persistence.

---

# 114. Final Interview Answer: How Do You Make a Spring Boot Application Production Ready?

> I focus on centralized error handling, validation, secure configuration and secrets, database migrations, health checks, logging, metrics, tracing, appropriate timeouts, connection-pool monitoring, automated tests, CI/CD, and a clear deployment and rollback strategy.

---

# 115. Final Interview Rule

> **Keep controllers thin, services responsible for business workflows, repositories focused on persistence, DTOs separate from entities, and configuration/security/observability outside business logic. Use the simplest architecture that solves the actual problem and avoid adding infrastructure just because it is available.**

Next:

```text
16-spring-boot-interview-questions.md
```

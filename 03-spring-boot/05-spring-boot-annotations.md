# Spring Boot — Important Annotations

Spring Boot applications rely heavily on annotations. An annotation usually tells Spring what a class, method, field, parameter, or configuration element is intended to do.

The goal is not to memorize annotations blindly. Understand:

```text
What does it do?
Where is it used?
Why is it needed?
What problem does it solve?
```

---

# 1. @SpringBootApplication

The main Spring Boot annotation:

```java
@SpringBootApplication
public class EcommerceApplication {

    public static void main(String[] args) {
        SpringApplication.run(
            EcommerceApplication.class,
            args
        );
    }
}
```

It combines:

```text
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

---

# 2. @Configuration

Marks a class as a configuration source.

```java
@Configuration
public class AppConfig {

}
```

It is commonly used with:

```java
@Bean
```

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public Clock clock() {
        return Clock.systemUTC();
    }
}
```

---

# 3. @Bean

Registers the object returned by a method as a Spring bean.

```java
@Configuration
public class AppConfig {

    @Bean
    public Clock clock() {
        return Clock.systemUTC();
    }
}
```

Useful for:

```text
Third-party classes
Custom object creation
Explicit configuration
Multiple implementations
```

---

# 4. @Component

Generic Spring-managed component:

```java
@Component
public class EmailClient {

}
```

Spring discovers it through component scanning.

---

# 5. @Service

Used for business/service logic:

```java
@Service
public class OrderService {

}
```

It is a specialization of:

```java
@Component
```

---

# 6. @Repository

Used for persistence/data-access components:

```java
@Repository
public class ProductRepository {

}
```

With Spring Data JPA, repository interfaces are commonly defined as:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long> {

}
```

Spring Data creates the implementation.

---

# 7. @Controller

Used for Spring MVC controllers:

```java
@Controller
public class HomeController {

}
```

---

# 8. @RestController

Used for REST APIs:

```java
@RestController
public class ProductController {

}
```

It combines:

```java
@Controller
@ResponseBody
```

Returned objects are normally written directly to the HTTP response body.

---

# 9. @ResponseBody

Indicates that a method's return value should be written to the HTTP response body.

```java
@GetMapping("/hello")
@ResponseBody
public String hello() {

    return "Hello";
}
```

Because `@RestController` already includes `@ResponseBody`, you usually do not need it on individual REST controller methods.

---

# 10. @RequestMapping

Defines request mappings.

```java
@RequestMapping("/api/products")
```

Can be used at class level:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

}
```

Then:

```java
@GetMapping("/{id}")
```

maps:

```text
GET /api/products/{id}
```

---

# 11. HTTP Mapping Annotations

Spring provides specialized mappings:

```text
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

Example:

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return service.getProduct(id);
}
```

---

# 12. @GetMapping

Used for HTTP GET requests.

```java
@GetMapping
public List<ProductResponse> getProducts() {

    return service.getProducts();
}
```

Typical purpose:

```text
Read data
```

---

# 13. @PostMapping

Used for HTTP POST requests.

```java
@PostMapping
public ProductResponse create(
        @RequestBody ProductRequest request) {

    return service.create(request);
}
```

Typical purpose:

```text
Create resource
```

---

# 14. @PutMapping

Used for HTTP PUT requests.

```java
@PutMapping("/{id}")
public ProductResponse update(
        @PathVariable Long id,
        @RequestBody ProductRequest request) {

    return service.update(id, request);
}
```

Typically represents replacement/update semantics.

---

# 15. @PatchMapping

Used for partial updates.

```java
@PatchMapping("/{id}")
public ProductResponse updatePrice(
        @PathVariable Long id,
        @RequestBody PriceUpdateRequest request) {

    return service.updatePrice(
        id,
        request
    );
}
```

---

# 16. @DeleteMapping

Used for HTTP DELETE.

```java
@DeleteMapping("/{id}")
public void delete(
        @PathVariable Long id) {

    service.delete(id);
}
```

---

# 17. @PathVariable

Reads a value from the URL path.

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return service.getProduct(id);
}
```

Request:

```text
GET /api/products/101
```

Then:

```text
id = 101
```

---

# 18. @RequestParam

Reads a query parameter.

```java
@GetMapping
public List<ProductResponse> search(
        @RequestParam String category) {

    return service.search(category);
}
```

Request:

```text
GET /api/products?category=electronics
```

Then:

```text
category = electronics
```

---

# 19. Optional Request Parameter

```java
@GetMapping
public List<ProductResponse> search(
        @RequestParam(
            required = false
        )
        String category) {

    return service.search(category);
}
```

You can also define a default:

```java
@RequestParam(
    defaultValue = "all"
)
String category
```

---

# 20. @RequestBody

Converts the HTTP request body into a Java object.

```java
@PostMapping
public ProductResponse create(
        @RequestBody ProductRequest request) {

    return service.create(request);
}
```

JSON:

```json
{
  "name": "Laptop",
  "price": 75000
}
```

Spring commonly uses Jackson for JSON serialization/deserialization.

---

# 21. @RequestHeader

Reads an HTTP header.

```java
@GetMapping
public String getUser(
        @RequestHeader("Authorization")
        String authorization) {

    return authorization;
}
```

Useful for:

```text
Authorization
Correlation ID
Custom headers
Content negotiation
```

---

# 22. @CookieValue

Reads a cookie:

```java
@GetMapping
public String getSession(
        @CookieValue("SESSION_ID")
        String sessionId) {

    return sessionId;
}
```

This is more common in session/cookie-based applications.

---

# 23. @ResponseStatus

Sets the HTTP response status.

```java
@ResponseStatus(HttpStatus.CREATED)
@PostMapping
public ProductResponse create(
        @RequestBody ProductRequest request) {

    return service.create(request);
}
```

For more control over headers/body/status, use:

```java
ResponseEntity<T>
```

---

# 24. ResponseEntity

Allows explicit control over:

```text
Status
Headers
Body
```

Example:

```java
@GetMapping("/{id}")
public ResponseEntity<ProductResponse> get(
        @PathVariable Long id) {

    ProductResponse product =
        service.getProduct(id);

    return ResponseEntity
        .ok(product);
}
```

---

# 25. ResponseEntity Created

```java
@PostMapping
public ResponseEntity<ProductResponse> create(
        @RequestBody ProductRequest request) {

    ProductResponse response =
        service.create(request);

    return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(response);
}
```

---

# 26. @Valid

Triggers Bean Validation on a request object.

```java
@PostMapping
public ProductResponse create(
        @Valid
        @RequestBody ProductRequest request) {

    return service.create(request);
}
```

DTO:

```java
public class ProductRequest {

    @NotBlank
    private String name;

    @Positive
    private BigDecimal price;
}
```

---

# 27. @Validated

`@Validated` is a Spring validation annotation that can be used for validation scenarios beyond basic `@Valid`, including validation groups and method-level validation when configured.

Example:

```java
@Validated
@Service
public class ProductService {

}
```

---

# 28. @NotNull

Value must not be `null`.

```java
@NotNull
private Long productId;
```

It does not reject an empty string.

---

# 29. @NotBlank

For strings:

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

---

# 30. @NotEmpty

Rejects:

```text
null
empty string
empty collection
```

depending on the field type.

Whitespace-only strings are not rejected by `@NotEmpty`.

---

# 31. @Size

Controls size:

```java
@Size(min = 3, max = 50)
private String name;
```

For collections:

```java
@Size(min = 1)
private List<Long> productIds;
```

---

# 32. @Min and @Max

For numeric constraints:

```java
@Min(1)
private int quantity;

@Max(100)
private int discount;
```

---

# 33. @Positive and @PositiveOrZero

```java
@Positive
private BigDecimal price;
```

Requires:

```text
price > 0
```

`@PositiveOrZero` allows:

```text
price >= 0
```

---

# 34. @Email

Validates an email-like string:

```java
@Email
private String email;
```

Usually combine it with:

```java
@NotBlank
```

if null/blank values should also be rejected.

---

# 35. @ExceptionHandler

Handles a specific exception.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

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
}
```

---

# 36. @ControllerAdvice

Provides centralized controller-level behavior.

Common use:

```text
Exception handling
Model binding
Shared controller behavior
```

For REST APIs, commonly use:

```java
@RestControllerAdvice
```

---

# 37. @RestControllerAdvice

Combines controller advice with REST response-body behavior.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

}
```

This is a common choice for centralized REST exception handling.

---

# 38. @CrossOrigin

Controls CORS behavior.

Example:

```java
@CrossOrigin(
    origins = "http://localhost:4200"
)
@RestController
public class ProductController {

}
```

For production applications, prefer a centralized CORS configuration rather than scattering permissive `@CrossOrigin` annotations everywhere.

---

# 39. @Autowired

Used for dependency injection.

Example:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
            paymentService;
    }
}
```

For a single constructor, explicit `@Autowired` is usually unnecessary.

---

# 40. @Qualifier

Selects a specific bean when multiple candidates exist.

```java
public OrderService(
        @Qualifier("stripePaymentService")
        PaymentService paymentService) {

    this.paymentService =
        paymentService;
}
```

---

# 41. @Primary

Marks a bean as the default candidate.

```java
@Service
@Primary
public class StripePaymentService
        implements PaymentService {

}
```

---

# 42. @Lazy

Delays bean initialization.

```java
@Lazy
@Service
public class ExpensiveService {

}
```

Useful when initialization is expensive or the bean is rarely needed.

---

# 43. @Scope

Defines bean scope.

```java
@Component
@Scope("prototype")
public class ReportGenerator {

}
```

Common scopes:

```text
singleton
prototype
request
session
application
websocket
```

---

# 44. @PostConstruct

Runs initialization logic after dependency injection.

```java
@PostConstruct
public void initialize() {

    loadCache();
}
```

Modern Spring Boot applications generally use:

```java
jakarta.annotation.PostConstruct
```

---

# 45. @PreDestroy

Runs cleanup logic before bean destruction.

```java
@PreDestroy
public void cleanup() {

    closeResources();
}
```

Use:

```java
jakarta.annotation.PreDestroy
```

in modern Jakarta-based Spring applications.

---

# 46. @Transactional

Defines a transaction boundary.

```java
@Transactional
public void placeOrder() {

    saveOrder();
    updateInventory();
}
```

If the transaction fails according to the configured transaction rules, the transaction can be rolled back.

Important topics:

```text
Propagation
Isolation
Rollback rules
Read-only transactions
Proxy behavior
```

---

# 47. @Transactional on Class

```java
@Service
@Transactional
public class OrderService {

}
```

This can apply transaction semantics to public methods in the class, subject to Spring's proxying rules.

Method-level configuration can override class-level settings.

---

# 48. @Transactional(readOnly = true)

For read-only service operations:

```java
@Transactional(readOnly = true)
public ProductResponse getProduct(
        Long id) {

    return repository.findById(id)
        .map(mapper::toResponse)
        .orElseThrow();
}
```

`readOnly` is primarily a transaction hint; it should not be treated as a universal guarantee that no writes can occur.

---

# 49. @Entity

Marks a JPA entity.

```java
@Entity
public class Product {

}
```

The entity is mapped to a database table.

---

# 50. @Table

Specifies table details:

```java
@Entity
@Table(name = "products")
public class Product {

}
```

---

# 51. @Id

Marks the primary key:

```java
@Id
private Long id;
```

---

# 52. @GeneratedValue

Defines ID generation strategy.

```java
@Id
@GeneratedValue(
    strategy = GenerationType.IDENTITY
)
private Long id;
```

The exact strategy should match the database and application requirements.

---

# 53. JPA Relationship Annotations

Common relationships:

```text
@OneToOne
@OneToMany
@ManyToOne
@ManyToMany
```

Example:

```java
@ManyToOne(fetch = FetchType.LAZY)
private Category category;
```

Be careful with:

```text
Cascade settings
Fetch strategy
Owning side
JSON serialization
N+1 queries
```

---

# 54. @JoinColumn

Defines a foreign-key column.

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "category_id")
private Category category;
```

---

# 55. @Enumerated

Stores an enum.

Prefer:

```java
@Enumerated(EnumType.STRING)
private OrderStatus status;
```

instead of ordinal storage.

Example:

```java
public enum OrderStatus {
    CREATED,
    PAID,
    SHIPPED,
    CANCELLED
}
```

`EnumType.STRING` is generally safer because changing enum order does not silently change stored numeric meanings.

---

# 56. @Version

Used for optimistic locking.

```java
@Version
private Long version;
```

Useful when multiple transactions may update the same record concurrently.

---

# 57. @Transient

Marks a field that should not be persisted by JPA.

```java
@Transient
private BigDecimal calculatedDiscount;
```

This field is not mapped to a database column.

---

# 58. Spring Data Repository Annotations

Common annotations include:

```text
@Repository
@Query
@Modifying
@Param
```

Example:

```java
@Query("""
    SELECT p
    FROM Product p
    WHERE p.category = :category
""")
List<Product> findByCategory(
    @Param("category")
    String category
);
```

---

# 59. @Modifying

Used for modifying queries such as:

```text
UPDATE
DELETE
```

Example:

```java
@Modifying
@Query("""
    UPDATE Product p
    SET p.price = :price
    WHERE p.id = :id
""")
int updatePrice(
    @Param("id") Long id,
    @Param("price") BigDecimal price
);
```

Such operations normally need appropriate transaction management.

---

# 60. @Param

Binds a method parameter to a named query parameter.

```java
@Query("""
    SELECT p
    FROM Product p
    WHERE p.name = :name
""")
List<Product> findByName(
    @Param("name") String name
);
```

---

# 61. @ConfigurationProperties

Binds structured configuration to a typed object.

```java
@ConfigurationProperties(
    prefix = "payment"
)
public record PaymentProperties(
    String baseUrl,
    Duration timeout
) {}
```

Configuration:

```yaml
payment:
  base-url: https://payment.example.com
  timeout: 5s
```

---

# 62. @ConfigurationPropertiesScan

Enables scanning for configuration property classes.

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class Application {

}
```

---

# 63. @EnableConfigurationProperties

Explicitly registers configuration property classes.

```java
@Configuration
@EnableConfigurationProperties(
    PaymentProperties.class
)
public class PaymentConfig {

}
```

---

# 64. @Profile

Activates a bean/configuration for a specific profile.

```java
@Bean
@Profile("dev")
public PaymentClient fakeClient() {

    return new FakePaymentClient();
}
```

---

# 65. @ConditionalOnProperty

Creates configuration conditionally.

```java
@Bean
@ConditionalOnProperty(
    name = "payment.enabled",
    havingValue = "true"
)
public PaymentClient paymentClient() {

    return new PaymentClient();
}
```

This is commonly used in auto-configuration.

---

# 66. @ConditionalOnMissingBean

Creates a bean only if a matching bean does not already exist.

```java
@Bean
@ConditionalOnMissingBean
public Clock clock() {

    return Clock.systemUTC();
}
```

This supports Spring Boot's default-configuration-and-back-off model.

---

# 67. @ConditionalOnClass

Applies configuration when a class is available on the classpath.

Conceptually:

```text
If dependency exists
    ↓
enable configuration
```

Spring Boot uses this heavily in auto-configuration.

---

# 68. @EnableScheduling

Enables scheduled tasks.

```java
@Configuration
@EnableScheduling
public class SchedulingConfig {

}
```

Then:

```java
@Scheduled(
    fixedRate = 60000
)
public void refresh() {

}
```

---

# 69. @Scheduled

Runs a method according to a schedule.

Example:

```java
@Scheduled(cron = "0 0 * * * *")
public void refreshData() {

}
```

Avoid long-running scheduled tasks without considering concurrency and distributed deployment.

---

# 70. @EnableAsync

Enables Spring's asynchronous method execution.

```java
@Configuration
@EnableAsync
public class AsyncConfig {

}
```

Then:

```java
@Async
public void sendEmail() {

}
```

Production applications should configure appropriate executors instead of relying blindly on defaults.

---

# 71. @Async

Runs a method asynchronously through Spring's task execution infrastructure.

```java
@Async
public void sendNotification() {

}
```

Important:

> `@Async` is proxy-based, so self-invocation from another method in the same bean does not normally trigger the asynchronous proxy behavior.

---

# 72. @Cacheable

Caches a method result.

```java
@Cacheable("products")
public ProductResponse getProduct(
        Long id) {

    return repository.findById(id)
        .map(mapper::toResponse)
        .orElseThrow();
}
```

If configured correctly, repeated calls can use the cache instead of executing the method again.

---

# 73. @CachePut

Always executes the method and updates the cache with the result.

```java
@CachePut(
    value = "products",
    key = "#result.id"
)
public ProductResponse update(
        Long id,
        ProductRequest request) {

    return service.update(
        id,
        request
    );
}
```

---

# 74. @CacheEvict

Removes cache entries.

```java
@CacheEvict(
    value = "products",
    key = "#id"
)
public void delete(Long id) {

    repository.deleteById(id);
}
```

---

# 75. @EnableCaching

Enables Spring's cache abstraction.

```java
@Configuration
@EnableCaching
public class CacheConfig {

}
```

The actual cache provider might be:

```text
Redis
Caffeine
Other supported provider
```

---

# 76. Security Annotations

Common Spring Security annotations include:

```text
@EnableMethodSecurity
@PreAuthorize
@PostAuthorize
@Secured
@RolesAllowed
```

Example:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteProduct(Long id) {

}
```

---

# 77. @EnableMethodSecurity

Enables method-level security.

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

}
```

Then:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteProduct(Long id) {

}
```

---

# 78. @PreAuthorize

Checks authorization before method execution.

```java
@PreAuthorize(
    "hasRole('ADMIN')"
)
public void deleteProduct(Long id) {

}
```

It supports Spring Security expressions.

---

# 79. @PostAuthorize

Checks authorization after method execution.

```java
@PostAuthorize(
    "returnObject.ownerId == authentication.name"
)
public OrderResponse getOrder(
        Long id) {

    return service.getOrder(id);
}
```

Use carefully, especially when returned objects contain sensitive data.

---

# 80. @Secured

Another method-security annotation:

```java
@Secured("ROLE_ADMIN")
public void deleteProduct(Long id) {

}
```

Modern Spring Security applications often use `@PreAuthorize` because it provides more expressive authorization rules.

---

# 81. @MockBean and Modern Testing

Older Spring Boot test examples often use:

```java
@MockBean
```

to add Mockito mocks to the Spring test context.

With newer Spring Boot generations, Spring's testing support also provides:

```text
@MockitoBean
@MockitoSpyBean
```

depending on the Spring Boot/Spring Framework version.

Always check the version used by the project before copying testing annotations from older tutorials.

---

# 82. @SpringBootTest

Loads the Spring Boot application context for integration-style testing.

```java
@SpringBootTest
class ApplicationTest {

}
```

Useful when you want to test broader application wiring.

It is heavier than a simple unit test.

---

# 83. @WebMvcTest

Focuses on the Spring MVC/web layer.

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {

}
```

Useful when testing controller behavior without loading the entire application.

---

# 84. @DataJpaTest

Focuses on JPA repository/database-related testing.

```java
@DataJpaTest
class ProductRepositoryTest {

}
```

Useful for repository behavior.

---

# 85. Important Annotation Groups

### Application

```text
@SpringBootApplication
@Configuration
@Bean
@ComponentScan
@EnableAutoConfiguration
```

### Dependency Injection

```text
@Autowired
@Qualifier
@Primary
```

### Components

```text
@Component
@Service
@Repository
@Controller
@RestController
```

### REST

```text
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
@PathVariable
@RequestParam
@RequestBody
@RequestHeader
@ResponseStatus
```

### Validation

```text
@Valid
@Validated
@NotNull
@NotBlank
@NotEmpty
@Size
@Positive
@Email
```

### Persistence

```text
@Entity
@Table
@Id
@GeneratedValue
@OneToMany
@ManyToOne
@OneToOne
@ManyToMany
@JoinColumn
@Enumerated
@Version
@Transient
```

### Transactions

```text
@Transactional
```

### Configuration

```text
@Value
@ConfigurationProperties
@ConfigurationPropertiesScan
@Profile
@ConditionalOnProperty
@ConditionalOnMissingBean
```

### Lifecycle

```text
@PostConstruct
@PreDestroy
@Lazy
```

### Caching

```text
@EnableCaching
@Cacheable
@CachePut
@CacheEvict
```

### Scheduling / Async

```text
@EnableScheduling
@Scheduled
@EnableAsync
@Async
```

### Security

```text
@EnableMethodSecurity
@PreAuthorize
@PostAuthorize
@Secured
```

### Testing

```text
@SpringBootTest
@WebMvcTest
@DataJpaTest
@MockitoBean
```

---

# 86. Annotation Interview Question

### What is @SpringBootApplication?

Human-written answer:

> `@SpringBootApplication` is a convenience annotation that combines `@Configuration`, `@EnableAutoConfiguration`, and `@ComponentScan`. It is normally placed on the main application class.

---

# 87. @Component vs @Service vs @Repository

Human-written answer:

> All three are Spring component stereotypes, but they communicate different responsibilities. `@Component` is generic, `@Service` is intended for business logic, and `@Repository` is intended for persistence or data access.

---

# 88. @Controller vs @RestController

Human-written answer:

> `@Controller` is mainly used for Spring MVC controllers, while `@RestController` is designed for REST APIs and combines `@Controller` with `@ResponseBody`.

---

# 89. @RequestParam vs @PathVariable

Human-written answer:

> `@PathVariable` reads a value from the URL path, such as `/products/10`, while `@RequestParam` reads a query parameter, such as `/products?id=10`.

---

# 90. @RequestBody

Human-written answer:

> `@RequestBody` tells Spring to deserialize the HTTP request body into a Java object, usually using a message converter such as Jackson for JSON.

---

# 91. @Valid vs @Validated

Human-written answer:

> `@Valid` is commonly used to trigger Bean Validation on an object. `@Validated` is Spring's extension that also supports validation groups and is commonly used for method-level validation.

---

# 92. @Transactional

Human-written answer:

> `@Transactional` defines transaction behavior around a method or class. It lets Spring manage transaction boundaries so related database operations can commit or roll back as a unit according to the configured transaction rules.

---

# 93. @Primary vs @Qualifier

Human-written answer:

> `@Primary` marks a default bean when multiple candidates exist. `@Qualifier` explicitly selects a particular bean at the injection point.

---

# 94. @Bean vs @Component

Human-written answer:

> `@Component` allows Spring to discover a class through component scanning. `@Bean` explicitly registers the object returned by a configuration method. I use `@Bean` especially for third-party or custom-configured objects.

---

# 95. What Is @Profile?

Human-written answer:

> `@Profile` allows a bean or configuration class to be active only for a particular environment, such as development or production.

---

# 96. What Is @ConfigurationProperties?

Human-written answer:

> `@ConfigurationProperties` binds related external configuration values into a typed Java object. I prefer it when multiple related settings need to be managed together.

---

# 97. What Is @Cacheable?

Human-written answer:

> `@Cacheable` allows Spring to cache a method's result. If a matching cache entry exists, the method can be skipped and the cached value returned, depending on the cache configuration.

---

# 98. What Is @Async?

Human-written answer:

> `@Async` tells Spring to execute a method asynchronously using its task-execution infrastructure. Because it is proxy-based, self-invocation within the same bean normally does not trigger the asynchronous behavior.

---

# 99. Common Annotation Mistakes

### Mistake 1

Using:

```java
@RestController
```

and also unnecessarily adding:

```java
@ResponseBody
```

to every method.

### Mistake 2

Using:

```java
@Autowired
```

on every field.

Prefer constructor injection.

### Mistake 3

Using:

```java
@CrossOrigin("*")
```

without understanding the security implications.

### Mistake 4

Putting:

```java
@Transactional
```

randomly on every method.

Transactions should reflect business/data consistency boundaries.

### Mistake 5

Using:

```java
@Cacheable
```

without understanding cache invalidation.

### Mistake 6

Using `@Async` and expecting it to work on self-invocation.

### Mistake 7

Using JPA relationship annotations without considering:

```text
Fetch strategy
N+1 queries
Cascade
Serialization
```

---

# 100. Annotation Selection Cheat Sheet

```text
Need a Spring component?
        ↓
@Component

Business logic?
        ↓
@Service

Persistence?
        ↓
@Repository

MVC controller?
        ↓
@Controller

REST API?
        ↓
@RestController

Explicit bean?
        ↓
@Bean

Dependency?
        ↓
Constructor Injection

Multiple implementations?
        ↓
@Primary / @Qualifier

HTTP path variable?
        ↓
@PathVariable

Query parameter?
        ↓
@RequestParam

JSON request body?
        ↓
@RequestBody

Validate request?
        ↓
@Valid

Centralized REST errors?
        ↓
@RestControllerAdvice

Transaction?
        ↓
@Transactional

External configuration?
        ↓
@ConfigurationProperties

Environment-specific bean?
        ↓
@Profile

Cache?
        ↓
@Cacheable / @CachePut / @CacheEvict

Scheduled task?
        ↓
@Scheduled

Async method?
        ↓
@Async

Method security?
        ↓
@PreAuthorize
```

---

# 101. Final Interview Rule

> **Don't try to memorize every Spring annotation. Group them by responsibility: application configuration, dependency injection, REST, validation, persistence, transactions, security, caching, and testing. If you understand the problem each annotation solves, remembering the syntax becomes much easier.**

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
```

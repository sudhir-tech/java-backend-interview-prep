# Spring Boot — Project Structure

A clean project structure makes a Spring Boot application easier to understand, test, maintain, and scale.

A common backend structure is:

```text
src/
├── main/
│   ├── java/
│   │   └── com.example.ecommerce/
│   │       ├── EcommerceApplication.java
│   │       ├── controller/
│   │       ├── service/
│   │       ├── repository/
│   │       ├── entity/
│   │       ├── dto/
│   │       ├── exception/
│   │       ├── config/
│   │       └── mapper/
│   │
│   └── resources/
│       ├── application.properties
│       ├── application.yml
│       ├── static/
│       └── templates/
│
└── test/
    └── java/
```

---

# 1. Standard Spring Boot Structure

The most common layered structure is:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

Additional packages support the layers:

```text
DTO
Entity
Exception
Configuration
Mapper
Security
```

---

# 2. Main Application Class

Example:

```java
package com.example.ecommerce;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

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

This is normally placed at the root package:

```text
com.example.ecommerce
```

so that component scanning can discover subpackages.

---

# 3. Why Package Placement Matters

Suppose:

```text
com.example.ecommerce
├── EcommerceApplication.java
├── controller
├── service
└── repository
```

Spring's component scanning can discover:

```text
controller
service
repository
```

because they are subpackages.

But if the application class is placed under:

```text
com.example.ecommerce.config
```

then sibling packages may not automatically be scanned.

General rule:

> Keep the main `@SpringBootApplication` class near the root of the application package.

---

# 4. Controller Package

The controller layer handles HTTP requests.

Example:

```text
controller/
└── ProductController.java
```

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

Responsibilities:

```text
Receive HTTP request
Validate/request-bind input
Call service
Return HTTP response
```

Avoid putting complex business logic here.

---

# 5. Service Package

The service layer contains business logic.

Example:

```text
service/
└── ProductService.java
```

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
                    () -> new ProductNotFoundException(id)
                );

        return new ProductResponse(
            product.getId(),
            product.getName(),
            product.getPrice()
        );
    }
}
```

Responsibilities:

```text
Business rules
Transaction boundaries
Orchestration
Calling repositories
Calling external services
Mapping domain data when appropriate
```

---

# 6. Repository Package

Repositories handle data access.

Example:

```text
repository/
└── ProductRepository.java
```

With Spring Data JPA:

```java
@Repository
public interface ProductRepository
        extends JpaRepository<Product, Long> {

}
```

In Spring Data, `@Repository` is often optional for repository interfaces because the framework creates and manages the repository proxy.

Responsibilities:

```text
Database access
Queries
Persistence operations
```

Business rules should generally not live in repositories.

---

# 7. Entity Package

Entities represent persistent domain objects.

Example:

```text
entity/
└── Product.java
```

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy =
        GenerationType.IDENTITY)
    private Long id;

    private String name;

    private BigDecimal price;

    // constructors, getters, setters
}
```

An entity is generally mapped to a database table.

---

# 8. Entity vs DTO

This distinction is important in interviews.

### Entity

Represents:

```text
Database/domain persistence model
```

### DTO

Represents:

```text
API input/output contract
```

Example:

```text
Product Entity
      ↓
ProductResponse DTO
      ↓
JSON Response
```

Do not automatically expose entities directly from every REST endpoint.

---

# 9. DTO Package

DTO means:

```text
Data Transfer Object
```

Typical structure:

```text
dto/
├── ProductRequest.java
└── ProductResponse.java
```

Request:

```java
public class ProductRequest {

    private String name;

    private BigDecimal price;

    // getters/setters
}
```

Response:

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {}
```

DTOs make API contracts explicit.

---

# 10. Request DTO vs Response DTO

It is often useful to keep them separate.

```text
ProductRequest
    ↓
Client → Server

ProductResponse
    ↓
Server → Client
```

Why?

The fields accepted from clients may differ from fields returned by the server.

For example:

```text
Request:
name
price

Response:
id
name
price
createdAt
```

---

# 11. Exception Package

Centralize custom exceptions.

Example:

```text
exception/
├── ProductNotFoundException.java
├── GlobalExceptionHandler.java
└── ErrorResponse.java
```

Custom exception:

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

---

# 12. Global Exception Handler

Use:

```java
@RestControllerAdvice
```

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(
        ProductNotFoundException.class
    )
    public ResponseEntity<ErrorResponse>
    handleProductNotFound(
            ProductNotFoundException ex) {

        ErrorResponse response =
            new ErrorResponse(
                "PRODUCT_NOT_FOUND",
                ex.getMessage()
            );

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(response);
    }
}
```

This avoids repeating exception handling in every controller.

---

# 13. Configuration Package

Application-specific configuration can live under:

```text
config/
```

Examples:

```text
SecurityConfig
RedisConfig
JacksonConfig
WebConfig
OpenApiConfig
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

# 14. Mapper Package

A mapper converts between:

```text
Entity ↔ DTO
```

Example:

```text
mapper/
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

For larger projects, dedicated mapping tools such as MapStruct can reduce repetitive mapping code.

---

# 15. Typical Ecommerce Structure

For an ecommerce backend:

```text
com.example.ecommerce
│
├── controller
│   ├── AuthController
│   ├── ProductController
│   ├── CartController
│   └── OrderController
│
├── service
│   ├── AuthService
│   ├── ProductService
│   ├── CartService
│   └── OrderService
│
├── repository
│   ├── UserRepository
│   ├── ProductRepository
│   ├── CartRepository
│   └── OrderRepository
│
├── entity
│   ├── User
│   ├── Product
│   ├── Cart
│   └── Order
│
├── dto
│   ├── request
│   └── response
│
├── exception
│
├── config
│
└── mapper
```

This is a good structure for a medium-sized backend project.

---

# 16. Request Flow in Ecommerce

For:

```text
POST /api/orders
```

the flow can be:

```text
OrderController
      ↓
OrderService
      ↓
ProductRepository
      ↓
Inventory / business logic
      ↓
OrderRepository
      ↓
Database
```

The controller should not directly coordinate all database operations.

---

# 17. Resources Directory

Spring Boot application resources normally live under:

```text
src/main/resources/
```

Common files:

```text
application.properties
application.yml
messages.properties
static/
templates/
```

---

# 18. application.properties

Example:

```properties
spring.application.name=ecommerce-service

server.port=8080

spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

Avoid committing real credentials.

---

# 19. application.yml

Equivalent YAML:

```yaml
spring:
  application:
    name: ecommerce-service

server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/ecommerce
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

In actual configuration, do not duplicate the `spring:` root key; combine the sections:

```yaml
spring:
  application:
    name: ecommerce-service

  datasource:
    url: jdbc:mysql://localhost:3306/ecommerce
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

server:
  port: 8080
```

---

# 20. Static Directory

```text
src/main/resources/static/
```

can contain static resources such as:

```text
CSS
JavaScript
Images
```

For a backend REST API with a separate Angular frontend, this directory may not be heavily used.

---

# 21. Templates Directory

```text
src/main/resources/templates/
```

is commonly used with server-side template engines such as:

```text
Thymeleaf
```

For a REST backend serving an Angular frontend separately, you may not need it.

---

# 22. Test Structure

Tests normally mirror the main package structure:

```text
src/test/java/
└── com.example.ecommerce/
    ├── controller/
    ├── service/
    └── repository/
```

Example:

```text
src/main/java/
└── com.example.ecommerce/
    └── service/
        └── ProductService.java

src/test/java/
└── com.example.ecommerce/
    └── service/
        └── ProductServiceTest.java
```

This makes tests easy to locate.

---

# 23. Maven Project Structure

A typical Maven Spring Boot project:

```text
ecommerce-backend/
│
├── pom.xml
├── mvnw
├── mvnw.cmd
├── .gitignore
│
└── src/
    ├── main/
    │   ├── java/
    │   └── resources/
    │
    └── test/
        └── java/
```

---

# 24. pom.xml

The Maven configuration contains:

```text
Project metadata
Dependencies
Build plugins
Properties
```

Example:

```xml
<parent>
    <groupId>
        org.springframework.boot
    </groupId>

    <artifactId>
        spring-boot-starter-parent
    </artifactId>

    <version>
        YOUR_SPRING_BOOT_VERSION
    </version>
</parent>
```

For a current project, use the Spring Boot version selected by your project's requirements rather than blindly copying an old version.

---

# 25. Dependency Section

Example:

```xml
<dependencies>

    <dependency>
        <groupId>
            org.springframework.boot
        </groupId>

        <artifactId>
            spring-boot-starter-web
        </artifactId>
    </dependency>

    <dependency>
        <groupId>
            org.springframework.boot
        </groupId>

        <artifactId>
            spring-boot-starter-data-jpa
        </artifactId>
    </dependency>

</dependencies>
```

---

# 26. Build Plugin

Spring Boot Maven projects commonly use:

```xml
<plugin>
    <groupId>
        org.springframework.boot
    </groupId>

    <artifactId>
        spring-boot-maven-plugin
    </artifactId>
</plugin>
```

It supports packaging and running Spring Boot applications.

---

# 27. Layered Architecture

A traditional Spring Boot backend often follows:

```text
Presentation Layer
       ↓
Business Layer
       ↓
Data Access Layer
       ↓
Database
```

Mapped to packages:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

---

# 28. Why Layered Architecture?

Advantages:

```text
Separation of concerns
Testability
Maintainability
Clear responsibilities
Easier debugging
```

For example:

```text
Controller
→ HTTP concerns

Service
→ Business concerns

Repository
→ Persistence concerns
```

---

# 29. Thin Controller Principle

A controller should generally be thin.

Bad:

```java
@PostMapping
public Order createOrder(...) {

    // validate inventory
    // calculate discounts
    // process payment
    // update stock
    // save order
    // send email

}
```

Better:

```java
@PostMapping
public OrderResponse createOrder(
        @RequestBody OrderRequest request) {

    return orderService.createOrder(request);
}
```

Business logic belongs in the service layer.

---

# 30. Fat Service Warning

The service layer should contain business logic, but avoid turning one service into a giant class.

Instead of:

```text
EcommerceService
```

with hundreds of unrelated methods, split responsibilities:

```text
ProductService
CartService
OrderService
PaymentService
InventoryService
```

---

# 31. Repository Responsibility

Repositories should focus on persistence.

Good:

```java
productRepository
    .findByCategory(category);
```

Avoid putting business decisions such as:

```text
Should premium users get 10% discount?
```

inside repository methods.

That belongs in business logic.

---

# 32. Package-by-Layer vs Package-by-Feature

There are two common approaches.

### Package by Layer

```text
controller/
service/
repository/
entity/
dto/
```

### Package by Feature

```text
product/
    ProductController
    ProductService
    ProductRepository
    ProductMapper

order/
    OrderController
    OrderService
    OrderRepository
    OrderMapper
```

---

# 33. Package-by-Feature

For larger applications, package-by-feature can improve cohesion.

Example:

```text
com.example.ecommerce
│
├── product
│   ├── ProductController
│   ├── ProductService
│   ├── ProductRepository
│   ├── Product
│   └── ProductResponse
│
├── order
│   ├── OrderController
│   ├── OrderService
│   ├── OrderRepository
│   └── Order
│
└── user
    ├── UserController
    ├── UserService
    ├── UserRepository
    └── User
```

This keeps related code together.

---

# 34. Which Structure Should You Use?

For a small learning project:

```text
Package by Layer
```

is simple and easy to understand.

For a larger application:

```text
Package by Feature
```

can scale better.

There is no universal rule.

Choose based on:

```text
Team conventions
Application size
Domain boundaries
Maintainability
```

---

# 35. Domain-Based Organization

A mature application may organize around business domains:

```text
order
payment
inventory
customer
product
```

rather than technical layers alone.

This is especially useful in:

```text
Microservices
Modular Monoliths
Domain-Driven Design
```

---

# 36. Configuration Separation

Keep configuration classes separate:

```text
config/
├── SecurityConfig
├── RedisConfig
├── JacksonConfig
└── DatabaseConfig
```

Do not create one giant:

```text
ApplicationConfig
```

with unrelated configuration if the application becomes large.

---

# 37. Security Package

Security configuration can be organized as:

```text
security/
├── SecurityConfig.java
├── JwtAuthenticationFilter.java
├── JwtService.java
└── CustomUserDetailsService.java
```

For larger projects, keeping security-related components together makes the security boundary clearer.

---

# 38. Utility Package

A `util` package can be useful for genuinely generic utilities.

Example:

```text
util/
├── DateUtils
└── StringUtils
```

But avoid using `util` as a dumping ground.

If a class belongs to a specific domain, keep it near that domain.

---

# 39. Constants

Avoid creating huge classes such as:

```text
Constants.java
```

containing unrelated values.

Prefer constants close to the domain that owns them.

For example:

```java
public final class SecurityConstants {

    public static final String
        AUTHORIZATION_HEADER =
        "Authorization";

    private SecurityConstants() {
    }
}
```

---

# 40. Configuration Properties Package

For typed configuration:

```text
config/
└── properties/
    ├── DatabaseProperties
    └── PaymentProperties
```

Example:

```java
@ConfigurationProperties(
    prefix = "payment"
)
public record PaymentProperties(
    String baseUrl,
    Duration timeout
) {}
```

This is cleaner than injecting many unrelated individual strings.

---

# 41. Resources by Environment

A common approach:

```text
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

Shared configuration:

```text
application.yml
```

Environment-specific configuration:

```text
application-dev.yml
application-prod.yml
```

Secrets should preferably come from the environment or a secrets-management system rather than source control.

---

# 42. Logging Configuration

Logging configuration can also be externalized.

For example:

```yaml
logging:
  level:
    root: INFO
    com.example.ecommerce: DEBUG
```

Avoid leaving excessive debug logging enabled in production.

---

# 43. Naming Conventions

Use clear names:

```text
ProductController
ProductService
ProductRepository
ProductRequest
ProductResponse
ProductMapper
ProductNotFoundException
```

Avoid vague names:

```text
CommonService
Helper
Manager
UtilityClass
Processor
```

unless their responsibility is genuinely clear.

---

# 44. Dependency Direction

A clean layered dependency direction is:

```text
Controller
    ↓
Service
    ↓
Repository
```

The repository should not call the controller.

The service should not depend on the controller.

This prevents circular architectural dependencies.

---

# 45. Avoid Circular Dependencies

Bad:

```text
Service A
   ↓
Service B
   ↓
Service A
```

Circular dependencies often indicate unclear responsibilities.

Refactor shared logic into:

```text
Another service
Domain component
Utility
Policy
```

depending on the problem.

---

# 46. Controller Dependencies

A controller may depend on:

```text
Service
Validator
Mapper
```

Example:

```java
@RestController
public class ProductController {

    private final ProductService service;
    private final ProductMapper mapper;

    public ProductController(
            ProductService service,
            ProductMapper mapper) {

        this.service = service;
        this.mapper = mapper;
    }
}
```

Avoid injecting repositories directly into controllers.

---

# 47. Service Dependencies

A service may depend on:

```text
Repository
Other Services
External Client
Mapper
Domain Component
```

Example:

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;

    public OrderService(
            OrderRepository orderRepository,
            InventoryService inventoryService,
            PaymentService paymentService) {

        this.orderRepository =
            orderRepository;

        this.inventoryService =
            inventoryService;

        this.paymentService =
            paymentService;
    }
}
```

---

# 48. Repository Dependencies

A repository usually interacts with:

```text
Database
JPA
Hibernate
JDBC
```

Application code normally calls the repository rather than directly using JDBC everywhere.

---

# 49. Layered Request Example

Request:

```text
POST /api/products
```

Flow:

```text
JSON
 ↓
ProductController
 ↓
ProductRequest
 ↓
ProductService
 ↓
ProductRepository
 ↓
Product Entity
 ↓
Database
 ↓
Product Entity
 ↓
ProductMapper
 ↓
ProductResponse
 ↓
JSON
```

---

# 50. Why DTOs Matter

Suppose the entity contains:

```text
id
name
price
internalCost
supplierId
createdAt
updatedAt
```

The API might only need:

```text
id
name
price
```

Returning the entity directly could expose internal fields.

A response DTO gives explicit control.

---

# 51. Entity Serialization Risk

If JPA entities contain relationships:

```java
@OneToMany
private List<OrderItem> items;
```

directly serializing entities can sometimes lead to:

```text
Large payloads
Lazy loading problems
Circular references
Unintended database access
Tight API/database coupling
```

DTOs can help avoid these problems.

---

# 52. Package Structure for Your Ecommerce Project

A strong structure for your backend:

```text
EcommerceBackend/
│
├── pom.xml
│
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com.example.ecommerce/
    │   │       ├── EcommerceApplication.java
    │   │       │
    │   │       ├── controller/
    │   │       ├── service/
    │   │       ├── repository/
    │   │       ├── entity/
    │   │       ├── dto/
    │   │       ├── exception/
    │   │       ├── config/
    │   │       ├── security/
    │   │       └── mapper/
    │   │
    │   └── resources/
    │       ├── application.yml
    │       └── db/
    │
    └── test/
        └── java/
            └── com.example.ecommerce/
```

---

# 53. Recommended Ecommerce Feature Structure

As the project grows, package-by-feature can become cleaner:

```text
com.example.ecommerce
│
├── auth/
│   ├── AuthController
│   ├── AuthService
│   └── JwtService
│
├── product/
│   ├── ProductController
│   ├── ProductService
│   ├── ProductRepository
│   ├── Product
│   ├── ProductRequest
│   └── ProductResponse
│
├── cart/
│   ├── CartController
│   ├── CartService
│   ├── CartRepository
│   └── Cart
│
├── order/
│   ├── OrderController
│   ├── OrderService
│   ├── OrderRepository
│   └── Order
│
└── common/
    ├── exception/
    ├── config/
    └── mapper/
```

This is closer to how a larger application can be organized.

---

# 54. Modular Monolith

A large application does not always need to become microservices immediately.

A modular monolith can have:

```text
Product Module
Order Module
Payment Module
Inventory Module
User Module
```

inside one Spring Boot application.

Each module can maintain clear boundaries.

This can be a useful architectural step before microservices.

---

# 55. Spring Boot Project Structure — Interview Answer

Question:

### How do you structure a Spring Boot project?

Human-written answer:

> I usually separate the application into controller, service, repository, entity, DTO, exception, and configuration layers. Controllers handle HTTP requests, services contain business logic, repositories handle persistence, and DTOs define the API contract. For larger applications, I prefer organizing code by business feature so related components stay together.

---

# 56. Why Keep Controllers Thin?

Human-written answer:

> Controllers should mainly deal with HTTP concerns. Keeping business logic in services makes the code easier to test, reuse, and maintain.

---

# 57. Why Use DTOs?

Human-written answer:

> DTOs separate the API contract from the database entity. They help control which fields are exposed and make the API less tightly coupled to the persistence model.

---

# 58. Package-by-Layer vs Package-by-Feature

Human-written answer:

> Package-by-layer is simple and works well for smaller applications. For larger applications, package-by-feature can be better because related business components stay together and the codebase becomes easier to navigate.

---

# 59. Where Should @SpringBootApplication Be?

Human-written answer:

> I normally place the main application class at the root package of the application so component scanning can discover controllers, services, repositories, and other components in its subpackages.

---

# 60. Project Structure Checklist

```text
□ Root application package
□ Main @SpringBootApplication class
□ Controller layer
□ Service layer
□ Repository layer
□ Entity/domain layer
□ Request DTOs
□ Response DTOs
□ Exception handling
□ Configuration
□ Security when required
□ Mapper when useful
□ application.yml/properties
□ Test package
□ Maven/Gradle configuration
□ Environment-specific configuration
```

---

# 61. Quick Revision

```text
Spring Boot Project
│
├── src/main/java
│   │
│   └── Root Package
│       ├── Application
│       ├── Controller
│       ├── Service
│       ├── Repository
│       ├── Entity
│       ├── DTO
│       ├── Exception
│       ├── Config
│       ├── Security
│       └── Mapper
│
├── src/main/resources
│   ├── application.yml
│   ├── application.properties
│   ├── static
│   └── templates
│
└── src/test/java
    └── Tests
```

---

# 62. Most Important Interview Concepts

Prioritize:

```text
1. Controller-Service-Repository
2. Entity vs DTO
3. Package-by-layer
4. Package-by-feature
5. Component scanning
6. Root package placement
7. application.properties/YAML
8. Configuration package
9. Exception package
10. Test structure
11. Thin controllers
12. Dependency direction
13. Avoiding circular dependencies
14. Modular organization
15. DTO/API contract separation
```

---

# 63. Final Interview Rule

> **A good Spring Boot project structure is not about creating many packages. It is about keeping responsibilities clear, dependencies flowing in the right direction, and business features easy to find and change.**

Next:

```text
01 Fundamentals
      ↓
02 Project Structure
      ↓
03 Dependency Injection & IoC
      ↓
04 Spring Beans & Configuration
```

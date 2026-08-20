# Spring Boot — Fundamentals

Spring Boot is a framework built on top of the Spring Framework that makes it easier to create production-ready Java applications with minimal configuration.

It is widely used for:

- REST APIs
- Backend applications
- Microservices
- Enterprise applications
- Database-driven applications
- Event-driven systems

Spring Boot focuses on:

```text
Convention over Configuration
+
Auto Configuration
+
Starter Dependencies
+
Embedded Servers
+
Production-ready Features
```

---

# 1. What Is Spring?

Spring is a Java application framework designed around:

```text
IoC — Inversion of Control
DI  — Dependency Injection
AOP — Aspect-Oriented Programming
```

Spring helps developers build loosely coupled and maintainable applications.

Example:

```java
class OrderService {

    private PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Instead of creating the dependency manually:

```java
new PaymentService()
```

Spring can create and inject it.

---

# 2. What Is Spring Boot?

Spring Boot simplifies Spring application development.

Traditional Spring applications often required considerable configuration.

Spring Boot provides:

```text
Auto Configuration
Starter Dependencies
Embedded Server
Externalized Configuration
Production-ready Actuator
```

The goal is:

```text
Create → Configure → Run
```

with minimal boilerplate.

---

# 3. Spring vs Spring Boot

| Spring Framework | Spring Boot |
|---|---|
| Core application framework | Simplifies Spring development |
| More manual configuration | Auto configuration |
| Server often configured separately | Embedded server |
| Dependencies configured individually | Starter dependencies |
| More setup | Faster project creation |
| Flexible | Convention-oriented |

Spring Boot does **not replace Spring**.

It builds on top of Spring.

---

# 4. Why Use Spring Boot?

Common advantages:

### 1. Less configuration

Spring Boot automatically configures many components based on the dependencies available.

### 2. Embedded server

Applications can run directly:

```bash
java -jar application.jar
```

### 3. Starter dependencies

Instead of manually adding many dependencies, use starters.

### 4. Production readiness

Spring Boot provides tools such as:

```text
Actuator
Health Checks
Metrics
Externalized Configuration
```

### 5. Easy integration

Spring Boot works well with:

```text
JPA
Hibernate
Spring Security
Redis
Kafka
MongoDB
Docker
Cloud platforms
```

---

# 5. Spring Boot Architecture

A typical backend application can be organized as:

```text
Client
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

Example:

```text
POST /orders
       ↓
OrderController
       ↓
OrderService
       ↓
OrderRepository
       ↓
MySQL
```

Each layer has a responsibility.

---

# 6. Typical Project Structure

A common structure:

```text
src/
├── main/
│   ├── java/
│   │   └── com.example.app/
│   │       ├── Application.java
│   │       ├── controller/
│   │       ├── service/
│   │       ├── repository/
│   │       ├── entity/
│   │       ├── dto/
│   │       ├── exception/
│   │       └── config/
│   │
│   └── resources/
│       ├── application.properties
│       ├── application.yml
│       ├── static/
│       └── templates/
│
└── test/
```

This is not mandatory.

It is a common convention.

---

# 7. Main Application Class

A Spring Boot application normally starts from a class containing:

```java
@SpringBootApplication
```

Example:

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {

        SpringApplication.run(
            DemoApplication.class,
            args
        );
    }
}
```

---

# 8. What Does @SpringBootApplication Do?

`@SpringBootApplication` is a combination of:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

Conceptually:

```text
@SpringBootApplication
       |
       +-- @Configuration
       |
       +-- @EnableAutoConfiguration
       |
       +-- @ComponentScan
```

---

# 9. @Configuration

Marks a class as a source of Spring bean definitions.

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Spring can use this configuration to create the bean.

---

# 10. @EnableAutoConfiguration

Tells Spring Boot to configure the application automatically based on:

```text
Classpath
+
Configuration
+
Existing Beans
```

For example, if Spring MVC dependencies are present, Spring Boot can configure many MVC components automatically.

---

# 11. @ComponentScan

Tells Spring where to look for Spring-managed components.

For example:

```java
@Component
@Service
@Repository
@Controller
@RestController
```

If the main application class is:

```text
com.example.app
```

Spring commonly scans that package and its subpackages.

---

# 12. Package Structure Matters

Recommended:

```text
com.example.app
├── Application.java
├── controller
├── service
├── repository
└── entity
```

Avoid placing the main application class too deep:

```text
com.example.app.service
    Application.java
```

because component scanning may not discover sibling packages automatically.

---

# 13. Spring Boot Starters

A starter is a convenient dependency bundle.

Examples:

```text
spring-boot-starter-web
spring-boot-starter-data-jpa
spring-boot-starter-security
spring-boot-starter-validation
spring-boot-starter-test
```

Instead of adding every required library manually, starters provide a curated dependency set.

---

# 14. Spring Boot Web Starter

For REST APIs:

```xml
<dependency>
    <groupId>
        org.springframework.boot
    </groupId>

    <artifactId>
        spring-boot-starter-web
    </artifactId>
</dependency>
```

This provides the infrastructure needed for typical Spring MVC web applications.

---

# 15. Spring Data JPA Starter

For database persistence:

```xml
<dependency>
    <groupId>
        org.springframework.boot
    </groupId>

    <artifactId>
        spring-boot-starter-data-jpa
    </artifactId>
</dependency>
```

It integrates:

```text
Spring Data JPA
+
JPA
+
Hibernate
```

with Spring Boot's configuration model.

---

# 16. Spring Boot Starter Test

For testing:

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

It provides commonly used testing infrastructure such as:

```text
JUnit
Mockito
Spring Test
AssertJ
```

depending on the Spring Boot version.

---

# 17. Embedded Server

Spring Boot web applications commonly use an embedded server.

For example:

```text
Tomcat
```

The application can start the server automatically.

You do not need to deploy the application manually to an external application server for a typical Spring Boot REST API.

---

# 18. Running a Spring Boot Application

Using Maven:

```bash
./mvnw spring-boot:run
```

or:

```bash
mvn spring-boot:run
```

After packaging:

```bash
java -jar target/application.jar
```

---

# 19. Maven Wrapper

A generated Spring Boot project commonly contains:

```text
mvnw
mvnw.cmd
```

The Maven Wrapper lets the project use a specified Maven version without requiring the developer to install that exact Maven version globally.

On macOS/Linux:

```bash
./mvnw clean install
```

On Windows:

```bash
mvnw.cmd clean install
```

---

# 20. application.properties

Spring Boot configuration can be placed in:

```text
src/main/resources/application.properties
```

Example:

```properties
server.port=8081

spring.application.name=order-service
```

Then the application runs on:

```text
http://localhost:8081
```

---

# 21. application.yml

YAML can also be used:

```yaml
server:
  port: 8081

spring:
  application:
    name: order-service
```

Do not normally maintain both files with conflicting configuration for the same properties.

---

# 22. Externalized Configuration

Application configuration should not always be hardcoded.

Examples:

```text
Database URL
Username
Password
Port
API keys
Feature flags
Timeouts
```

can be supplied through:

```text
application.properties
application.yml
Environment variables
Command-line arguments
External configuration
```

This allows the same application artifact to run in different environments.

---

# 23. Environment Variables

Example:

```bash
export DB_URL="jdbc:mysql://localhost:3306/orders"
```

Then reference it:

```properties
spring.datasource.url=${DB_URL}
```

This is useful because secrets and environment-specific values do not need to be committed directly into source code.

---

# 24. Profiles

Spring profiles allow environment-specific configuration.

Common profiles:

```text
dev
test
prod
```

Example:

```text
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

Activate a profile:

```properties
spring.profiles.active=dev
```

For production deployments, the active profile is often supplied externally rather than hardcoded.

---

# 25. REST Controller

A REST endpoint can be created using:

```java
@RestController
```

Example:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping
    public String getProducts() {
        return "Products";
    }
}
```

Request:

```text
GET /api/products
```

---

# 26. @RestController

`@RestController` effectively combines:

```java
@Controller
@ResponseBody
```

It tells Spring that the controller methods normally return data directly as the HTTP response body.

For REST APIs, this is the usual choice.

---

# 27. @RequestMapping

Defines a base URL or request mapping.

Example:

```java
@RequestMapping("/api/products")
```

Then:

```java
@GetMapping
```

maps:

```text
GET /api/products
```

and:

```java
@GetMapping("/{id}")
```

maps:

```text
GET /api/products/{id}
```

---

# 28. HTTP Methods

Common mappings:

```java
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

Typical REST usage:

```text
GET     → Read
POST    → Create
PUT     → Replace
PATCH   → Partial update
DELETE  → Delete
```

---

# 29. Service Layer

Business logic should generally live in a service rather than directly inside the controller.

Example:

```java
@Service
public class ProductService {

    public String getProduct() {
        return "Product";
    }
}
```

Controller:

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductService service;

    public ProductController(
            ProductService service) {

        this.service = service;
    }

    @GetMapping
    public String getProduct() {
        return service.getProduct();
    }
}
```

---

# 30. Dependency Injection

Spring injects the service into the controller.

Preferred style:

```java
private final ProductService service;

public ProductController(
        ProductService service) {

    this.service = service;
}
```

This is:

```text
Constructor Injection
```

It is generally preferred over field injection.

---

# 31. Repository Layer

The repository handles persistence operations.

Example:

```java
@Repository
public class ProductRepository {

}
```

With Spring Data JPA, repositories are often interfaces:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long> {

}
```

Spring Data creates the implementation automatically.

---

# 32. Entity

An entity represents a persistent database object.

Example:

```java
@Entity
public class Product {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private double price;
}
```

JPA/Hibernate maps the object to a database table.

---

# 33. DTO

A DTO means:

```text
Data Transfer Object
```

DTOs are commonly used to control what enters or leaves the API.

Example:

```java
public class ProductResponse {

    private Long id;
    private String name;
    private double price;
}
```

Avoid exposing internal entities directly when a dedicated API contract is more appropriate.

---

# 34. Typical Request Flow

A typical Spring Boot API:

```text
HTTP Request
     ↓
Controller
     ↓
DTO Validation
     ↓
Service
     ↓
Repository
     ↓
Database
     ↓
Repository
     ↓
Service
     ↓
Response DTO
     ↓
HTTP Response
```

This separation makes the application easier to test and maintain.

---

# 35. Dependency Injection Example

Instead of:

```java
public class OrderService {

    private PaymentService paymentService =
        new PaymentService();
}
```

use:

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

Spring manages the dependency.

---

# 36. Bean

A Spring Bean is an object that is:

```text
created
+
configured
+
managed
```

by the Spring IoC container.

Examples:

```text
Service
Repository
Controller
Configuration Bean
```

---

# 37. IoC Container

The Spring IoC container manages application objects.

Conceptually:

```text
Application starts
      ↓
Spring creates beans
      ↓
Spring resolves dependencies
      ↓
Spring injects dependencies
      ↓
Application is ready
```

The commonly used container interface is:

```text
ApplicationContext
```

---

# 38. ApplicationContext

`ApplicationContext` provides the Spring application's managed environment.

It handles capabilities such as:

```text
Bean creation
Dependency injection
Configuration
Events
Resource access
```

You usually do not manually retrieve beans from it in normal application code.

Prefer dependency injection.

---

# 39. Bean Lifecycle

Simplified lifecycle:

```text
Bean definition
      ↓
Bean instantiation
      ↓
Dependency injection
      ↓
Initialization callbacks
      ↓
Bean ready
      ↓
Application runs
      ↓
Destruction callbacks
```

Spring manages this lifecycle.

---

# 40. @Component

Generic Spring-managed component:

```java
@Component
public class EmailClient {

}
```

Spring detects it through component scanning.

---

# 41. @Service

`@Service` is a specialization of `@Component` intended for service/business logic.

```java
@Service
public class OrderService {

}
```

It communicates the class's role clearly.

---

# 42. @Repository

`@Repository` indicates a persistence/data-access component.

```java
@Repository
public class ProductRepository {

}
```

With Spring Data, repository interfaces are commonly used instead.

---

# 43. @Controller

Used for Spring MVC controllers.

```java
@Controller
public class HomeController {

}
```

For REST APIs, normally use:

```java
@RestController
```

---

# 44. @Bean

Use `@Bean` when you want to explicitly register an object with Spring.

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

This is useful for:

```text
Third-party classes
Custom configuration
Explicit bean creation
```

---

# 45. Auto Configuration

Spring Boot tries to configure common infrastructure automatically.

Conceptually:

```text
Dependencies
     +
Application configuration
     +
Existing beans
     ↓
Auto Configuration
```

For example, adding a web starter enables a set of web-related auto-configuration behavior.

---

# 46. Conditional Configuration

Spring Boot often configures components conditionally.

Examples of concepts include:

```text
ConditionalOnClass
ConditionalOnMissingBean
ConditionalOnProperty
```

This allows Boot to say:

```text
If dependency X exists
and bean Y does not exist,
configure Z.
```

---

# 47. Overriding Auto Configuration

Auto configuration is not absolute.

If you provide your own appropriate bean or configuration, Spring Boot can often back off from its default configuration.

This is an important interview concept:

```text
Auto Configuration gives defaults,
not restrictions.
```

---

# 48. Spring Boot Startup

Simplified startup:

```text
main()
  ↓
SpringApplication.run()
  ↓
Create ApplicationContext
  ↓
Read configuration
  ↓
Component scanning
  ↓
Auto configuration
  ↓
Create beans
  ↓
Inject dependencies
  ↓
Start embedded server
  ↓
Application ready
```

---

# 49. Embedded Tomcat

For a standard Spring MVC application, Spring Boot can start an embedded servlet container such as Tomcat.

The application therefore behaves as a standalone process:

```bash
java -jar app.jar
```

This is one reason Spring Boot applications are convenient to deploy.

---

# 50. Default Port

Spring Boot's default HTTP port is:

```text
8080
```

Change it:

```properties
server.port=8081
```

or:

```yaml
server:
  port: 8081
```

---

# 51. Health and Production Readiness

Spring Boot applications can use:

```text
Spring Boot Actuator
```

to expose operational information such as:

```text
Health
Metrics
Application information
Mappings
```

Actuator is covered in detail in a later file.

---

# 52. Logging

Spring Boot provides logging infrastructure through its default logging setup.

Example:

```java
private static final Logger log =
    LoggerFactory.getLogger(
        OrderService.class
    );

log.info(
    "Processing order {}",
    orderId
);
```

Prefer structured logging with useful context over `System.out.println()`.

---

# 53. Exception Handling

A REST application should return meaningful HTTP responses when errors occur.

Spring provides:

```text
@ExceptionHandler
@ControllerAdvice
@RestControllerAdvice
```

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(
        ProductNotFoundException.class
    )
    public ResponseEntity<String>
    handleNotFound(
            ProductNotFoundException ex) {

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(ex.getMessage());
    }
}
```

Detailed exception handling is covered later.

---

# 54. Validation

Spring Boot applications commonly use Bean Validation.

Example:

```java
public class ProductRequest {

    @NotBlank
    private String name;

    @Positive
    private double price;
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

Validation deserves its own detailed topic.

---

# 55. Configuration Properties

Instead of scattering configuration values:

```java
private final String url =
    "https://example.com";
```

externalize them:

```properties
payment.url=https://example.com
```

Then bind them using:

```text
@ConfigurationProperties
```

This becomes especially useful for larger applications.

---

# 56. Spring Boot and REST

A common Spring Boot REST stack is:

```text
Spring Boot
    ↓
Spring MVC
    ↓
Jackson
    ↓
JSON
```

Example:

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return service.findById(id);
}
```

Spring converts the returned Java object into JSON using configured HTTP message converters, commonly Jackson for JSON.

---

# 57. Request Mapping Example

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping("/{id}")
    public ProductResponse getProduct(
            @PathVariable Long id) {

        return service.findById(id);
    }

    @PostMapping
    public ProductResponse createProduct(
            @RequestBody ProductRequest request) {

        return service.create(request);
    }

    @DeleteMapping("/{id}")
    public void deleteProduct(
            @PathVariable Long id) {

        service.delete(id);
    }
}
```

---

# 58. Spring Boot and Database

A typical database stack:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Spring Data JPA
    ↓
Hibernate
    ↓
JDBC Driver
    ↓
MySQL
```

Each layer has a different responsibility.

---

# 59. Transaction Management

Business operations involving multiple database changes often require transactions.

Example:

```java
@Transactional
public void placeOrder() {

    createOrder();

    reduceInventory();

    createPayment();
}
```

If the transaction fails, the intended transactional work can be rolled back according to transaction rules.

Detailed transaction behavior is covered separately.

---

# 60. Spring Boot and Microservices

Spring Boot is frequently used to build microservices.

Example:

```text
User Service
Order Service
Payment Service
Inventory Service
```

Each service can be:

```text
independently built
independently deployed
independently scaled
```

Spring Cloud and related infrastructure can be added when distributed-system capabilities are required.

---

# 61. Production Architecture

A typical production backend may look like:

```text
Client
   ↓
Load Balancer
   ↓
API / Spring Boot Application
   ↓
Service Layer
   ↓
Database
```

Additional components:

```text
Redis
Kafka
Monitoring
Logging
Tracing
API Gateway
Service Discovery
```

---

# 62. Spring Boot vs Spring MVC

Spring MVC is a web framework/module inside the Spring ecosystem.

Spring Boot is the application bootstrapping and auto-configuration framework that makes it easier to build and run Spring applications.

Typical relationship:

```text
Spring Boot
    ↓
Spring Framework
    ↓
Spring MVC
```

Spring Boot can configure Spring MVC for you when the appropriate dependencies are present.

---

# 63. Spring Boot vs Spring Cloud

Spring Boot:

```text
Build and run applications.
```

Spring Cloud:

```text
Distributed-system patterns and infrastructure integration.
```

Examples of cloud/distributed concerns:

```text
Service Discovery
Configuration Management
API Gateway
Circuit Breakers
Distributed Systems
```

---

# 64. Why Constructor Injection?

Preferred:

```java
private final OrderService service;

public OrderController(
        OrderService service) {

    this.service = service;
}
```

Advantages:

```text
Dependencies are explicit
Fields can be final
Easier unit testing
No reflection-based field injection requirement
Object cannot exist without required dependencies
```

---

# 65. Field Injection

Example:

```java
@Autowired
private OrderService service;
```

It works, but constructor injection is generally preferred.

Interview answer:

```text
I prefer constructor injection because dependencies are explicit,
the fields can be final, and the class is easier to unit test.
```

---

# 66. Spring Boot Testing

Common testing layers:

```text
Unit Test
Integration Test
Web Layer Test
Repository Test
Full Application Test
```

Common annotations include:

```text
@SpringBootTest
@WebMvcTest
@DataJpaTest
```

Testing is covered in a separate file.

---

# 67. Common Spring Boot Annotations

Important annotations to know:

```text
@SpringBootApplication
@Configuration
@Bean
@Component
@Service
@Repository
@Controller
@RestController
@Autowired
@Qualifier
@Primary
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
@PathVariable
@RequestParam
@RequestBody
@Valid
@Transactional
```

Do not memorize annotations without understanding what problem each solves.

---

# 68. Common Interview Question

### What is Spring Boot?

Human-written answer:

> Spring Boot is built on top of the Spring Framework and simplifies application development by providing auto-configuration, starter dependencies, embedded servers, and production-ready features. It helps us build and deploy Spring applications with much less configuration.

---

# 69. Why Spring Boot Instead of Traditional Spring?

Human-written answer:

> Spring Boot reduces boilerplate configuration. It provides starters, auto-configuration, embedded servers, and sensible defaults, so we can focus more on business logic instead of infrastructure setup.

---

# 70. What Is Auto Configuration?

Human-written answer:

> Auto configuration means Spring Boot automatically configures common application components based on the dependencies available on the classpath, application properties, and existing beans.

---

# 71. What Is @SpringBootApplication?

Human-written answer:

> `@SpringBootApplication` is a convenience annotation that combines `@Configuration`, `@EnableAutoConfiguration`, and `@ComponentScan`.

---

# 72. What Is a Spring Bean?

Human-written answer:

> A Spring Bean is an object whose lifecycle is managed by the Spring IoC container. Spring creates the bean, injects its dependencies, and manages it according to its configuration.

---

# 73. What Is Dependency Injection?

Human-written answer:

> Dependency Injection means an object receives the dependencies it needs from the Spring container instead of creating those dependencies itself. This reduces coupling and makes the code easier to test.

---

# 74. What Is IoC?

Human-written answer:

> Inversion of Control means the responsibility for creating and managing application objects is moved from our code to the Spring container.

---

# 75. What Is a Starter?

Human-written answer:

> A Spring Boot starter is a convenient dependency bundle for a particular capability, such as web development, JPA, security, or testing. It reduces the need to manually manage many related dependencies.

---

# 76. What Is Embedded Server?

Human-written answer:

> Spring Boot can package an application with an embedded web server such as Tomcat, so the application can run as a standalone JAR using `java -jar` without requiring a separately installed application server.

---

# 77. Why Is Spring Boot Popular for Backend Development?

Human-written answer:

> It provides a mature ecosystem, dependency injection, REST API support, database integration, security, testing, monitoring, and microservice support. It also reduces configuration and speeds up development.

---

# 78. Spring Boot Request Flow

A simple interview explanation:

```text
Client
  ↓
DispatcherServlet
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Database
```

The controller handles the HTTP request, the service handles business logic, and the repository handles data access.

---

# 79. DispatcherServlet

For Spring MVC applications, `DispatcherServlet` acts as the front controller.

Simplified flow:

```text
HTTP Request
     ↓
DispatcherServlet
     ↓
Find Handler
     ↓
Controller Method
     ↓
Business Logic
     ↓
Response
```

It coordinates request processing rather than containing the application's business logic itself.

---

# 80. Common Mistakes

### Mistake 1

Putting business logic directly inside controllers.

Prefer:

```text
Controller → Service
```

### Mistake 2

Using field injection everywhere.

Prefer:

```text
Constructor Injection
```

### Mistake 3

Hardcoding environment-specific configuration.

Prefer:

```text
Externalized Configuration
```

### Mistake 4

Returning database entities directly without considering the API contract.

Prefer:

```text
DTOs
```

when appropriate.

### Mistake 5

Using `System.out.println()` for application logging.

Prefer:

```text
Logger
```

### Mistake 6

Putting all application logic into one class.

Keep responsibilities separated.

---

# 81. Production Best Practices

```text
Use constructor injection
Use DTOs for API contracts
Validate incoming requests
Centralize exception handling
Externalize configuration
Never hardcode secrets
Use proper logging
Use transactions where required
Add automated tests
Use Actuator for operational visibility
Keep controllers thin
Keep business logic in services
Use repositories for persistence
```

---

# 82. Fundamentals Checklist

Before moving to advanced Spring Boot topics, understand:

```text
□ Spring vs Spring Boot
□ IoC
□ Dependency Injection
□ Spring Bean
□ ApplicationContext
□ @Component
□ @Service
□ @Repository
□ @Controller
□ @RestController
□ @Configuration
□ @Bean
□ @SpringBootApplication
□ Auto Configuration
□ Component Scanning
□ Starter Dependencies
□ Embedded Server
□ application.properties
□ application.yml
□ Profiles
□ Externalized Configuration
□ Basic REST APIs
□ Controller → Service → Repository
□ DTOs
□ Basic validation
□ Basic exception handling
□ Basic transactions
```

---

# 83. Quick Revision

```text
Spring Boot
│
├── Spring Framework
│   ├── IoC
│   ├── DI
│   └── AOP
│
├── Boot Features
│   ├── Auto Configuration
│   ├── Starters
│   ├── Embedded Server
│   └── Externalized Configuration
│
├── Application Layers
│   ├── Controller
│   ├── Service
│   ├── Repository
│   └── Database
│
├── Configuration
│   ├── properties
│   ├── YAML
│   └── Profiles
│
└── Production
    ├── Actuator
    ├── Logging
    ├── Testing
    └── Monitoring
```

---

# 84. Most Important Interview Concepts

If you have limited time, prioritize:

```text
1. Spring vs Spring Boot
2. IoC and Dependency Injection
3. @SpringBootApplication
4. Auto Configuration
5. Component Scanning
6. Spring Beans
7. Constructor Injection
8. Starter Dependencies
9. Embedded Tomcat
10. REST Controller
11. Controller-Service-Repository architecture
12. application.properties / YAML
13. Profiles
14. Externalized Configuration
15. DispatcherServlet
```

---

# 85. Final Interview Rule

> **Don't describe Spring Boot as just a framework that makes Java development easier. Explain what actually makes it easier: auto-configuration, starter dependencies, embedded servers, externalized configuration, and the Spring ecosystem.**

This foundation will make the next topics much easier:

```text
01 Fundamentals
      ↓
02 Project Structure
      ↓
03 IoC & Dependency Injection
      ↓
04 Beans & Configuration
      ↓
05 Spring Boot Annotations
      ↓
06 Configuration & Profiles
      ↓
07 REST APIs
      ↓
08 Validation
      ↓
09 Exception Handling
      ↓
10 Spring Data JPA
```

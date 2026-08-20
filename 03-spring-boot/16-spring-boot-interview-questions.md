# Spring Boot — Interview Questions and Answers

This file is designed for experienced Java/Spring Boot backend interviews.

The answers are intentionally short, natural, and human-written so they are easier to speak during an interview.

---

# 1. What is Spring Boot?

> Spring Boot is built on top of the Spring Framework and simplifies application development by providing auto-configuration, starter dependencies, embedded servers, and production-ready features. It reduces the amount of configuration we need to write manually.

---

# 2. Why do we use Spring Boot?

Main benefits:

```text
Auto-configuration
Starter dependencies
Embedded server
Externalized configuration
Actuator
Easy testing
Production-ready features
```

Human answer:

> The main reason I use Spring Boot is that it removes a lot of boilerplate configuration and lets me focus more on business logic.

---

# 3. Spring Framework vs Spring Boot

Spring Framework provides:

```text
Dependency Injection
IoC
AOP
MVC
Transaction management
Data access
Security integration
```

Spring Boot adds:

```text
Auto-configuration
Starters
Embedded servers
Externalized configuration
Actuator
Simplified setup
```

Interview answer:

> Spring Framework provides the core features, while Spring Boot makes it much easier to configure and run Spring applications with sensible defaults.

---

# 4. What is IoC?

IoC means:

```text
Inversion of Control
```

Instead of a class creating its dependencies:

```java
ProductService service =
    new ProductService();
```

Spring manages the dependency lifecycle.

```text
Spring Container
      ↓
Creates Bean
      ↓
Injects Dependency
      ↓
Application uses Bean
```

Interview answer:

> IoC means the responsibility for creating and managing objects is transferred from our code to the Spring container.

---

# 5. What is Dependency Injection?

Dependency Injection means a class receives the objects it depends on rather than creating them itself.

Example:

```java
@Service
public class ProductService {

    private final ProductRepository repository;

    public ProductService(
            ProductRepository repository) {

        this.repository = repository;
    }
}
```

Interview answer:

> Dependency Injection allows Spring to provide the required dependencies to a class. I generally prefer constructor injection because the dependencies are explicit and the class is easier to test.

---

# 6. Why Constructor Injection?

Advantages:

```text
Explicit dependencies
Final fields
Easy unit testing
No field reflection required
Prevents partially initialized objects
```

Interview answer:

> I prefer constructor injection because dependencies are explicit, fields can be final, and unit testing becomes straightforward.

---

# 7. What Is a Spring Bean?

A Bean is an object managed by the Spring IoC container.

Examples:

```java
@Service
public class ProductService {
}
```

```java
@Repository
public interface ProductRepository
        extends JpaRepository<Product, Long> {
}
```

Spring creates and manages these objects.

---

# 8. What Is ApplicationContext?

`ApplicationContext` is Spring's IoC container.

It manages:

```text
Bean creation
Dependency injection
Bean lifecycle
Configuration
Events
Resources
```

Interview answer:

> ApplicationContext is the main Spring container responsible for creating and managing beans and resolving their dependencies.

---

# 9. What Is @SpringBootApplication?

It is a convenience annotation combining:

```text
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

Example:

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

---

# 10. What Is Auto-Configuration?

Spring Boot automatically configures components based on:

```text
Classpath
Properties
Existing beans
Application configuration
```

Example:

If Spring Boot detects:

```text
Spring MVC
```

it can automatically configure common MVC infrastructure.

Interview answer:

> Auto-configuration provides sensible defaults based on the dependencies and configuration present in the application.

---

# 11. What Are Starter Dependencies?

Starters provide convenient dependency bundles.

Example:

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

Common starters:

```text
spring-boot-starter-web
spring-boot-starter-data-jpa
spring-boot-starter-security
spring-boot-starter-validation
spring-boot-starter-test
```

---

# 12. Why Embedded Tomcat?

Spring Boot can package an application with an embedded server.

For example:

```text
Spring Boot
    ↓
Embedded Tomcat
    ↓
java -jar application.jar
```

No separate application-server installation is required for a typical executable JAR deployment.

---

# 13. What Is Component Scanning?

Spring scans configured packages for components such as:

```text
@Component
@Service
@Repository
@Controller
@RestController
```

and registers them as beans.

---

# 14. @Component vs @Service

Both can create Spring-managed components.

`@Service` communicates intent:

```text
Business/service layer
```

`@Component` is more generic.

Interview answer:

> Technically `@Service` is a specialized component stereotype. I use it for service classes because it makes the role of the class clearer.

---

# 15. @Repository

`@Repository` indicates a persistence/data-access component.

It also participates in Spring's exception translation mechanism for supported persistence technologies.

Example:

```java
@Repository
public interface ProductRepository
        extends JpaRepository<Product, Long> {
}
```

---

# 16. @Controller vs @RestController

`@Controller` is typically used with Spring MVC views.

`@RestController` is effectively:

```text
@Controller
+
@ResponseBody
```

For REST APIs:

```java
@RestController
```

is usually the natural choice.

---

# 17. What Is @RequestMapping?

It maps HTTP requests to controller methods/classes.

Example:

```java
@RequestMapping("/api/products")
```

Then:

```java
@GetMapping("/{id}")
```

results in:

```text
GET /api/products/{id}
```

---

# 18. @GetMapping vs @PostMapping

Examples:

```java
@GetMapping
```

for retrieving resources.

```java
@PostMapping
```

for creating resources or performing operations represented by POST.

Other mappings:

```text
@PutMapping
@PatchMapping
@DeleteMapping
```

---

# 19. @PathVariable

Used to extract a value from the URL.

```java
@GetMapping("/{id}")
public ProductResponse get(
        @PathVariable Long id) {
    ...
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

# 20. @RequestParam

Used for query parameters.

```java
@GetMapping
public List<ProductResponse> search(
        @RequestParam String category) {
    ...
}
```

Request:

```text
GET /api/products?category=electronics
```

---

# 21. @RequestBody

Maps the request body to a Java object.

```java
@PostMapping
public ProductResponse create(
        @Valid
        @RequestBody CreateProductRequest request) {
    ...
}
```

---

# 22. What Is Bean Validation?

Bean Validation validates object fields against constraints.

Example:

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

---

# 23. @Valid vs @Validated

`@Valid` is commonly used for request-body validation.

`@Validated` is Spring's variant and is useful for:

```text
Method validation
Validation groups
Spring-specific validation scenarios
```

Interview answer:

> For normal request DTO validation I commonly use `@Valid`. I use `@Validated` when I need method-level validation or validation groups.

---

# 24. What Is @RestControllerAdvice?

It provides centralized exception handling for REST controllers.

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
}
```

Then:

```java
@ExceptionHandler(
    ProductNotFoundException.class
)
```

can map exceptions to API responses.

---

# 25. Why Global Exception Handling?

Without centralized handling, controllers may contain repetitive:

```java
try {
    ...
} catch (...) {
    ...
}
```

A global handler gives:

```text
Consistent errors
Centralized logic
Cleaner controllers
Correct HTTP statuses
```

---

# 26. Custom Exception

Example:

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

Then:

```java
throw new ProductNotFoundException(id);
```

---

# 27. What HTTP Status for Missing Resource?

Usually:

```text
404 Not Found
```

Example:

```text
GET /api/products/999
```

when product 999 doesn't exist.

---

# 28. What HTTP Status for Validation Failure?

Usually:

```text
400 Bad Request
```

Example:

```json
{
  "name": "",
  "price": -10
}
```

---

# 29. What HTTP Status for Duplicate Resource?

Commonly:

```text
409 Conflict
```

Example:

```text
SKU already exists
```

---

# 30. 401 vs 403

```text
401 → Authentication is missing/invalid
403 → Authenticated but not allowed
```

Interview answer:

> 401 means the request isn't successfully authenticated. 403 means the user is authenticated but doesn't have permission to perform the operation.

---

# 31. What Is Spring Data JPA?

Spring Data JPA simplifies data access using:

```text
Repositories
Entity mapping
Query methods
JPQL
Specifications
Pagination
Sorting
```

Example:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long> {
}
```

---

# 32. What Is JpaRepository?

`JpaRepository` provides common persistence operations.

Examples:

```text
save()
findById()
findAll()
delete()
count()
existsById()
```

It also supports JPA-specific functionality and pagination/sorting through its inherited interfaces.

---

# 33. What Is an Entity?

An entity is a Java object mapped to a database table.

Example:

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(
        strategy = GenerationType.IDENTITY
    )
    private Long id;

    private String name;
}
```

---

# 34. @Id

Marks the entity's primary key.

```java
@Id
private Long id;
```

---

# 35. @GeneratedValue

Defines how the identifier is generated.

Example:

```java
@GeneratedValue(
    strategy = GenerationType.IDENTITY
)
```

The appropriate strategy depends on the database and application requirements.

---

# 36. Lazy vs Eager Fetching

Lazy:

```text
Relationship loaded when accessed
```

Eager:

```text
Relationship loaded immediately
```

Interview answer:

> I generally prefer lazy loading for associations and explicitly fetch the data required by a use case, because careless eager loading can create large queries and performance problems.

---

# 37. What Is the N+1 Problem?

Example:

```text
Query 100 orders
    ↓
For every order query customer
```

Result:

```text
1 + 100 queries
```

Solutions can include:

```text
Fetch join
Entity graph
DTO projection
Batch fetching
```

---

# 38. What Is a Transaction?

A transaction groups operations into a unit of work.

Example:

```java
@Transactional
public void placeOrder(...) {
    ...
}
```

If the transaction rolls back:

```text
Changes are undone
```

according to the transaction and database semantics.

---

# 39. @Transactional

`@Transactional` defines transactional behavior.

Example:

```java
@Transactional
public void createOrder(...) {
    ...
}
```

Interview answer:

> I place transaction boundaries around business operations that need atomic database behavior, rather than putting `@Transactional` on every method.

---

# 40. Rollback Behavior

By default, Spring's declarative transaction management generally rolls back for:

```text
RuntimeException
Error
```

Checked exceptions do not automatically trigger rollback in the same default way.

Use:

```java
@Transactional(
    rollbackFor = IOException.class
)
```

when appropriate.

---

# 41. What Is Optimistic Locking?

Optimistic locking detects concurrent modifications.

Example:

```java
@Version
private Long version;
```

Flow:

```text
A reads version 5
B reads version 5

A updates → version 6

B tries update using version 5
→ conflict
```

This prevents silent lost updates.

---

# 42. What Is Pessimistic Locking?

Pessimistic locking obtains a database lock to prevent conflicting operations during the transaction.

Example:

```java
@Lock(
    LockModeType.PESSIMISTIC_WRITE
)
```

Use carefully because excessive locking can reduce concurrency.

---

# 43. What Is a DTO?

DTO:

```text
Data Transfer Object
```

It represents data exchanged between application boundaries.

Example:

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {
}
```

---

# 44. Why Not Return Entities Directly?

Returning entities can cause:

```text
Sensitive field exposure
Tight API/database coupling
Serialization problems
Lazy-loading issues
Uncontrolled API evolution
```

DTOs provide a cleaner API contract.

---

# 45. What Is Pagination?

Instead of loading:

```text
100,000 products
```

request:

```text
page = 0
size = 20
```

Example:

```java
Page<Product> products =
    repository.findAll(
        PageRequest.of(0, 20)
    );
```

---

# 46. What Is Caching?

Caching stores frequently accessed data closer to the application.

Example:

```text
Request
  ↓
Redis
  ↓ miss
Database
  ↓
Redis
```

Benefits:

```text
Lower latency
Reduced database load
Better scalability
```

---

# 47. Redis

Redis is commonly used for:

```text
Caching
Rate limiting
Distributed locks
Short-lived data
```

It should not automatically replace the durable database.

---

# 48. What Is Spring Security?

Spring Security provides authentication and authorization features.

It can handle:

```text
Authentication
Authorization
Password hashing
OAuth2
JWT resource-server support
Method security
Security filters
```

---

# 49. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

Example:

```text
Login → Authentication
Delete product → Authorization
```

---

# 50. JWT

JWT:

```text
JSON Web Token
```

A token can carry claims such as:

```text
subject
roles/authorities
expiration
issuer
```

A typical flow:

```text
Login
  ↓
Authentication
  ↓
JWT issued
  ↓
Client sends Authorization header
  ↓
Backend validates token
```

---

# 51. JWT Statelessness

With bearer-token authentication, the server can validate the token without maintaining a traditional server-side session for each request.

This can make horizontal scaling easier.

However, token revocation, expiration, refresh, and key management still need to be designed properly.

---

# 52. Password Hashing

Never store plaintext passwords.

Use:

```java
PasswordEncoder
```

Example:

```java
PasswordEncoder encoder =
    new BCryptPasswordEncoder();

String hash =
    encoder.encode(password);
```

Store:

```text
hash
```

not:

```text
password
```

---

# 53. What Is Spring Boot Actuator?

Actuator provides production-oriented endpoints.

Examples:

```text
/actuator/health
/actuator/metrics
/info
```

Expose only appropriate endpoints.

---

# 54. Health vs Readiness

Liveness:

```text
Should the application be restarted?
```

Readiness:

```text
Should traffic be sent to this instance?
```

These are particularly useful in Kubernetes-style deployments.

---

# 55. What Is Microservices Architecture?

Microservices split an application into independently deployable services around business capabilities.

Example:

```text
User Service
Product Service
Order Service
Payment Service
Inventory Service
```

---

# 56. Microservices Advantages

```text
Independent deployment
Independent scaling
Team ownership
Domain separation
Failure isolation
```

But they also introduce:

```text
Network failures
Distributed transactions
Operational complexity
Observability challenges
Consistency challenges
```

---

# 57. API Gateway

An API Gateway provides a common entry point for clients.

Responsibilities can include:

```text
Routing
Authentication integration
Rate limiting
CORS
Observability
```

Keep business logic in the services.

---

# 58. Service Discovery

Service discovery lets services find available service instances dynamically.

Example:

```text
ORDER-SERVICE
     ↓
Service Discovery
     ↓
PAYMENT-SERVICE instances
```

Technologies can include:

```text
Eureka
Kubernetes Service discovery
Cloud-native discovery
```

---

# 59. Circuit Breaker

Circuit breaker prevents repeated calls to an unhealthy service.

States:

```text
CLOSED
OPEN
HALF_OPEN
```

Interview answer:

> When failures cross a configured threshold, the circuit opens and calls fail fast. After a wait period, limited calls test whether the dependency has recovered.

---

# 60. Retry

Retry can help with temporary failures.

Use:

```text
Limited attempts
Exponential backoff
Jitter
Idempotency
Timeouts
```

Do not blindly retry non-idempotent operations such as payments.

---

# 61. Saga Pattern

Saga handles distributed business workflows through local transactions.

Example:

```text
Create Order
     ↓
Reserve Inventory
     ↓
Process Payment
     ↓
Confirm Order
```

If payment fails:

```text
Cancel Order
     ↓
Release Inventory
```

---

# 62. Outbox Pattern

The Outbox Pattern stores:

```text
Business change
+
Event
```

in the same local database transaction.

Then:

```text
Outbox Publisher
      ↓
Message Broker
```

This reduces the risk of losing an event after a successful database transaction.

---

# 63. Kafka

Kafka is commonly used for:

```text
Event streaming
Asynchronous communication
High-throughput messaging
Data pipelines
```

Concepts:

```text
Topic
Partition
Producer
Consumer
Consumer group
Offset
```

---

# 64. Eventual Consistency

In an event-driven architecture:

```text
Order DB
   ↓
Event
   ↓
Inventory Service
```

the two databases may temporarily have different states.

Eventually:

```text
Events processed
     ↓
State converges
```

---

# 65. Idempotency

If the same event arrives twice:

```text
OrderPaid
OrderPaid
```

the consumer should avoid applying the business effect twice.

Common approach:

```text
eventId
processed-event store
```

---

# 66. Distributed Tracing

Tracing follows a request across services.

```text
Gateway
   ↓
Order
   ↓
Payment
   ↓
Inventory
```

Tools include:

```text
OpenTelemetry
Jaeger
Zipkin
APM platforms
```

---

# 67. Correlation ID

Example:

```text
X-Correlation-Id: ABC123
```

Use it in logs across services.

This makes production troubleshooting easier.

---

# 68. What Is Spring Cloud?

Spring Cloud provides tools and integrations for distributed systems.

Examples:

```text
Spring Cloud Gateway
Spring Cloud Config
Spring Cloud LoadBalancer
OpenFeign
Service discovery integrations
Resilience integrations
```

---

# 69. What Is OpenFeign?

OpenFeign provides a declarative HTTP client.

Example:

```java
@FeignClient(
    name = "payment-service"
)
public interface PaymentClient {

    @PostMapping("/payments")
    PaymentResponse pay(
        @RequestBody PaymentRequest request
    );
}
```

---

# 70. RestClient vs WebClient vs Feign

Simple comparison:

```text
RestClient
→ Synchronous HTTP client

WebClient
→ Non-blocking/reactive HTTP client

OpenFeign
→ Declarative HTTP client
```

Choose based on the application's architecture and requirements.

---

# 71. What Is Spring Cloud Gateway?

It is an API gateway built for Spring applications.

It can provide:

```text
Routing
Filters
Rate limiting
Authentication integration
Observability
```

---

# 72. What Is Spring Cloud Config?

It provides centralized configuration management for distributed applications.

Conceptually:

```text
Config Server
     |
+----+----+----+
|    |    |    |
A    B    C    D
```

---

# 73. How Do You Handle Service Failure?

Human answer:

> I start with sensible timeouts and use retries only for safe transient failures. Depending on the dependency, I can add circuit breakers, bulkheads, asynchronous messaging, and safe fallback behavior to prevent cascading failures.

---

# 74. How Do You Debug Production Issues?

Human answer:

> I first check the impact using metrics such as error rate and latency. Then I use correlation IDs and distributed traces to identify the affected service, check application logs and database performance, and compare the issue with recent deployments or configuration changes.

---

# 75. How Do You Improve Database Performance?

Human answer:

> I first identify the slow query using monitoring or query analysis. Then I check execution plans, indexes, joins, N+1 queries, pagination, and connection-pool behavior. I optimize based on measurements rather than guessing.

---

# 76. How Do You Handle High Traffic?

Human answer:

> I first identify the bottleneck. Depending on the workload, I can use horizontal scaling, caching, database optimization, connection-pool tuning, asynchronous processing, rate limiting, and load balancing.

---

# 77. How Do You Design a Production REST API?

Human answer:

> I start with clear resource-oriented endpoints and DTOs, validate requests, use appropriate HTTP status codes, centralize exception handling, secure the endpoints, add pagination for large datasets, and make the API observable with logs, metrics, and tracing.

---

# 78. How Do You Make a Spring Boot Application Production Ready?

Human answer:

> I focus on configuration and secret management, database migrations, centralized error handling, security, health checks, structured logging, metrics, tracing, appropriate timeouts, automated tests, CI/CD, and a clear deployment and rollback strategy.

---

# 79. How Do You Handle a Career Project in an Interview?

For your ecommerce backend, a natural answer is:

> I built an ecommerce backend using Java, Spring Boot, MySQL, Spring Data JPA, REST APIs, and JWT-based authentication. The application has modules for users, products, cart, and orders. I followed a layered architecture with controllers, services, repositories, DTOs, and centralized exception handling.

---

# 80. Explain Your Ecommerce Architecture

Human answer:

> The client communicates with REST controllers. Controllers validate requests and delegate to the service layer. The service layer contains the business logic and transaction boundaries, while repositories handle database operations using Spring Data JPA. I use DTOs to keep the API model separate from the database entities.

---

# 81. Why Did You Use Spring Boot?

Human answer:

> I chose Spring Boot because it gives me auto-configuration, embedded Tomcat, starter dependencies, easy REST API development, Spring Data integration, security support, and production features through Actuator.

---

# 82. Why MySQL?

Human answer:

> I used MySQL because the ecommerce data is highly relational. Products, users, orders, and order items have clear relationships, and MySQL provides transactions, indexing, constraints, and reliable relational data management.

---

# 83. Why Spring Data JPA?

Human answer:

> Spring Data JPA reduces boilerplate repository code and gives me standard CRUD operations, query methods, pagination, sorting, and integration with Hibernate.

---

# 84. Why DTOs in Your Project?

Human answer:

> I use DTOs so I don't expose JPA entities directly through the API. It gives me control over which fields are exposed and allows the API contract to evolve independently from the database model.

---

# 85. How Did You Handle Exceptions?

Human answer:

> I created meaningful custom exceptions such as product-not-found and insufficient-stock exceptions and handled them centrally using `@RestControllerAdvice`. This gives the API consistent error responses and appropriate HTTP status codes.

---

# 86. How Did You Validate Requests?

Human answer:

> I used Bean Validation annotations such as `@NotBlank`, `@NotNull`, `@Positive`, and `@Size` on request DTOs, then used `@Valid` in the controller. Business rules such as stock availability are validated in the service layer.

---

# 87. How Did You Handle Database Transactions?

Human answer:

> I use `@Transactional` around business operations that need atomic database changes. I keep the transaction boundary around the actual database workflow and avoid holding transactions open unnecessarily during slow external calls.

---

# 88. How Would You Improve the Ecommerce Project?

Good answer:

> I would add stronger automated integration testing, Redis caching for suitable read-heavy data, better observability with metrics and tracing, asynchronous event processing for non-critical workflows, and resilience around external services.

---

# 89. What Was a Difficult Problem?

Natural answer:

> One area I focused on was handling failures cleanly. Instead of letting database or business exceptions reach the client directly, I introduced custom exceptions and centralized exception handling so the API returns meaningful and consistent responses.

---

# 90. What Would You Do Differently?

Human answer:

> If I were taking the project further, I would invest more in integration testing and observability early. It is easy to focus on functionality first, but production debugging becomes much easier when logs, metrics, tracing, and automated tests are designed from the beginning.

---

# 91. Interview: Tell Me About Yourself

For your Java backend profile:

> I'm a Java backend developer with around three years of experience working with Java-based applications. My main experience is with Java, Spring Boot, REST APIs, SQL, and Spring Data JPA. I've also worked with tools such as Git, Jenkins, SonarQube, and ELK. Recently I've been strengthening my DSA, system design, and Spring Boot skills through my master's program and hands-on projects, including an ecommerce backend. I'm now looking for a backend role where I can contribute to production systems and continue growing as a software engineer.

---

# 92. Interview: Why Should We Hire You?

> I have a solid Java backend foundation and practical experience with Spring Boot, REST APIs, databases, debugging, and production environments. I also understand the importance of writing maintainable code, testing properly, and solving problems systematically. I'm actively strengthening my DSA and system-design skills, so I can contribute both to implementation and to solving backend engineering problems.

---

# 93. Interview: Why Are You Looking for a New Role?

> I'm looking for a role where I can take on stronger backend development responsibilities, work on modern Java and Spring Boot systems, and continue growing in areas like system design, distributed systems, and scalable application development.

---

# 94. Interview: Why This Company?

A good structure:

```text
Company
   +
Role
   +
Technology
   +
Growth
```

Example:

> I'm interested because the role aligns well with my Java and backend experience, and I also like the opportunity to work on systems at a larger scale. I'm looking for an environment where I can contribute technically while continuing to grow as a backend engineer.

Customize the company-specific part for each interview.

---

# 95. Interview: Explain a Production Issue You Solved

Use:

```text
Problem
→ Investigation
→ Root cause
→ Fix
→ Result
```

Example:

> We had an application issue where requests were taking longer than expected. I checked the application logs and monitoring dashboards, traced the issue to database behavior, reviewed the query and connection usage, and worked on the fix. After the change, the application behavior improved and the issue stopped recurring.

Keep the answer truthful to your actual experience.

---

# 96. Interview: How Do You Debug a Java Application?

Human answer:

> I first reproduce or understand the issue, then check logs, stack traces, metrics, and recent changes. I narrow the problem down to the application, database, external dependency, or infrastructure layer and then verify the root cause before making the fix.

---

# 97. Interview: How Do You Debug a Memory Problem?

> I first check memory and JVM metrics and look for patterns such as increasing heap usage or frequent garbage collection. Then I would use heap dumps or profiling tools to identify objects retaining memory and confirm whether there is a leak or simply insufficient memory for the workload.

---

# 98. Interview: How Do You Debug High CPU?

> I first confirm the CPU increase through monitoring. Then I check thread activity, application logs, recent deployments, and profiling information to identify CPU-intensive code, loops, excessive serialization, GC activity, or other bottlenecks.

---

# 99. Interview: What Is Your Approach to Code Review?

> I first check correctness and business behavior, then readability, maintainability, error handling, security, performance, and test coverage. I try to give specific feedback and distinguish blocking issues from suggestions.

---

# 100. Interview: How Do You Handle a Disagreement in Code Review?

> I focus on the technical requirement rather than making it personal. I explain the reasoning behind my suggestion, listen to the other approach, and use project standards, tests, or measurements when possible to reach a decision.

---

# 101. Interview: How Do You Prioritize Bugs?

I consider:

```text
Production impact
Number of users affected
Business impact
Security impact
Data integrity
Availability
Workaround availability
```

Human answer:

> I prioritize based on business and production impact. A security issue or data-integrity problem would generally be more urgent than a cosmetic issue.

---

# 102. Interview: What Is Your Strength?

A good backend-oriented answer:

> One of my strengths is systematic debugging. I try to understand the actual root cause instead of immediately applying a workaround. That approach has helped me when working with application logs, databases, and production issues.

---

# 103. Interview: What Is Your Weakness?

Natural answer:

> Earlier I sometimes spent too much time trying to make a solution perfect. I've been improving by first delivering a clean, correct solution and then optimizing it based on actual requirements and measurements.

---

# 104. Interview: Where Do You See Yourself?

> I want to grow into a strong backend engineer who can independently design and build reliable Java services, and gradually take more responsibility for architecture and system design.

---

# 105. Final Interview Preparation Checklist

Before a Java/Spring Boot interview, revise:

```text
Java 17
OOP
Collections
Streams
Exceptions
Multithreading
Concurrency
JVM basics
DSA
SQL
Indexes
Transactions
JPA/Hibernate
N+1
Spring IoC
Dependency Injection
Spring Boot
REST APIs
Validation
Exception handling
Spring Security
JWT
Caching
Redis
Microservices
Kafka
Spring Cloud
API Gateway
Service Discovery
Circuit Breaker
System Design
Testing
Git
CI/CD
Docker
Kubernetes basics
Observability
```

---

# 106. Final Rule for Interview Answers

Don't try to sound like a textbook.

Use this structure:

```text
Direct answer
    ↓
Short explanation
    ↓
Real example
    ↓
Tradeoff if relevant
```

Example:

> `@Transactional` defines a transaction boundary. I normally place it around a business operation that needs atomic database changes. For example, when creating an order and updating related inventory, I use a transaction where those database changes need to succeed or fail together.

That sounds much more natural than giving a long definition.

---

# 107. Final Interview Mindset

```text
Understand
    ↓
Explain simply
    ↓
Give practical example
    ↓
Mention tradeoff
    ↓
Stop
```

Do not keep talking after you have answered the question.

A strong interview answer is usually:

```text
Clear
Concise
Technically correct
Practical
Based on real experience
```

Next:

```text
17-spring-boot-real-world-scenarios.md
```

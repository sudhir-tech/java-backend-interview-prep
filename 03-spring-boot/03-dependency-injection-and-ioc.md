# Spring Boot — Dependency Injection and IoC

Dependency Injection (DI) and Inversion of Control (IoC) are two of the most important concepts in Spring.

If you understand:

```text
IoC
↓
Spring Container
↓
Beans
↓
Dependency Injection
↓
Constructor Injection
```

you understand the foundation of how Spring applications are wired together.

---

# 1. What Is IoC?

IoC means:

```text
Inversion of Control
```

Normally, a class creates the objects it depends on.

Example:

```java
public class OrderService {

    private PaymentService paymentService =
        new PaymentService();
}
```

Here:

```text
OrderService
    ↓
creates PaymentService
```

The class controls the creation of its dependency.

With Spring:

```text
Spring Container
    ↓
creates PaymentService
    ↓
injects it into OrderService
```

The control has been inverted.

---

# 2. Simple Definition of IoC

Interview-friendly definition:

> Inversion of Control means the responsibility for creating and managing application objects is transferred from the application code to a container such as the Spring IoC container.

---

# 3. What Is Dependency Injection?

Dependency Injection is the mechanism used to implement IoC.

Suppose:

```java
class OrderService {

    private PaymentService paymentService;
}
```

`OrderService` depends on:

```text
PaymentService
```

Instead of creating it:

```java
new PaymentService()
```

the dependency is provided from outside.

That is:

```text
Dependency Injection
```

---

# 4. Dependency

A dependency is an object that another object needs to perform its work.

Example:

```java
public class OrderService {

    private final PaymentService paymentService;
}
```

Here:

```text
OrderService
    ↓
depends on
    ↓
PaymentService
```

Therefore:

```text
PaymentService = dependency
```

---

# 5. Without Dependency Injection

Example:

```java
public class OrderService {

    private PaymentService paymentService;

    public OrderService() {
        this.paymentService =
            new PaymentService();
    }
}
```

Problems:

```text
Tight coupling
Difficult testing
Hard to replace implementation
Class controls dependency creation
```

---

# 6. With Dependency Injection

```java
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
            paymentService;
    }
}
```

Now:

```text
OrderService
    ↓
receives dependency
```

It does not create it.

---

# 7. Why DI Is Useful

Dependency Injection provides:

```text
Loose coupling
Better testability
Clear dependencies
Easier maintenance
Easier replacement of implementations
Better separation of concerns
```

---

# 8. Example with Interface

Suppose:

```java
public interface PaymentService {

    void pay(double amount);
}
```

Implementation:

```java
@Service
public class StripePaymentService
        implements PaymentService {

    @Override
    public void pay(double amount) {

        System.out.println(
            "Payment processed"
        );
    }
}
```

Consumer:

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

Spring injects an available implementation.

---

# 9. Interface-Based Design

Prefer depending on:

```text
interface
```

rather than:

```text
concrete implementation
```

Example:

```java
private final PaymentService paymentService;
```

instead of:

```java
private final StripePaymentService paymentService;
```

This improves flexibility.

---

# 10. Three Types of Dependency Injection

Spring commonly supports:

```text
1. Constructor Injection
2. Setter Injection
3. Field Injection
```

Constructor injection is generally preferred for required dependencies.

---

# 11. Constructor Injection

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

Spring sees the constructor and supplies the dependency.

---

# 12. Why Constructor Injection Is Preferred

Advantages:

```text
Dependencies are explicit
Fields can be final
Easy unit testing
Supports immutability
Object cannot be constructed without required dependencies
```

It also makes the class's dependencies visible immediately.

---

# 13. Constructor Injection and Testing

Without DI:

```java
OrderService service =
    new OrderService();
```

may internally create:

```text
PaymentService
```

which makes unit testing harder.

With constructor injection:

```java
PaymentService payment =
    mock(PaymentService.class);

OrderService service =
    new OrderService(payment);
```

The dependency can easily be replaced with a mock.

---

# 14. Setter Injection

Example:

```java
@Service
public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(
            PaymentService paymentService) {

        this.paymentService =
            paymentService;
    }
}
```

Useful when:

```text
Dependency is optional
Dependency may be changed
```

But required dependencies are usually better expressed through constructors.

---

# 15. Field Injection

Example:

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

It is simple but generally not preferred.

Problems include:

```text
Dependencies are less explicit
Fields cannot naturally be final
Plain unit testing is less convenient
Class construction depends on framework injection
```

---

# 16. Interview Answer: Why Constructor Injection?

> I prefer constructor injection because it makes dependencies explicit, allows fields to be final, improves testability, and ensures required dependencies are provided when the object is created.

---

# 17. What Is a Spring Bean?

A Spring Bean is an object that is:

```text
created
configured
and managed
```

by the Spring IoC container.

Examples:

```text
Controller
Service
Repository
Configuration Bean
```

---

# 18. How Does a Class Become a Bean?

Common approaches include:

```java
@Component
@Service
@Repository
@Controller
@RestController
```

or explicitly:

```java
@Bean
```

inside a configuration class.

---

# 19. @Component

Example:

```java
@Component
public class EmailClient {

}
```

Spring detects it through component scanning and registers it as a bean.

---

# 20. @Service

Used for service/business logic:

```java
@Service
public class OrderService {

}
```

`@Service` is a specialization of `@Component`.

---

# 21. @Repository

Used for persistence components:

```java
@Repository
public class ProductRepository {

}
```

It is also a specialization of `@Component`.

Spring Data repository interfaces are usually detected and proxied by Spring Data.

---

# 22. @Controller

Used for MVC controllers:

```java
@Controller
public class HomeController {

}
```

---

# 23. @RestController

Used for REST APIs:

```java
@RestController
public class ProductController {

}
```

It combines the semantics of:

```java
@Controller
@ResponseBody
```

---

# 24. @Bean

`@Bean` explicitly registers an object with Spring.

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

Spring manages the returned `Clock` object.

---

# 25. When Should You Use @Bean?

Use `@Bean` when:

```text
You need explicit configuration
The class is from a third-party library
You need custom construction
You want to control bean creation
```

Example:

```java
@Bean
public ObjectMapper objectMapper() {

    return new ObjectMapper();
}
```

---

# 26. @Component vs @Bean

### @Component

Used directly on the class:

```java
@Component
public class EmailClient {

}
```

Spring discovers it through component scanning.

### @Bean

Used on a method:

```java
@Bean
public EmailClient emailClient() {

    return new EmailClient();
}
```

Spring registers the returned object.

---

# 27. Interview Answer

> `@Component` is used for component scanning on a class, while `@Bean` is used inside configuration to explicitly register an object with the Spring container. I typically use `@Bean` when I need to configure or register a third-party class.

---

# 28. Spring IoC Container

The Spring container is responsible for managing beans.

Conceptually:

```text
Application starts
      ↓
Spring creates ApplicationContext
      ↓
Scans components
      ↓
Creates beans
      ↓
Resolves dependencies
      ↓
Injects dependencies
      ↓
Application is ready
```

---

# 29. ApplicationContext

`ApplicationContext` is a central Spring container interface.

It provides:

```text
Bean management
Dependency injection
Configuration
Application events
Resource loading
```

Example:

```java
ApplicationContext context =
    SpringApplication.run(
        EcommerceApplication.class,
        args
    );
```

Normally, application code should prefer dependency injection over repeatedly calling:

```java
context.getBean(...)
```

---

# 30. Bean Creation

Suppose:

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

Spring needs to create:

```text
OrderService
```

But first it needs:

```text
PaymentService
```

Therefore the dependency graph might look like:

```text
OrderController
       ↓
OrderService
       ↓
PaymentService
```

Spring resolves this graph during application startup.

---

# 31. Dependency Graph

A Spring application can be viewed as a graph:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
DataSource
```

Spring creates the objects and connects the graph.

This is one of the core reasons Spring is called a:

```text
Dependency Injection Container
```

---

# 32. Component Scanning

Spring scans packages for classes such as:

```text
@Component
@Service
@Repository
@Controller
@RestController
```

Example:

```java
@Component
public class EmailClient {

}
```

If the package is scanned, Spring registers:

```text
EmailClient
```

as a bean.

---

# 33. Component Scan Boundary

Suppose:

```text
com.example.app
├── Application.java
├── service
└── repository
```

If the application class is in:

```text
com.example.app
```

Spring can normally scan:

```text
com.example.app.*
```

This is why keeping the main application class at the root package is important.

---

# 34. Explicit Component Scan

You can explicitly configure scanning:

```java
@SpringBootApplication(
    scanBasePackages = "com.example"
)
public class Application {

}
```

But this should not be added unnecessarily.

Good package organization usually makes the default scan sufficient.

---

# 35. Dependency Resolution

Suppose Spring sees:

```java
public OrderService(
        PaymentService paymentService)
```

It asks:

```text
Do I have a PaymentService bean?
```

If exactly one matching bean exists:

```text
Inject it.
```

If none exists:

```text
Application startup fails.
```

If multiple candidates exist:

```text
Spring needs additional information.
```

This leads to:

```text
@Primary
@Qualifier
```

---

# 36. Multiple Implementations

Suppose:

```java
public interface PaymentService {
    void pay(double amount);
}
```

Two implementations:

```java
@Service
public class StripePaymentService
        implements PaymentService {
}
```

and:

```java
@Service
public class PaypalPaymentService
        implements PaymentService {
}
```

Now:

```java
public OrderService(
        PaymentService paymentService) {
}
```

is ambiguous.

Spring sees:

```text
PaymentService
       ↓
StripePaymentService
PaypalPaymentService
```

Which one should it inject?

---

# 37. @Primary

Mark one implementation as the default:

```java
@Service
@Primary
public class StripePaymentService
        implements PaymentService {
}
```

Now Spring chooses it when there is no more specific qualifier.

---

# 38. @Qualifier

Instead of relying on a default, explicitly choose the bean.

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
        @Qualifier("stripePaymentService")
        PaymentService paymentService) {

        this.paymentService =
            paymentService;
    }
}
```

This tells Spring exactly which bean to inject.

---

# 39. @Primary vs @Qualifier

### @Primary

Means:

```text
Use this bean by default.
```

### @Qualifier

Means:

```text
Use this specific bean here.
```

If both apply, a matching qualifier is more specific.

---

# 40. Custom Bean Name

By default, Spring commonly derives a bean name from the class name.

Example:

```java
@Service
public class StripePaymentService {

}
```

Bean name is commonly:

```text
stripePaymentService
```

You can specify a name:

```java
@Service("stripe")
public class StripePaymentService {

}
```

Then:

```java
@Qualifier("stripe")
```

can select it.

---

# 41. Interface Injection

A common pattern:

```java
public interface NotificationService {

    void send(String message);
}
```

Implementation:

```java
@Service
public class EmailNotificationService
        implements NotificationService {

    @Override
    public void send(String message) {

    }
}
```

Consumer:

```java
@Service
public class OrderService {

    private final NotificationService notificationService;

    public OrderService(
            NotificationService notificationService) {

        this.notificationService =
            notificationService;
    }
}
```

The service depends on the abstraction.

---

# 42. Loose Coupling

Without interface-based DI:

```text
OrderService
     ↓
EmailNotificationService
```

With abstraction:

```text
OrderService
     ↓
NotificationService
     ↑
EmailNotificationService
```

This makes implementation replacement easier.

---

# 43. Dependency Inversion Principle

Dependency Injection supports the Dependency Inversion Principle.

High-level modules should depend on abstractions rather than concrete implementations.

Instead of:

```java
private StripePaymentService paymentService;
```

prefer:

```java
private PaymentService paymentService;
```

when the abstraction is meaningful.

---

# 44. Dependency Injection and SOLID

DI is closely related to:

```text
D — Dependency Inversion Principle
```

It can also support:

```text
Single Responsibility
Open/Closed Principle
Interface Segregation
```

by encouraging cleaner boundaries.

---

# 45. Bean Scope

A Spring bean has a scope.

Common scopes:

```text
singleton
prototype
request
session
application
websocket
```

The most common default scope is:

```text
singleton
```

---

# 46. Singleton Scope

By default, Spring creates one bean instance per Spring application context.

Example:

```java
@Service
public class ProductService {

}
```

Normally:

```text
One ProductService instance
per ApplicationContext
```

This does not mean one instance for the entire JVM in every possible setup.

---

# 47. Prototype Scope

Prototype means Spring creates a new bean instance when the bean is requested from the container.

Example:

```java
@Component
@Scope("prototype")
public class ReportGenerator {

}
```

Important:

> Prototype does not automatically mean a new instance every time it is injected into an existing singleton.

This distinction is an important interview point.

---

# 48. Request Scope

In web applications:

```text
One bean instance per HTTP request
```

Example:

```java
@Component
@RequestScope
public class RequestContext {

}
```

Useful for request-specific state.

---

# 49. Session Scope

A session-scoped bean lives for the lifecycle of an HTTP session.

Example:

```java
@Component
@SessionScope
public class UserSession {

}
```

Use carefully in stateless REST architectures.

---

# 50. Singleton Beans and Thread Safety

A singleton bean can be used by many requests concurrently.

Therefore:

```text
Do not store mutable request-specific state
in singleton service fields.
```

Bad:

```java
@Service
public class OrderService {

    private String currentUser;
}
```

This can create concurrency problems.

Prefer method-local variables and immutable/shared-safe state.

---

# 51. Bean Lifecycle

Simplified lifecycle:

```text
Bean Definition
      ↓
Instantiation
      ↓
Dependency Injection
      ↓
Aware callbacks
      ↓
BeanPostProcessor
      ↓
Initialization
      ↓
Ready
      ↓
Destruction
```

The exact lifecycle contains more internal steps, but this model is enough for most interviews.

---

# 52. @PostConstruct

A method can run after dependency injection and initialization.

Example:

```java
@PostConstruct
public void initialize() {

    System.out.println(
        "Bean initialized"
    );
}
```

Use it for lightweight initialization.

---

# 53. @PreDestroy

A method can run before bean destruction.

```java
@PreDestroy
public void cleanup() {

    System.out.println(
        "Cleaning up"
    );
}
```

Avoid using lifecycle callbacks for long-running or fragile shutdown work unless there is a clear need.

---

# 54. Bean Initialization Example

```java
@Component
public class CacheManager {

    @PostConstruct
    public void init() {

        loadCache();
    }

    private void loadCache() {

    }

    @PreDestroy
    public void shutdown() {

        clearResources();
    }

    private void clearResources() {

    }
}
```

---

# 55. @Lazy

By default, singleton beans are generally initialized eagerly during application startup.

You can use:

```java
@Lazy
```

to defer initialization until the bean is needed.

Example:

```java
@Service
@Lazy
public class ExpensiveService {

}
```

Use lazy initialization intentionally because it can move failures from startup time to first use.

---

# 56. Circular Dependency

Example:

```text
ServiceA
   ↓
ServiceB
   ↓
ServiceA
```

This is a circular dependency.

Example:

```java
@Service
class A {

    A(B b) {
    }
}
```

```java
@Service
class B {

    B(A a) {
    }
}
```

This design is usually a sign that responsibilities need to be reconsidered.

---

# 57. How to Fix Circular Dependencies

Possible solutions:

```text
Refactor responsibilities
Extract shared logic
Introduce another service
Redesign dependencies
```

Do not treat:

```text
@Lazy
```

as the default architectural solution.

---

# 58. @Autowired

Spring can use `@Autowired` for dependency injection.

Example:

```java
@Autowired
public OrderService(
        PaymentService paymentService) {

    this.paymentService =
        paymentService;
}
```

For a single constructor, modern Spring can generally use that constructor without explicitly adding `@Autowired`.

Therefore this is usually enough:

```java
public OrderService(
        PaymentService paymentService) {

    this.paymentService =
        paymentService;
}
```

---

# 59. Multiple Constructors

If a class has multiple constructors, Spring may need help identifying the injection constructor.

Example:

```java
@Autowired
public OrderService(
        PaymentService paymentService) {

    this.paymentService =
        paymentService;
}
```

The annotation makes the intended constructor explicit.

---

# 60. Optional Dependencies

Some dependencies may be optional.

One approach is:

```java
@Autowired(required = false)
```

But in modern code, prefer clearer designs such as:

```text
Optional
ObjectProvider
Conditional configuration
Separate implementations
```

depending on the requirement.

---

# 61. ObjectProvider

`ObjectProvider<T>` can be useful when you need controlled access to a dependency or optional/lazy resolution.

Example:

```java
@Component
public class NotificationManager {

    private final ObjectProvider<
        NotificationService> provider;

    public NotificationManager(
            ObjectProvider<
                NotificationService> provider) {

        this.provider = provider;
    }
}
```

This is an advanced dependency-management tool.

---

# 62. @Qualifier with @Bean

You can also qualify configuration beans:

```java
@Configuration
public class PaymentConfig {

    @Bean
    public PaymentClient stripeClient() {
        return new PaymentClient("stripe");
    }

    @Bean
    public PaymentClient paypalClient() {
        return new PaymentClient("paypal");
    }
}
```

Then:

```java
public PaymentService(
        @Qualifier("stripeClient")
        PaymentClient client) {

    this.client = client;
}
```

---

# 63. @Primary with @Bean

```java
@Bean
@Primary
public PaymentClient defaultClient() {

    return new PaymentClient("stripe");
}
```

Now it becomes the default candidate for that type.

---

# 64. BeanFactory

`BeanFactory` is a core Spring container interface.

It provides basic bean management.

`ApplicationContext` builds on the container capabilities and adds more application-level features.

Interview distinction:

```text
BeanFactory
→ basic container

ApplicationContext
→ richer application container
```

In normal Spring Boot applications, you typically work with:

```text
ApplicationContext
```

rather than directly using `BeanFactory`.

---

# 65. Dependency Injection Flow

A useful mental model:

```text
Class
 ↓
Declares dependency
 ↓
Spring scans class
 ↓
Spring registers bean
 ↓
Spring finds required dependency
 ↓
Spring resolves candidate
 ↓
Spring creates/injects dependency
 ↓
Bean becomes ready
```

---

# 66. Constructor Injection Example

Complete example:

```java
public interface PaymentGateway {

    boolean charge(
        BigDecimal amount
    );
}
```

Implementation:

```java
@Service
public class StripePaymentGateway
        implements PaymentGateway {

    @Override
    public boolean charge(
            BigDecimal amount) {

        return true;
    }
}
```

Consumer:

```java
@Service
public class OrderService {

    private final PaymentGateway gateway;

    public OrderService(
            PaymentGateway gateway) {

        this.gateway = gateway;
    }

    public void placeOrder(
            BigDecimal amount) {

        gateway.charge(amount);
    }
}
```

Spring connects:

```text
OrderService
      ↓
PaymentGateway
      ↑
StripePaymentGateway
```

---

# 67. Unit Testing with DI

Constructor injection makes unit testing straightforward:

```java
@Test
void shouldPlaceOrder() {

    PaymentGateway gateway =
        mock(PaymentGateway.class);

    OrderService service =
        new OrderService(gateway);

    service.placeOrder(
        BigDecimal.TEN
    );

    verify(gateway)
        .charge(BigDecimal.TEN);
}
```

The test does not need to start the entire Spring application.

---

# 68. DI vs Service Locator

Service Locator:

```java
PaymentService service =
    context.getBean(
        PaymentService.class
    );
```

Dependency Injection:

```java
public OrderService(
        PaymentService service) {

    this.service = service;
}
```

DI makes dependencies explicit in the class API.

Service Locator hides dependencies behind container lookups.

Constructor injection is generally cleaner for normal application dependencies.

---

# 69. DI vs Manual Object Creation

Manual creation:

```java
new PaymentService()
```

DI:

```java
PaymentService
```

provided by Spring.

Use DI for managed application dependencies.

Manual construction is still appropriate for:

```text
Simple value objects
DTOs
Local helper objects
Objects that should not be Spring-managed
```

Not every object needs to be a Spring bean.

---

# 70. Do We Need Spring for Every Class?

No.

A common mistake is:

```java
@Component
public class EverySmallObject {

}
```

Not every class needs to be a bean.

Use Spring-managed beans when the object participates in:

```text
Application wiring
Configuration
Business services
Persistence
Web layer
Cross-cutting infrastructure
```

Simple immutable objects can often be created normally.

---

# 71. DI and Immutability

Constructor injection works well with immutable fields:

```java
private final PaymentGateway gateway;
```

The reference is assigned once during construction.

This makes the class easier to reason about.

---

# 72. DI and Testability

Good DI:

```text
Production
OrderService
    ↓
RealPaymentGateway

Test
OrderService
    ↓
MockPaymentGateway
```

The business class does not need to know whether the dependency is:

```text
real
mock
fake
stub
alternative implementation
```

---

# 73. DI and Configuration

Spring can inject configuration values too.

Example:

```java
@Value("${payment.timeout}")
private Duration timeout;
```

For larger configuration sets, prefer:

```text
@ConfigurationProperties
```

rather than scattering `@Value` across many classes.

---

# 74. Common IoC Interview Question

### What is the difference between IoC and DI?

Human-written answer:

> IoC is the broader principle where control of object creation and management is transferred to the container. Dependency Injection is the mechanism Spring uses to provide an object's dependencies.

---

# 75. What Is a Spring Container?

Human-written answer:

> The Spring container creates and manages Spring beans, resolves their dependencies, and controls their lifecycle. In a typical Spring Boot application, `ApplicationContext` is the main container we interact with.

---

# 76. What Is a Bean?

Human-written answer:

> A Spring Bean is an object managed by the Spring container. Spring creates it, injects its dependencies, and manages its lifecycle according to its configuration.

---

# 77. Constructor vs Field Injection

Human-written answer:

> I prefer constructor injection because dependencies are explicit, fields can remain final, and unit testing is easier. Field injection hides dependencies and makes plain object construction harder.

---

# 78. @Component vs @Bean

Human-written answer:

> `@Component` lets Spring discover a class through component scanning, while `@Bean` explicitly registers the object returned by a configuration method. I generally use `@Bean` when I need to configure or register a third-party or specially constructed object.

---

# 79. @Primary vs @Qualifier

Human-written answer:

> `@Primary` defines the default bean when multiple candidates exist. `@Qualifier` lets me explicitly select a specific bean at an injection point.

---

# 80. What Happens If Two Beans Have the Same Type?

Human-written answer:

> Spring cannot choose automatically if multiple beans match the required type. I can resolve the ambiguity using `@Primary` for a default implementation or `@Qualifier` when I need a specific implementation.

---

# 81. What Happens If a Required Bean Is Missing?

Human-written answer:

> If Spring cannot find a required dependency, application context creation normally fails during startup because Spring cannot construct the dependent bean.

---

# 82. What Is the Default Bean Scope?

Human-written answer:

> The default scope is singleton, meaning Spring normally creates one bean instance per application context.

---

# 83. Are Singleton Beans Thread-Safe?

Human-written answer:

> Not automatically. Singleton means one shared instance per application context; it does not make the object's mutable state thread-safe. I avoid storing request-specific mutable state in singleton services.

---

# 84. What Is a Circular Dependency?

Human-written answer:

> A circular dependency occurs when Bean A depends on Bean B while Bean B directly or indirectly depends on Bean A. It usually indicates that the responsibilities should be refactored into clearer components.

---

# 85. Common Mistakes

### Mistake 1

Creating Spring-managed dependencies manually:

```java
new OrderService(...)
```

inside production code.

### Mistake 2

Using field injection everywhere.

### Mistake 3

Putting mutable request state inside singleton beans.

### Mistake 4

Using `@Lazy` to hide architectural circular dependencies.

### Mistake 5

Creating every small class as a Spring bean.

### Mistake 6

Depending on concrete implementations unnecessarily.

---

# 86. Best Practices

```text
Use constructor injection
Prefer interfaces for meaningful abstractions
Keep required dependencies explicit
Use final fields
Keep singleton services stateless
Use @Qualifier for explicit implementations
Use @Primary for sensible defaults
Use @Bean for explicit/third-party configuration
Avoid unnecessary application-context lookups
Avoid circular dependencies
Do not make every class a Spring bean
```

---

# 87. Dependency Injection Checklist

```text
□ Understand IoC
□ Understand DI
□ Understand Spring IoC container
□ Understand ApplicationContext
□ Understand Spring Bean
□ Know @Component
□ Know @Service
□ Know @Repository
□ Know @Controller
□ Know @RestController
□ Know @Bean
□ Prefer constructor injection
□ Understand setter injection
□ Understand field injection
□ Know @Primary
□ Know @Qualifier
□ Understand bean scopes
□ Understand singleton scope
□ Understand prototype scope
□ Understand bean lifecycle
□ Know @PostConstruct
□ Know @PreDestroy
□ Understand circular dependencies
□ Understand component scanning
□ Understand dependency resolution
```

---

# 88. Quick Revision

```text
IoC
 ↓
Spring Container
 ↓
Bean Creation
 ↓
Dependency Resolution
 ↓
Dependency Injection
 ↓
Bean Lifecycle
```

Dependency injection styles:

```text
Constructor   ← Preferred
Setter        ← Optional dependencies
Field         ← Generally avoid
```

Multiple implementations:

```text
@Primary
    or
@Qualifier
```

Bean creation:

```text
@Component
@Service
@Repository
@Controller
@RestController
@Bean
```

---

# 89. Final Interview Rule

> **Think of Spring as an object wiring system. Your classes declare what they need, and the Spring container creates the managed objects and connects those dependencies. Constructor injection makes those dependencies explicit and keeps the code easier to test and maintain.**

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
```

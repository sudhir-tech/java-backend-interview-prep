# Spring Boot — Design Patterns

This file covers the design patterns most useful for Java and Spring Boot backend development and interviews.

The goal is not to memorize pattern names. Understand:

```text
Problem
→ Pattern
→ Why it helps
→ Tradeoff
→ Spring Boot example
```

---

# 1. What Is a Design Pattern?

A design pattern is a reusable solution to a recurring software design problem.

It is not a copy-paste implementation.

Examples:

```text
Singleton
Factory
Builder
Strategy
Observer
Adapter
Decorator
Template Method
Proxy
Facade
Repository
Dependency Injection
```

---

# 2. Why Design Patterns Matter

Patterns help with:

```text
Maintainability
Extensibility
Loose coupling
Testability
Readability
Reuse
```

But patterns can also add unnecessary abstraction.

Use a pattern when it solves a real design problem.

---

# 3. SOLID and Design Patterns

Patterns work well with SOLID principles.

```text
S → Single Responsibility
O → Open/Closed
L → Liskov Substitution
I → Interface Segregation
D → Dependency Inversion
```

Spring itself heavily uses dependency inversion, interfaces, proxies, factories, templates, and strategies.

---

# 4. Singleton Pattern

Singleton ensures that a class has one logical instance.

Traditional Java:

```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

But manually implementing Singleton is usually unnecessary in Spring.

---

# 5. Singleton Scope in Spring

Spring beans are singleton by default.

```java
@Service
public class OrderService {
}
```

By default, Spring creates one bean instance per application context.

Important:

> Spring singleton means one bean instance per Spring application context, not one object globally across every JVM or server.

---

# 6. Singleton Thread Safety

A singleton bean may serve many requests concurrently.

Therefore avoid mutable request-specific state:

```java
@Service
public class OrderService {

    private String currentOrderId; // BAD
}
```

Prefer stateless services:

```java
@Service
public class OrderService {

    public OrderResponse create(OrderRequest request) {
        ...
    }
}
```

---

# 7. Factory Pattern

Factory centralizes object creation.

Without factory:

```text
Controller
   |
if type A → new A()
if type B → new B()
if type C → new C()
```

With factory:

```text
Controller
   |
Factory
   |
Implementation
```

The caller doesn't need to know construction details.

---

# 8. Simple Factory Example

```java
public interface PaymentProcessor {
    void process();
}
```

Implementations:

```java
class CardPaymentProcessor implements PaymentProcessor {
    public void process() {
        ...
    }
}

class UpiPaymentProcessor implements PaymentProcessor {
    public void process() {
        ...
    }
}
```

Factory:

```java
@Component
public class PaymentProcessorFactory {

    public PaymentProcessor getProcessor(String type) {

        return switch (type) {
            case "CARD" -> new CardPaymentProcessor();
            case "UPI" -> new UpiPaymentProcessor();
            default -> throw new IllegalArgumentException(
                "Unsupported payment type"
            );
        };
    }
}
```

In Spring, a better implementation can inject a map of beans rather than manually creating dependencies.

---

# 9. Factory with Spring Beans

Example:

```java
public interface PaymentProcessor {

    String type();

    void process(PaymentRequest request);
}
```

Implementations:

```java
@Component
public class CardPaymentProcessor
        implements PaymentProcessor {

    public String type() {
        return "CARD";
    }

    public void process(PaymentRequest request) {
        ...
    }
}
```

```java
@Component
public class UpiPaymentProcessor
        implements PaymentProcessor {

    public String type() {
        return "UPI";
    }

    public void process(PaymentRequest request) {
        ...
    }
}
```

Factory:

```java
@Component
public class PaymentProcessorFactory {

    private final Map<String, PaymentProcessor> processors;

    public PaymentProcessorFactory(
            List<PaymentProcessor> processors) {

        this.processors = processors.stream()
            .collect(Collectors.toMap(
                PaymentProcessor::type,
                Function.identity()
            ));
    }

    public PaymentProcessor get(String type) {
        return processors.get(type);
    }
}
```

This is more extensible than a large `switch`.

---

# 10. Strategy Pattern

Strategy allows different algorithms or behaviors to be selected at runtime.

Example:

```text
Payment
   |
+-- Card Strategy
+-- UPI Strategy
+-- Wallet Strategy
```

Interface:

```java
public interface PaymentStrategy {

    void pay(PaymentRequest request);
}
```

Implementations:

```java
@Component
class CardPaymentStrategy
        implements PaymentStrategy {

    public void pay(PaymentRequest request) {
        ...
    }
}
```

```java
@Component
class UpiPaymentStrategy
        implements PaymentStrategy {

    public void pay(PaymentRequest request) {
        ...
    }
}
```

---

# 11. Strategy vs Factory

They are often used together.

Factory:

```text
Which strategy should I use?
```

Strategy:

```text
How does that behavior work?
```

Example:

```text
PaymentFactory
      ↓
CardPaymentStrategy
```

---

# 12. Strategy Example — Discounts

```java
public interface DiscountStrategy {

    BigDecimal calculate(Order order);
}
```

Implementations:

```text
RegularDiscount
PremiumDiscount
FestivalDiscount
```

Then:

```java
DiscountStrategy strategy =
    discountFactory.get(customerType);

BigDecimal discount =
    strategy.calculate(order);
```

This avoids large conditional logic.

---

# 13. Strategy vs if-else

Before:

```java
if (type.equals("CARD")) {
    ...
} else if (type.equals("UPI")) {
    ...
} else if (type.equals("WALLET")) {
    ...
}
```

After:

```text
type
 ↓
Strategy selection
 ↓
Implementation
```

Benefits:

```text
Open for extension
Less conditional logic
Easier testing
```

---

# 14. Builder Pattern

Builder creates complex objects step by step.

Example:

```java
Order order = Order.builder()
    .customerId(100)
    .items(items)
    .totalAmount(amount)
    .status(OrderStatus.PENDING)
    .build();
```

Useful when an object has:

```text
Many fields
Optional fields
Complex construction
```

---

# 15. Builder with Lombok

Common Spring Boot usage:

```java
@Builder
@Getter
public class OrderResponse {

    private Long id;
    private String status;
    private BigDecimal total;
}
```

Then:

```java
OrderResponse response =
    OrderResponse.builder()
        .id(100L)
        .status("CONFIRMED")
        .total(amount)
        .build();
```

---

# 16. Builder Advantages

```text
Readable construction
Optional parameters
Avoids huge constructors
Immutable objects can be easier to create
```

Tradeoff:

```text
More generated/boilerplate code
Can hide required-field validation if used carelessly
```

---

# 17. Prototype Pattern

Prototype creates new objects based on an existing object.

Conceptually:

```text
Existing Object
      |
     copy
      ↓
New Object
```

Useful when object creation is expensive or when cloning has clear value.

It is less common in typical Spring Boot CRUD applications.

---

# 18. Adapter Pattern

Adapter makes incompatible interfaces work together.

Example:

```text
Your application
      |
   Adapter
      |
Third-party Payment API
```

Suppose your application expects:

```java
interface PaymentGateway {
    PaymentResult charge(PaymentRequest request);
}
```

Third-party library exposes:

```java
ThirdPartyResponse makeCharge(
    ThirdPartyRequest request
);
```

Adapter translates between them.

---

# 19. Adapter Example

```java
@Component
public class PaymentGatewayAdapter
        implements PaymentGateway {

    private final ThirdPartyClient client;

    public PaymentGatewayAdapter(
            ThirdPartyClient client) {
        this.client = client;
    }

    @Override
    public PaymentResult charge(
            PaymentRequest request) {

        ThirdPartyRequest thirdPartyRequest =
            map(request);

        ThirdPartyResponse response =
            client.makeCharge(thirdPartyRequest);

        return map(response);
    }
}
```

Your domain remains independent of the external API.

---

# 20. Why Adapter Is Useful

Without adapter:

```text
Business logic
      ↓
Third-party SDK classes everywhere
```

With adapter:

```text
Business logic
      ↓
Your interface
      ↓
Adapter
      ↓
Third-party API
```

This improves testability and reduces vendor coupling.

---

# 21. Decorator Pattern

Decorator adds behavior around an existing object without changing its core implementation.

Conceptually:

```text
Service
   ↓
Logging Decorator
   ↓
Caching Decorator
   ↓
Original Service
```

Spring uses similar ideas through proxies and interceptors.

---

# 22. Proxy Pattern

Proxy provides an intermediary around an object.

Spring heavily uses proxies for:

```text
@Transactional
@Cacheable
@Async
Method security
AOP
```

Conceptually:

```text
Client
  ↓
Proxy
  ↓
Target Object
```

---

# 23. @Transactional and Proxy

Example:

```java
@Transactional
public void createOrder() {
    ...
}
```

Spring can create a proxy around the bean.

Conceptually:

```text
Caller
  ↓
Spring Proxy
  ↓
Begin transaction
  ↓
Target method
  ↓
Commit / rollback
```

---

# 24. Self-Invocation Problem

Example:

```java
public void methodA() {
    methodB();
}

@Transactional
public void methodB() {
    ...
}
```

If `methodA()` calls `methodB()` directly on `this`, the call can bypass the Spring proxy.

Therefore the transactional interceptor may not run as expected.

This is a common Spring interview question.

---

# 25. Facade Pattern

Facade provides a simplified interface to a complex subsystem.

Example:

```text
CheckoutFacade
    |
+---+---+---+
|   |   |   |
Cart Payment Inventory
```

Controller:

```java
checkoutFacade.checkout(request);
```

instead of coordinating many services itself.

---

# 26. Facade Example

```java
@Service
public class CheckoutFacade {

    private final CartService cartService;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;

    public CheckoutFacade(
            CartService cartService,
            InventoryService inventoryService,
            PaymentService paymentService) {

        this.cartService = cartService;
        this.inventoryService = inventoryService;
        this.paymentService = paymentService;
    }

    public CheckoutResult checkout(
            CheckoutRequest request) {

        var cart = cartService.getCart(request.userId());

        inventoryService.reserve(cart);

        return paymentService.pay(cart);
    }
}
```

The controller remains simple.

---

# 27. Template Method Pattern

Template Method defines the overall algorithm while allowing subclasses to customize specific steps.

Conceptually:

```text
process()
  |
  +-- validate()
  +-- execute()
  +-- audit()
```

Spring's template classes use this idea.

---

# 28. JdbcTemplate

Instead of manually handling:

```text
Connection
Statement
ResultSet
Exception handling
Close resources
```

you use:

```java
jdbcTemplate.query(...);
```

Spring's template abstraction handles common infrastructure concerns.

---

# 29. RestTemplate / RestClient Style

Template/client abstractions similarly hide repetitive HTTP infrastructure.

Modern Spring applications should generally prefer `RestClient` for imperative HTTP calls, while `WebClient` is appropriate for reactive use cases.

---

# 30. Repository Pattern

Repository abstracts persistence operations.

Example:

```java
public interface OrderRepository
        extends JpaRepository<Order, Long> {
}
```

Service:

```java
orderRepository.save(order);
```

The service doesn't need to know the low-level SQL implementation for common persistence operations.

---

# 31. Repository Benefits

```text
Persistence abstraction
Testability
Cleaner service layer
Centralized data access
```

But don't create unnecessary repository methods for every possible query.

---

# 32. Service Layer Pattern

Service layer contains business logic.

Typical architecture:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

Example:

```java
@Service
public class OrderService {

    public OrderResponse create(
            OrderRequest request) {

        // business logic

        return ...;
    }
}
```

---

# 33. Controller Responsibility

Controller should generally handle:

```text
HTTP request
Validation triggering
Authentication context
Response mapping
HTTP status
```

Avoid putting large business workflows in controllers.

Bad:

```java
@PostMapping
public ResponseEntity<?> create(...) {

    // 200 lines of business logic
}
```

Better:

```java
@PostMapping
public ResponseEntity<OrderResponse> create(
        @Valid @RequestBody OrderRequest request) {

    return ResponseEntity.ok(
        orderService.create(request)
    );
}
```

---

# 34. Dependency Injection Pattern

Spring's Dependency Injection is based heavily on the Dependency Inversion Principle.

Instead of:

```java
class OrderService {

    private PaymentService payment =
        new PaymentService();
}
```

use:

```java
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring supplies the dependency.

---

# 35. Constructor Injection

Preferred approach:

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
Immutable dependency reference
Easy unit testing
Required dependencies are explicit
No reflection-based field injection needed
```

---

# 36. Observer Pattern

Observer allows one object to notify interested consumers when something happens.

Conceptually:

```text
Order
 |
 +--> Inventory
 +--> Notification
 +--> Analytics
```

Spring events provide a local application-level implementation.

---

# 37. Spring Application Events

Publisher:

```java
applicationEventPublisher.publishEvent(
    new OrderCreatedEvent(orderId)
);
```

Listener:

```java
@EventListener
public void handle(OrderCreatedEvent event) {
    ...
}
```

This is useful for decoupling components inside the same application.

---

# 38. Application Event vs Kafka Event

Spring application event:

```text
Same JVM/application context
```

Kafka event:

```text
Distributed
Persistent
Asynchronous
Cross-service
```

Do not treat Spring application events as a replacement for distributed messaging.

---

# 39. Event Listener and Transactions

If event processing depends on a successful database transaction, understand the timing.

For example:

```java
@Transactional
public void createOrder() {

    saveOrder();

    publishEvent();
}
```

A regular event listener may execute before the transaction has committed.

For post-commit behavior, Spring provides mechanisms such as:

```text
@TransactionalEventListener
```

---

# 40. @TransactionalEventListener

Example:

```java
@TransactionalEventListener(
    phase = TransactionPhase.AFTER_COMMIT
)
public void handle(OrderCreatedEvent event) {
    ...
}
```

This listener runs after the surrounding transaction commits.

Important:

> This is still an in-process event mechanism. It is not equivalent to a durable Kafka event.

---

# 41. Chain of Responsibility

A request passes through a sequence of handlers.

Conceptually:

```text
Request
 ↓
Authentication
 ↓
Validation
 ↓
Authorization
 ↓
Business processing
```

Spring filters and interceptor chains use similar concepts.

---

# 42. Spring Security Filter Chain

Conceptually:

```text
HTTP Request
     ↓
Security Filters
     ↓
Authentication
     ↓
Authorization
     ↓
Controller
```

Each filter can inspect or modify the request.

---

# 43. Command Pattern

Command encapsulates an operation as an object.

Example:

```java
public record CreateOrderCommand(
    Long userId,
    List<OrderItemRequest> items
) {}
```

A handler processes it:

```java
public OrderResult handle(
        CreateOrderCommand command) {
    ...
}
```

Useful in:

```text
CQRS
Workflow processing
Command buses
Undoable operations
```

---

# 44. CQRS

CQRS means:

```text
Command Query Responsibility Segregation
```

Separate:

```text
Write model
Read model
```

Example:

```text
Commands → Order DB
Queries  → Read model
```

Useful when read and write workloads have significantly different requirements.

Don't introduce CQRS for simple CRUD without a reason.

---

# 45. CQRS Example

Write:

```text
POST /orders
    ↓
Order Service
    ↓
Transactional DB
```

Read:

```text
GET /orders
    ↓
Read Model
    ↓
Optimized query
```

Events synchronize the read model.

---

# 46. Event Sourcing

Event sourcing stores changes as events rather than only storing the latest state.

Example:

```text
AccountCreated
MoneyDeposited
MoneyWithdrawn
```

Current state is reconstructed from the event history.

---

# 47. Event Sourcing Tradeoffs

Benefits:

```text
Complete history
Replay
Auditability
Temporal reconstruction
```

Challenges:

```text
Complexity
Event schema evolution
Storage
Replay performance
Debugging
```

Don't use event sourcing simply because Kafka exists.

---

# 48. Specification Pattern

Specification encapsulates business rules that can be combined.

Example:

```text
Product is:
  Active
  AND
  In stock
  AND
  Price < 50000
```

Useful for complex filtering and reusable business predicates.

Spring Data JPA supports specifications for dynamic queries.

---

# 49. Specification Example

```java
Specification<Product> active =
    (root, query, cb) ->
        cb.isTrue(root.get("active"));
```

Combine:

```java
Specification<Product> result =
    active.and(inStock);
```

This is useful when search/filter combinations become dynamic.

---

# 50. Null Object Pattern

Instead of returning `null`, provide an object representing "no operation" or "no result" where appropriate.

Example:

```java
public interface NotificationSender {
    void send(String message);
}
```

No-op implementation:

```java
class NoOpNotificationSender
        implements NotificationSender {

    public void send(String message) {
        // intentionally do nothing
    }
}
```

This can reduce null checks.

---

# 51. DTO Pattern

DTO:

```text
Data Transfer Object
```

Example:

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {}
```

Benefits:

```text
API contract separation
Avoid exposing entities
Control response fields
Prevent accidental persistence coupling
```

---

# 52. Entity vs DTO

Entity:

```text
Database model
```

DTO:

```text
API/data-transfer model
```

Avoid blindly returning JPA entities from controllers.

---

# 53. Mapper Pattern

Map:

```text
Entity → DTO
DTO → Entity
```

Example:

```java
public ProductResponse toResponse(
        Product product) {

    return new ProductResponse(
        product.getId(),
        product.getName(),
        product.getPrice()
    );
}
```

For larger applications, mapping can be centralized using dedicated mapper classes or tools such as MapStruct.

---

# 54. Dependency Inversion

High-level business logic should depend on abstractions rather than concrete infrastructure.

Bad:

```text
OrderService
    ↓
StripeClient
```

Better:

```text
OrderService
    ↓
PaymentGateway
    ↑
StripePaymentGateway
```

Now the business layer is less coupled to the vendor.

---

# 55. Hexagonal Architecture

Also called:

```text
Ports and Adapters
```

Concept:

```text
          REST Adapter
               |
               v
        +--------------+
        |   Domain     |
        |   Core       |
        +--------------+
          ^          ^
          |          |
     DB Adapter   Kafka Adapter
```

The domain is isolated from infrastructure.

---

# 56. Ports and Adapters

Port:

```java
public interface PaymentGateway {
    PaymentResult charge(PaymentRequest request);
}
```

Adapter:

```java
@Component
public class StripePaymentAdapter
        implements PaymentGateway {
    ...
}
```

The domain depends on the port.

Infrastructure implements the adapter.

---

# 57. Clean Architecture

Typical layers:

```text
Entities
Use Cases
Interface Adapters
Infrastructure
```

Dependency direction should point toward business rules.

The exact layer names are less important than keeping business logic independent from infrastructure.

---

# 58. Layered Architecture

Common Spring Boot structure:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

This is simple and appropriate for many applications.

---

# 59. Layered vs Hexagonal

Layered:

```text
Simple
Easy to understand
Good for many CRUD applications
```

Hexagonal:

```text
Stronger isolation
Better infrastructure independence
Useful for complex domains
More abstraction
```

Choose based on complexity.

---

# 60. Anti-Corruption Layer

When integrating with an external or legacy system:

```text
Your Domain
    |
Anti-Corruption Layer
    |
Legacy System
```

The layer translates external concepts into your internal domain model.

This prevents legacy concepts from spreading through your application.

---

# 61. Example — Legacy Payment API

Legacy:

```text
LEGACY_PAY
CUSTOMER_REF
TXN_CODE
```

Your domain:

```text
Payment
CustomerId
TransactionId
```

Adapter/ACL translates between the models.

---

# 62. Facade vs Adapter

Facade:

```text
Simplifies a complex subsystem
```

Adapter:

```text
Makes incompatible interfaces compatible
```

Example:

```text
Facade → CheckoutFacade
Adapter → StripePaymentAdapter
```

---

# 63. Strategy vs State

Strategy:

```text
Choose an algorithm/behavior
```

State:

```text
Behavior changes based on current state
```

Example:

```text
PaymentStrategy → Card / UPI

OrderState → Pending / Confirmed / Cancelled
```

---

# 64. Template Method vs Strategy

Template Method:

```text
Algorithm structure fixed
Some steps customizable
```

Strategy:

```text
Entire behavior can be replaced
```

Spring often uses both template abstractions and injectable strategies.

---

# 65. Decorator vs Proxy

Decorator:

```text
Adds behavior
```

Proxy:

```text
Controls/accesses an underlying object
```

The implementations can look similar, and the distinction is often about intent.

Spring AOP commonly uses proxies.

---

# 66. Factory vs Builder

Factory:

```text
Which object should be created?
```

Builder:

```text
How should a complex object be constructed?
```

They solve different construction problems.

---

# 67. Singleton vs Prototype Scope

Spring:

```java
@Scope("singleton")
```

Default:

```text
One bean per application context
```

Prototype:

```java
@Scope("prototype")
```

A new bean instance is created when Spring obtains a prototype instance.

Important:

> Injecting a prototype bean into a singleton does not automatically give the singleton a new prototype instance on every method call.

---

# 68. Request Scope

For web applications:

```java
@RequestScope
```

creates a bean associated with an HTTP request.

Useful for request-specific state.

Don't store request-specific state in singleton beans.

---

# 69. Strategy Pattern with Spring

A clean Spring approach:

```java
public interface PaymentStrategy {

    String type();

    PaymentResult pay(PaymentRequest request);
}
```

Inject:

```java
List<PaymentStrategy>
```

Build:

```text
Map<String, PaymentStrategy>
```

Then:

```text
CARD → CardStrategy
UPI  → UpiStrategy
```

This avoids a growing `if/else` chain.

---

# 70. Factory + Strategy

A common real-world combination:

```text
Request
  ↓
Factory
  ↓
Strategy
  ↓
Business operation
```

Example:

```text
PaymentFactory
     ↓
CardPaymentStrategy
```

The factory chooses; the strategy performs.

---

# 71. Repository + Service + Controller

A common Spring structure:

```text
Controller
     ↓
Service
     ↓
Repository
     ↓
JPA/Hibernate
     ↓
Database
```

Responsibilities:

```text
Controller → HTTP
Service → business logic
Repository → persistence
```

---

# 72. Why Not Put Everything in Service?

A service can become a "god class":

```text
OrderService
  3000 lines
```

Responsibilities include:

```text
Payment
Inventory
Notification
Pricing
Reporting
```

Split responsibilities around business boundaries.

---

# 73. God Object

A god object knows too much and does too much.

Symptoms:

```text
Huge class
Many dependencies
Many unrelated methods
Hard to test
Frequent merge conflicts
```

Fix through meaningful responsibility boundaries.

---

# 74. Anemic Domain Model

An anemic domain model contains entities with mostly fields and little business behavior.

This isn't always wrong for CRUD systems.

For complex domain logic, meaningful behavior can live closer to the domain model.

Choose based on complexity.

---

# 75. Overengineering

Bad:

```text
Factory
FactoryFactory
StrategyFactory
BuilderFactory
```

for:

```text
Simple CRUD
```

Patterns should reduce complexity, not increase it.

---

# 76. Interview: What Design Patterns Does Spring Use?

Strong answer:

> Spring uses many design patterns internally. Dependency Injection supports inversion of control, proxies are used for features such as transactions and AOP, template classes reduce repetitive infrastructure code, factories are used throughout bean creation, and strategy-style abstractions appear in areas such as resource resolution and handler selection.

---

# 77. Interview: What Is Factory Pattern?

> Factory centralizes object creation and hides construction details from the caller. In Spring, I can combine a factory with injected strategy implementations, for example selecting a CardPaymentStrategy or UpiPaymentStrategy based on the payment type.

---

# 78. Interview: What Is Strategy Pattern?

> Strategy allows different implementations of an algorithm or business behavior to be selected at runtime. In Spring Boot, I can define an interface, create multiple `@Component` implementations, inject them as a list or map, and select the required implementation based on the request.

---

# 79. Interview: Why Constructor Injection?

> Constructor injection makes dependencies explicit, allows fields to remain final, makes unit testing easier, and ensures the object cannot be created without its required dependencies.

---

# 80. Interview: What Is Proxy Pattern in Spring?

> Spring commonly creates proxies around beans to add cross-cutting behavior such as transactions, caching, security, and AOP. The caller interacts with the proxy, which performs the additional behavior before or after delegating to the target object.

---

# 81. Interview: Why Does @Transactional Sometimes Not Work?

> One common reason is self-invocation. If one method in a bean directly calls another transactional method using `this`, the call can bypass the Spring proxy, so the transactional interceptor may not execute.

---

# 82. Interview: What Is Adapter Pattern?

> Adapter translates one interface into another expected by the application. I commonly use it around third-party APIs so the business layer depends on my own interface rather than directly depending on a vendor SDK.

---

# 83. Interview: What Is Facade Pattern?

> Facade provides a simple interface over a complex subsystem. For example, a CheckoutFacade can coordinate cart, inventory, payment, and order operations so the controller doesn't need to know the internal workflow.

---

# 84. Interview: What Is Builder Pattern?

> Builder is useful for constructing objects with many optional fields or complex construction rules. It makes object creation readable and avoids very large constructors.

---

# 85. Interview: Factory vs Strategy?

> Factory is mainly responsible for selecting or creating the appropriate implementation. Strategy represents the interchangeable behavior itself. In a payment system, the factory can select the payment strategy and the strategy performs the actual payment logic.

---

# 86. Interview: Repository Pattern?

> Repository abstracts persistence operations from business logic. In Spring Data JPA, interfaces such as `JpaRepository` provide common CRUD and query functionality while the service layer focuses on business rules.

---

# 87. Interview: What Is Dependency Injection?

> Dependency Injection means an object receives its dependencies from an external container or caller instead of constructing them itself. Spring's IoC container manages these dependencies and their lifecycle.

---

# 88. Interview: What Is Observer Pattern in Spring?

> Spring application events provide an observer-like mechanism. One component publishes an event and other components can listen using `@EventListener`. It's useful for decoupling components within the same application, while Kafka is more appropriate for durable distributed events.

---

# 89. Interview: What Is Template Method?

> Template Method defines the overall workflow while allowing specific steps to vary. Spring's template classes such as `JdbcTemplate` encapsulate repetitive infrastructure operations and let developers focus on the actual database work.

---

# 90. Interview: What Is Hexagonal Architecture?

> Hexagonal architecture separates the domain from infrastructure through ports and adapters. The business layer defines interfaces such as `PaymentGateway`, while infrastructure implementations connect those ports to databases, external APIs, or messaging systems.

---

# 91. Interview: Layered vs Hexagonal?

> Layered architecture is simpler and works well for many CRUD applications. Hexagonal architecture provides stronger isolation from infrastructure and can be useful when the domain is complex or when multiple external adapters need to be supported. I would choose based on complexity rather than applying hexagonal architecture everywhere.

---

# 92. Interview: Should Every Project Use Design Patterns?

> No. Patterns should solve real recurring problems. Adding abstractions without a reason can make a simple application harder to understand. I prefer starting with the simplest clean design and introducing a pattern when the code demonstrates the problem the pattern solves.

---

# 93. Practical Ecommerce Design

Payment architecture:

```text
Checkout Service
      |
 PaymentGateway
      |
+-----+-----+
|           |
Card       UPI
Adapter    Adapter
|           |
Provider A Provider B
```

This combines:

```text
Dependency Inversion
Adapter
Strategy
Factory
```

---

# 94. Practical Notification Design

```text
NotificationService
       |
NotificationChannel
       |
 +-----+------+
 |            |
Email        SMS
Strategy     Strategy
```

Selection:

```text
NotificationFactory
       ↓
EmailStrategy / SmsStrategy
```

---

# 95. Practical Discount Design

```text
DiscountStrategy
      |
+-----+-----+
|     |     |
Regular Premium Festival
```

Then:

```java
DiscountStrategy strategy =
    factory.get(customerType);

strategy.calculate(order);
```

This keeps discount rules separate.

---

# 96. Practical Payment Design

```text
PaymentController
       ↓
PaymentService
       ↓
PaymentStrategy
       ↓
PaymentGateway
       ↓
Adapter
       ↓
External Provider
```

Responsibilities remain separated.

---

# 97. Pattern Selection Guide

Problem:

```text
Many interchangeable algorithms
```

Use:

```text
Strategy
```

Problem:

```text
Need centralized implementation selection
```

Use:

```text
Factory
```

Problem:

```text
Complex object construction
```

Use:

```text
Builder
```

Problem:

```text
External incompatible API
```

Use:

```text
Adapter
```

Problem:

```text
Complex subsystem
```

Use:

```text
Facade
```

Problem:

```text
Cross-cutting behavior
```

Often use:

```text
Proxy / AOP
```

Problem:

```text
Persistence abstraction
```

Use:

```text
Repository
```

---

# 98. Pattern Selection — Continued

Problem:

```text
Common algorithm skeleton
```

Use:

```text
Template Method
```

Problem:

```text
Notify multiple listeners
```

Use:

```text
Observer
```

Problem:

```text
Separate domain from infrastructure
```

Use:

```text
Ports and Adapters
```

Problem:

```text
Separate read/write models
```

Consider:

```text
CQRS
```

Problem:

```text
Need complete state history
```

Consider:

```text
Event Sourcing
```

---

# 99. Design Pattern Checklist

```text
□ SOLID
□ Dependency Injection
□ Singleton
□ Factory
□ Strategy
□ Builder
□ Adapter
□ Decorator
□ Proxy
□ Facade
□ Template Method
□ Repository
□ Observer
□ Chain of Responsibility
□ Command
□ CQRS
□ Specification
□ DTO
□ Mapper
□ Hexagonal Architecture
□ Clean Architecture
□ Anti-Corruption Layer
```

---

# 100. Final Mental Model

```text
                    DESIGN PROBLEM
                          |
                          v
                    Identify intent
                          |
          +---------------+---------------+
          |               |               |
      Creation        Behavior        Structure
          |               |               |
       Factory        Strategy         Adapter
       Builder        Observer         Facade
       Singleton      Template         Decorator
                                      Proxy
                          |
                          v
                    Spring Framework
                          |
       +------------------+------------------+
       |                  |                  |
      Beans           AOP/Proxy         Persistence
       |                  |                  |
       DI             Transaction       Repository
       Scope          Security          JPA
       Factory        Cache
                          |
                          v
                     Clean Design
```

---

# 101. Final Rule

> **Don't say "I used Strategy Pattern" just to sound advanced. Explain the problem it solved. In Spring Boot, the strongest design answers connect patterns to real requirements such as loose coupling, testability, extensibility, transaction boundaries, external integrations, and maintainability.**

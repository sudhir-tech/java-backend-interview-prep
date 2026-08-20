# SOLID & OOP Design Principles

SOLID principles help us design Java code that is easier to understand, test, extend and maintain.

For Java backend interviews, focus on:

- OOP fundamentals
- Encapsulation
- Abstraction
- Inheritance
- Polymorphism
- SOLID principles
- Composition vs inheritance
- Dependency Injection
- Programming to interfaces
- Common design mistakes
- Backend examples

---

# 1. What Is OOP?

Object-Oriented Programming organizes software around objects that contain:

```text
State
+
Behavior
```

For example:

```java
class Product {

    private String name;
    private double price;

    public void applyDiscount(double percentage) {
        price -= price * percentage / 100;
    }
}
```

The object contains state:

```text
name
price
```

and behavior:

```text
applyDiscount()
```

---

# 2. Four Pillars of OOP

The four commonly discussed pillars are:

```text
Encapsulation
Abstraction
Inheritance
Polymorphism
```

---

# 3. Encapsulation

Encapsulation means keeping an object's internal state protected and controlling access through well-defined methods.

Example:

```java
class BankAccount {

    private double balance;

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public double getBalance() {
        return balance;
    }
}
```

The field:

```java
private double balance;
```

cannot be directly modified from outside.

Instead:

```java
deposit()
```

controls how the value changes.

---

# 4. Why Encapsulation?

Encapsulation provides:

- Data protection
- Controlled modification
- Validation
- Easier maintenance
- Reduced coupling

Without encapsulation:

```java
account.balance = -100000;
```

could potentially put the object into an invalid state.

With encapsulation:

```java
account.deposit(1000);
```

the class controls the operation.

---

# 5. Abstraction

Abstraction means exposing what an object can do while hiding unnecessary implementation details.

Example:

```java
interface PaymentService {

    void pay(double amount);
}
```

The caller knows:

```text
pay()
```

but does not need to know the internal payment-processing steps.

Implementation:

```java
class CardPaymentService
        implements PaymentService {

    @Override
    public void pay(double amount) {
        // validation
        // card processing
        // transaction creation
    }
}
```

---

# 6. Encapsulation vs Abstraction

### Encapsulation

Focuses on:

```text
How data is protected
```

Example:

```java
private double balance;
```

### Abstraction

Focuses on:

```text
What functionality is exposed
```

Example:

```java
void pay(double amount);
```

### Interview Answer

> Encapsulation protects an object's internal state and controls how it is modified, while abstraction hides implementation details and exposes only the required behavior.

---

# 7. Inheritance

Inheritance allows one class to reuse or specialize behavior from another class.

Example:

```java
class Vehicle {

    void start() {
        System.out.println("Starting");
    }
}

class Car extends Vehicle {

    void drive() {
        System.out.println("Driving");
    }
}
```

Now:

```java
Car car = new Car();

car.start();
car.drive();
```

---

# 8. IS-A Relationship

Inheritance represents an IS-A relationship.

Example:

```text
Car IS-A Vehicle
Dog IS-A Animal
CardPayment IS-A Payment
```

If the relationship doesn't make logical sense, inheritance is probably the wrong choice.

---

# 9. HAS-A Relationship

Composition represents a HAS-A relationship.

Example:

```java
class Order {

    private PaymentService paymentService;
}
```

An:

```text
Order HAS-A PaymentService
```

relationship exists.

---

# 10. Composition

Composition means building a class using other objects instead of inheriting their implementation.

Example:

```java
class OrderService {

    private final PaymentService paymentService;

    OrderService(
        PaymentService paymentService
    ) {
        this.paymentService = paymentService;
    }
}
```

This is common in Spring applications.

---

# 11. Composition vs Inheritance

### Inheritance

```text
IS-A
```

Example:

```java
class Car extends Vehicle
```

### Composition

```text
HAS-A
```

Example:

```java
class OrderService {

    private PaymentService paymentService;
}
```

In many application designs, composition is preferred because it reduces tight coupling.

---

# 12. Polymorphism

Polymorphism means the same interface or parent type can represent different implementations.

Example:

```java
PaymentService payment;

payment = new CardPaymentService();
payment.pay(1000);

payment = new UpiPaymentService();
payment.pay(1000);
```

The same method:

```java
pay()
```

can behave differently depending on the runtime implementation.

---

# 13. Compile-Time Polymorphism

Method overloading is commonly described as compile-time polymorphism.

Example:

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
}
```

The compiler determines which method should be called based on the arguments.

---

# 14. Runtime Polymorphism

Method overriding provides runtime polymorphism.

Example:

```java
class PaymentService {

    void pay() {
        System.out.println("Payment");
    }
}

class CardPaymentService
        extends PaymentService {

    @Override
    void pay() {
        System.out.println(
            "Card payment"
        );
    }
}
```

Then:

```java
PaymentService service =
    new CardPaymentService();

service.pay();
```

The overridden method in `CardPaymentService` executes.

---

# 15. Dynamic Method Dispatch

The JVM determines the overridden instance method implementation at runtime based on the actual object.

Example:

```java
PaymentService service =
    new CardPaymentService();
```

Reference type:

```text
PaymentService
```

Actual object:

```text
CardPaymentService
```

The actual object's overridden method is selected for the virtual call.

---

# 16. What Is SOLID?

SOLID is a group of five object-oriented design principles:

```text
S → Single Responsibility Principle
O → Open/Closed Principle
L → Liskov Substitution Principle
I → Interface Segregation Principle
D → Dependency Inversion Principle
```

They help create maintainable and flexible software.

---

# 17. S — Single Responsibility Principle

### Definition

A class should have one clear responsibility and one reason to change.

It does not necessarily mean a class can have only one method.

---

# 18. SRP — Bad Example

```java
class OrderService {

    void createOrder() {
        // create order
    }

    void sendEmail() {
        // send email
    }

    void generateInvoice() {
        // generate invoice
    }

    void saveToDatabase() {
        // database logic
    }
}
```

This class has multiple responsibilities:

```text
Order processing
Email
Invoice generation
Persistence
```

Changes in any of these areas can force changes to the same class.

---

# 19. SRP — Better Design

Separate responsibilities:

```java
class OrderService {
    void createOrder() {
    }
}
```

```java
class EmailService {
    void sendEmail() {
    }
}
```

```java
class InvoiceService {
    void generateInvoice() {
    }
}
```

```java
class OrderRepository {
    void save() {
    }
}
```

Now each component has a clearer responsibility.

---

# 20. SRP Backend Example

A Spring Boot application can separate:

```text
Controller
    ↓
Service
    ↓
Repository
```

For example:

```java
@RestController
class ProductController {
}
```

handles HTTP concerns.

```java
@Service
class ProductService {
}
```

handles business logic.

```java
@Repository
class ProductRepository {
}
```

handles persistence.

This separation supports SRP and separation of concerns.

---

# 21. O — Open/Closed Principle

### Definition

Software entities should be:

```text
Open for extension
Closed for modification
```

In practical terms, new behavior should often be added by introducing new implementations rather than repeatedly modifying stable existing logic.

---

# 22. OCP — Bad Example

```java
class PaymentService {

    void pay(String type) {

        if (type.equals("CARD")) {
            // card payment
        }
        else if (type.equals("UPI")) {
            // UPI payment
        }
        else if (type.equals("CASH")) {
            // cash payment
        }
    }
}
```

Every new payment type requires changing this class.

---

# 23. OCP — Better Design

Define an abstraction:

```java
interface PaymentMethod {

    void pay(double amount);
}
```

Implement different methods:

```java
class CardPayment
        implements PaymentMethod {

    public void pay(double amount) {
        // card payment
    }
}
```

```java
class UpiPayment
        implements PaymentMethod {

    public void pay(double amount) {
        // UPI payment
    }
}
```

Now a new payment method can be added:

```java
class WalletPayment
        implements PaymentMethod {

    public void pay(double amount) {
        // wallet payment
    }
}
```

without modifying the existing payment implementations.

---

# 24. L — Liskov Substitution Principle

### Definition

Objects of a subtype should be usable wherever the parent type is expected without breaking the correctness of the program.

The subtype should honor the behavioral contract of the abstraction.

---

# 25. LSP — Classic Example

Suppose:

```java
class Bird {

    void fly() {
    }
}
```

Then:

```java
class Penguin extends Bird {

    @Override
    void fly() {
        throw new UnsupportedOperationException();
    }
}
```

This is a design problem.

A `Penguin` cannot satisfy the expected behavior of:

```java
Bird.fly()
```

---

# 26. LSP — Better Design

Separate capabilities:

```java
interface Bird {
}
```

```java
interface FlyingBird {

    void fly();
}
```

Then:

```java
class Sparrow
        implements Bird, FlyingBird {

    public void fly() {
    }
}
```

```java
class Penguin
        implements Bird {
}
```

Now the abstraction does not force penguins to provide unsupported behavior.

---

# 27. LSP Backend Example

Suppose:

```java
interface Storage {

    void save(File file);
}
```

If one implementation silently ignores `save()` or throws an unsupported-operation exception, callers cannot safely treat all implementations as `Storage`.

The abstraction should represent a behavior all implementations can actually provide.

---

# 28. I — Interface Segregation Principle

### Definition

Clients should not be forced to depend on methods they do not need.

Instead of one huge interface, prefer smaller, focused interfaces when appropriate.

---

# 29. ISP — Bad Example

```java
interface Worker {

    void work();

    void eat();

    void sleep();
}
```

Suppose a machine implements:

```java
class Robot implements Worker {
}
```

The robot may not need:

```text
eat()
sleep()
```

---

# 30. ISP — Better Design

Split interfaces:

```java
interface Workable {

    void work();
}
```

```java
interface Eatable {

    void eat();
}
```

```java
interface Sleepable {

    void sleep();
}
```

Now classes implement only what they need.

---

# 31. ISP Backend Example

Instead of:

```java
interface UserOperations {

    void createUser();

    void deleteUser();

    void exportReports();

    void processPayments();

    void manageInventory();
}
```

consider smaller abstractions where the domain actually requires them:

```text
UserService
ReportService
PaymentService
InventoryService
```

This reduces unnecessary coupling.

---

# 32. D — Dependency Inversion Principle

### Definition

High-level modules should not depend directly on low-level implementation details.

Both should depend on abstractions.

Abstractions should not depend on details.

Details should depend on abstractions.

---

# 33. DIP — Bad Example

```java
class OrderService {

    private MySQLOrderRepository repository =
        new MySQLOrderRepository();

    void saveOrder() {
        repository.save();
    }
}
```

`OrderService` is tightly coupled to:

```text
MySQLOrderRepository
```

---

# 34. DIP — Better Design

Create an abstraction:

```java
interface OrderRepository {

    void save(Order order);
}
```

Implementation:

```java
class MySQLOrderRepository
        implements OrderRepository {

    public void save(Order order) {
        // MySQL implementation
    }
}
```

Service:

```java
class OrderService {

    private final OrderRepository repository;

    OrderService(
        OrderRepository repository
    ) {
        this.repository = repository;
    }
}
```

Now the service depends on:

```text
OrderRepository
```

rather than a concrete database implementation.

---

# 35. Dependency Injection

Dependency Injection is a technique used to provide an object's dependencies from outside instead of having the object construct them itself.

Example:

```java
class OrderService {

    private final PaymentService paymentService;

    OrderService(
        PaymentService paymentService
    ) {
        this.paymentService = paymentService;
    }
}
```

The dependency is injected through the constructor.

---

# 36. Constructor Injection in Spring

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
        PaymentService paymentService
    ) {
        this.paymentService = paymentService;
    }
}
```

Spring resolves the dependency and supplies it when creating the bean.

Constructor injection is generally preferred because:

- Dependencies are explicit
- Fields can be final
- Objects can be easier to test
- Required dependencies cannot be accidentally omitted after construction

---

# 37. SOLID Summary

```text
S
Single Responsibility
One clear responsibility

O
Open/Closed
Extend behavior without repeatedly modifying stable code

L
Liskov Substitution
Subtypes should honor the parent abstraction's contract

I
Interface Segregation
Prefer focused interfaces

D
Dependency Inversion
Depend on abstractions rather than concrete details
```

---

# 38. SOLID Does Not Mean "More Classes"

A common mistake is to interpret SOLID as:

```text
More classes = better design
```

Not necessarily.

Good design balances:

```text
Cohesion
Coupling
Complexity
Readability
Testability
Changeability
```

The goal is maintainable software, not maximum abstraction.

---

# 39. Coupling

Coupling describes how strongly components depend on one another.

High coupling:

```text
A → concrete B
```

Changes in B may require changes in A.

Lower coupling:

```text
A → Interface
       ↑
       B
```

A can work with different implementations.

---

# 40. Cohesion

Cohesion describes how closely related the responsibilities inside a module are.

High cohesion:

```text
ProductService
    ↓
Product-related business operations
```

Low cohesion:

```text
ProductService
    ↓
Products
Emails
Reports
Payments
Logging
```

High cohesion is generally desirable.

---

# 41. High Cohesion + Low Coupling

A common design goal is:

```text
High cohesion
+
Low coupling
```

This makes systems easier to:

- Understand
- Test
- Change
- Reuse
- Maintain

---

# 42. Programming to an Interface

Prefer:

```java
List<String> names =
    new ArrayList<>();
```

instead of:

```java
ArrayList<String> names =
    new ArrayList<>();
```

The variable depends on the abstraction:

```text
List
```

rather than the implementation:

```text
ArrayList
```

This makes changing the implementation easier.

---

# 43. Example — Repository Abstraction

Instead of:

```java
private MySQLUserRepository repository;
```

prefer:

```java
private UserRepository repository;
```

where:

```java
interface UserRepository {
}
```

and implementations can vary.

This is especially useful for testing and architectural flexibility.

---

# 44. Dependency Inversion vs Dependency Injection

These are related but not identical.

### Dependency Inversion Principle

A design principle:

```text
Depend on abstractions.
```

### Dependency Injection

A technique:

```text
Provide dependencies from outside.
```

Spring uses dependency injection extensively, which can help implement designs consistent with DIP.

---

# 45. Interface vs Abstract Class

### Interface

Useful for defining a contract or capability.

```java
interface Payment {

    void pay();
}
```

### Abstract Class

Useful when related classes share state or common implementation.

```java
abstract class BasePayment {

    protected double amount;

    abstract void pay();

    void log() {
        System.out.println("Payment");
    }
}
```

Choose based on the relationship and design needs.

---

# 46. Multiple Interfaces

A Java class can implement multiple interfaces.

```java
class UserService
        implements Auditable, Loggable {
}
```

Java does not support multiple inheritance of classes.

This allows multiple contracts/capabilities without inheriting implementation from multiple classes.

---

# 47. Why Prefer Composition?

Composition usually gives more flexibility because behavior can be assembled from independent components.

Example:

```java
class OrderService {

    private final PaymentService paymentService;
    private final NotificationService notificationService;
}
```

The service can work with different implementations of both dependencies.

---

# 48. Strategy Pattern and OCP

SOLID often leads naturally to design patterns.

For payment processing:

```java
interface PaymentStrategy {

    void pay(double amount);
}
```

Implementations:

```text
CardPaymentStrategy
UpiPaymentStrategy
WalletPaymentStrategy
```

The service can depend on:

```java
PaymentStrategy
```

This is a common use of the Strategy pattern.

---

# 49. Factory Pattern

A factory centralizes object creation.

Example:

```java
class PaymentFactory {

    static PaymentStrategy create(
        String type
    ) {

        return switch (type) {
            case "CARD" ->
                new CardPaymentStrategy();

            case "UPI" ->
                new UpiPaymentStrategy();

            default ->
                throw new IllegalArgumentException(
                    "Unsupported payment type"
                );
        };
    }
}
```

This can keep object-creation logic separate from business logic.

---

# 50. SOLID in a Spring Boot Application

A typical backend can look like:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

With abstractions:

```text
Controller
    ↓
Service Interface
    ↓
Service Implementation
    ↓
Repository Interface
    ↓
Repository Implementation
```

However, don't add interfaces solely for the sake of having an interface. Add abstractions where they provide meaningful decoupling or represent a useful contract.

---

# 51. Example — E-Commerce Order Flow

A simplified design:

```text
OrderController
       ↓
OrderService
       ↓
OrderRepository
       ↓
Database
```

Payment:

```text
OrderService
       ↓
PaymentService
       ↓
PaymentStrategy
       ↓
Card / UPI / Wallet
```

Notification:

```text
OrderService
       ↓
NotificationService
       ↓
Email / SMS / Push
```

This separation helps keep responsibilities focused.

---

# 52. Example — Bad Order Service

Avoid putting everything into one class:

```java
class OrderService {

    void createOrder() {
        // validate user
        // calculate price
        // save order
        // charge payment
        // send email
        // generate invoice
        // update inventory
        // publish event
    }
}
```

This class becomes difficult to:

```text
Test
Maintain
Change
Reuse
Understand
```

---

# 53. Better Order Design

Separate domain responsibilities:

```text
OrderService
PaymentService
InventoryService
NotificationService
InvoiceService
OrderRepository
```

Then coordinate them:

```java
class OrderService {

    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;

    void createOrder() {

        // coordinate business workflow
    }
}
```

The exact boundaries depend on the system's requirements.

---

# 54. Common SOLID Interview Question

### What is the Single Responsibility Principle?

### Answer

> A class should have one clear responsibility and therefore one primary reason to change. It doesn't mean the class can have only one method; it means unrelated responsibilities should not be mixed together.

---

# 55. What Is Open/Closed Principle?

### Answer

> Code should be designed so new behavior can often be added through extension or new implementations rather than repeatedly modifying stable existing logic.

---

# 56. What Is Liskov Substitution Principle?

### Answer

> A subtype should be usable wherever its parent abstraction is expected without violating the expected behavior or contract.

---

# 57. What Is Interface Segregation Principle?

### Answer

> Clients should not be forced to depend on methods they do not need. Smaller, focused interfaces are often better than large interfaces containing unrelated responsibilities.

---

# 58. What Is Dependency Inversion Principle?

### Answer

> High-level business logic should depend on abstractions rather than concrete implementation details. Both high-level and low-level components should be designed around appropriate abstractions.

---

# 59. What Is Dependency Injection?

### Answer

> Dependency Injection is a technique where an object's dependencies are provided from outside instead of the object creating them itself. In Spring, constructor injection is a common approach.

---

# 60. Why Is Constructor Injection Preferred?

### Answer

> Constructor injection makes required dependencies explicit, supports immutable fields, improves testability and prevents an object from being constructed without its required dependencies.

---

# 61. Composition vs Inheritance Interview Answer

> I prefer composition when I need to assemble behavior from independent components because it usually gives lower coupling and more flexibility. I use inheritance when there is a genuine IS-A relationship and the subtype can correctly satisfy the parent abstraction.

---

# 62. Abstraction vs Encapsulation Interview Answer

> Abstraction focuses on hiding implementation complexity and exposing required behavior, while encapsulation focuses on protecting an object's internal state and controlling access to it.

---

# 63. Overloading vs Overriding

### Overloading

Same method name:

```java
add(int a, int b)
add(double a, double b)
```

Usually resolved at compile time.

### Overriding

Subclass provides a new implementation:

```java
@Override
void pay() {
}
```

Resolved through runtime polymorphism for applicable instance method calls.

---

# 64. Can Static Methods Be Overridden?

No.

Static methods belong to the class rather than being dynamically dispatched instance methods.

A subclass can declare a static method with the same signature, but that is method hiding, not overriding.

---

# 65. Can Private Methods Be Overridden?

No.

Private methods are not inherited as overridable methods by subclasses.

A subclass can define another method with the same name, but it is not an override.

---

# 66. Can Final Methods Be Overridden?

No.

A method declared:

```java
final
```

cannot be overridden by a subclass.

---

# 67. Why Use final?

`final` can communicate that something should not be reassigned or overridden.

Examples:

```java
final class Utility {
}
```

```java
final int value = 10;
```

```java
final void process() {
}
```

In backend code, final fields are especially useful for immutable object state and constructor-injected dependencies.

---

# 68. Immutability

An immutable object cannot have its state changed after construction.

Example:

```java
public final class User {

    private final Long id;
    private final String name;

    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}
```

There are no setters.

---

# 69. Benefits of Immutability

Immutable objects are generally:

- Easier to reason about
- Safer to share
- Less prone to accidental mutation
- Easier to use in concurrent code

---

# 70. Immutability vs final

`final` alone does not make an object immutable.

Example:

```java
final List<String> names =
    new ArrayList<>();
```

The reference cannot be reassigned:

```java
names = anotherList; // not allowed
```

But the list can still be modified:

```java
names.add("Java");
```

So:

```text
final reference ≠ immutable object
```

---

# 71. DTO Design

A DTO carries data between application boundaries.

Example:

```java
public record ProductResponse(
    Long id,
    String name,
    BigDecimal price
) {
}
```

For suitable use cases, records provide a concise immutable data-carrier model.

---

# 72. Domain Model vs DTO

### Domain Entity

Represents application/domain state and often maps to persistence.

```java
@Entity
class Product {
}
```

### DTO

Represents data exchanged between application layers or APIs.

```java
record ProductResponse(
    Long id,
    String name
) {}
```

Keeping DTOs separate from persistence entities can prevent internal database structure from leaking directly into API contracts.

---

# 73. Common Design Smells

Watch for:

```text
God classes
Long methods
Large interfaces
Deep inheritance hierarchies
Too many conditionals for behavior selection
Tight coupling
Hidden dependencies
Duplicated business logic
Anemic abstractions
Unnecessary abstractions
```

---

# 74. God Class

A God class tries to do everything.

Example:

```text
OrderService
    ↓
Orders
Payments
Users
Emails
Reports
Inventory
Logging
Database
```

Better:

```text
OrderService
PaymentService
UserService
NotificationService
ReportService
InventoryService
```

---

# 75. Long Conditional Smell

Code like:

```java
if (type.equals("CARD")) {
}
else if (type.equals("UPI")) {
}
else if (type.equals("WALLET")) {
}
else if (type.equals("CRYPTO")) {
}
```

may indicate that a strategy or polymorphic design could be useful.

But don't introduce a pattern automatically; consider complexity and expected change.

---

# 76. Dependency Injection Makes Testing Easier

With constructor injection:

```java
OrderService service =
    new OrderService(mockPaymentService);
```

A unit test can provide a mock or fake dependency.

Without DI:

```java
new OrderService()
```

might internally create:

```java
new MySQLPaymentService()
```

which makes isolated testing harder.

---

# 77. SOLID and Unit Testing

Good design often improves testability.

For example:

```text
OrderService
    ↓
PaymentService interface
```

A unit test can supply:

```text
MockPaymentService
```

instead of calling a real payment provider.

---

# 78. SOLID and Microservices

SOLID mainly applies to object and component design, but its ideas can also influence larger service boundaries.

For example:

```text
Order Service
Payment Service
Inventory Service
Notification Service
```

Each service should have a meaningful responsibility rather than becoming a giant service containing unrelated domains.

Don't blindly map every class to a microservice.

---

# 79. Interview Scenario

### Question

You have:

```java
PaymentService
```

and need to support:

```text
Card
UPI
Wallet
```

How would you design it?

### Good Answer

> I would define a PaymentStrategy interface with a common `pay()` contract and create separate implementations for Card, UPI and Wallet. The order or payment orchestration service would depend on the abstraction rather than concrete implementations. A factory or dependency-injection-based strategy selection mechanism can select the appropriate implementation. This keeps the design extensible and aligns with OCP and DIP.

---

# 80. Interview Scenario — Notification

Requirement:

```text
Email
SMS
Push Notification
```

Design:

```java
interface NotificationService {

    void send(
        String recipient,
        String message
    );
}
```

Implementations:

```text
EmailNotificationService
SmsNotificationService
PushNotificationService
```

The business service depends on the interface.

---

# 81. Interview Scenario — Storage

Requirement:

```text
MySQL
MongoDB
```

Possible abstraction:

```java
interface UserRepository {

    User findById(Long id);

    void save(User user);
}
```

Implementations can provide different persistence mechanisms where the abstraction's contract makes sense.

---

# 82. SOLID Quick Revision

```text
S — Single Responsibility
Keep unrelated responsibilities separate.

O — Open/Closed
Add new behavior through extension where appropriate.

L — Liskov Substitution
Subtypes must preserve the abstraction's expected behavior.

I — Interface Segregation
Prefer focused interfaces.

D — Dependency Inversion
Depend on suitable abstractions.
```

---

# 83. Most Important Interview Topics

Before a Java backend interview, be comfortable explaining:

```text
OOP pillars
Encapsulation
Abstraction
Inheritance
Polymorphism

Composition vs inheritance
Overloading vs overriding
Interface vs abstract class

SOLID
SRP
OCP
LSP
ISP
DIP

Dependency Injection
Constructor Injection
Coupling
Cohesion
Immutability

Strategy Pattern
Factory Pattern
Programming to interfaces
DTO vs Entity
```

---

# 84. Final Interview Answer

If an interviewer asks:

### "How do you apply SOLID in your backend projects?"

A natural answer:

> I try to keep responsibilities separated, keep business logic independent from infrastructure details, and depend on interfaces where that gives meaningful flexibility. For example, in a Spring Boot backend I would keep controllers focused on HTTP concerns, services focused on business logic and repositories focused on persistence. For behavior that can vary, such as payment methods, I would use an interface with separate implementations rather than growing a large conditional block. I also prefer constructor injection because it makes dependencies explicit and improves testability.

---

# 85. Practical Rule

Don't say:

> "I always use SOLID."

A better engineering mindset is:

> "I use SOLID principles when they make the code easier to change, test and understand."

Good design is not about adding abstractions everywhere.

The goal is:

```text
Simple
Understandable
Testable
Maintainable
Change-friendly
```

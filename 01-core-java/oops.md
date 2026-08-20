# Object-Oriented Programming in Java

Object-Oriented Programming (OOP) is a programming approach where software is designed around **objects that contain data and behavior**.

Java is primarily an object-oriented programming language and is built around four major OOP principles:

1. Encapsulation
2. Inheritance
3. Polymorphism
4. Abstraction

---

## 1. Encapsulation

Encapsulation means **bundling data and the methods that operate on that data inside a class**, while restricting direct access to the internal state.

### Example

```java
public class BankAccount {

    private double balance;

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
}
```

Here, `balance` is private and cannot be modified directly from outside the class.

### Why use encapsulation?

- Protects object state
- Prevents invalid modifications
- Improves maintainability
- Provides controlled access through methods

### Interview Answer

> Encapsulation is the practice of hiding an object's internal state and exposing controlled access through methods.

---

## 2. Inheritance

Inheritance allows one class to acquire properties and behavior from another class.

### Example

```java
class Animal {

    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {

    void bark() {
        System.out.println("Barking");
    }
}
```

`Dog` inherits the `eat()` method from `Animal`.

### Interview Answer

> Inheritance allows a child class to reuse and extend the properties and behavior of a parent class.

### Types of inheritance supported by Java

Java supports:

- Single inheritance
- Multilevel inheritance
- Hierarchical inheritance

Java does **not** support multiple inheritance through classes.

However, Java allows a class to implement multiple interfaces.

---

## 3. Polymorphism

Polymorphism means **one interface or reference can represent different forms of behavior**.

There are two common types in Java:

- Compile-time polymorphism
- Runtime polymorphism

---

### 3.1 Compile-Time Polymorphism

Compile-time polymorphism is achieved through **method overloading**.

### Example

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

The method name is the same, but the parameters are different.

The compiler determines which method should be called.

### Interview Answer

> Method overloading is compile-time polymorphism where multiple methods have the same name but different parameter lists.

---

### 3.2 Runtime Polymorphism

Runtime polymorphism is achieved through **method overriding**.

### Example

```java
class Animal {

    void sound() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Bark");
    }
}
```

```java
Animal animal = new Dog();

animal.sound();
```

Output:

```text
Bark
```

The reference type is `Animal`, but the actual object is `Dog`.

Therefore, the overridden `Dog.sound()` method is executed at runtime.

### Interview Answer

> Runtime polymorphism occurs when a parent class reference refers to a child class object and the overridden method is resolved at runtime.

---

## 4. Abstraction

Abstraction means **hiding implementation details and exposing only the essential functionality**.

Java provides abstraction mainly through:

- Abstract classes
- Interfaces

### Example using an abstract class

```java
abstract class Payment {

    abstract void pay();

    void receipt() {
        System.out.println("Generating receipt");
    }
}
```

A subclass provides the implementation:

```java
class CreditCardPayment extends Payment {

    @Override
    void pay() {
        System.out.println("Processing credit card payment");
    }
}
```

### Interview Answer

> Abstraction hides unnecessary implementation details and exposes only the functionality required by the user.

---

# Encapsulation vs Abstraction

| Encapsulation | Abstraction |
|---|---|
| Hides internal state | Hides implementation complexity |
| Focuses on data protection | Focuses on what an object does |
| Uses access modifiers | Uses interfaces and abstract classes |
| Example: `private balance` | Example: `Payment.pay()` |

---

# Inheritance vs Composition

Inheritance represents an **"IS-A"** relationship.

```java
class Dog extends Animal {
}
```

A dog **is an** animal.

Composition represents a **"HAS-A"** relationship.

```java
class Car {

    private Engine engine;
}
```

A car **has an** engine.

### Interview Tip

In many real-world designs, **composition is preferred over inheritance** because it reduces tight coupling and makes behavior easier to change.

---

# Method Overloading vs Method Overriding

| Overloading | Overriding |
|---|---|
| Same method name | Same method signature |
| Different parameters | Same parameters |
| Compile-time polymorphism | Runtime polymorphism |
| Usually within the same class | Requires inheritance |
| Return type alone cannot overload a method | Return type must be compatible |

### Example: Overloading

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

### Example: Overriding

```java
class Animal {

    void sound() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Bark");
    }
}
```

---

# Interface vs Abstract Class

## Interface

Use an interface when you want to define a contract that different classes can implement.

```java
interface Payment {

    void pay();
}
```

Multiple unrelated classes can implement the same interface.

```java
class CreditCardPayment implements Payment {

    @Override
    public void pay() {
        System.out.println("Credit card payment");
    }
}
```

---

## Abstract Class

Use an abstract class when you want to share common state or implementation between related classes.

```java
abstract class Vehicle {

    String brand;

    abstract void start();

    void stop() {
        System.out.println("Vehicle stopped");
    }
}
```

A child class can extend it:

```java
class Car extends Vehicle {

    @Override
    void start() {
        System.out.println("Car started");
    }
}
```

### Quick Comparison

| Interface | Abstract Class |
|---|---|
| Defines a contract | Can define contract + shared implementation |
| Supports multiple implementation | A class can extend only one class |
| Can contain default/static methods | Can contain instance methods |
| Implemented using `implements` | Extended using `extends` |

---

# Common Interview Questions

## 1. What are the four pillars of OOP?

### Answer

> The four pillars of OOP are Encapsulation, Inheritance, Polymorphism and Abstraction.

---

## 2. What is encapsulation?

### Answer

> Encapsulation means hiding an object's internal state and providing controlled access to it through methods.

---

## 3. What is inheritance?

### Answer

> Inheritance allows a child class to reuse and extend the properties and behavior of a parent class.

---

## 4. What is polymorphism?

### Answer

> Polymorphism allows the same interface or method to behave differently depending on the object or context.

---

## 5. What is abstraction?

### Answer

> Abstraction hides implementation details and exposes only the essential functionality to the user.

---

## 6. Does Java support multiple inheritance?

### Answer

> Java does not support multiple inheritance through classes because it can create ambiguity. However, a class can implement multiple interfaces.

---

## 7. Why doesn't Java support multiple inheritance through classes?

If both parent classes contain the same method, the child class could have ambiguity about which implementation to use.

Java avoids this problem by not allowing multiple class inheritance.

---

## 8. Why is composition often preferred over inheritance?

### Answer

> Composition reduces tight coupling and allows behavior to be changed more flexibly. It also makes systems easier to maintain and extend.

---

## 9. Can an abstract class have a constructor?

### Answer

> Yes. An abstract class can have a constructor. The constructor is executed when an object of a concrete subclass is created.

Example:

```java
abstract class Animal {

    Animal() {
        System.out.println("Animal constructor");
    }
}

class Dog extends Animal {

    Dog() {
        System.out.println("Dog constructor");
    }
}
```

---

## 10. Can an interface have implemented methods?

### Answer

> Yes. Modern Java interfaces can contain `default` and `static` methods with implementations.

Example:

```java
interface Vehicle {

    void start();

    default void stop() {
        System.out.println("Vehicle stopped");
    }
}
```

---

# Important Follow-Up Questions

If an interviewer asks about OOP, be prepared for these follow-ups:

- What is runtime polymorphism?
- What is compile-time polymorphism?
- Why can't Java support multiple class inheritance?
- Interface vs abstract class?
- Composition vs inheritance?
- What is dynamic method dispatch?
- Can static methods be overridden?
- Can private methods be overridden?
- Can constructors be inherited?
- What is the difference between IS-A and HAS-A?
- Can an abstract class be instantiated?
- Can an interface have variables?
- What is method hiding?
- What is the difference between association, aggregation and composition?
- How does OOP help in designing maintainable applications?

---

# Common Mistakes

### Mistake 1: Saying overloading is runtime polymorphism

❌ Incorrect:

> Method overloading is runtime polymorphism.

✅ Correct:

> Method overloading is compile-time polymorphism.

---

### Mistake 2: Saying Java supports multiple inheritance

❌ Incorrect:

> Java supports multiple inheritance.

✅ Correct:

> Java does not support multiple inheritance through classes, but a class can implement multiple interfaces.

---

### Mistake 3: Confusing abstraction with encapsulation

Remember:

```text
Encapsulation → Protect internal state

Abstraction   → Hide implementation complexity
```

---

### Mistake 4: Thinking inheritance is always better

Inheritance creates a strong relationship between parent and child classes.

In many cases, composition provides better flexibility.

```text
Inheritance → IS-A

Composition → HAS-A
```

---

# Real-World Example

Consider an e-commerce application.

We could have:

```text
Payment
   |
   ├── CreditCardPayment
   ├── UPIPayment
   └── WalletPayment
```

The application can depend on the `Payment` interface:

```java
interface Payment {

    void pay(double amount);
}
```

Different implementations can provide different behavior:

```java
class CreditCardPayment implements Payment {

    @Override
    public void pay(double amount) {
        System.out.println("Paid using credit card: " + amount);
    }
}
```

```java
class UPIPayment implements Payment {

    @Override
    public void pay(double amount) {
        System.out.println("Paid using UPI: " + amount);
    }
}
```

Then:

```java
Payment payment = new UPIPayment();

payment.pay(1000);
```

This demonstrates **polymorphism** because the same `Payment` interface can represent different payment implementations.

It also demonstrates **abstraction** because the caller only needs to know about `pay()` and does not need to know how the payment is processed internally.

---

# 🎯 Quick Revision

```text
Encapsulation
    ↓
Protect the data

Inheritance
    ↓
Reuse and extend behavior

Polymorphism
    ↓
Same interface, different behavior

Abstraction
    ↓
Hide implementation details
```

## Interview Mindset

Don't just memorize definitions.

For every OOP concept, be able to explain:

```text
What is it?
      ↓
Why do we need it?
      ↓
How does Java implement it?
      ↓
Can you give a code example?
      ↓
Where would you use it in a real project?
      ↓
What follow-up questions can the interviewer ask?
```

This is the difference between **knowing OOP** and being able to **answer an OOP interview question confidently**.

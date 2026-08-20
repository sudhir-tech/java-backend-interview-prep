# Core Java Interview Questions — Java Backend Developer

A practical interview-focused collection of Core Java questions and concise answers for Java Backend Developer interviews.

---

# 1. What are the main features of Java?

Java is:

- Object-oriented
- Platform independent
- Strongly typed
- Garbage collected
- Multithreaded
- Secure by design
- Rich in standard libraries

A common explanation is:

```text
Write Once, Run Anywhere
```

Java source code is compiled into bytecode, which runs on the JVM.

---

# 2. JDK vs JRE vs JVM

### JVM

Runs Java bytecode.

### JRE

Conceptually:

```text
JVM + Java runtime libraries
```

### JDK

Conceptually:

```text
JRE/runtime + development tools
```

Examples of JDK tools:

```text
javac
java
javadoc
jdb
jar
```

Modern Java distributions are generally installed as JDKs; the historical standalone JRE distribution is no longer the normal installation model for current Java releases.

---

# 3. Why is Java platform independent?

Java source code is compiled into bytecode:

```text
Java source
    ↓
javac
    ↓
Bytecode
    ↓
JVM
    ↓
Operating system
```

The same bytecode can run on different operating systems as long as a compatible JVM is available.

---

# 4. What is bytecode?

Bytecode is the intermediate instruction format produced by the Java compiler.

Example:

```text
Hello.java
    ↓
javac
    ↓
Hello.class
```

The JVM executes or compiles this bytecode at runtime.

---

# 5. What is the JVM?

The Java Virtual Machine executes Java bytecode and provides runtime services such as:

```text
Memory management
Garbage collection
Class loading
JIT compilation
Thread management
Security/runtime checks
```

---

# 6. What is JIT?

JIT stands for:

```text
Just-In-Time compilation
```

The JVM can identify frequently executed code and compile it into optimized native machine code during runtime.

Conceptually:

```text
Bytecode
   ↓
JVM execution
   ↓
Hot code detected
   ↓
JIT compilation
   ↓
Optimized native code
```

---

# 7. What is a class?

A class is a blueprint that defines data and behavior.

Example:

```java
class User {

    private String name;

    public void login() {
        System.out.println("Login");
    }
}
```

---

# 8. What is an object?

An object is an instance of a class.

```java
User user =
    new User();
```

Here:

```text
User → class
user → reference
new User() → object
```

---

# 9. Class vs Object

### Class

Defines structure and behavior.

### Object

Represents an actual runtime instance.

Example:

```text
Class → User
Objects → user1, user2, user3
```

---

# 10. What are the four pillars of OOP?

```text
Encapsulation
Inheritance
Polymorphism
Abstraction
```

---

# 11. What is encapsulation?

Encapsulation means keeping an object's state controlled through its public interface.

Example:

```java
class Account {

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

The field is not directly exposed.

---

# 12. What is inheritance?

Inheritance allows a class to reuse and specialize behavior from another class.

```java
class Animal {
    void eat() {
    }
}

class Dog extends Animal {
    void bark() {
    }
}
```

`Dog` inherits accessible behavior from `Animal`.

---

# 13. What is polymorphism?

Polymorphism means the same interface can represent different implementations.

Example:

```java
Animal animal =
    new Dog();

animal.eat();
```

A common backend example:

```java
Payment payment =
    new CardPayment();
```

The reference type is `Payment`, while the actual object is `CardPayment`.

---

# 14. Compile-time vs runtime polymorphism

### Compile-time

Usually associated with method overloading.

```java
void print(int x)
void print(String x)
```

### Runtime

Usually associated with method overriding.

```java
Animal animal =
    new Dog();

animal.sound();
```

The overridden implementation is selected at runtime.

---

# 15. What is abstraction?

Abstraction means exposing the necessary behavior while hiding implementation details.

Example:

```java
interface Payment {
    void pay();
}
```

Different implementations:

```java
class CardPayment
        implements Payment {
}

class UpiPayment
        implements Payment {
}
```

The caller depends on the abstraction.

---

# 16. Interface vs Abstract Class

### Interface

Useful for defining a contract.

```java
interface Payment {
    void pay();
}
```

A class can implement multiple interfaces.

### Abstract class

Useful when related classes share state or implementation.

```java
abstract class Vehicle {

    protected String number;

    abstract void start();
}
```

A class can extend only one class.

---

# 17. Can an interface have methods with implementation?

Yes.

Modern Java interfaces can contain:

```text
default methods
static methods
private methods
```

Example:

```java
interface Logger {

    default void log() {
        System.out.println("Log");
    }
}
```

---

# 18. What is method overloading?

Same method name with different parameter lists.

```java
void add(int a, int b) {
}

void add(int a, int b, int c) {
}
```

Return type alone cannot distinguish overloaded methods.

This is compile-time polymorphism.

---

# 19. What is method overriding?

A subclass provides its own implementation of an inherited instance method.

```java
class Animal {

    void sound() {
        System.out.println("Animal");
    }
}

class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Dog");
    }
}
```

---

# 20. Rules for method overriding

Important rules include:

- Method signature must be compatible.
- Return type can be covariant.
- Access cannot be reduced.
- `final` methods cannot be overridden.
- `static` methods are hidden, not overridden.
- `private` methods are not overridden.
- Constructors are not overridden.

---

# 21. Can static methods be overridden?

No.

Static methods belong to the class.

A subclass can define a static method with the same signature, but this is method hiding rather than runtime overriding.

---

# 22. Can private methods be overridden?

No.

Private methods are not inherited as overridable methods.

A subclass can define another method with the same name, but it is not an override.

---

# 23. Can final methods be overridden?

No.

```java
final void calculate() {
}
```

A subclass cannot override a final method.

---

# 24. What is a constructor?

A constructor initializes an object when it is created.

```java
class User {

    User(String name) {
        this.name = name;
    }

    private String name;
}
```

A constructor:

- Has the same name as the class.
- Has no return type.
- Runs during object construction.
- Can be overloaded.

---

# 25. Can a constructor be inherited?

No.

Constructors belong to the class that declares them.

A subclass can invoke a superclass constructor using:

```java
super();
```

---

# 26. this vs super

### this

Refers to the current object.

```java
this.name = name;
```

### super

Refers to the superclass portion of the object or invokes superclass constructors/methods.

```java
super();
```

---

# 27. What is constructor chaining?

Constructors can call other constructors.

Within the same class:

```java
this();
```

Superclass constructor:

```java
super();
```

A constructor call must appear as the first statement of the constructor.

---

# 28. What is the Object class?

`java.lang.Object` is the root class of the Java class hierarchy.

Important methods include:

```java
toString()
equals()
hashCode()
getClass()
wait()
notify()
notifyAll()
```

---

# 29. equals() vs ==

### ==

For objects, normally compares whether two references refer to the same object.

```java
a == b
```

### equals()

Can compare logical equality when the class provides an appropriate implementation.

```java
a.equals(b)
```

Example:

```java
String a = new String("Java");
String b = new String("Java");

a == b       // false
a.equals(b)  // true
```

---

# 30. equals() and hashCode() contract

If:

```java
a.equals(b)
```

is true, then:

```java
a.hashCode() == b.hashCode()
```

must also be true.

The reverse is not required:

```text
same hash code
≠
objects must be equal
```

This is critical for:

```text
HashMap
HashSet
HashTable-like hashing structures
```

---

# 31. Why override equals() and hashCode() together?

If you override only `equals()` but not `hashCode()`, logically equal objects may produce different hash codes.

That can break expected behavior in hash-based collections.

Correct pattern:

```java
@Override
public boolean equals(Object obj) {
    // equality logic
}

@Override
public int hashCode() {
    // matching hash logic
}
```

---

# 32. What is String?

`String` represents immutable sequences of characters.

Example:

```java
String name =
    "Sudhir";
```

Once a String object exists, its contents cannot be changed.

---

# 33. Why is String immutable?

Immutability provides benefits such as:

```text
Thread safety
String pool optimization
Security
Stable hash codes
Safe use as HashMap keys
```

Example:

```java
String s = "Java";

s.concat(" Backend");
```

The original String is unchanged because `concat()` returns a new String.

---

# 34. String pool

Java maintains a pool of String literals.

Example:

```java
String a = "Java";
String b = "Java";
```

The literals can refer to the same pooled String object.

But:

```java
String c =
    new String("Java");
```

explicitly creates a new String object.

---

# 35. String vs StringBuilder vs StringBuffer

### String

Immutable.

### StringBuilder

Mutable and generally preferred for repeated string modifications in single-threaded code.

### StringBuffer

Mutable and synchronized; generally less preferred than StringBuilder when synchronization is unnecessary.

Example:

```java
StringBuilder builder =
    new StringBuilder();

builder.append("Java");
builder.append(" Backend");
```

---

# 36. Why is StringBuilder faster for repeated concatenation?

Repeated String concatenation can create multiple intermediate String objects.

`StringBuilder` maintains mutable character data and can reduce unnecessary intermediate allocations.

---

# 37. What is the String Constant Pool?

It is a JVM-managed area used to canonicalize String literals and other interned strings.

Example:

```java
String a = "hello";
String b = "hello";
```

The literals can share the same pooled object.

---

# 38. What does intern() do?

`intern()` returns the canonical representation of a String from the string pool.

```java
String a =
    new String("Java");

String b =
    a.intern();
```

Use it carefully; unnecessary interning can increase memory pressure.

---

# 39. What are access modifiers?

Java has:

```text
public
protected
default/package-private
private
```

### public

Accessible wherever the containing type is accessible.

### protected

Accessible within the same package and through inheritance rules from other packages.

### package-private

Accessible only within the same package.

### private

Accessible only within the declaring class.

---

# 40. final keyword

`final` can apply to variables, methods and classes.

### final variable

Cannot be reassigned.

```java
final int x = 10;
```

### final method

Cannot be overridden.

### final class

Cannot be extended.

```java
final class Utility {
}
```

---

# 41. final reference vs immutable object

Important distinction:

```java
final User user =
    new User();
```

The reference cannot point to another object:

```java
user = new User(); // not allowed
```

But the object's internal state may still be mutable.

So:

```text
final reference
≠
immutable object
```

---

# 42. static keyword

`static` means a member belongs to the class rather than a particular instance.

Example:

```java
class Counter {

    static int count;
}
```

Access:

```java
Counter.count;
```

---

# 43. Static block

A static initializer runs when the class is initialized.

```java
static {
    System.out.println("Initialized");
}
```

It is commonly used for class-level initialization, although constructors or explicit initialization methods are often clearer for application code.

---

# 44. Instance block

An instance initializer runs during object construction as part of the constructor initialization process.

```java
{
    System.out.println("Instance block");
}
```

It runs for each object created.

---

# 45. What is an immutable class?

An immutable object cannot have its observable state changed after construction.

Typical design:

```text
Class final
Fields private final
No setters
Defensive copies for mutable fields
State initialized in constructor
```

Example:

```java
public final class User {

    private final String name;

    public User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

---

# 46. What is a record?

A record is a concise way to model immutable-style data carriers.

Example:

```java
public record User(
    Long id,
    String name
) {
}
```

The compiler provides members such as:

```text
accessors
constructor
equals()
hashCode()
toString()
```

Records are especially useful for DTOs and value-like data.

---

# 47. What is an enum?

An enum represents a fixed set of constants.

```java
enum Status {
    PENDING,
    PAID,
    CANCELLED
}
```

Enums can also have fields, constructors and methods.

---

# 48. What is autoboxing?

Automatic conversion from primitive to wrapper.

```java
Integer x = 10;
```

Conceptually:

```java
Integer.valueOf(10);
```

---

# 49. What is unboxing?

Conversion from wrapper to primitive.

```java
Integer x = 10;

int y = x;
```

Conceptually:

```java
x.intValue();
```

Be careful:

```java
Integer x = null;

int y = x;
```

can cause:

```text
NullPointerException
```

---

# 50. Primitive vs Wrapper

### Primitive

```text
int
long
double
boolean
char
```

### Wrapper

```text
Integer
Long
Double
Boolean
Character
```

Wrappers are objects and are useful where Java requires reference types, such as generic collections.

---

# 51. What is an exception?

An exception represents an abnormal condition that interrupts normal program flow.

Examples:

```text
NullPointerException
IOException
SQLException
IllegalArgumentException
```

---

# 52. Checked vs Unchecked Exceptions

### Checked

Compiler requires handling or declaring them.

Examples:

```text
IOException
SQLException
```

### Unchecked

Subclasses of `RuntimeException`.

Examples:

```text
NullPointerException
IllegalArgumentException
IndexOutOfBoundsException
```

---

# 53. Error vs Exception

`Error` generally represents serious JVM/system-level problems.

Examples:

```text
OutOfMemoryError
StackOverflowError
```

Applications generally should not try to recover from every Error.

Exceptions generally represent conditions application code may handle or propagate.

---

# 54. throw vs throws

### throw

Actually throws an exception.

```java
throw new IllegalArgumentException(
    "Invalid amount"
);
```

### throws

Declares that a method may propagate exceptions.

```java
void readFile()
        throws IOException {
}
```

---

# 55. try-catch-finally

```java
try {
    // risky operation
}
catch (IOException e) {
    // handle
}
finally {
    // cleanup
}
```

With resources, prefer try-with-resources when applicable.

---

# 56. Can finally be skipped?

Usually it runs when control leaves the try/catch construct, but there are exceptional cases such as:

```text
System.exit()
JVM crash
Process termination
Power failure
```

Do not rely on `finally` for actions that must survive process termination.

---

# 57. What is a custom exception?

An application-specific exception that represents a domain or business condition.

```java
class InsufficientBalanceException
        extends RuntimeException {

    public InsufficientBalanceException(
            String message) {
        super(message);
    }
}
```

---

# 58. What is a stack trace?

A stack trace shows the sequence of method calls leading to an exception.

Example:

```text
main()
  ↓
service()
  ↓
repository()
  ↓
exception
```

It helps identify where the failure originated.

---

# 59. What is garbage collection?

Garbage collection automatically identifies objects that are no longer reachable and reclaims their memory.

Conceptually:

```text
Create object
    ↓
Object reachable
    ↓
References removed
    ↓
Object becomes unreachable
    ↓
GC may reclaim memory
```

You cannot rely on an exact GC execution time.

---

# 60. Can we force garbage collection?

You can request it:

```java
System.gc();
```

but this is only a request to the JVM.

The JVM is free to decide whether and when to perform garbage collection.

---

# 61. What makes an object eligible for GC?

An object becomes eligible when it is no longer reachable through references from GC roots.

Common GC roots include:

```text
Active thread references
Static references
JNI references
Live stack references
```

---

# 62. Stack vs Heap

### Stack

Generally contains:

```text
Method frames
Local variables/references
Call state
```

Each thread has its own stack.

### Heap

Contains objects and arrays managed by the JVM's memory management system.

The heap is shared across threads.

---

# 63. StackOverflowError vs OutOfMemoryError

### StackOverflowError

Can happen with excessively deep recursion.

```java
void recurse() {
    recurse();
}
```

### OutOfMemoryError

Can happen when the JVM cannot satisfy memory allocation requirements.

---

# 64. What is a memory leak in Java?

Java has garbage collection, but an application can still retain references to objects that it no longer needs.

Example:

```java
static List<Object> cache =
    new ArrayList<>();
```

If objects are continuously added and never removed, memory usage can grow.

Common causes:

```text
Unbounded caches
Listeners not deregistered
Static collections
ThreadLocal misuse
Long-lived references
```

---

# 65. What is a ThreadLocal?

`ThreadLocal` provides thread-specific values.

```java
ThreadLocal<String> user =
    new ThreadLocal<>();
```

Each thread can have its own value.

Useful examples:

```text
Request context
Tracing context
Thread-scoped state
```

But in thread pools, values should be removed when no longer needed:

```java
try {
    user.set("Sudhir");
}
finally {
    user.remove();
}
```

This is especially important because pooled threads are reused.

---

# 66. What is a thread?

A thread is an execution path within a process.

Java supports concurrent execution through:

```text
Thread
Runnable
Callable
ExecutorService
CompletableFuture
Virtual threads
```

---

# 67. Process vs Thread

### Process

An independent running program with its own process-level resources.

### Thread

An execution unit within a process.

Threads in the same process generally share:

```text
Heap
Process resources
```

but each has its own stack and execution state.

---

# 68. Runnable vs Callable

### Runnable

Does not return a result.

```java
Runnable task =
    () -> System.out.println("Hello");
```

### Callable

Can return a result and throw checked exceptions.

```java
Callable<Integer> task =
    () -> 10 + 20;
```

---

# 69. ExecutorService

`ExecutorService` manages task execution using a pool or executor strategy.

Example:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(4);

executor.submit(() ->
    processOrder()
);

executor.shutdown();
```

For modern applications, choose executor configuration based on workload rather than blindly using a fixed pool.

---

# 70. synchronized

`synchronized` provides mutual exclusion and memory-visibility guarantees around a monitor.

Example:

```java
public synchronized void increment() {
    count++;
}
```

Only one thread at a time can execute the synchronized region for the same monitor.

---

# 71. synchronized method vs block

### Method

```java
public synchronized void update() {
}
```

Locks the relevant object monitor.

### Block

```java
synchronized (lock) {
    update();
}
```

Allows more precise control over what is synchronized and which monitor is used.

---

# 72. What is volatile?

`volatile` ensures that reads and writes of the variable have the required visibility/order guarantees across threads.

Example:

```java
private volatile boolean running;
```

Important:

> `volatile` does not make compound operations such as `count++` atomic.

---

# 73. AtomicInteger

For atomic numeric updates:

```java
AtomicInteger counter =
    new AtomicInteger();

counter.incrementAndGet();
```

Useful when you need atomic operations without using a synchronized block for that particular state.

---

# 74. Race condition

A race condition occurs when the result depends on the timing/interleaving of concurrent operations.

Example:

```java
count++;
```

is not one indivisible operation.

Conceptually:

```text
read
modify
write
```

Two threads can interleave these operations and lose updates.

---

# 75. Deadlock

Deadlock occurs when threads wait indefinitely for resources held by each other.

Example:

```text
Thread A
  locks A
  waits for B

Thread B
  locks B
  waits for A
```

Prevention strategies include:

```text
Consistent lock ordering
Reducing lock scope
Avoiding unnecessary nested locks
Using timeout-based locking where appropriate
```

---

# 76. ConcurrentHashMap

`ConcurrentHashMap` is designed for concurrent access.

Example:

```java
ConcurrentHashMap<String, Integer>
    counts = new ConcurrentHashMap<>();
```

It provides thread-safe operations without synchronizing the entire map for every operation.

---

# 77. HashMap vs ConcurrentHashMap

### HashMap

Not thread-safe.

### ConcurrentHashMap

Designed for concurrent access.

Also:

```text
HashMap → permits one null key and null values
ConcurrentHashMap → does not permit null keys or values
```

---

# 78. What is a fail-fast iterator?

Some collection iterators detect structural modification during iteration and may throw:

```text
ConcurrentModificationException
```

Example:

```java
for (String item : list) {
    list.remove(item);
}
```

This is not a synchronization mechanism.

---

# 79. ArrayList vs LinkedList

### ArrayList

Backed by a dynamically resizable array.

Good for:

```text
Fast random access
Iteration
Appending at the end
```

### LinkedList

Node-based linked structure.

Can be useful for certain deque operations, but `ArrayDeque` is usually a better choice for queue/deque workloads.

---

# 80. HashMap internal concept

A `HashMap` uses hashing to locate entries.

Conceptually:

```text
key
 ↓
hash
 ↓
bucket
 ↓
entry
```

Collisions can occur when multiple keys map to the same bucket.

Modern Java implementations can use tree structures for heavily collided buckets under certain conditions.

---

# 81. HashMap key requirements

A good key should have:

```text
Stable hashCode
Consistent equals/hashCode
State that does not change while used as a key
```

If key fields used by `equals()` or `hashCode()` change after insertion, lookup can fail.

---

# 82. Comparable vs Comparator

### Comparable

Defines natural ordering inside the class.

```java
class User
        implements Comparable<User> {
}
```

Method:

```java
compareTo()
```

### Comparator

Defines external/custom ordering.

```java
Comparator<User> byName =
    Comparator.comparing(
        User::getName
    );
```

---

# 83. What is Optional?

`Optional<T>` represents a value that may or may not be present.

Example:

```java
Optional<User> user =
    repository.findById(id);
```

Useful for representing absence explicitly.

Avoid using Optional indiscriminately for every field or parameter.

---

# 84. map vs flatMap in Optional

### map

Transforms the contained value.

```java
optional.map(User::getName);
```

### flatMap

Useful when the mapping function already returns an Optional.

```java
optional.flatMap(
    user -> repository.findAddress(user)
);
```

---

# 85. What is a lambda expression?

A lambda is a concise way to represent behavior.

```java
(a, b) -> a + b
```

It is commonly used with functional interfaces.

---

# 86. Functional interface

An interface with exactly one abstract method.

Example:

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}
```

Can be used with:

```java
Calculator c =
    (a, b) -> a + b;
```

It may still contain default and static methods.

---

# 87. Common functional interfaces

From `java.util.function`:

```text
Predicate<T>
Function<T,R>
Consumer<T>
Supplier<T>
UnaryOperator<T>
BinaryOperator<T>
```

Examples:

```java
Predicate<Integer> positive =
    x -> x > 0;

Function<String, Integer> length =
    String::length;
```

---

# 88. Stream API

Streams provide a declarative way to process data.

Example:

```java
List<String> names =
    users.stream()
         .map(User::getName)
         .filter(name ->
             name.startsWith("S"))
         .toList();
```

A stream is not a data structure; it represents a pipeline for processing data.

---

# 89. Intermediate vs terminal operations

### Intermediate

Return another stream.

Examples:

```text
map
filter
sorted
distinct
```

### Terminal

Produce a result or side effect and consume the stream.

Examples:

```text
collect
toList
forEach
count
reduce
```

---

# 90. Lazy evaluation in streams

Intermediate operations are generally lazy.

```java
users.stream()
     .filter(...)
     .map(...);
```

Nothing is actually processed until a terminal operation runs.

---

# 91. map vs flatMap in streams

### map

One input → one output.

```java
users.stream()
     .map(User::getName)
```

### flatMap

One input → multiple values, flattened into one stream.

```java
orders.stream()
      .flatMap(order ->
          order.getItems().stream())
```

---

# 92. Stream vs Collection

### Collection

Stores data.

### Stream

Processes data.

A stream generally does not own the underlying data source.

---

# 93. What is a parallel stream?

A parallel stream can split stream processing across multiple threads, typically using the common ForkJoinPool.

Example:

```java
users.parallelStream()
     .map(this::process)
     .toList();
```

Do not assume parallel streams are always faster.

They can be harmful for:

```text
Small datasets
Blocking I/O
Order-sensitive processing
Shared mutable state
CPU/resource contention
```

---

# 94. What is serialization?

Serialization converts an object into a representation that can be stored or transmitted.

Traditional Java serialization uses:

```java
Serializable
```

But backend applications often use formats such as:

```text
JSON
Protocol Buffers
Avro
```

depending on the system.

---

# 95. What is a DTO?

DTO stands for:

```text
Data Transfer Object
```

It represents data transferred between application boundaries.

Example:

```java
public record UserResponse(
    Long id,
    String name
) {
}
```

DTOs help prevent exposing internal domain/entity structures directly through APIs.

---

# 96. Entity vs DTO

### Entity

Represents persistence/domain state.

```java
@Entity
class User {
}
```

### DTO

Represents data transferred across an API or application boundary.

```java
record UserResponse(
    Long id,
    String name
) {}
```

Avoid exposing persistence entities directly when doing so creates unwanted coupling or security risks.

---

# 97. What is dependency injection?

Dependency injection means an object's dependencies are supplied from outside instead of the object constructing them itself.

Without DI:

```java
class OrderService {

    private PaymentService payment =
        new PaymentService();
}
```

With DI:

```java
class OrderService {

    private final PaymentService payment;

    OrderService(
        PaymentService payment
    ) {
        this.payment = payment;
    }
}
```

Spring commonly manages this dependency injection.

---

# 98. Why is constructor injection preferred?

Constructor injection:

```text
Makes dependencies explicit
Supports final fields
Improves testability
Prevents partially initialized objects
Makes required dependencies clear
```

Example:

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(
        PaymentService paymentService
    ) {
        this.paymentService = paymentService;
    }
}
```

---

# 99. What is immutability useful for in backend systems?

Immutable objects are useful because they:

```text
Reduce accidental state changes
Are easier to reason about
Can be safer in concurrent code
Work well as value objects
Can simplify caching
```

Examples:

```text
String
Records
Immutable DTOs
Value objects
```

---

# 100. What is SOLID?

SOLID represents five design principles:

```text
S — Single Responsibility
O — Open/Closed
L — Liskov Substitution
I — Interface Segregation
D — Dependency Inversion
```

---

# 101. Single Responsibility Principle

A class should have one clear responsibility and one primary reason to change.

Bad:

```text
OrderService
  + business logic
  + database code
  + email sending
  + PDF generation
```

Better:

```text
OrderService
OrderRepository
EmailService
PdfService
```

---

# 102. Open/Closed Principle

Software entities should generally be:

```text
Open for extension
Closed for modification
```

Example:

```java
interface Payment {
    void pay();
}
```

Add:

```java
class CardPayment
        implements Payment {
}
```

without changing existing payment logic.

---

# 103. Liskov Substitution Principle

A subtype should be usable wherever its base abstraction is expected without breaking the expected behavior.

If a subclass violates the assumptions of the base abstraction, inheritance may be the wrong design.

---

# 104. Interface Segregation Principle

Clients should not be forced to depend on methods they do not need.

Prefer focused interfaces:

```java
interface Printer {
    void print();
}

interface Scanner {
    void scan();
}
```

rather than one huge interface containing unrelated operations.

---

# 105. Dependency Inversion Principle

High-level modules should depend on abstractions rather than concrete low-level implementations.

Example:

```java
class OrderService {

    private final PaymentGateway gateway;

    OrderService(
        PaymentGateway gateway
    ) {
        this.gateway = gateway;
    }
}
```

The service depends on:

```text
PaymentGateway
```

rather than a concrete payment provider.

---

# 106. Composition vs Inheritance

Composition means building behavior using contained objects.

```java
class OrderService {

    private final PaymentService payment;
}
```

Inheritance:

```java
class Dog extends Animal {
}
```

Composition often provides more flexibility and lower coupling than deep inheritance hierarchies.

---

# 107. What is dependency inversion vs dependency injection?

They are related but different.

### Dependency Inversion

A design principle:

```text
Depend on abstractions
```

### Dependency Injection

A mechanism for supplying dependencies:

```text
Constructor injection
Setter injection
Framework-managed injection
```

Spring commonly uses DI to implement designs that follow dependency inversion.

---

# 108. What is a design pattern?

A design pattern is a reusable approach to a recurring software design problem.

Examples:

```text
Factory
Builder
Strategy
Observer
Adapter
Decorator
Singleton
```

Patterns should solve actual design problems rather than being added just to make code look sophisticated.

---

# 109. Singleton

A Singleton ensures that a class has a controlled single instance within the intended scope.

However, careless Singleton usage can create:

```text
Global state
Testing difficulties
Hidden dependencies
Concurrency issues
```

In Spring, singleton bean scope means one bean instance per Spring application context by default; that is not exactly the same concept as a JVM-wide GoF Singleton.

---

# 110. Factory Pattern

Factory encapsulates object creation.

Example:

```java
Payment payment =
    PaymentFactory.create(type);
```

The caller does not need to know the concrete implementation creation details.

---

# 111. Strategy Pattern

Strategy allows interchangeable algorithms behind an abstraction.

```java
interface PricingStrategy {
    double calculate(Order order);
}
```

Implementations:

```text
RegularPricing
DiscountPricing
PremiumPricing
```

The service can select the strategy at runtime.

---

# 112. Builder Pattern

Builder is useful for constructing objects with many optional fields.

Example:

```java
User user =
    User.builder()
        .name("Sudhir")
        .age(28)
        .build();
```

Records or constructors can sometimes provide a simpler alternative.

---

# 113. What is defensive copying?

Defensive copying prevents callers from modifying internal mutable state.

Bad:

```java
public List<String> getItems() {
    return items;
}
```

Potentially safer:

```java
public List<String> getItems() {
    return List.copyOf(items);
}
```

This is particularly important when designing immutable objects.

---

# 114. What is shallow copy vs deep copy?

### Shallow copy

Copies the outer object but references may still point to the same nested objects.

### Deep copy

Copies the nested object graph as well.

Deep copying can be expensive and should be used only when required.

---

# 115. What is dependency scope?

In a backend application, dependency scope determines where and how a dependency is available.

For Maven, examples include:

```text
compile
provided
runtime
test
```

Understanding dependency scope helps keep application packages and runtime dependencies correct.

---

# 116. What is class loading?

The JVM loads class definitions when needed.

Conceptually:

```text
Loading
  ↓
Linking
  ↓
Initialization
```

Class loading is handled through class loaders.

---

# 117. ClassLoader

A `ClassLoader` loads class definitions into the JVM.

Examples include:

```text
Bootstrap class loader
Platform class loader
Application class loader
```

Custom class loaders are also possible.

---

# 118. What is parent delegation?

A class loader generally delegates class-loading requests to its parent before attempting to load the class itself.

Conceptually:

```text
Application ClassLoader
        ↓
Platform ClassLoader
        ↓
Bootstrap ClassLoader
```

This helps prevent application code from replacing core Java classes unexpectedly.

---

# 119. What is reflection?

Reflection allows runtime inspection and interaction with:

```text
Classes
Methods
Fields
Constructors
Annotations
```

Example:

```java
Class<?> clazz =
    User.class;
```

Frameworks such as Spring use reflection and related runtime infrastructure extensively.

---

# 120. What are annotations?

Annotations provide metadata.

Example:

```java
@Service
class UserService {
}
```

The annotation can be processed by:

```text
Compiler
Annotation processor
Framework
Runtime reflection
```

---

# 121. What is @Retention?

It controls how long an annotation is retained:

```text
SOURCE
CLASS
RUNTIME
```

Runtime annotations can be inspected through reflection.

---

# 122. What is @Target?

Defines where an annotation can be applied.

Examples:

```text
TYPE
METHOD
FIELD
PARAMETER
CONSTRUCTOR
```

---

# 123. What is garbage collector generational thinking?

Many JVM collectors exploit the observation that many objects die young.

Conceptually:

```text
New objects
   ↓
Young generation / young collection
   ↓
Long-lived objects
   ↓
Older generations / regions
```

The exact implementation depends on the selected garbage collector and Java version.

Do not assume every modern JVM uses the same generation layout internally.

---

# 124. What is a stop-the-world pause?

During certain JVM operations, application threads may be paused while the JVM performs required work.

Garbage collection can involve stop-the-world phases.

Modern collectors aim to reduce pause times, but they do not eliminate all pauses.

---

# 125. What is G1 GC?

G1, or Garbage-First Garbage Collector, divides the heap into regions and aims to provide predictable pause-time behavior while managing large heaps.

It is a common general-purpose collector in modern Java.

---

# 126. What is a thread pool?

A thread pool maintains reusable worker threads.

Conceptually:

```text
Tasks
 ↓
Queue
 ↓
Worker threads
 ↓
Execute
```

Benefits:

```text
Avoid repeated thread creation
Control concurrency
Improve resource usage
```

---

# 127. What is Executor vs ExecutorService?

### Executor

Basic interface for executing tasks.

```java
executor.execute(task);
```

### ExecutorService

Adds lifecycle management and task submission APIs.

```java
Future<?> future =
    executor.submit(task);
```

It can be shut down using:

```java
executor.shutdown();
```

---

# 128. Future vs CompletableFuture

### Future

Represents a result that may become available later.

```java
Future<Integer> future =
    executor.submit(task);
```

### CompletableFuture

Supports asynchronous composition.

```java
CompletableFuture
    .supplyAsync(this::getUser)
    .thenApply(this::mapUser);
```

It supports chaining, combining and exception handling.

---

# 129. What is CompletableFuture?

It represents a value that may be completed asynchronously.

Example:

```java
CompletableFuture<String> future =
    CompletableFuture.supplyAsync(
        () -> "Java"
    );

future.thenApply(String::toUpperCase)
      .thenAccept(System.out::println);
```

---

# 130. thenApply vs thenCompose

### thenApply

Transforms one result:

```java
future.thenApply(
    User::getName
);
```

### thenCompose

Chains another asynchronous operation and flattens nested futures.

```java
getUser()
    .thenCompose(
        user -> getOrders(user.id())
    );
```

---

# 131. thenCombine

Combines two independent asynchronous results.

```java
userFuture.thenCombine(
    orderFuture,
    (user, orders) ->
        new UserSummary(user, orders)
);
```

---

# 132. Exception handling in CompletableFuture

Common methods:

```text
exceptionally
handle
whenComplete
```

Example:

```java
future.exceptionally(
    ex -> "fallback"
);
```

---

# 133. What are virtual threads?

Virtual threads are lightweight Java threads managed by the JVM rather than directly mapping every application thread to a dedicated OS thread.

They are particularly useful for high-concurrency workloads with blocking I/O.

Example:

```java
Thread.startVirtualThread(
    () -> processRequest()
);
```

They do not make CPU-bound operations inherently faster.

---

# 134. Platform thread vs virtual thread

### Platform thread

Typically backed by an operating-system thread.

### Virtual thread

Lightweight JVM-managed thread designed to make large numbers of concurrent tasks easier to handle.

Use virtual threads primarily to simplify high-concurrency I/O-bound code.

---

# 135. What is synchronization?

Synchronization controls concurrent access to shared mutable state.

Tools include:

```text
synchronized
Lock
Atomic classes
Concurrent collections
volatile
```

Choose based on the actual concurrency requirement.

---

# 136. ReentrantLock vs synchronized

`ReentrantLock` provides additional capabilities such as:

```text
tryLock()
Timed lock acquisition
Interruptible lock acquisition
Multiple conditions
```

`synchronized` is simpler and automatically releases the monitor when leaving the synchronized region.

---

# 137. What is atomicity?

An operation is atomic when it appears indivisible to other threads.

Example:

```java
AtomicInteger count =
    new AtomicInteger();

count.incrementAndGet();
```

Atomicity is different from visibility and ordering.

---

# 138. Visibility vs Atomicity

### Visibility

One thread sees another thread's updates.

`volatile` can provide visibility guarantees.

### Atomicity

An operation happens as one indivisible unit.

`volatile count++` is still not atomic.

---

# 139. What is immutability in concurrency?

Immutable objects do not change after construction, so multiple threads can safely share them without coordinating changes to their state.

This is one reason immutable value objects are useful in concurrent systems.

---

# 140. What is a race condition?

A race condition occurs when correctness depends on unpredictable timing between concurrent operations.

Typical prevention:

```text
Synchronization
Atomic operations
Immutable state
Concurrent data structures
Proper ownership
```

---

# 141. What is deadlock?

Two or more threads wait forever for resources held by one another.

Common prevention:

```text
Consistent lock ordering
Small critical sections
Avoid nested locks
Timed lock acquisition
```

---

# 142. What is livelock?

Threads are active but continuously respond to each other without making useful progress.

Unlike deadlock:

```text
Deadlock → threads are blocked
Livelock → threads keep running but don't progress
```

---

# 143. What is starvation?

A thread waits indefinitely because other threads repeatedly get access to the resource or execution opportunity it needs.

---

# 144. How do you make a class thread-safe?

Possible approaches:

```text
Make it immutable
Avoid shared mutable state
Synchronize critical sections
Use concurrent collections
Use atomic variables
Use locks carefully
```

The best solution depends on the state and access pattern.

---

# 145. What is a defensive copy?

If a class stores a mutable object:

```java
private final Date date;
```

returning the original reference can expose internal state.

Instead, return a copy or immutable representation.

Modern Java often favors immutable types such as:

```java
Instant
LocalDate
LocalDateTime
```

over mutable legacy date types.

---

# 146. Why prefer java.time?

The `java.time` API provides immutable and thread-safe date/time types.

Examples:

```text
LocalDate
LocalTime
LocalDateTime
Instant
ZonedDateTime
Duration
Period
```

---

# 147. LocalDate vs LocalDateTime vs Instant

### LocalDate

Date without time:

```text
2026-08-20
```

### LocalDateTime

Date + time without a zone/offset:

```text
2026-08-20T18:30
```

### Instant

A point on the UTC timeline.

Useful for timestamps exchanged between distributed systems.

---

# 148. Why is timezone handling important?

Distributed applications can run across different time zones.

Prefer:

```text
UTC / Instant
```

for many machine-generated timestamps.

Convert to a user's local time zone at the presentation boundary when required.

---

# 149. What is BigDecimal used for?

`BigDecimal` is useful for decimal arithmetic where exact decimal representation matters, such as monetary calculations.

Avoid:

```java
double price = 0.1 + 0.2;
```

when exact decimal arithmetic is required.

Use:

```java
BigDecimal price =
    new BigDecimal("0.10");
```

---

# 150. Why use BigDecimal(String)?

Prefer:

```java
new BigDecimal("0.1");
```

over:

```java
new BigDecimal(0.1);
```

because the `double` already contains a binary floating-point approximation.

---

# 151. What is an idempotent operation?

An operation is idempotent when performing it multiple times has the same intended effect as performing it once.

Examples in API design can include:

```text
PUT
DELETE
```

depending on the implementation and business semantics.

This is especially important in distributed systems and retry logic.

---

# 152. What is defensive exception handling?

Good exception handling should:

```text
Preserve useful context
Avoid swallowing failures
Use appropriate exception types
Log at the correct boundary
Avoid exposing sensitive details
```

Bad:

```java
catch (Exception e) {
    // ignore
}
```

---

# 153. What is clean code?

Clean code is code that is:

```text
Readable
Maintainable
Testable
Focused
Consistent
Easy to change
```

Examples:

```text
Meaningful names
Small focused methods
Clear responsibilities
Low coupling
High cohesion
```

---

# 154. What is high cohesion?

A class has high cohesion when its responsibilities are closely related.

Example:

```text
OrderService
```

should primarily contain order-related business behavior rather than unrelated email, reporting and file-processing logic.

---

# 155. What is low coupling?

Low coupling means components have minimal unnecessary dependencies on each other.

Example:

```java
OrderService
    ↓
PaymentGateway interface
```

rather than:

```java
OrderService
    ↓
SpecificPaymentVendorImplementation
```

---

# 156. What is dependency injection in unit testing?

Constructor injection makes testing simple:

```java
PaymentGateway mockGateway =
    mock(PaymentGateway.class);

OrderService service =
    new OrderService(mockGateway);
```

The service does not need to create the dependency itself.

---

# 157. What is mocking?

Mocking replaces a dependency with a test-controlled implementation.

Example:

```text
OrderService
    ↓
PaymentGateway mock
```

This allows testing business logic without calling the real external payment system.

---

# 158. Unit test vs integration test

### Unit test

Tests a small unit in isolation.

```text
Service
 ↓
Mock dependencies
```

### Integration test

Tests interaction between components or external infrastructure.

Examples:

```text
Application + database
Application + REST API
Application + message broker
```

---

# 159. What is defensive programming?

Designing software to handle invalid or unexpected inputs safely.

Examples:

```text
Validate API input
Check nullability
Validate ranges
Handle external failures
Set timeouts
Limit resource usage
```

---

# 160. What makes backend code production-ready?

A strong backend implementation considers:

```text
Correctness
Security
Validation
Error handling
Logging
Monitoring
Performance
Scalability
Testing
Database behavior
Concurrency
Resource management
```

---

# 161. Java Backend Scenario: API is slow

If an API becomes slow, don't immediately optimize Java code.

Investigate:

```text
Application logs
Database query time
External API latency
Connection pool
Thread pool
CPU
Memory
GC
Network
Caching
```

Then identify the actual bottleneck.

---

# 162. Java Backend Scenario: High CPU

Possible causes:

```text
Infinite loops
Heavy algorithms
Excessive serialization
High request volume
CPU-heavy business logic
GC pressure
Thread contention
```

Approach:

```text
Measure
 ↓
Profile
 ↓
Identify hot path
 ↓
Optimize
 ↓
Benchmark
```

---

# 163. Java Backend Scenario: High memory

Investigate:

```text
Heap usage
GC behavior
Large collections
Unbounded caches
Large request payloads
Thread count
Object retention
Memory leaks
```

A heap dump can help identify retained objects.

---

# 164. Java Backend Scenario: Database connection pool exhausted

Possible causes:

```text
Slow queries
Connections not released
Long transactions
Too much concurrency
Pool too small
Database overloaded
```

Investigate:

```text
HikariCP metrics
Database performance
Connection acquisition time
Transaction duration
Application logs
```

Do not simply increase the pool size without finding the bottleneck.

---

# 165. Java Backend Scenario: Memory leak

Approach:

```text
Observe heap growth
 ↓
Take heap dump
 ↓
Analyze retained objects
 ↓
Identify reference chain
 ↓
Fix ownership/lifecycle issue
 ↓
Verify with load testing
```

---

# 166. Java Backend Scenario: ConcurrentHashMap or synchronized Map?

If multiple threads need concurrent access to a shared map:

```text
ConcurrentHashMap
```

is usually more scalable than synchronizing every operation on a standard HashMap.

But choose based on the actual operations and consistency requirements.

---

# 167. Java Backend Scenario: ArrayList or Set?

Use:

### List

When:

```text
Order matters
Duplicates are allowed
Index-based access is useful
```

### Set

When:

```text
Uniqueness matters
Membership checks are important
```

---

# 168. Java Backend Scenario: HashMap or TreeMap?

### HashMap

Good for average-case fast lookup.

### TreeMap

Maintains keys in sorted order and provides tree-based operations.

Use TreeMap when sorted-map behavior is required.

---

# 169. Java Backend Scenario: HashSet or ArrayList?

If you frequently need:

```text
contains()
```

and uniqueness is required, a HashSet can be appropriate.

If order and indexed access matter, use a List.

Always consider the actual workload.

---

# 170. Java Backend Scenario: StringBuilder or String?

Use:

```text
String
```

for immutable text values.

Use:

```text
StringBuilder
```

for repeated modifications within a method or local operation.

---

# 171. Java Backend Scenario: synchronized or AtomicInteger?

For a simple atomic numeric update:

```java
AtomicInteger
```

may be appropriate.

For protecting multiple related pieces of shared state as one invariant:

```java
synchronized
```

or a lock may be more appropriate.

Choose based on the critical section, not the popularity of the API.

---

# 172. Java Backend Scenario: Optional.empty() or null?

`Optional` can make absence explicit in APIs such as repository lookups.

Example:

```java
Optional<User> findById(Long id);
```

However, don't automatically use Optional for every internal field, method parameter or data structure.

---

# 173. Java Backend Scenario: checked or unchecked exception?

A useful rule:

### Checked

When callers are realistically expected to recover from a condition and the API benefits from forcing explicit handling.

### Unchecked

Often used for programming errors and many business/application exceptions where explicit declaration at every layer adds little value.

Consistency within the application matters.

---

# 174. Java Backend Scenario: How do you handle external API failure?

Use:

```text
Timeouts
Retries where safe
Exponential backoff
Circuit breakers where appropriate
Fallbacks where meaningful
Logging
Metrics
Tracing
Idempotency
```

Never retry blindly.

Retries can amplify an outage.

---

# 175. Java Backend Scenario: How do you improve API performance?

Start with measurement.

Possible improvements:

```text
Database query optimization
Indexes
Pagination
Caching
Connection pool tuning
Reduce unnecessary network calls
Batch operations
Efficient serialization
Avoid N+1 queries
```

---

# 176. Java Backend Scenario: What is N+1 query problem?

An application executes:

```text
1 query for parent records
+
N queries for related records
```

Example:

```text
1 query → 100 orders
100 queries → order items
```

This can create unnecessary database load.

Possible solutions include:

```text
Fetch joins
Entity graphs
Batch fetching
Explicit queries
DTO projections
```

Choose according to the use case.

---

# 177. Java Backend Scenario: What is lazy loading?

Lazy loading delays loading related data until it is actually accessed.

This can reduce unnecessary database work, but can also cause:

```text
N+1 queries
LazyInitializationException
Unexpected database calls
```

Use it deliberately.

---

# 178. Java Backend Scenario: What is connection pooling?

Instead of opening a new database connection for every request:

```text
Connection pool
  ↓
Reusable database connections
```

Benefits:

```text
Lower connection creation overhead
Controlled database concurrency
Better throughput
```

Common Java backend pool:

```text
HikariCP
```

---

# 179. Java Backend Scenario: What is caching?

Caching stores frequently used data closer to the application.

Example:

```text
Request
 ↓
Cache
 ├─ hit → return
 └─ miss → database → cache → return
```

Caching can improve latency and reduce database load.

---

# 180. Cache invalidation

Caching introduces consistency questions.

Common strategies:

```text
TTL
Cache-aside
Write-through
Write-behind
Explicit invalidation
```

A common Spring backend pattern is cache-aside behavior.

---

# 181. Java Backend Scenario: What is REST?

REST is an architectural style for distributed systems.

Common HTTP methods:

```text
GET
POST
PUT
PATCH
DELETE
```

Resources are represented through URLs and operations through HTTP semantics.

---

# 182. GET vs POST

### GET

Typically retrieves a resource and should be safe/idempotent.

### POST

Typically creates a resource or triggers a processing action and is not inherently idempotent.

---

# 183. PUT vs PATCH

### PUT

Usually represents replacement of a resource representation and is generally intended to be idempotent.

### PATCH

Usually represents a partial modification.

The exact semantics depend on the API contract.

---

# 184. HTTP status codes

Common codes:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Content
429 Too Many Requests
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

---

# 185. 401 vs 403

### 401 Unauthorized

The client has not provided valid authentication credentials.

### 403 Forbidden

The client is authenticated but does not have permission for the requested operation.

---

# 186. Authentication vs Authorization

### Authentication

```text
Who are you?
```

### Authorization

```text
What are you allowed to do?
```

Example:

```text
JWT authentication
+
ROLE_ADMIN authorization
```

---

# 187. JWT

JWT stands for:

```text
JSON Web Token
```

A typical JWT contains:

```text
Header
Payload
Signature
```

A backend can validate the signature and claims before allowing access.

Never put sensitive secrets directly into the JWT payload because the payload is normally readable by the token holder.

---

# 188. What is idempotency in payment APIs?

If a client retries:

```text
POST /payments
```

because of a network timeout, the server should avoid creating duplicate charges.

An idempotency key can help:

```text
Idempotency-Key: abc123
```

The server stores the result associated with the key and safely handles repeated requests.

---

# 189. What is pagination?

Pagination limits how many records are returned.

Example:

```text
?page=0&size=20
```

It prevents huge database queries and huge API responses.

For very large datasets, cursor/keyset pagination can be more efficient than deep offset pagination.

---

# 190. Offset vs cursor pagination

### Offset

```text
LIMIT 20 OFFSET 10000
```

Simple, but can become expensive for deep pages and can behave inconsistently as data changes.

### Cursor/keyset

```text
WHERE id > lastSeenId
LIMIT 20
```

Often better for large ordered datasets.

---

# 191. What is API rate limiting?

Rate limiting controls how frequently a client can call an API.

Example:

```text
100 requests/minute
```

Benefits:

```text
Protect backend
Prevent abuse
Control traffic spikes
Improve fairness
```

---

# 192. What is graceful degradation?

When part of a system fails, the application continues providing reduced functionality rather than failing completely.

Example:

```text
Recommendation service unavailable
        ↓
Checkout still works
        ↓
Recommendations omitted
```

---

# 193. What is a circuit breaker?

A circuit breaker prevents repeated calls to an unhealthy downstream service.

Conceptually:

```text
Closed
  ↓ failures
Open
  ↓ timeout
Half-open
  ↓ test request
Closed/Open
```

It helps prevent cascading failures.

---

# 194. What is backpressure?

Backpressure prevents producers from overwhelming consumers.

Example:

```text
Producer
   ↓
Queue
   ↓
Consumer
```

If consumers cannot keep up, the system needs a strategy such as:

```text
Bounded queue
Rate limiting
Batching
Dropping
Retry policies
Flow control
```

---

# 195. What is observability?

Observability helps understand system behavior through:

```text
Logs
Metrics
Traces
```

Common backend tooling:

```text
ELK
Prometheus
Grafana
OpenTelemetry
```

---

# 196. What is structured logging?

Instead of:

```text
Order failed
```

structured logging can include:

```text
orderId
userId
timestamp
service
errorCode
```

Often represented as JSON.

This makes searching and analysis easier.

---

# 197. What is correlation ID?

A correlation ID identifies a request across services.

Example:

```text
API Gateway
    ↓ correlationId=abc
Order Service
    ↓ abc
Payment Service
    ↓ abc
Notification Service
```

This makes distributed troubleshooting much easier.

---

# 198. What is a transaction?

A transaction groups operations into a unit with transactional guarantees.

The classic ACID properties are:

```text
Atomicity
Consistency
Isolation
Durability
```

---

# 199. What is transaction isolation?

Isolation controls how concurrent transactions interact.

Common levels:

```text
READ UNCOMMITTED
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

Higher isolation can reduce anomalies but may reduce concurrency.

---

# 200. Final Java Backend Interview Checklist

Before an interview, be able to explain these without memorizing a textbook answer:

## Core Java

```text
JDK / JRE / JVM
Bytecode
JIT
OOP
Class / Object
Inheritance
Polymorphism
Abstraction
Encapsulation
Interface / Abstract class
Overloading / Overriding
final / static
equals / hashCode
String immutability
StringBuilder
Access modifiers
Exceptions
Garbage collection
Stack / Heap
Class loading
Reflection
Annotations
```

## Collections

```text
ArrayList
LinkedList
HashMap
HashSet
TreeMap
TreeSet
ConcurrentHashMap
Comparable
Comparator
Iterator
Fail-fast behavior
```

## Modern Java

```text
Lambda
Functional interfaces
Streams
Optional
Records
java.time
CompletableFuture
Virtual threads
```

## Concurrency

```text
Thread
ExecutorService
synchronized
volatile
AtomicInteger
Locks
Race condition
Deadlock
Starvation
Livelock
ThreadLocal
```

## Backend Engineering

```text
REST
HTTP methods
Status codes
Authentication
Authorization
JWT
Pagination
Caching
Transactions
Connection pooling
N+1 queries
Idempotency
Rate limiting
Circuit breakers
Backpressure
Observability
```

---

# 201. Rapid-Fire Questions

Try answering these aloud in 30–60 seconds each:

1. Why is String immutable?
2. Why override hashCode when overriding equals?
3. HashMap internal working?
4. ArrayList vs LinkedList?
5. HashMap vs ConcurrentHashMap?
6. Comparable vs Comparator?
7. What is fail-fast behavior?
8. What is Optional?
9. map vs flatMap?
10. What is a functional interface?
11. What is a stream?
12. Why are streams lazy?
13. What is a race condition?
14. volatile vs synchronized?
15. AtomicInteger vs synchronized?
16. What causes deadlock?
17. How do you prevent deadlock?
18. What is ExecutorService?
19. Future vs CompletableFuture?
20. thenApply vs thenCompose?
21. What are virtual threads?
22. What is garbage collection?
23. What causes memory leaks in Java?
24. Stack vs heap?
25. What is JIT?
26. What is reflection?
27. What are annotations?
28. What is dependency injection?
29. Why constructor injection?
30. What is SOLID?
31. Composition vs inheritance?
32. What is a DTO?
33. What is connection pooling?
34. What is caching?
35. What is N+1?
36. What is idempotency?
37. 401 vs 403?
38. PUT vs PATCH?
39. Offset vs cursor pagination?
40. How would you debug a slow API?

---

# 202. Strong Interview Mindset

For experienced Java backend interviews, don't answer only with definitions.

Use this structure:

```text
Definition
   ↓
Why it matters
   ↓
Small example
   ↓
Backend use case
   ↓
Trade-off
```

Example:

### Interviewer:

Why use ConcurrentHashMap?

### Better answer:

> `ConcurrentHashMap` is designed for concurrent access to a map. It allows multiple threads to work with the map safely without using one global synchronized lock for every operation. I'd use it when multiple backend threads need shared map state, but I'd still check whether I actually need shared mutable state because avoiding shared state can be simpler.

This sounds much stronger than:

> "ConcurrentHashMap is thread-safe."

---

# 203. Final Preparation Rule

For your Java Backend interviews, prioritize these topics first:

```text
1. Collections
2. String / equals / hashCode
3. OOP
4. Exceptions
5. Java 8+ features
6. Streams
7. Multithreading
8. Concurrent collections
9. JVM / GC
10. Spring Boot
11. REST APIs
12. SQL
13. Microservices
14. System Design
15. Debugging / production scenarios
```

The goal is not to memorize every Java API.

The goal is to be able to explain:

```text
Why?
How?
When?
Trade-offs?
Real backend example?
```

That is what separates a memorized interview answer from an experienced engineering answer.

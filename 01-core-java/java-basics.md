# Java Basics — Interview Preparation

A practical Core Java guide covering the fundamentals that Java Backend Developers are expected to know in interviews.

---

# 1. What Is Java?

Java is a high-level, object-oriented programming language designed to be portable, robust and suitable for building applications ranging from backend services to enterprise systems.

A major Java concept is:

```text
Write Once, Run Anywhere
```

Java source code is compiled into bytecode, which runs on a Java Virtual Machine (JVM).

```text
Java Source Code
       ↓
    javac
       ↓
   Bytecode
       ↓
      JVM
       ↓
Machine-specific execution
```

---

# 2. Why Is Java Platform Independent?

Java source code is not compiled directly into one specific operating system's machine code.

Instead:

```text
.java
  ↓
bytecode
  ↓
JVM
  ↓
OS-specific execution
```

Different operating systems have different JVM implementations.

Therefore, the same Java bytecode can run on different platforms as long as a compatible JVM exists.

---

# 3. JDK vs JRE vs JVM

This is one of the most common Java interview questions.

## JVM

JVM stands for:

```text
Java Virtual Machine
```

It executes Java bytecode.

Responsibilities include:

- Bytecode execution
- Memory management
- Garbage collection
- Runtime environment
- JIT compilation

---

## JRE

JRE stands for:

```text
Java Runtime Environment
```

Conceptually:

```text
JRE = JVM + runtime libraries
```

It provides what is needed to run Java applications.

---

## JDK

JDK stands for:

```text
Java Development Kit
```

Conceptually:

```text
JDK = JRE/runtime + development tools
```

It includes tools such as:

```text
javac
java
javadoc
jar
jdb
```

Modern Java distributions have evolved beyond the old simple JDK/JRE packaging distinction, but the conceptual difference remains useful for interviews.

---

# 4. javac vs java

### javac

Compiles Java source code:

```bash
javac Main.java
```

Output:

```text
Main.class
```

### java

Runs the compiled application:

```bash
java Main
```

The JVM loads and executes the bytecode.

---

# 5. Basic Java Program

```java
public class Main {

    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

Important parts:

```text
public
static
void
main
String[]
args
```

---

# 6. main() Method

The traditional Java application entry point is:

```java
public static void main(String[] args)
```

### public

The JVM needs to be able to access the method.

### static

The JVM can invoke it without creating an instance of `Main`.

### void

The method doesn't return a value.

### String[] args

Contains command-line arguments.

---

# 7. Class

A class is a blueprint used to create objects.

Example:

```java
class Product {

    String name;
    double price;

    void display() {
        System.out.println(name);
    }
}
```

---

# 8. Object

An object is an instance of a class.

```java
Product product =
    new Product();
```

Now:

```text
Product
   ↓
Object
```

The object has its own instance state.

---

# 9. Constructor

A constructor initializes an object when it is created.

```java
class Product {

    private String name;

    Product(String name) {
        this.name = name;
    }
}
```

Create:

```java
Product product =
    new Product("Laptop");
```

---

# 10. Constructor Rules

Important rules:

- Constructor name must match the class name.
- Constructors do not have a return type.
- Constructors are called during object creation.
- Constructors can be overloaded.
- Constructors are not inherited.
- Constructors cannot be overridden.

Example:

```java
class User {

    User() {
    }

    User(String name) {
    }
}
```

---

# 11. Default Constructor

If you don't declare any constructor, the compiler can provide a no-argument constructor.

Example:

```java
class User {
}
```

Conceptually:

```java
User() {
}
```

But if you declare your own constructor:

```java
class User {

    User(String name) {
    }
}
```

Java does not automatically add a no-argument constructor.

Therefore:

```java
new User();
```

would not compile unless you explicitly define one.

---

# 12. this Keyword

`this` refers to the current object.

Example:

```java
class User {

    private String name;

    User(String name) {
        this.name = name;
    }
}
```

Here:

```text
this.name
```

refers to the instance field.

---

# 13. Common Uses of this

`this` can be used to:

- Refer to current object's fields
- Call another constructor
- Pass the current object
- Return the current object

Constructor chaining:

```java
class User {

    private String name;
    private int age;

    User() {
        this("Unknown", 0);
    }

    User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

---

# 14. super Keyword

`super` refers to the parent class portion of the current object.

Example:

```java
class Animal {

    String name = "Animal";
}

class Dog extends Animal {

    String name = "Dog";

    void printNames() {
        System.out.println(name);
        System.out.println(super.name);
    }
}
```

Output conceptually:

```text
Dog
Animal
```

---

# 15. super Constructor

A child class can invoke a parent constructor using:

```java
super();
```

Example:

```java
class Animal {

    Animal(String name) {
    }
}

class Dog extends Animal {

    Dog(String name) {
        super(name);
    }
}
```

A superclass constructor call must occur as the first statement in the constructor.

---

# 16. Access Modifiers

Java provides four main access levels:

```text
public
protected
default/package-private
private
```

---

# 17. public

Accessible from anywhere where the containing type is accessible.

```java
public class User {

    public void login() {
    }
}
```

---

# 18. private

Accessible only inside the declaring class.

```java
class User {

    private String password;
}
```

This is commonly used for encapsulation.

---

# 19. protected

Accessible:

- Within the same package
- In subclasses, including subclasses in other packages subject to Java's protected-access rules

Example:

```java
class Parent {

    protected int value;
}
```

---

# 20. Default / Package-Private

If no access modifier is specified:

```java
class User {

    String name;
}
```

the member is package-private.

It is accessible within the same package.

---

# 21. Access Modifier Summary

```text
Modifier       Same Class   Same Package   Subclass   Other Package
--------------------------------------------------------------------
private        Yes          No             No         No
default        Yes          Yes            Package    No
protected      Yes          Yes            Yes*       Yes*
public         Yes          Yes            Yes        Yes
```

`protected` access from another package has additional restrictions: access is through inheritance, not arbitrary package-level access.

---

# 22. Primitive Data Types

Java has eight primitive types:

```text
byte
short
int
long
float
double
char
boolean
```

---

# 23. Integer Types

```java
byte b = 10;
short s = 100;
int i = 1000;
long l = 100000L;
```

Typical sizes:

```text
byte   → 8 bits
short  → 16 bits
int    → 32 bits
long   → 64 bits
```

---

# 24. Floating-Point Types

```java
float price = 10.5f;
double amount = 100.50;
```

Typical sizes:

```text
float  → 32 bits
double → 64 bits
```

For financial calculations, `BigDecimal` is generally preferred over floating-point types because decimal floating-point arithmetic can introduce precision issues.

---

# 25. char

`char` represents a UTF-16 code unit.

Example:

```java
char grade = 'A';
```

Important:

```text
'A'
```

is a character literal.

```text
"A"
```

is a String.

---

# 26. boolean

```java
boolean active = true;
```

It represents:

```text
true
false
```

Do not assume its storage size from implementation details when answering interviews; the Java language specifies the type's behavior rather than exposing a simple memory-size contract like C primitives.

---

# 27. Reference Types

Examples:

```java
String name;
User user;
Product product;
int[] numbers;
List<String> names;
```

Reference variables hold references to objects rather than storing the entire object directly in the variable.

---

# 28. Primitive vs Reference Types

### Primitive

```java
int age = 25;
```

Contains a primitive value.

### Reference

```java
User user =
    new User();
```

The variable refers to an object.

Important interview point:

> Java is always pass-by-value. For object references, the value being passed is the reference value.

---

# 29. Wrapper Classes

Primitive types have corresponding wrapper classes:

```text
byte    → Byte
short   → Short
int     → Integer
long    → Long
float   → Float
double  → Double
char    → Character
boolean → Boolean
```

Example:

```java
Integer number = 100;
```

---

# 30. Autoboxing

Autoboxing converts a primitive to its wrapper type.

```java
int x = 10;

Integer y = x;
```

Conceptually:

```text
int
 ↓
Integer
```

---

# 31. Unboxing

Unboxing converts a wrapper object to a primitive.

```java
Integer x = 10;

int y = x;
```

Conceptually:

```text
Integer
   ↓
int
```

Be careful with `null`:

```java
Integer value = null;

int number = value;
```

This can throw:

```text
NullPointerException
```

because Java must unbox `null`.

---

# 32. Integer Caching

Java commonly caches certain `Integer` values, especially values in the range:

```text
-128 to 127
```

Therefore:

```java
Integer a = 100;
Integer b = 100;

System.out.println(a == b);
```

can print:

```text
true
```

But:

```java
Integer a = 1000;
Integer b = 1000;

System.out.println(a == b);
```

should not be relied upon for value comparison.

Use:

```java
a.equals(b)
```

for wrapper value comparison.

---

# 33. == vs equals()

For primitives:

```java
int a = 10;
int b = 10;

a == b
```

compares values.

For object references:

```java
User a = new User();
User b = new User();

a == b
```

compares whether the references refer to the same object.

`equals()` is intended for logical equality when a class implements it appropriately.

---

# 34. Type Casting

Casting converts a value from one type to another where Java permits it.

Example:

```java
double price = 100.5;

int value = (int) price;
```

Result:

```text
100
```

The fractional part is discarded.

---

# 35. Widening Conversion

A smaller numeric type can generally be converted to a larger compatible numeric type automatically.

```java
int x = 100;

long y = x;
```

Conceptually:

```text
int → long
```

---

# 36. Narrowing Conversion

A larger numeric type to a smaller type generally requires explicit casting.

```java
long x = 100;

int y = (int) x;
```

Be careful about overflow.

---

# 37. String

`String` is a class representing text.

Example:

```java
String name = "Sudhir";
```

Strings are immutable.

Operations that appear to modify a String actually create another String object when necessary.

---

# 38. String Immutability

Example:

```java
String name = "Java";

name.concat(" Backend");
```

The original String is not changed.

Correct:

```java
name =
    name.concat(" Backend");
```

Now `name` refers to the new String.

---

# 39. String Pool

Java maintains a pool of interned strings.

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

Therefore:

```java
a == b
```

can be true, while:

```java
a == c
```

is false.

For text equality:

```java
a.equals(c)
```

---

# 40. StringBuilder

Use `StringBuilder` when repeatedly constructing or modifying strings in a single-threaded context.

Example:

```java
StringBuilder builder =
    new StringBuilder();

builder.append("Java");
builder.append(" ");
builder.append("Backend");

String result =
    builder.toString();
```

It is generally more efficient than repeatedly concatenating strings in a loop.

---

# 41. StringBuffer

`StringBuffer` provides synchronized methods and is designed for thread-safe mutable string operations.

For most normal application code:

```text
StringBuilder
```

is preferred unless synchronized StringBuffer behavior is specifically required.

---

# 42. static

`static` members belong to the class rather than a particular instance.

Example:

```java
class Counter {

    static int count;
}
```

Access:

```java
Counter.count
```

---

# 43. Static Method

```java
class MathUtil {

    static int add(int a, int b) {
        return a + b;
    }
}
```

Call:

```java
MathUtil.add(2, 3);
```

A static method cannot directly access instance fields because there is no implicit `this`.

---

# 44. Static Block

A static block runs when the class is initialized.

```java
class Config {

    static {
        System.out.println(
            "Class initialized"
        );
    }
}
```

Use static initialization carefully; application frameworks generally provide better lifecycle mechanisms for complex initialization.

---

# 45. final Variable

A final variable can be assigned only once.

```java
final int MAX = 100;
```

After assignment:

```java
MAX = 200;
```

is not allowed.

---

# 46. final Reference

A final reference cannot point to another object after assignment.

```java
final List<String> names =
    new ArrayList<>();
```

This is allowed:

```java
names.add("Java");
```

But this is not:

```java
names = new ArrayList<>();
```

Therefore:

```text
final reference ≠ immutable object
```

---

# 47. final Class

A final class cannot be extended.

```java
final class SecurityUtil {
}
```

This can be useful when a class is not intended for inheritance.

---

# 48. final Method

A final method cannot be overridden.

```java
class Parent {

    final void process() {
    }
}
```

A subclass cannot provide an override for `process()`.

---

# 49. Enum

An enum represents a fixed set of constants.

```java
enum OrderStatus {
    CREATED,
    PAID,
    SHIPPED,
    DELIVERED,
    CANCELLED
}
```

Use:

```java
OrderStatus status =
    OrderStatus.PAID;
```

Enums are useful for domain states that have a known finite set of values.

---

# 50. Enum with Fields

Enums can contain fields and methods.

```java
enum Priority {

    LOW(1),
    MEDIUM(2),
    HIGH(3);

    private final int value;

    Priority(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

---

# 51. Packages

Packages organize related classes and help avoid naming conflicts.

Example:

```java
package com.example.order;
```

A backend application might use:

```text
com.example.controller
com.example.service
com.example.repository
com.example.dto
com.example.entity
```

---

# 52. import

`import` allows you to use a type without writing its fully qualified name.

Instead of:

```java
java.util.ArrayList<String>
```

you can write:

```java
import java.util.ArrayList;
```

and then:

```java
ArrayList<String> list;
```

---

# 53. Object Class

Every Java class ultimately derives from:

```java
java.lang.Object
```

Important methods include:

```text
toString()
equals()
hashCode()
getClass()
```

Other methods include:

```text
wait()
notify()
notifyAll()
```

which are related to object monitor coordination.

---

# 54. toString()

`toString()` provides a string representation of an object.

Example:

```java
@Override
public String toString() {
    return "User{name='" +
        name + "'}";
}
```

Useful for:

```text
Logging
Debugging
Diagnostics
```

---

# 55. equals() and hashCode()

If you override:

```java
equals()
```

you should normally also override:

```java
hashCode()
```

The key contract is:

> If two objects are equal according to `equals()`, they must return the same hash code.

This is especially important when objects are used in:

```text
HashMap
HashSet
```

---

# 56. Pass-by-Value in Java

Java is always:

```text
Pass-by-value
```

For primitives:

```java
void update(int x) {
    x = 20;
}
```

The caller's primitive is not changed.

For objects:

```java
void update(User user) {
    user.setName("Alex");
}
```

the copied reference value points to the same object, so modifying that object's state can be visible to the caller.

But reassigning the parameter does not change the caller's reference:

```java
void update(User user) {
    user = new User();
}
```

---

# 57. Example of Pass-by-Value

```java
class User {
    String name;
}

static void change(User user) {

    user.name = "Alex";

    user = new User();
    user.name = "Bob";
}
```

If:

```java
User user = new User();
user.name = "Sudhir";

change(user);
```

after the method:

```text
user.name = Alex
```

The first mutation affects the shared object.

The parameter reassignment does not change the caller's variable.

---

# 58. Method Overloading

Same method name with different parameter lists.

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

Return type alone cannot distinguish overloaded methods.

This is invalid:

```java
int add(int a, int b) {
}

double add(int a, int b) {
}
```

---

# 59. Method Overriding

A subclass provides a compatible implementation of an inherited instance method.

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

# 60. @Override

Use:

```java
@Override
```

when overriding a method.

It helps the compiler detect mistakes.

For example, if the method signature does not actually override a parent method, the compiler reports an error.

---

# 61. Interface

An interface defines a contract that implementing classes agree to provide.

```java
interface Payment {

    void pay(double amount);
}
```

Implementation:

```java
class CardPayment
        implements Payment {

    @Override
    public void pay(double amount) {
        System.out.println(
            "Card payment"
        );
    }
}
```

Interfaces can also contain default, static and private methods in modern Java.

---

# 62. Abstract Class

An abstract class cannot normally be instantiated directly.

```java
abstract class Payment {

    abstract void pay();

    void log() {
        System.out.println(
            "Payment started"
        );
    }
}
```

A subclass can implement the abstract method.

---

# 63. Interface vs Abstract Class

### Interface

Useful for:

```text
Contract
Capability
Multiple implementations
Multiple interface inheritance
```

### Abstract Class

Useful when related classes need:

```text
Shared state
Shared implementation
Protected members
Common base behavior
```

Choose based on the actual design.

---

# 64. Multiple Inheritance

Java does not support multiple inheritance of classes.

This is not allowed:

```java
class C extends A, B {
}
```

But Java supports implementing multiple interfaces:

```java
class C
        implements A, B {
}
```

This avoids many ambiguities associated with multiple class inheritance.

---

# 65. Default Methods in Interfaces

Interfaces can provide default implementations.

```java
interface Logger {

    default void log() {
        System.out.println("Log");
    }
}
```

A class can use or override the default method.

---

# 66. Static Interface Methods

Interfaces can contain static methods.

```java
interface MathUtil {

    static int add(int a, int b) {
        return a + b;
    }
}
```

Call:

```java
MathUtil.add(2, 3);
```

The static method belongs to the interface.

---

# 67. Abstract Method

An abstract method declares behavior without providing the implementation.

```java
abstract class Animal {

    abstract void sound();
}
```

The concrete subclass provides the implementation.

---

# 68. Exception Basics

Java has a hierarchy based on:

```text
Throwable
   ├── Error
   └── Exception
```

`RuntimeException` is a subclass of `Exception`.

Examples:

```text
NullPointerException
IllegalArgumentException
IOException
SQLException
```

Exception handling is covered in detail in:

```text
exception-handling.md
```

---

# 69. Checked vs Unchecked Exceptions

### Checked

Generally subclasses of `Exception` other than `RuntimeException`.

The compiler requires them to be caught or declared.

Example:

```java
IOException
```

### Unchecked

`RuntimeException` and its subclasses.

Example:

```java
NullPointerException
IllegalArgumentException
```

They are not required to be explicitly declared or caught.

---

# 70. Error

`Error` represents serious conditions generally outside normal application recovery.

Examples:

```text
OutOfMemoryError
StackOverflowError
```

Application code generally should not try to handle these as ordinary business exceptions.

---

# 71. Garbage Collection

Java automatically manages memory using garbage collection.

Objects that are no longer reachable can become eligible for collection.

Example:

```java
User user =
    new User();

user = null;
```

If there are no other references to the object, it may become eligible for garbage collection.

More detail is covered in:

```text
jvm-memory-gc.md
```

---

# 72. Stack vs Heap

A simplified interview model:

```text
Stack
  ↓
Method frames
Local variables
Execution state

Heap
  ↓
Objects
Arrays
Instance data
```

Each thread has its own stack.

The heap is shared among threads.

This model is simplified; actual JVM implementation details can be more complex.

---

# 73. Null

`null` means a reference variable does not currently refer to an object.

Example:

```java
User user = null;
```

Calling an instance method through a null reference can result in:

```text
NullPointerException
```

---

# 74. var

Java supports local variable type inference with `var`.

Example:

```java
var name = "Java";
var count = 10;
```

The compiler determines the static type.

`var` does not make Java dynamically typed.

This is valid:

```java
var name = "Java";
```

but not:

```java
var name;
```

because the compiler needs an initializer to infer the type.

---

# 75. var Limitations

`var` can be used for local variables and certain loop variables.

It cannot be used as:

```text
Class fields
Method parameters
Method return types
```

Example:

```java
class User {

    var name; // invalid
}
```

---

# 76. Diamond Operator

Java can infer generic type arguments.

Instead of:

```java
List<String> names =
    new ArrayList<String>();
```

write:

```java
List<String> names =
    new ArrayList<>();
```

The `<>` is called the diamond operator.

---

# 77. LocalDate vs Date

For modern Java applications, prefer the `java.time` API.

Examples:

```java
LocalDate
LocalTime
LocalDateTime
Instant
ZonedDateTime
Duration
Period
```

Example:

```java
LocalDate today =
    LocalDate.now();
```

The modern API is generally preferable to the legacy `java.util.Date`/`Calendar` APIs.

---

# 78. BigDecimal

For precise decimal arithmetic, especially financial values:

```java
BigDecimal price =
    new BigDecimal("99.99");
```

Prefer constructing from a String when exact decimal representation is required.

Avoid:

```java
new BigDecimal(0.1);
```

because the binary floating-point value may not represent decimal `0.1` exactly.

---

# 79. Java Naming Conventions

### Class

```java
ProductService
```

PascalCase.

### Method

```java
calculatePrice()
```

camelCase.

### Variable

```java
totalPrice
```

camelCase.

### Constant

```java
MAX_RETRY_COUNT
```

UPPER_SNAKE_CASE.

### Package

```text
com.example.product
```

lowercase.

---

# 80. Clean Java Class Structure

A common structure:

```java
public class ProductService {

    private final ProductRepository repository;

    public ProductService(
        ProductRepository repository
    ) {
        this.repository = repository;
    }

    public Product getProduct(Long id) {
        return repository.findById(id);
    }

    private void validateProduct(Product product) {
        // validation
    }
}
```

Typical order:

```text
Fields
Constructors
Public methods
Private helper methods
```

Consistency matters more than a rigid universal ordering rule.

---

# 81. Java Bean

A traditional JavaBean generally follows conventions such as:

```text
Private fields
No-argument constructor
Getters/setters
Serializable support in traditional definitions
```

Example:

```java
public class User {

    private String name;

    public User() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

Not every modern Java DTO needs to follow all traditional JavaBean conventions.

---

# 82. Record

Modern Java provides records for concise data carriers.

Example:

```java
public record UserResponse(
    Long id,
    String name
) {
}
```

Records automatically provide useful implementations for:

```text
Accessors
equals()
hashCode()
toString()
```

and are designed around final record components.

---

# 83. Sealed Classes

Modern Java supports sealed types.

Example:

```java
public sealed interface Payment
    permits CardPayment, UpiPayment {
}
```

Only permitted types can directly implement the sealed interface.

This is useful when the set of allowed subtypes should be explicitly controlled.

---

# 84. Pattern Matching

Modern Java versions provide pattern matching features.

Example:

```java
if (obj instanceof String text) {
    System.out.println(
        text.toUpperCase()
    );
}
```

This combines type checking and variable binding.

---

# 85. Switch Expressions

Modern Java supports switch expressions.

```java
String result =
    switch (status) {
        case "PAID" ->
            "Payment successful";

        case "FAILED" ->
            "Payment failed";

        default ->
            "Unknown";
    };
```

This can be cleaner than older switch statement patterns.

---

# 86. Text Blocks

Java supports multi-line string literals using text blocks.

```java
String json = """
    {
      "name": "Java",
      "type": "backend"
    }
    """;
```

Useful for:

```text
JSON
SQL
Multi-line text
```

---

# 87. Optional

`Optional<T>` represents a value that may or may not be present.

Example:

```java
Optional<User> user =
    repository.findById(id);
```

Use:

```java
user.orElseThrow();
```

or:

```java
user.orElse(defaultUser);
```

Avoid blindly using:

```java
optional.get();
```

without checking presence or having a clear reason.

---

# 88. Functional Interfaces

A functional interface has one abstract method.

Example:

```java
@FunctionalInterface
interface Calculator {

    int calculate(int a, int b);
}
```

It can be used with a lambda:

```java
Calculator add =
    (a, b) -> a + b;
```

Common functional interfaces:

```text
Predicate<T>
Function<T, R>
Consumer<T>
Supplier<T>
UnaryOperator<T>
BinaryOperator<T>
```

More detail is covered in:

```text
java-8-lambda-streams.md
```

---

# 89. Lambda Expression

A lambda provides a concise implementation of a functional interface.

Example:

```java
List<String> names =
    List.of("Java", "Spring");

names.forEach(
    name -> System.out.println(name)
);
```

Lambda and Stream details are covered separately.

---

# 90. Immutable vs Mutable

### Mutable

State can change:

```java
List<String> list =
    new ArrayList<>();

list.add("Java");
```

### Immutable

State cannot be changed after creation.

Examples include:

```text
String
Integer
LocalDate
```

and properly designed immutable classes.

---

# 91. Why Is String Immutable?

Common reasons include:

```text
String pool sharing
Security
Thread safety
Hashing
Predictable behavior
```

Because a String cannot change after creation, it is safe to share its value across multiple references.

---

# 92. Why Is String a Good HashMap Key?

Strings are immutable.

Once a String is used as a key:

```java
Map<String, Integer> map =
    new HashMap<>();

map.put("Java", 1);
```

its hash code and equality behavior do not change due to mutation of the String itself.

This makes it suitable as a map key.

---

# 93. What Happens When You Create an Object?

For:

```java
User user =
    new User();
```

Conceptually:

```text
Evaluate constructor arguments
        ↓
Allocate/initialize object
        ↓
Initialize fields
        ↓
Run constructor initialization
        ↓
Return object reference
        ↓
Assign reference to user
```

The exact JVM allocation details are implementation-specific.

---

# 94. Initialization Order

For class initialization and object creation, a simplified order is useful to remember.

For a class:

```text
Static fields / static initialization
        ↓
Class initialization
```

For object creation:

```text
Superclass initialization
        ↓
Superclass instance initialization
        ↓
Superclass constructor
        ↓
Subclass instance initialization
        ↓
Subclass constructor
```

The exact details include field initializers and initialization blocks.

---

# 95. Static vs Instance

### Static

Belongs to the class:

```java
MathUtil.add(1, 2);
```

### Instance

Belongs to an object:

```java
User user =
    new User();

user.getName();
```

Static members should be used for behavior/state that logically belongs to the class rather than an individual object.

---

# 96. Can a Static Method Access Instance Variables?

Not directly.

Example:

```java
class User {

    private String name;

    static void printName() {
        System.out.println(name);
    }
}
```

This does not compile because there is no particular `User` instance associated with the static method.

It could access an instance through an explicit object reference.

---

# 97. Can an Instance Method Access Static Variables?

Yes.

```java
class Counter {

    static int count;

    void increment() {
        count++;
    }
}
```

The instance method can access class-level state.

---

# 98. Is Java 100% Object-Oriented?

A common interview answer:

> Java is strongly object-oriented, but it is not purely object-oriented because it has primitive types such as `int`, `boolean` and `char` that are not objects.

Wrapper classes provide object representations for primitives.

---

# 99. Why Does Java Not Support Operator Overloading?

Java intentionally keeps operator behavior relatively simple and predictable.

The language does not provide user-defined operator overloading like C++.

One notable built-in behavior is:

```java
String + String
```

for string concatenation.

---

# 100. Why Is Java Pass-by-Value?

Java passes the value of every argument.

For an object:

```java
User user
```

the value passed is the reference value.

Therefore:

```text
Java
 ↓
Always pass-by-value

Primitive argument
 ↓
copy of primitive value

Object argument
 ↓
copy of reference value
```

This is one of the most important Java interview concepts.

---

# 101. Core Java Quick Revision

```text
Java
 ↓
Source code → bytecode → JVM

JDK
 ↓
Development tools + runtime

JVM
 ↓
Executes bytecode

Class
 ↓
Blueprint

Object
 ↓
Class instance

this
 ↓
Current object

super
 ↓
Parent class portion

static
 ↓
Class-level member

final
 ↓
Cannot be reassigned / overridden / extended depending on usage

private
 ↓
Class-only access

protected
 ↓
Package + subclass access

public
 ↓
Broad access

String
 ↓
Immutable text

StringBuilder
 ↓
Mutable string construction

Wrapper
 ↓
Object form of primitive

Autoboxing
 ↓
Primitive → Wrapper

Unboxing
 ↓
Wrapper → Primitive

equals()
 ↓
Logical equality

==
 ↓
Primitive value or reference identity

interface
 ↓
Contract

abstract class
 ↓
Shared abstraction/state/implementation

enum
 ↓
Fixed set of constants

record
 ↓
Concise data carrier

Optional
 ↓
Explicitly represent possible absence

var
 ↓
Local type inference

BigDecimal
 ↓
Precise decimal arithmetic

LocalDate / java.time
 ↓
Modern date/time API
```

---

# 102. Top Java Basics Interview Questions

Before a Java Backend interview, be ready for these:

1. What is Java?
2. Why is Java platform independent?
3. What is JVM?
4. Difference between JDK, JRE and JVM?
5. What happens when Java code is compiled?
6. What is bytecode?
7. What is a class?
8. What is an object?
9. What is a constructor?
10. What is `this`?
11. What is `super`?
12. Difference between static and instance members?
13. What are access modifiers?
14. What are primitive data types?
15. Primitive vs reference types?
16. What is autoboxing?
17. What is unboxing?
18. What is String immutability?
19. What is String pool?
20. `==` vs `equals()`?
21. StringBuilder vs StringBuffer?
22. What is method overloading?
23. What is method overriding?
24. Can static methods be overridden?
25. Can private methods be overridden?
26. Can final methods be overridden?
27. Interface vs abstract class?
28. Does Java support multiple inheritance?
29. What is pass-by-value?
30. What is `final`?
31. What is an enum?
32. What is `var`?
33. What is Optional?
34. What is BigDecimal?
35. What is a record?
36. What are sealed classes?
37. What is a functional interface?
38. What is a lambda expression?
39. What is garbage collection?
40. Stack vs heap?

---

# 103. Strong Interview Answer — JDK, JRE, JVM

> JVM is responsible for executing Java bytecode. JRE conceptually provides the JVM and runtime libraries needed to run Java applications. JDK provides the development tools needed to build Java applications in addition to the runtime environment. In modern Java distributions, the packaging details have evolved, but this conceptual distinction is still useful.

---

# 104. Strong Interview Answer — Why Java Is Platform Independent

> Java source code is compiled into platform-independent bytecode. That bytecode is executed by a JVM, and different operating systems have their own JVM implementations. So the same compiled Java application can run across platforms with a compatible JVM.

---

# 105. Strong Interview Answer — == vs equals()

> For primitives, `==` compares values. For object references, `==` checks whether two references point to the same object. `equals()` is used for logical equality when the class provides an appropriate implementation.

---

# 106. Strong Interview Answer — String Immutability

> String is immutable, meaning its value cannot be changed after creation. This makes String safe to share, supports string pooling, provides stable hashing and helps with security and thread-safety.

---

# 107. Strong Interview Answer — Pass-by-Value

> Java is always pass-by-value. For primitives, the primitive value is copied. For objects, the reference value is copied, so both the caller and method parameter can refer to the same object. Mutating that object can be visible to the caller, but reassigning the method parameter does not change the caller's reference.

---

# 108. Strong Interview Answer — Interface vs Abstract Class

> I use an interface when I primarily need a contract or capability that can have multiple implementations. I use an abstract class when closely related classes need shared state or common implementation. The choice depends on the relationship and design requirements.

---

# 109. Strong Interview Answer — Why Use BigDecimal?

> For financial or other exact decimal calculations, I prefer BigDecimal because binary floating-point types such as double cannot represent many decimal fractions exactly. I also avoid constructing BigDecimal directly from a double when exact decimal input is required.

---

# 110. Final Java Basics Checklist

Before moving on, make sure you can explain these without memorizing definitions:

```text
✓ JDK / JRE / JVM
✓ Bytecode
✓ Class / Object
✓ Constructor
✓ this / super
✓ Access modifiers
✓ Primitive / Reference
✓ Wrapper classes
✓ Autoboxing / Unboxing
✓ == / equals()
✓ String pool
✓ String immutability
✓ StringBuilder
✓ static
✓ final
✓ Overloading
✓ Overriding
✓ Interface
✓ Abstract class
✓ Enum
✓ Pass-by-value
✓ Stack / Heap
✓ Garbage collection
✓ var
✓ Optional
✓ BigDecimal
✓ java.time
✓ Records
✓ Sealed classes
✓ Functional interfaces
✓ Lambdas
```

This file is the foundation for the rest of the Core Java section. For deeper topics, refer to:

```text
collections.md
equals-hashcode.md
exception-handling.md
generics.md
java-8-features.md
java-8-lambda-streams.md
jvm-memory-gc.md
multithreading-concurrency.md
oops.md
strings.md
```

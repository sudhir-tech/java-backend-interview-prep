# Exception Handling in Java

Exception handling is a mechanism used to detect, handle and recover from abnormal situations that occur while a Java program is running.

Instead of allowing an application to fail unexpectedly, exception handling allows us to define how errors should be handled.

---

# 1. What is an Exception?

An exception is an event that disrupts the normal flow of program execution.

Example:

```java
int result = 10 / 0;
```

This causes:

```text
ArithmeticException: / by zero
```

Without handling the exception, the current execution flow is interrupted.

---

# 2. Exception Hierarchy

A simplified Java exception hierarchy looks like this:

```text
                    Throwable
                       |
             __________________
            |                  |
          Error             Exception
                               |
                    ______________________
                   |                      |
          RuntimeException        Checked Exceptions
                   |
        ______________________
       |          |           |
 Arithmetic   NullPointer   IndexOutOfBounds
 Exception     Exception      Exception
```

`Throwable` is the root class for exceptions and errors.

The two major branches are:

- `Error`
- `Exception`

---

# 3. Error vs Exception

## Error

Errors generally represent serious problems that applications usually should not try to recover from.

Examples:

```text
OutOfMemoryError
StackOverflowError
NoClassDefFoundError
```

## Exception

Exceptions represent conditions that an application may be able to handle.

Examples:

```text
IOException
SQLException
NullPointerException
IllegalArgumentException
```

### Interview Answer

> Errors generally represent serious JVM or system-level problems, while exceptions represent conditions that an application can potentially handle.

---

# 4. Checked Exceptions

Checked exceptions are checked by the compiler.

Examples:

- IOException
- SQLException
- FileNotFoundException
- ClassNotFoundException

Example:

```java
import java.io.FileReader;
import java.io.IOException;

public class Example {

    public static void main(String[] args) throws IOException {

        FileReader reader = new FileReader("data.txt");

        reader.close();
    }
}
```

You must either:

- Handle the exception using `try-catch`
- Or declare it using `throws`

### Interview Answer

> Checked exceptions are exceptions that the compiler requires us to handle or declare.

---

# 5. Unchecked Exceptions

Unchecked exceptions are subclasses of `RuntimeException`.

The compiler does not require them to be explicitly handled.

Examples:

- NullPointerException
- ArithmeticException
- IllegalArgumentException
- IndexOutOfBoundsException
- NumberFormatException

Example:

```java
String name = null;

System.out.println(name.length());
```

This causes:

```text
NullPointerException
```

### Interview Answer

> Unchecked exceptions are RuntimeException subclasses and are not checked by the compiler.

---

# 6. Checked vs Unchecked Exceptions

| Checked | Unchecked |
|---|---|
| Checked at compile time | Occur during runtime |
| Must be handled or declared | No mandatory handling |
| Subclasses of Exception excluding RuntimeException | RuntimeException and its subclasses |
| Often represent external/recoverable conditions | Often represent programming errors or invalid state |
| Example: IOException | Example: NullPointerException |

---

# 7. try-catch

The `try` block contains code that may throw an exception.

The `catch` block handles it.

```java
try {

    int result = 10 / 0;

} catch (ArithmeticException e) {

    System.out.println("Cannot divide by zero");
}
```

Output:

```text
Cannot divide by zero
```

---

# 8. Multiple catch Blocks

A `try` block can have multiple catch blocks.

```java
try {

    int[] numbers = {10, 20, 30};

    System.out.println(numbers[5]);

} catch (ArithmeticException e) {

    System.out.println("Arithmetic error");

} catch (ArrayIndexOutOfBoundsException e) {

    System.out.println("Invalid array index");
}
```

### Important Rule

More specific exceptions should generally be caught before broader exceptions.

Correct:

```java
try {

    // code

} catch (NullPointerException e) {

    // specific

} catch (RuntimeException e) {

    // broader
}
```

Incorrect:

```java
try {

    // code

} catch (RuntimeException e) {

    // broader

} catch (NullPointerException e) {

    // unreachable
}
```

---

# 9. finally

The `finally` block is used for cleanup code.

It generally executes whether an exception occurs or not.

```java
try {

    System.out.println("Opening resource");

} catch (Exception e) {

    System.out.println("Handling exception");

} finally {

    System.out.println("Cleanup");
}
```

### Interview Answer

> The finally block is generally used for cleanup operations and normally executes whether an exception is thrown or not.

---

# 10. Can finally be skipped?

Yes, there are exceptional situations where `finally` may not execute.

For example:

```java
System.exit(0);
```

or if the JVM terminates abnormally.

Therefore, avoid saying:

> finally always executes.

A better answer is:

> Finally normally executes when control leaves the try/catch structure, except in situations such as JVM termination.

---

# 11. try-with-resources

Try-with-resources automatically closes resources that implement `AutoCloseable`.

Example:

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class Example {

    public static void main(String[] args) {

        try (BufferedReader reader =
                     new BufferedReader(new FileReader("data.txt"))) {

            System.out.println(reader.readLine());

        } catch (IOException e) {

            System.out.println("File error: " + e.getMessage());
        }
    }
}
```

The reader is automatically closed.

### Interview Answer

> Try-with-resources automatically closes AutoCloseable resources and helps prevent resource leaks.

---

# 12. Why is try-with-resources preferred?

Traditional approach:

```java
BufferedReader reader = null;

try {

    reader = new BufferedReader(new FileReader("data.txt"));

} finally {

    if (reader != null) {
        reader.close();
    }
}
```

Try-with-resources:

```java
try (BufferedReader reader =
         new BufferedReader(new FileReader("data.txt"))) {

    // use reader
}
```

It is:

- Cleaner
- Safer
- Easier to maintain
- Less error-prone

---

# 13. throw

The `throw` keyword is used to explicitly throw an exception.

```java
public void withdraw(double amount) {

    if (amount <= 0) {
        throw new IllegalArgumentException(
            "Amount must be positive"
        );
    }
}
```

### Interview Answer

> `throw` is used to explicitly create and throw an exception from a specific point in the program.

---

# 14. throws

The `throws` keyword declares that a method may throw one or more exceptions.

```java
public void readFile() throws IOException {

    FileReader reader = new FileReader("data.txt");

    reader.close();
}
```

The caller is responsible for handling or declaring the exception.

---

# 15. throw vs throws

| throw | throws |
|---|---|
| Actually throws an exception | Declares possible exceptions |
| Used inside method body | Used in method signature |
| Throws one exception at a time | Can declare multiple exceptions |
| Example: `throw new RuntimeException()` | Example: `throws IOException` |

### Easy way to remember

```text
throw
  ↓
"I am throwing this exception."

throws
  ↓
"This method may throw these exceptions."
```

---

# 16. Custom Exceptions

You can create your own exception classes when domain-specific errors need to be represented.

Example:

```java
public class InsufficientBalanceException
        extends RuntimeException {

    public InsufficientBalanceException(String message) {
        super(message);
    }
}
```

Usage:

```java
public void withdraw(double amount) {

    if (amount > balance) {

        throw new InsufficientBalanceException(
            "Insufficient balance"
        );
    }
}
```

This is particularly useful in backend applications where business rules need meaningful exceptions.

---

# 17. Checked Custom Exception

You can also create a checked exception by extending `Exception`.

```java
public class PaymentException extends Exception {

    public PaymentException(String message) {
        super(message);
    }
}
```

Then:

```java
public void processPayment()
        throws PaymentException {

    throw new PaymentException("Payment failed");
}
```

---

# 18. Exception Propagation

If an exception is not handled in the current method, it can propagate up the call stack.

Example:

```java
public void methodA() {

    methodB();
}

public void methodB() {

    methodC();
}

public void methodC() {

    int result = 10 / 0;
}
```

The exception starts in:

```text
methodC()
   ↓
methodB()
   ↓
methodA()
   ↓
caller
```

If no method handles it, the thread terminates with an exception.

### Interview Answer

> Exception propagation occurs when an exception moves up the call stack until it is handled or reaches the thread's uncaught exception handler.

---

# 19. Exception Stack Trace

A stack trace helps identify where an exception occurred and how execution reached that point.

Example:

```text
java.lang.ArithmeticException: / by zero
    at Calculator.divide(Calculator.java:10)
    at OrderService.calculate(OrderService.java:25)
    at OrderController.create(OrderController.java:40)
```

Read it from the top to understand the exception and the call path.

---

# 20. getMessage() vs printStackTrace()

### getMessage()

Returns the exception message.

```java
catch (Exception e) {

    System.out.println(e.getMessage());
}
```

### printStackTrace()

Prints the exception and stack trace.

```java
catch (Exception e) {

    e.printStackTrace();
}
```

In production applications, use a proper logging framework rather than relying on `printStackTrace()`.

---

# 21. Exception Chaining

Exception chaining means preserving the original cause when creating another exception.

Example:

```java
try {

    processPayment();

} catch (SQLException e) {

    throw new PaymentException(
        "Unable to process payment",
        e
    );
}
```

A custom exception can provide the original cause:

```java
public class PaymentException extends RuntimeException {

    public PaymentException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

This preserves the original exception for debugging.

### Interview Answer

> Exception chaining allows us to wrap a lower-level exception with a higher-level exception while preserving the original cause.

---

# 22. Exception Handling in Spring Boot

In REST APIs, handling every exception inside every controller is usually not ideal.

Instead, we can centralize exception handling using:

```text
@RestControllerAdvice
```

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleNotFound(
            ResourceNotFoundException ex) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(ex.getMessage());
    }
}
```

Now controllers can focus on business logic while common exception handling is centralized.

---

# 23. Example Backend Exception Flow

Consider an e-commerce API:

```text
Client
   |
   ↓
Controller
   |
   ↓
Service
   |
   ↓
Repository
   |
   ↓
Database
```

Suppose the requested product doesn't exist:

```text
Repository
   ↓
Product not found
   ↓
Service throws ResourceNotFoundException
   ↓
@RestControllerAdvice
   ↓
HTTP 404
```

Example response:

```json
{
    "status": 404,
    "message": "Product not found"
}
```

This produces a consistent API response instead of exposing a raw stack trace.

---

# 24. Good Exception Handling Practices

### 1. Don't catch Exception unnecessarily

Avoid:

```java
try {

    // code

} catch (Exception e) {

    System.out.println("Something went wrong");
}
```

This can hide important problems.

Prefer catching a meaningful exception:

```java
catch (SQLException e) {
    // handle database problem
}
```

---

### 2. Don't swallow exceptions

Bad:

```java
try {

    process();

} catch (Exception e) {

}
```

The exception disappears and debugging becomes difficult.

---

### 3. Preserve the original cause

Prefer:

```java
throw new PaymentException(
    "Payment processing failed",
    e
);
```

rather than:

```java
throw new PaymentException(
    "Payment processing failed"
);
```

when the original cause is important.

---

### 4. Use meaningful exception names

Good:

```text
UserNotFoundException
PaymentFailedException
InsufficientBalanceException
InvalidOrderStateException
```

Avoid vague names such as:

```text
SomethingWentWrongException
MyException
ErrorException
```

---

### 5. Don't use exceptions for normal control flow

Avoid:

```java
try {

    return map.get("user").getName();

} catch (NullPointerException e) {

    return "Unknown";
}
```

Prefer explicit validation or safe APIs.

---

### 6. Log useful context

Instead of logging only:

```text
Payment failed
```

include useful non-sensitive context:

```text
Payment processing failed for orderId=12345
```

Avoid logging passwords, tokens or other sensitive information.

---

# 25. Exception vs Error

| Exception | Error |
|---|---|
| Usually application-level problem | Usually serious JVM/system-level problem |
| Can often be handled | Usually not intended to be recovered from |
| Examples: IOException, SQLException | Examples: OutOfMemoryError, StackOverflowError |
| Application may recover | Recovery is often difficult or inappropriate |

---

# 26. final vs finally vs finalize

This is a classic interview question.

## final

`final` is a keyword.

Examples:

```java
final int MAX_USERS = 100;
```

A final variable cannot be reassigned.

A final method cannot be overridden.

A final class cannot be extended.

---

## finally

`finally` is a block used with exception handling.

```java
try {

    // code

} finally {

    // cleanup
}
```

---

## finalize()

`finalize()` was a legacy mechanism associated with garbage collection and has been deprecated for removal in modern Java.

Do not rely on it for resource cleanup.

Use:

- try-with-resources
- AutoCloseable
- explicit cleanup where appropriate

### Quick Answer

```text
final
    ↓
Keyword

finally
    ↓
Exception-handling block

finalize()
    ↓
Legacy cleanup mechanism; deprecated for removal
```

---

# 27. Can We Have try Without catch?

Yes, if there is a `finally` block.

```java
try {

    System.out.println("Executing");

} finally {

    System.out.println("Cleanup");
}
```

You cannot have a `try` block completely by itself.

It must be followed by:

- `catch`
- `finally`
- Or both

---

# 28. Can We Have Multiple finally Blocks?

No.

A single `try` statement can have at most one `finally` block.

---

# 29. Can We Have Multiple catch Blocks?

Yes.

```java
try {

    // code

} catch (IOException e) {

    // handle IO

} catch (SQLException e) {

    // handle database

}
```

---

# 30. Can a finally Block Have a return Statement?

Technically yes, but it is strongly discouraged.

Example:

```java
public int test() {

    try {
        return 10;

    } finally {
        return 20;
    }
}
```

The method returns:

```text
20
```

A `return` inside `finally` can override a return or exception from the `try` block and make debugging extremely confusing.

### Interview Answer

> Yes, but it should be avoided because a return from finally can suppress an exception or override a return value from the try/catch block.

---

# 31. Multiple Exceptions in One Catch

Modern Java allows multiple exception types in one catch block.

```java
try {

    // code

} catch (IOException | SQLException e) {

    System.out.println("Operation failed");
}
```

This is called a **multi-catch** statement.

---

# 32. Exception Handling Interview Questions

## Core Questions

1. What is an exception?
2. What is the difference between Error and Exception?
3. Checked vs unchecked exceptions?
4. What is RuntimeException?
5. What is try-catch?
6. What is finally?
7. Can finally be skipped?
8. What is try-with-resources?
9. What is AutoCloseable?
10. Difference between throw and throws?
11. Can we create custom exceptions?
12. What is exception propagation?
13. What is exception chaining?
14. What is a stack trace?
15. What is multi-catch?
16. Can we have try without catch?
17. Can we have multiple catch blocks?
18. Can finally have a return statement?
19. final vs finally vs finalize?
20. Why should we avoid catching generic Exception?
21. What happens if an exception is never handled?
22. How do you handle exceptions in Spring Boot?
23. What is `@ExceptionHandler`?
24. What is `@RestControllerAdvice`?
25. How would you design a consistent error response for REST APIs?

---

# 33. Spring Boot Interview Example

### Question

**How would you handle exceptions globally in Spring Boot?**

### Interview Answer

> I would use `@RestControllerAdvice` with `@ExceptionHandler` methods to centralize exception handling. This keeps controllers clean and allows us to return consistent HTTP status codes and error responses across the application.

---

# 34. REST API Error Response

A production API should return useful, structured error information.

Example:

```json
{
    "timestamp": "2026-08-20T10:30:00Z",
    "status": 404,
    "error": "Not Found",
    "message": "Product not found",
    "path": "/api/products/101"
}
```

Avoid returning:

```json
{
    "stackTrace": "...."
}
```

Stack traces should not normally be exposed to clients.

---

# 35. Quick Revision

```text
Exception
    ↓
Abnormal condition that disrupts execution

Checked Exception
    ↓
Compiler requires handling or declaration

Unchecked Exception
    ↓
RuntimeException hierarchy

try
    ↓
Code that may fail

catch
    ↓
Handle exception

finally
    ↓
Cleanup

throw
    ↓
Explicitly throw exception

throws
    ↓
Declare possible exceptions

try-with-resources
    ↓
Automatically close resources

Custom Exception
    ↓
Represent domain-specific problems

@RestControllerAdvice
    ↓
Global REST exception handling
```

---

# 36. Interview Mindset

Don't memorize only:

> "Checked exceptions are compile-time and unchecked exceptions are runtime."

Be ready to explain:

```text
What is the difference?
        ↓
Why does the distinction exist?
        ↓
When would you use each?
        ↓
How does exception propagation work?
        ↓
How do you preserve the original cause?
        ↓
How would you handle it in Spring Boot?
        ↓
What should the REST API return?
```

A strong backend developer should be able to connect Java exception handling to **real application behavior, logging, resource management and REST API design**.

# JVM, Memory and Garbage Collection in Java

JVM internals are common Java backend interview topics because they explain how Java applications use memory, execute code and manage objects.

For backend interviews, focus on understanding:

- JVM architecture
- Stack vs Heap
- Metaspace
- String Pool
- Class loading
- Garbage Collection
- Memory leaks
- OutOfMemoryError
- StackOverflowError
- GC tuning basics
- JIT compilation

---

# 1. What is the JVM?

JVM stands for:

```text
Java Virtual Machine
```

It executes Java bytecode.

Typical flow:

```text
Java Source Code
      ↓
javac
      ↓
Bytecode (.class)
      ↓
JVM
      ↓
Machine instructions
```

The JVM is responsible for:

- Loading classes
- Executing bytecode
- Managing memory
- Garbage collection
- Runtime security checks
- JIT compilation

---

# 2. JDK vs JRE vs JVM

### JVM

Runs Java bytecode.

### JRE

Historically:

```text
JRE = JVM + Java runtime libraries
```

Modern JDK distributions are generally what developers install; standalone JRE distributions are no longer the normal way Java is packaged for current releases.

### JDK

Provides the tools required to develop Java applications.

Conceptually:

```text
JDK
 ├── JVM
 ├── Java libraries
 └── Development tools
```

Important tools include:

```text
javac
java
javadoc
jar
jdb
```

---

# 3. Why Is Java Platform Independent?

Java source code is compiled into bytecode.

```text
Java Code
    ↓
Bytecode
    ↓
JVM for the target platform
    ↓
Machine Code
```

The bytecode can run on different operating systems as long as a compatible JVM exists.

This leads to:

```text
Write Once, Run Anywhere
```

---

# 4. JVM Architecture

A simplified JVM looks like:

```text
                 JVM
                  |
        ---------------------
        |                   |
   Class Loader        Runtime Data
                            |
        --------------------------------
        |       |        |             |
      Heap    Stack    Metaspace    PC Register
        |
   Garbage Collector

                  +
            Execution Engine
             /           \
        Interpreter       JIT
```

---

# 5. Class Loader

The Class Loader loads class definitions into the JVM.

Conceptually:

```text
.class file
    ↓
Class Loader
    ↓
Class metadata
    ↓
JVM
```

Class loading involves stages such as:

```text
Loading
   ↓
Linking
   ├── Verification
   ├── Preparation
   └── Resolution
   ↓
Initialization
```

---

# 6. Class Loading

When a class is needed, the JVM loads it.

Example:

```java
User user =
        new User();
```

The JVM needs the `User` class definition.

A class loader locates and loads the class.

---

# 7. Parent Delegation Model

Java class loaders generally follow parent delegation.

Conceptually:

```text
Application ClassLoader
          ↓
Platform ClassLoader
          ↓
Bootstrap ClassLoader
```

A class loader normally asks its parent to load a class before attempting to load it itself.

This helps prevent application code from replacing core Java classes.

---

# 8. Bootstrap Class Loader

The Bootstrap Class Loader loads core Java runtime classes.

Examples include classes from the core platform modules.

It is implemented as part of the JVM rather than as an ordinary Java class.

---

# 9. Platform Class Loader

The Platform Class Loader loads platform classes that are not loaded by the bootstrap loader.

It replaced the old terminology of the extension class loader in modern Java.

---

# 10. Application Class Loader

The Application Class Loader loads application classes and classes available through the application's class path/module path.

For a typical application:

```text
YourApplication.class
     ↓
Application Class Loader
```

---

# 11. Heap

The heap is the main memory area where objects are allocated.

Example:

```java
User user =
        new User();
```

The object is allocated in heap memory.

The heap is shared among threads.

```text
Thread 1 ──┐
Thread 2 ──┼──> Heap
Thread 3 ──┘
```

---

# 12. Stack

Each thread has its own JVM stack.

A stack contains stack frames for method invocations.

Example:

```java
public void process() {
    int count = 10;
    calculate(count);
}
```

Conceptually:

```text
Thread
  ↓
Stack
  ├── process() frame
  └── calculate() frame
```

Each frame contains information needed for the method execution, including local variables and operand stack state.

---

# 13. Stack vs Heap

| Stack | Heap |
|---|---|
| Per-thread | Shared |
| Stores stack frames | Stores objects |
| Method execution state | Object data |
| Usually short-lived frame data | Managed by GC |
| StackOverflowError possible | OutOfMemoryError possible |

---

# 14. Important Clarification About References

Consider:

```java
User user =
        new User();
```

A simplified explanation is:

```text
Reference variable → points to object
                     in heap
```

Where the reference itself is stored depends on the context.

For a local variable, the reference is part of the method's execution frame.

Do not oversimplify this as:

```text
all references are always on stack
```

because fields, static variables and other references have different storage contexts.

---

# 15. Method Area and Metaspace

The JVM needs memory for class-related metadata.

In modern HotSpot JVMs, class metadata is stored in:

```text
Metaspace
```

Metaspace replaced the old permanent generation (PermGen) in Java 8.

It can grow based on native memory availability, subject to configured limits.

---

# 16. PermGen vs Metaspace

Before Java 8:

```text
PermGen
```

Java 8 and later HotSpot:

```text
Metaspace
```

Main difference:

```text
PermGen → JVM-managed fixed-style region
Metaspace → Native memory for class metadata
```

Interview point:

> Metaspace replaced PermGen in Java 8.

---

# 17. String Pool

Java maintains a pool of interned strings.

Example:

```java
String a = "Java";
String b = "Java";
```

The JVM can reuse the same interned string object.

Therefore:

```java
a == b
```

can be:

```text
true
```

because both references can point to the same pooled String object.

---

# 18. new String()

Consider:

```java
String a = "Java";

String b =
        new String("Java");
```

`b` refers to a distinct String object created by the constructor.

Therefore:

```java
a == b
```

is:

```text
false
```

while:

```java
a.equals(b)
```

is:

```text
true
```

---

# 19. String Immutability

Strings are immutable.

Example:

```java
String name = "Java";

name = name + " Backend";
```

The original String object is not modified.

A new String value is produced.

This contributes to String safety and makes String objects suitable for pooling.

---

# 20. Garbage Collection

Garbage Collection, or GC, automatically reclaims memory occupied by objects that are no longer reachable.

Example:

```java
User user =
        new User();

user = null;
```

If no other reachable reference points to that object, it may become eligible for garbage collection.

Important:

> Eligible for GC does not mean immediately collected.

---

# 21. Reachability

GC fundamentally works around object reachability.

Conceptually:

```text
GC Roots
   ↓
Reachable objects
   ↓
Live

Not reachable from roots
   ↓
Eligible for reclamation
```

Common GC roots include:

- Active thread stacks
- Static references
- JNI/native references
- Other JVM-managed root references

---

# 22. Can We Force Garbage Collection?

You can request GC:

```java
System.gc();
```

or:

```java
Runtime.getRuntime()
       .gc();
```

But this is only a request.

The JVM is not required to perform a full collection immediately.

### Interview Answer

> `System.gc()` is only a request to the JVM. It does not guarantee that garbage collection will happen immediately.

---

# 23. Finalization

Modern Java has deprecated finalization and it should not be used for normal resource management.

Do not write application logic that depends on:

```java
finalize()
```

Use explicit resource management instead:

```java
try (Resource resource = ...) {
    // use resource
}
```

---

# 24. try-with-resources

For resources such as:

- Files
- Streams
- Database connections
- Statements
- Result sets

use:

```java
try (InputStream input =
        new FileInputStream("data.txt")) {

    // read

}
```

The resource is automatically closed.

This is deterministic resource management and is separate from garbage collection.

---

# 25. Garbage Collection Generations

Many JVM garbage collectors historically use generational concepts such as:

```text
Young generation
Old generation
```

The general idea is based on the observation that many objects die young.

A simplified model:

```text
New objects
    ↓
Young generation
    ↓
survive collections
    ↓
Older generation
```

The exact memory layout and algorithms depend on the collector and JDK version.

---

# 26. Minor/Young Collection

A young-generation collection primarily deals with young objects.

Because many young objects become unreachable quickly, these collections can often reclaim a significant amount of memory.

Do not assume every modern collector exposes exactly the same collection terminology or behavior.

---

# 27. Old Generation

Objects that remain alive for longer can eventually be treated as old-generation objects in collectors that use generational designs.

Long-lived objects may include:

```text
Caches
Application configuration
Large object graphs
Long-lived services
```

---

# 28. Stop-The-World

A Stop-The-World pause means application threads are paused while the JVM performs certain runtime work.

Some GC phases can require pauses.

Modern collectors aim to minimize pause duration, but no production application should assume GC is always completely pause-free.

---

# 29. Common Garbage Collectors

Modern Java provides several collectors.

Common examples include:

```text
Serial GC
Parallel GC
G1 GC
ZGC
Shenandoah
```

The appropriate choice depends on:

- Heap size
- Latency requirements
- Throughput requirements
- Allocation rate
- Application workload
- JDK version

---

# 30. G1 Garbage Collector

G1 stands for:

```text
Garbage-First
```

G1 divides the heap into regions and tries to prioritize regions with more reclaimable garbage while considering pause-time goals.

It is designed for large heaps and predictable pause targets.

G1 is commonly used and is the default collector in many modern JDK configurations.

---

# 31. ZGC

ZGC is designed for very low pause times and large heaps.

It performs much of its work concurrently with application execution.

The important interview point is:

> ZGC is designed for low-latency garbage collection with very short pauses, even on large heaps.

Do not claim that ZGC eliminates all pauses.

---

# 32. Shenandoah

Shenandoah is another low-pause garbage collector.

It performs substantial GC work concurrently with application threads.

Its goal is to reduce pause times, particularly for large heaps.

---

# 33. Serial GC

Serial GC uses a simpler approach and is generally suitable for smaller applications or environments where simplicity matters more than maximum throughput.

It uses a single GC thread for collection work.

---

# 34. Parallel GC

Parallel GC uses multiple threads for garbage collection and focuses primarily on throughput.

Conceptually:

```text
Application
     ↓
Parallel GC workers
     ↓
Higher collection throughput
```

---

# 35. OutOfMemoryError

A common JVM failure is:

```text
java.lang.OutOfMemoryError
```

It means the JVM could not satisfy a memory allocation request.

Possible causes include:

- Heap exhaustion
- Metaspace exhaustion
- Native memory exhaustion
- Excessive direct-buffer usage
- Memory leaks
- Huge allocations

The exact message tells you more about the memory area involved.

---

# 36. Java Heap Space

Example:

```text
java.lang.OutOfMemoryError:
Java heap space
```

This commonly indicates that the heap cannot satisfy an allocation.

Possible causes:

```text
Too many live objects
Large allocations
Memory leak
Heap too small
```

---

# 37. Metaspace OutOfMemoryError

Example:

```text
java.lang.OutOfMemoryError:
Metaspace
```

Possible causes include:

- Excessive class loading
- Classloader leaks
- Dynamically generated classes
- Incorrect classloader lifecycle
- Metaspace limit being too low

---

# 38. StackOverflowError

A `StackOverflowError` commonly occurs due to excessive recursion.

Example:

```java
void recurse() {
    recurse();
}
```

Calling:

```java
recurse();
```

can eventually exhaust the thread's stack.

Result:

```text
java.lang.StackOverflowError
```

---

# 39. StackOverflowError vs OutOfMemoryError

| Error | Typical cause |
|---|---|
| StackOverflowError | Excessive stack usage, often recursion |
| OutOfMemoryError | JVM cannot satisfy a memory allocation |

---

# 40. Memory Leak in Java

Java has GC, but Java applications can still have memory leaks.

A memory leak occurs when objects are no longer logically needed but remain reachable.

Example:

```java
static List<Object> cache =
        new ArrayList<>();
```

If objects are continuously added and never removed:

```text
Objects remain reachable
       ↓
GC cannot reclaim them
       ↓
Memory usage grows
```

---

# 41. Common Java Memory Leak Causes

Examples:

- Static collections
- Unbounded caches
- Listener registrations
- ThreadLocal values retained by pooled threads
- Classloader leaks
- Incorrect lifecycle management
- Large collections that never shrink
- Objects accidentally retained by long-lived references

---

# 42. Memory Leak vs Garbage Collection

GC only removes unreachable objects.

If an object is still reachable:

```text
GC Root
  ↓
Object
```

then GC considers it live.

So:

```text
"Java has GC"
```

does not mean:

```text
"Java cannot have memory leaks"
```

---

# 43. JIT Compiler

JIT stands for:

```text
Just-In-Time Compiler
```

The JVM initially interprets bytecode and can compile frequently executed code into optimized native machine code.

Conceptually:

```text
Bytecode
   ↓
Interpreter
   ↓
Hot code detected
   ↓
JIT compilation
   ↓
Optimized machine code
```

This improves runtime performance for frequently executed code.

---

# 44. Interpreter vs JIT

### Interpreter

Executes bytecode instructions.

### JIT

Compiles frequently executed code into native instructions and applies runtime optimizations.

Modern JVMs use both techniques as part of execution.

---

# 45. Hot Code

The JVM identifies code paths that execute frequently.

These are often called:

```text
Hot spots
```

The JIT can optimize them using runtime profiling information.

Possible optimizations include:

- Method inlining
- Dead-code elimination
- Loop optimizations
- Escape analysis
- Other runtime-specific optimizations

---

# 46. Escape Analysis

Escape analysis determines whether an object or reference escapes the scope where it was created.

The JVM can use this information for optimizations.

Conceptually:

```text
Object stays local
      ↓
Does not escape
      ↓
JVM may optimize allocation/locking
```

Do not claim that every non-escaping object is automatically allocated on the stack. JVM optimizations are implementation-dependent.

---

# 47. JVM Memory Options

Common JVM options include:

```text
-Xms
-Xmx
-Xss
```

### `-Xms`

Initial heap size.

Example:

```bash
-Xms512m
```

### `-Xmx`

Maximum heap size.

Example:

```bash
-Xmx2g
```

### `-Xss`

Thread stack size.

Example:

```bash
-Xss1m
```

Exact behavior and defaults depend on the JVM/platform.

---

# 48. Example JVM Command

```bash
java \
  -Xms512m \
  -Xmx2g \
  -jar application.jar
```

This configures:

```text
Initial heap → 512 MB
Maximum heap → 2 GB
```

---

# 49. Heap Dump

A heap dump captures information about objects in JVM heap memory.

It is useful for investigating:

- Memory leaks
- Unexpected object retention
- Large collections
- Excessive object counts
- OutOfMemoryError

Tools such as:

```text
jcmd
JConsole
VisualVM
Eclipse MAT
```

can be useful depending on the investigation.

---

# 50. Thread Dump

A thread dump captures information about JVM threads and their states.

Useful for investigating:

- Deadlocks
- Thread contention
- Blocked threads
- High CPU symptoms
- Stuck requests
- Thread pool problems

Common tools include:

```bash
jcmd <pid> Thread.print
```

and:

```bash
jstack <pid>
```

---

# 51. JConsole

JConsole is a JMX-based monitoring tool.

It can expose information such as:

- Heap usage
- Threads
- Classes
- CPU
- JVM memory pools

It is useful for learning and troubleshooting JVM behavior.

---

# 52. VisualVM

VisualVM can help inspect a running JVM.

Useful areas include:

- CPU
- Memory
- Threads
- Heap dumps
- Profiling
- GC activity

It is a useful practical tool for JVM troubleshooting.

---

# 53. Production Troubleshooting Example

Suppose a Spring Boot application keeps throwing:

```text
OutOfMemoryError: Java heap space
```

A reasonable investigation flow is:

```text
Check metrics
    ↓
Confirm memory growth
    ↓
Capture heap dump
    ↓
Analyze retained objects
    ↓
Identify retaining reference
    ↓
Find application bug
    ↓
Fix leak
    ↓
Load test
    ↓
Monitor again
```

Do not simply increase:

```text
-Xmx
```

and assume the problem is solved.

Increasing heap can hide the underlying leak temporarily.

---

# 54. Production GC Troubleshooting

Suppose latency suddenly increases.

Possible investigation:

```text
Application latency
        ↓
Check GC metrics
        ↓
GC frequency?
Pause duration?
Allocation rate?
Heap occupancy?
        ↓
Analyze thread/GC behavior
        ↓
Tune application or JVM
```

The correct solution could involve:

- Reducing unnecessary allocations
- Fixing a memory leak
- Adjusting heap sizing
- Reviewing collector configuration
- Reducing object churn
- Optimizing application behavior

---

# 55. Strong Interview Question

### "Explain JVM memory areas."

A good answer:

> The JVM uses several runtime memory areas. The heap is shared between threads and stores objects. Each thread has its own stack containing method frames and local execution state. Class metadata is stored in Metaspace in modern HotSpot JVMs. There are also areas such as the program counter and native method stack. Garbage collection primarily manages heap objects.

---

# 56. Strong Interview Question

### "Where are objects stored?"

Typical answer:

> Objects are generally allocated in the JVM heap, while references to them can exist in different contexts such as local variables, object fields and static fields. The exact physical memory optimization is JVM implementation-dependent.

This is more accurate than saying:

```text
Object → heap
Reference → always stack
```

---

# 57. Strong Interview Question

### "How does Garbage Collection work?"

A strong concise answer:

> Garbage collection identifies objects that are no longer reachable from GC roots and reclaims memory associated with them. Modern collectors use different algorithms and may divide the heap into regions or generations. Collectors such as G1, ZGC and Shenandoah are designed to balance throughput and latency according to different goals.

---

# 58. Strong Interview Question

### "Can Java have memory leaks?"

Yes.

A strong answer:

> Yes. Java automatically collects unreachable objects, but an application can still leak memory by unintentionally keeping objects reachable. Common examples include static collections, unbounded caches, ThreadLocal misuse and classloader leaks.

---

# 59. Strong Interview Question

### "What causes StackOverflowError?"

> Usually excessive stack usage, most commonly infinite or very deep recursion. Each method invocation requires stack space, and eventually the thread's stack can be exhausted.

---

# 60. Strong Interview Question

### "What is the difference between heap and stack?"

> The heap is shared among threads and is primarily used for dynamically allocated objects managed by the garbage collector. Each thread has its own stack, which stores method invocation frames and local execution state. Heap pressure and stack exhaustion therefore lead to different problems.

---

# 61. Strong Interview Question

### "What is Metaspace?"

> Metaspace is the native-memory area used by modern HotSpot JVMs for class metadata. It replaced PermGen in Java 8. Excessive class loading or classloader leaks can cause Metaspace-related OutOfMemoryError.

---

# 62. Strong Interview Question

### "What is JIT?"

> JIT stands for Just-In-Time compilation. The JVM identifies frequently executed code and compiles it into optimized native machine instructions using runtime profiling information, improving performance after the application has warmed up.

---

# 63. Strong Interview Question

### "Can System.gc() force GC?"

> No. `System.gc()` only requests garbage collection. The JVM decides whether and when to perform collection.

---

# 64. Strong Interview Question

### "What is the String Pool?"

> The String Pool is a JVM-managed pool of interned strings. String literals can be reused from the pool, which avoids unnecessary duplicate String objects and is one reason String literals can behave differently from separately constructed String objects.

---

# 65. Backend Connection — Spring Boot

A Java backend application can create large numbers of objects through:

```text
HTTP requests
    ↓
DTOs
    ↓
Entities
    ↓
Collections
    ↓
JSON serialization
    ↓
Database results
```

High allocation rates can increase GC activity.

Therefore backend performance is affected by:

```text
Object allocation
+
Heap sizing
+
GC behavior
+
Database usage
+
Caching
+
Request concurrency
```

---

# 66. Backend Connection — Hibernate

Hibernate can create substantial object graphs:

```text
Entity
 ↓
Relationships
 ↓
Collections
 ↓
Associated entities
```

Poorly designed queries can accidentally load huge object graphs.

Example:

```text
One request
    ↓
10,000 entities
    ↓
Large object graph
    ↓
High heap usage
    ↓
More GC pressure
```

This is why pagination, projections, fetch strategies and query optimization matter.

---

# 67. Backend Connection — Caching

Caching improves performance but can increase memory usage.

Example:

```java
Map<Long, Product> cache =
        new HashMap<>();
```

If the cache grows without bounds:

```text
Requests
   ↓
More cached objects
   ↓
Heap usage increases
   ↓
GC pressure
   ↓
Possible OutOfMemoryError
```

Use bounded cache strategies where appropriate.

---

# 68. JVM Troubleshooting Checklist

When a Java application has memory problems, investigate:

```text
1. Heap usage
2. GC frequency
3. GC pause duration
4. Allocation rate
5. Heap dump
6. Dominator/retained objects
7. Large collections
8. Cache size
9. ThreadLocal usage
10. Classloader behavior
11. Native/direct memory
12. JVM configuration
```

---

# 69. Quick Revision

```text
JVM
 ↓
Executes Java bytecode

Heap
 ↓
Objects
 ↓
Shared across threads

Stack
 ↓
Per-thread method frames

Metaspace
 ↓
Class metadata

GC
 ↓
Reclaims unreachable objects

Memory Leak
 ↓
Unneeded objects still reachable

StackOverflowError
 ↓
Stack exhaustion

OutOfMemoryError
 ↓
Memory allocation failure

JIT
 ↓
Optimizes hot code

-Xms
 ↓
Initial heap

-Xmx
 ↓
Maximum heap

-Xss
 ↓
Thread stack size
```

---

# 70. Interview Mindset

When troubleshooting JVM problems, don't immediately jump to:

```text
Increase heap
```

Instead ask:

```text
What memory is exhausted?
        ↓
Why is memory growing?
        ↓
Are objects still reachable?
        ↓
Is there excessive allocation?
        ↓
Is GC struggling?
        ↓
Is the application retaining objects?
        ↓
What does the heap/thread dump show?
        ↓
Fix root cause
```

The best backend engineers use JVM knowledge to connect application behavior with runtime behavior.

---

# 71. Practical Commands

### Check Java version

```bash
java -version
```

### Find running Java processes

```bash
jps -l
```

### Print thread information

```bash
jcmd <pid> Thread.print
```

### Generate a heap dump

```bash
jcmd <pid> GC.heap_dump /tmp/heap.hprof
```

### Show JVM flags

```bash
jcmd <pid> VM.flags
```

These commands are useful for real-world Java troubleshooting.

---

# 72. Final Interview Summary

Know these extremely well:

```text
JVM
JDK vs JVM
Class loading
Parent delegation
Heap
Stack
Metaspace
String Pool
Garbage Collection
GC Roots
Reachability
Generational GC
G1
ZGC
Shenandoah
Stop-The-World
Memory leaks
OutOfMemoryError
StackOverflowError
JIT
JVM flags
Heap dumps
Thread dumps
```

If asked a JVM question in a backend interview, explain it in this order:

```text
What it is
    ↓
Why it exists
    ↓
How it works
    ↓
Practical example
    ↓
Backend impact
    ↓
Troubleshooting/performance consideration
```

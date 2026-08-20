# Multithreading and Concurrency in Java

Multithreading is the ability to execute multiple threads within a process.

Concurrency is one of the most important Java backend interview topics because backend applications commonly handle:

- Multiple HTTP requests
- Database operations
- Background jobs
- Message processing
- File processing
- Scheduled tasks
- Parallel computations

The most important idea is:

```text
Thread
   ↓
Unit of execution

Concurrency
   ↓
Multiple tasks making progress

Synchronization
   ↓
Control access to shared mutable state
```

---

# 1. Process vs Thread

A **process** is an independent running program.

A **thread** is a unit of execution within a process.

A process can contain multiple threads.

```text
Process
 ├── Thread 1
 ├── Thread 2
 ├── Thread 3
 └── Thread 4
```

Threads within the same process generally share the process's memory/resources, while each thread has its own execution stack.

### Interview Answer

> A process is an independent execution environment, while a thread is a lightweight unit of execution inside a process. Threads within the same process can share memory and resources.

---

# 2. Why Use Multiple Threads?

Multithreading can improve:

- Responsiveness
- Throughput
- Resource utilization
- Parallel execution of independent tasks

For example, a backend application may process several independent requests concurrently.

```text
Request A → Thread 1
Request B → Thread 2
Request C → Thread 3
```

However, creating more threads does not automatically make an application faster.

Too many threads can cause:

- Context switching
- Memory overhead
- CPU contention
- Lock contention
- Poor performance

---

# 3. Concurrency vs Parallelism

These terms are related but different.

### Concurrency

Multiple tasks are in progress during overlapping periods.

```text
Task A ───────
       Task B ───────
```

### Parallelism

Multiple tasks execute at the same time on different CPU cores.

```text
Core 1 → Task A
Core 2 → Task B
```

### Interview Answer

> Concurrency is about managing multiple tasks that overlap in progress, while parallelism means tasks are actually executing simultaneously, typically on multiple CPU cores.

---

# 4. Creating a Thread

Java provides the `Thread` class.

```java
Thread thread = new Thread(() -> {
    System.out.println("Running");
});

thread.start();
```

Important:

```java
thread.start();
```

starts a new thread.

Calling:

```java
thread.run();
```

directly does **not** start a new thread. It simply invokes the method on the current thread.

---

# 5. `start()` vs `run()`

This is a common interview question.

### `start()`

```java
thread.start();
```

Requests that the JVM schedule a new thread of execution.

### `run()`

```java
thread.run();
```

Executes the method normally on the calling thread.

### Interview Answer

> `start()` initiates a new thread of execution, whereas calling `run()` directly is just a normal method call on the current thread.

---

# 6. Runnable

A common way to define a task is:

```java
Runnable task = () -> {
    System.out.println("Task running");
};
```

Then:

```java
Thread thread = new Thread(task);

thread.start();
```

`Runnable` represents work that does not return a result.

---

# 7. Callable

`Callable` is useful when a task needs to return a result or throw checked exceptions.

```java
Callable<Integer> task = () -> {
    return 100;
};
```

It is commonly used with:

```text
ExecutorService
Future
```

Example:

```java
ExecutorService executor =
        Executors.newSingleThreadExecutor();

Future<Integer> future =
        executor.submit(task);

Integer result = future.get();

executor.shutdown();
```

---

# 8. Runnable vs Callable

| Runnable | Callable |
|---|---|
| `run()` | `call()` |
| Does not return a result | Returns a result |
| Cannot directly throw checked exceptions from `run()` | Can throw checked exceptions |
| Common for fire-and-forget tasks | Common for tasks producing results |

### Interview Answer

> I use Runnable when a task doesn't need to return a result, and Callable when I need a result or checked-exception support.

---

# 9. Thread Lifecycle

A Java thread can move through states represented by:

```java
Thread.State
```

Important states include:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

---

# 10. NEW

A thread is in `NEW` after creation but before `start()`.

```java
Thread thread =
        new Thread(task);
```

At this point:

```text
state = NEW
```

---

# 11. RUNNABLE

After:

```java
thread.start();
```

the thread becomes eligible to run.

Java's `RUNNABLE` state represents a thread that is runnable, including one that may currently be executing.

---

# 12. BLOCKED

A thread enters `BLOCKED` when it is waiting to acquire an intrinsic monitor lock.

Example:

```java
synchronized (lock) {
    // critical section
}
```

If another thread owns the same monitor, a waiting thread can become `BLOCKED`.

---

# 13. WAITING

A thread can enter `WAITING` when it waits indefinitely for another thread's action.

Examples include:

```java
Object.wait();
Thread.join();
```

without a timeout.

---

# 14. TIMED_WAITING

A thread can enter `TIMED_WAITING` when it waits for a bounded amount of time.

Examples:

```java
Thread.sleep(1000);
```

or:

```java
thread.join(1000);
```

---

# 15. TERMINATED

After the thread finishes execution:

```text
TERMINATED
```

It cannot be started again.

```java
thread.start();
thread.start();
```

The second attempt results in:

```text
IllegalThreadStateException
```

---

# 16. `sleep()`

Example:

```java
Thread.sleep(1000);
```

This pauses the current thread for approximately the specified duration.

Important:

> `sleep()` does not release intrinsic locks held by the thread.

This is a common interview trap.

---

# 17. `join()`

`join()` allows one thread to wait for another thread to finish.

Example:

```java
Thread worker = new Thread(task);

worker.start();

worker.join();
```

The calling thread waits until `worker` terminates.

---

# 18. Race Condition

A race condition occurs when the result depends on the timing/interleaving of concurrent operations.

Example:

```java
class Counter {

    int count = 0;

    void increment() {
        count++;
    }
}
```

This is not atomic.

Conceptually:

```text
read count
   ↓
add 1
   ↓
write count
```

Two threads can interleave these operations and lose an update.

---

# 19. Lost Update

Suppose:

```text
count = 0
```

Two threads execute:

```text
count++
```

Possible interleaving:

```text
Thread A reads 0
Thread B reads 0

Thread A writes 1
Thread B writes 1
```

Expected:

```text
2
```

Actual:

```text
1
```

This is a classic race condition.

---

# 20. Critical Section

A critical section is code that accesses shared state and must be protected from unsafe concurrent access.

Example:

```java
synchronized
```

can be used to protect a critical section.

---

# 21. `synchronized`

Java provides intrinsic locking through `synchronized`.

Example:

```java
public synchronized void increment() {
    count++;
}
```

Only one thread at a time can execute the synchronized instance method for a particular object monitor.

---

# 22. Synchronized Block

Instead of synchronizing the entire method:

```java
public void increment() {

    synchronized (this) {
        count++;
    }
}
```

This can make the protected region smaller.

A better design can sometimes use a dedicated lock object:

```java
private final Object lock =
        new Object();

public void increment() {

    synchronized (lock) {
        count++;
    }
}
```

---

# 23. Instance Synchronized Method

```java
public synchronized void save() {
    // critical section
}
```

For an instance method, the lock is associated with the current object's monitor.

Conceptually:

```java
synchronized (this) {
    // method body
}
```

---

# 24. Static Synchronized Method

```java
public static synchronized void save() {
}
```

The monitor is associated with the `Class` object for that class.

Conceptually:

```java
synchronized (MyClass.class) {
}
```

---

# 25. Synchronized Method vs Block

Method:

```java
public synchronized void process() {
    // entire method protected
}
```

Block:

```java
public void process() {

    // non-critical work

    synchronized (lock) {
        // critical work
    }
}
```

A synchronized block gives more precise control over:

- Lock object
- Critical section size
- Contention

---

# 26. Mutual Exclusion

Synchronization provides mutual exclusion for a given monitor.

```text
Thread A
   ↓
acquires lock
   ↓
critical section
   ↓
releases lock

Thread B
   ↓
waits
```

Only one thread can own that monitor at a time.

---

# 27. Reentrant Locking

Java intrinsic locks are reentrant.

If a thread already owns a monitor, it can acquire the same monitor again.

Example:

```java
public synchronized void methodA() {
    methodB();
}

public synchronized void methodB() {
}
```

The same thread can enter both methods without deadlocking on its own monitor.

---

# 28. Volatile

The `volatile` keyword provides visibility guarantees for a variable across threads.

Example:

```java
private volatile boolean running =
        true;
```

If one thread changes:

```java
running = false;
```

other threads reading `running` can observe the updated value according to the Java Memory Model's volatile rules.

---

# 29. What volatile Does Not Do

`volatile` does **not** make compound operations atomic.

This is still unsafe:

```java
volatile int count;

count++;
```

Because:

```text
read
+
write
```

is a compound operation.

For atomic increments, use:

```java
AtomicInteger
```

or suitable synchronization.

---

# 30. volatile vs synchronized

| volatile | synchronized |
|---|---|
| Provides visibility/order guarantees | Provides mutual exclusion + visibility |
| Does not provide general compound-operation atomicity | Protects critical sections |
| No monitor lock required | Uses intrinsic locking |
| Useful for state flags | Useful for shared mutable state |

### Interview Answer

> I use volatile when I need visibility of a variable across threads and don't need compound atomic operations. I use synchronized when multiple operations must be performed safely as one critical section.

---

# 31. Atomic Classes

Java provides atomic classes in:

```java
java.util.concurrent.atomic
```

Examples:

```text
AtomicInteger
AtomicLong
AtomicBoolean
AtomicReference
```

Example:

```java
AtomicInteger counter =
        new AtomicInteger();

counter.incrementAndGet();
```

This provides an atomic increment operation.

---

# 32. AtomicInteger

Example:

```java
AtomicInteger counter =
        new AtomicInteger(0);

counter.incrementAndGet();

System.out.println(
    counter.get()
);
```

Useful for simple atomic state updates without explicitly using a synchronized block.

---

# 33. Compare-And-Set

Atomic classes commonly use CAS:

```text
Compare And Set
```

Conceptually:

```text
current value == expected value?
        ↓
yes → update
no  → retry/fail
```

This supports lock-free atomic operations for suitable use cases.

---

# 34. ExecutorService

Creating raw threads for every task is usually not the best approach in backend applications.

Prefer:

```java
ExecutorService
```

Example:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(4);

executor.submit(() -> {
    System.out.println("Processing");
});

executor.shutdown();
```

The executor manages a pool of worker threads.

---

# 35. Why Use Thread Pools?

Thread pools can reduce overhead associated with repeatedly creating threads.

They provide:

- Thread reuse
- Task queuing
- Concurrency control
- Lifecycle management

Conceptually:

```text
Tasks
 ↓
Queue
 ↓
Thread Pool
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 └── Worker 4
```

---

# 36. Fixed Thread Pool

```java
ExecutorService executor =
        Executors.newFixedThreadPool(4);
```

The pool has a fixed number of worker threads.

Useful when you want bounded concurrency.

---

# 37. Single Thread Executor

```java
ExecutorService executor =
        Executors.newSingleThreadExecutor();
```

Tasks execute sequentially through one worker thread.

Useful when you want asynchronous task execution but don't need parallel workers.

---

# 38. Scheduled Executor

For scheduled tasks:

```java
ScheduledExecutorService scheduler =
        Executors.newScheduledThreadPool(2);
```

Example:

```java
scheduler.schedule(
    () -> System.out.println("Executed"),
    5,
    TimeUnit.SECONDS
);
```

---

# 39. Future

A `Future` represents the result of an asynchronous computation.

Example:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(2);

Future<Integer> future =
        executor.submit(() -> 42);

Integer result =
        future.get();

executor.shutdown();
```

Important:

```java
future.get();
```

can block until the result is available.

---

# 40. Future Methods

Common methods:

```java
get()
isDone()
isCancelled()
cancel()
```

Example:

```java
if (future.isDone()) {
    Integer result = future.get();
}
```

---

# 41. CompletableFuture

`CompletableFuture` supports asynchronous pipelines and composition.

Example:

```java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(
            () -> "Hello"
        );
```

Then:

```java
future.thenApply(
    value -> value + " Java"
);
```

---

# 42. CompletableFuture Pipeline

Example:

```java
CompletableFuture
    .supplyAsync(() -> fetchUser())
    .thenApply(user -> user.getName())
    .thenAccept(name ->
        System.out.println(name)
    );
```

Conceptually:

```text
fetch user
    ↓
extract name
    ↓
consume result
```

This is useful for composing asynchronous operations.

---

# 43. `thenApply()` vs `thenAccept()`

### thenApply()

Transforms a result.

```java
future.thenApply(
    value -> value.toUpperCase()
);
```

Returns another `CompletableFuture`.

### thenAccept()

Consumes a result without returning a transformed value.

```java
future.thenAccept(
    value -> System.out.println(value)
);
```

Returns:

```java
CompletableFuture<Void>
```

---

# 44. `thenCompose()`

Used to chain asynchronous operations where the next operation itself returns a `CompletableFuture`.

Example:

```java
getUser()
    .thenCompose(user ->
        getOrders(user.getId())
    );
```

Conceptually:

```text
Future<User>
     ↓
Future<List<Order>>
```

without creating a nested:

```text
Future<Future<List<Order>>>
```

---

# 45. `thenCombine()`

Used to combine two independent asynchronous results.

Example:

```java
CompletableFuture<User> user =
        getUser();

CompletableFuture<Account> account =
        getAccount();

CompletableFuture<String> result =
        user.thenCombine(
            account,
            (u, a) ->
                u.getName() + ":" + a.getId()
        );
```

---

# 46. `allOf()`

Waits for multiple futures.

```java
CompletableFuture<Void> all =
        CompletableFuture.allOf(
            future1,
            future2,
            future3
        );
```

Useful when multiple independent tasks need to complete before continuing.

---

# 47. Exception Handling with CompletableFuture

Example:

```java
future
    .thenApply(value -> process(value))
    .exceptionally(error -> {
        System.out.println(
            error.getMessage()
        );
        return "fallback";
    });
```

Other useful methods include:

```java
handle()
whenComplete()
exceptionally()
```

---

# 48. Concurrent Collections

Java provides collections designed for concurrent access.

Important examples:

```text
ConcurrentHashMap
CopyOnWriteArrayList
BlockingQueue
ConcurrentLinkedQueue
```

---

# 49. ConcurrentHashMap

Instead of:

```java
HashMap
```

for concurrent access, consider:

```java
ConcurrentHashMap
```

Example:

```java
ConcurrentHashMap<String, Integer>
        counts =
        new ConcurrentHashMap<>();
```

It is designed for concurrent access without requiring you to synchronize every operation externally.

---

# 50. HashMap vs ConcurrentHashMap

| HashMap | ConcurrentHashMap |
|---|---|
| Not thread-safe for concurrent mutation | Designed for concurrent access |
| External synchronization may be needed | Provides concurrency-safe operations |
| Allows one null key and null values | Does not allow null keys or null values |
| Suitable for non-concurrent use | Suitable for concurrent access patterns |

Do not simply replace every HashMap with ConcurrentHashMap. Choose based on the application's concurrency requirements.

---

# 51. CopyOnWriteArrayList

Useful when:

```text
Reads >> Writes
```

Example:

```java
CopyOnWriteArrayList<String> list =
        new CopyOnWriteArrayList<>();
```

When the list is modified, a new underlying array is created.

This makes reads safe and efficient for read-heavy scenarios but writes can be expensive.

---

# 52. BlockingQueue

A `BlockingQueue` is useful for producer-consumer systems.

Example:

```text
Producer
   ↓
BlockingQueue
   ↓
Consumer
```

Common implementations include:

```text
ArrayBlockingQueue
LinkedBlockingQueue
PriorityBlockingQueue
```

---

# 53. Producer-Consumer Example

Conceptually:

```java
BlockingQueue<String> queue =
        new LinkedBlockingQueue<>();

queue.put("Order-101");

String order =
        queue.take();
```

`put()` and `take()` can block depending on queue state.

This pattern is common in background processing systems.

---

# 54. Deadlock

A deadlock occurs when threads wait indefinitely for resources held by each other.

Example:

```text
Thread A
  locks A
  waits for B

Thread B
  locks B
  waits for A
```

Neither can continue.

---

# 55. Deadlock Example

```java
synchronized (lockA) {

    synchronized (lockB) {
        // work
    }
}
```

Another thread does:

```java
synchronized (lockB) {

    synchronized (lockA) {
        // work
    }
}
```

Possible result:

```text
Thread 1 → holds A → waits for B
Thread 2 → holds B → waits for A
```

Deadlock.

---

# 56. Preventing Deadlocks

Common strategies:

- Consistent lock ordering
- Avoid unnecessary nested locks
- Keep critical sections small
- Use timed lock acquisition with `Lock`
- Avoid calling unknown/external code while holding locks

### Strong Interview Answer

> A common way to prevent deadlocks is to acquire multiple locks in a consistent global order across all code paths.

---

# 57. ReentrantLock

Java provides:

```java
ReentrantLock
```

Example:

```java
Lock lock =
        new ReentrantLock();

lock.lock();

try {
    // critical section
} finally {
    lock.unlock();
}
```

The `finally` block is essential to ensure the lock is released.

---

# 58. `tryLock()`

`ReentrantLock` can attempt to acquire a lock without waiting indefinitely.

```java
if (lock.tryLock()) {

    try {
        // work
    } finally {
        lock.unlock();
    }
}
```

A timed version is also available.

This can help avoid certain indefinite waiting scenarios.

---

# 59. synchronized vs ReentrantLock

| synchronized | ReentrantLock |
|---|---|
| Simpler syntax | More explicit control |
| Lock automatically released when leaving synchronized block | Must unlock explicitly |
| No timed acquisition | Supports `tryLock()` and timed locking |
| Intrinsic monitor | Explicit lock object |
| Good default for simple critical sections | Useful when advanced lock features are needed |

### Interview Answer

> I prefer synchronized for straightforward mutual exclusion because it is simple and harder to misuse. I consider ReentrantLock when I need features such as timed acquisition, `tryLock()` or more explicit lock management.

---

# 60. ReadWriteLock

`ReadWriteLock` separates read and write access.

Conceptually:

```text
Many readers
     ↓
Read lock

One writer
     ↓
Write lock
```

Example:

```java
ReadWriteLock lock =
        new ReentrantReadWriteLock();

lock.readLock().lock();

try {
    // read
} finally {
    lock.readLock().unlock();
}
```

Useful when reads significantly outnumber writes and the workload benefits from concurrent readers.

---

# 61. Semaphore

A `Semaphore` controls the number of threads allowed to access a resource concurrently.

Example:

```java
Semaphore semaphore =
        new Semaphore(3);
```

At most three permits can be acquired simultaneously.

This can be useful for limiting access to scarce resources.

---

# 62. CountDownLatch

A `CountDownLatch` allows one or more threads to wait until a count reaches zero.

Example:

```java
CountDownLatch latch =
        new CountDownLatch(3);
```

Workers call:

```java
latch.countDown();
```

Another thread waits:

```java
latch.await();
```

Useful for one-time coordination.

---

# 63. CyclicBarrier

A `CyclicBarrier` allows a group of threads to wait until all participating threads reach a synchronization point.

Conceptually:

```text
Thread A ──┐
Thread B ──┼── barrier ── continue
Thread C ──┘
```

Unlike CountDownLatch, a CyclicBarrier can be reused.

---

# 64. Atomicity

Atomicity means an operation appears indivisible from the perspective of other threads.

For example:

```java
counter.incrementAndGet();
```

is an atomic operation for `AtomicInteger`.

But:

```java
counter = counter + 1;
```

is not generally atomic when `counter` is a shared plain integer.

---

# 65. Visibility

Visibility means that updates made by one thread become observable by another thread according to the Java Memory Model's synchronization rules.

Tools that establish visibility include:

- `volatile`
- `synchronized`
- Lock operations
- Thread start/join
- Concurrent utilities

---

# 66. Java Memory Model

The Java Memory Model defines rules for:

- Visibility
- Ordering
- Atomicity
- Inter-thread communication

A critical concept is:

```text
happens-before
```

If one action happens-before another, the effects of the first are guaranteed to be visible to the second under the JMM rules.

---

# 67. Happens-Before Examples

Important happens-before relationships include:

### Unlock → subsequent lock

An unlock on a monitor happens-before a subsequent lock on the same monitor.

### Volatile write → subsequent volatile read

A write to a volatile variable happens-before a subsequent read of that variable.

### Thread start

Actions before:

```java
thread.start();
```

happen-before actions in the started thread.

### Thread join

Actions in a thread happen-before another thread successfully returns from:

```java
thread.join();
```

These rules are useful when reasoning about visibility.

---

# 68. Thread Safety

A class is thread-safe when it behaves correctly when accessed concurrently according to its contract.

Thread safety can be achieved using:

- Immutability
- Synchronization
- Locks
- Atomic variables
- Concurrent collections
- Thread confinement
- Message passing

---

# 69. Immutability and Thread Safety

Immutable objects are naturally easier to share safely.

Example:

```java
public final class Config {

    private final String host;
    private final int port;

    public Config(String host, int port) {
        this.host = host;
        this.port = port;
    }
}
```

Once constructed, the object's state does not change.

This reduces synchronization requirements.

---

# 70. ThreadLocal

`ThreadLocal` provides each thread with its own independently initialized value.

Example:

```java
ThreadLocal<String> userContext =
        new ThreadLocal<>();
```

Then:

```java
userContext.set("Sudhir");
```

Each thread gets its own value.

Use it carefully in thread pools because pooled threads are reused.

Always remove values when appropriate:

```java
try {
    userContext.set("Sudhir");

    // work

} finally {
    userContext.remove();
}
```

---

# 71. Thread Confinement

Thread confinement means state is accessed by only one thread.

For example, a local variable:

```java
void process() {

    List<String> values =
            new ArrayList<>();

    // only this thread uses it
}
```

No synchronization is needed for that local mutable list if it never escapes to other threads.

---

# 72. Backend Example — Request Processing

A backend server may receive:

```text
Request 1
Request 2
Request 3
Request 4
```

The server can process them concurrently using worker threads.

```text
HTTP Requests
      ↓
Thread Pool
 ┌────┼────┬────┐
 T1   T2   T3   T4
```

This is why understanding thread pools and concurrency is important for Spring Boot developers.

---

# 73. Spring Boot Connection

Spring-managed singleton beans are typically shared across requests.

Example:

```java
@Service
public class OrderService {

    private int counter = 0;

    public void process() {
        counter++;
    }
}
```

If multiple requests access this singleton concurrently, mutable instance state can create race conditions.

Prefer stateless services:

```java
@Service
public class OrderService {

    public Order process(OrderRequest request) {
        // local variables
        // repository calls
        // business logic
    }
}
```

### Interview Answer

> Spring singleton beans are shared across requests, so I avoid mutable shared instance state in service classes. I prefer stateless services and use proper concurrency controls when shared state is unavoidable.

---

# 74. Common Concurrency Mistake

Do not assume:

```java
private int counter;
```

is safe just because it is inside a Spring service.

If the service is shared and multiple requests modify the field concurrently, it can have race conditions.

---

# 75. ExecutorService Shutdown

After submitting tasks, properly shut down an executor.

```java
executor.shutdown();
```

For more forceful shutdown:

```java
executor.shutdownNow();
```

`shutdownNow()` attempts to interrupt running tasks and returns tasks that were awaiting execution.

It does not guarantee that every running task stops immediately.

---

# 76. Graceful Shutdown

A common pattern:

```java
executor.shutdown();

try {

    if (!executor.awaitTermination(
            10,
            TimeUnit.SECONDS)) {

        executor.shutdownNow();
    }

} catch (InterruptedException e) {

    executor.shutdownNow();

    Thread.currentThread().interrupt();
}
```

Important:

> When catching InterruptedException, restore the interrupt status when you cannot fully handle the interruption.

---

# 77. Interrupts

A thread can be interrupted using:

```java
thread.interrupt();
```

An interrupt is a cooperative cancellation mechanism.

It does not forcibly kill the thread.

Code should respond appropriately to interruption.

Example:

```java
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

---

# 78. Common Interview Question — Can We Stop a Thread?

Do not use the deprecated:

```java
Thread.stop()
```

for normal application design.

Prefer cooperative cancellation using:

```text
interrupt()
```

and/or cancellation mechanisms provided by executors and futures.

---

# 79. Common Interview Question — Why Is count++ Not Atomic?

Because it is conceptually multiple operations:

```text
read
modify
write
```

Two threads can interleave those operations.

Use:

```java
AtomicInteger
```

or synchronization when appropriate.

---

# 80. Common Interview Question — Is volatile Atomic?

Not generally.

`volatile` provides visibility and ordering guarantees, but it does not make compound operations such as:

```java
count++;
```

atomic.

---

# 81. Common Interview Question — Does synchronized Guarantee Visibility?

Yes.

Entering and exiting synchronized blocks establishes the necessary happens-before relationships for visibility around the same monitor.

---

# 82. Common Interview Question — Does sleep() Release the Lock?

No.

If a thread holds an intrinsic lock and calls:

```java
Thread.sleep(...)
```

it keeps that lock while sleeping.

---

# 83. Common Interview Question — wait() vs sleep()

| wait() | sleep() |
|---|---|
| Method of Object | Static method of Thread |
| Used for thread coordination | Used to pause execution |
| Must be called while owning the object's monitor | Does not require owning a monitor |
| Releases the object's monitor while waiting | Does not release locks |
| Can be notified | Wakes after timeout or interruption |

---

# 84. Common Interview Question — notify() vs notifyAll()

`notify()` wakes one waiting thread.

```java
lock.notify();
```

`notifyAll()` wakes all threads waiting on that monitor.

```java
lock.notifyAll();
```

The scheduler determines which awakened thread proceeds.

In modern application design, higher-level concurrency utilities are often preferable to manually coordinating threads with `wait()`/`notify()`.

---

# 85. Common Interview Question — synchronized vs AtomicInteger

Use `AtomicInteger` when you need simple atomic operations such as:

```java
incrementAndGet()
compareAndSet()
getAndIncrement()
```

Use synchronization when multiple related operations must be protected as one critical section.

Example:

```java
synchronized (lock) {

    if (balance >= amount) {
        balance -= amount;
        recordTransaction();
    }
}
```

The whole sequence needs to be consistent.

---

# 86. Common Interview Question — How Do You Prevent Race Conditions?

Possible approaches:

1. Make the state immutable.
2. Avoid shared mutable state.
3. Use synchronized blocks.
4. Use locks.
5. Use atomic variables.
6. Use concurrent collections.
7. Use thread confinement.
8. Use message-passing/queue-based designs.

The best solution depends on the workload and consistency requirements.

---

# 87. Common Interview Question — What Is Deadlock?

A strong answer:

> Deadlock occurs when two or more threads wait indefinitely for resources held by each other. A common prevention technique is consistent lock ordering, where every code path acquires multiple locks in the same order.

---

# 88. Common Interview Question — What Is Starvation?

Starvation occurs when a thread repeatedly fails to get enough CPU time or access to a required resource because other threads keep taking priority or the resource.

Possible causes:

- Unfair locking
- Excessive contention
- High-priority threads
- Poor scheduling/design

---

# 89. Common Interview Question — What Is Livelock?

In livelock, threads remain active but make no useful progress.

Example:

```text
Thread A keeps backing off for B
Thread B keeps backing off for A
```

Unlike deadlock:

```text
Deadlock → threads are blocked
Livelock  → threads are active but not progressing
```

---

# 90. Common Interview Question — What Is Context Switching?

Context switching occurs when the CPU switches execution from one thread/task to another.

Too many threads can cause excessive context switching and reduce performance.

This is one reason thread-pool sizing matters.

---

# 91. Common Interview Question — How Many Threads Should a Pool Have?

There is no universal number.

It depends on:

- CPU cores
- CPU-bound vs I/O-bound work
- Task duration
- Blocking behavior
- External service latency
- Database connection pool size
- Memory limits
- Throughput requirements

### Interview Answer

> I would not choose a thread-pool size blindly. I consider whether the workload is CPU-bound or I/O-bound, the available CPU, blocking behavior and downstream limits such as the database connection pool, then validate the choice using load testing and metrics.

---

# 92. Important Backend Insight

Thread pool size and database connection pool size should be considered together.

For example:

```text
100 worker threads
        ↓
10 database connections
```

If all 100 threads frequently wait for database connections, increasing the thread count may only increase contention.

This is why backend concurrency must be designed across the entire system.

---

# 93. Virtual Threads

Modern Java also provides virtual threads.

Example:

```java
Thread.startVirtualThread(() -> {
    processRequest();
});
```

Virtual threads are lightweight threads managed by the Java runtime rather than mapping one-to-one with operating-system threads in the traditional way.

They are particularly useful for applications with many blocking I/O operations.

They do not make CPU-bound work magically faster.

---

# 94. Virtual Threads vs Platform Threads

### Platform threads

Traditional Java threads are backed by operating-system threads.

### Virtual threads

Lightweight Java threads designed to support very high numbers of concurrent tasks, especially blocking I/O workloads.

### Interview Answer

> Virtual threads are lightweight threads intended to make high-concurrency applications easier to build, especially when tasks spend significant time blocked on I/O. They don't increase CPU capacity, so CPU-bound workloads still depend on available processor resources.

---

# 95. Structured Concurrency

Modern Java also includes structured concurrency APIs in newer Java releases as a preview/incubating area depending on the JDK version.

The idea is to treat related concurrent tasks as a structured unit:

```text
Parent task
 ├── Child task A
 ├── Child task B
 └── Child task C
```

This can simplify:

- Cancellation
- Error handling
- Lifecycle management
- Observability

For interviews, know the concept and verify the exact API status for the Java version being used.

---

# 96. Common Concurrency Utilities

Know these:

```text
ExecutorService
ScheduledExecutorService
Future
CompletableFuture
ConcurrentHashMap
BlockingQueue
AtomicInteger
AtomicLong
AtomicReference
CountDownLatch
CyclicBarrier
Semaphore
ReentrantLock
ReadWriteLock
ThreadLocal
```

---

# 97. Quick Revision

```text
Thread
    ↓
Unit of execution

Concurrency
    ↓
Multiple tasks making progress

Parallelism
    ↓
Tasks executing simultaneously

Race Condition
    ↓
Unsafe shared-state interleaving

synchronized
    ↓
Mutual exclusion + visibility

volatile
    ↓
Visibility/order, not general atomicity

AtomicInteger
    ↓
Atomic operations

ExecutorService
    ↓
Thread pool management

Future
    ↓
Async result

CompletableFuture
    ↓
Composable async pipelines

ConcurrentHashMap
    ↓
Concurrent map access

BlockingQueue
    ↓
Producer-consumer coordination

Deadlock
    ↓
Threads waiting forever

ReentrantLock
    ↓
Explicit lock control

Virtual Threads
    ↓
Lightweight high-concurrency execution
```

---

# 98. Strong Interview Answer

### Question: "How do you handle concurrency in a Java backend application?"

> First, I try to minimize shared mutable state and keep services stateless. For concurrent tasks, I prefer managed executors or framework-managed thread pools instead of creating raw threads for every request. For shared state, I choose the simplest correct mechanism, such as immutable objects, atomic variables, synchronized blocks, locks or concurrent collections. For asynchronous workflows, CompletableFuture can help compose independent operations. I also consider downstream limits such as database connection pools and monitor the application under load.

---

# 99. Backend Design Example

Suppose an e-commerce backend needs to process an order:

```text
Order Request
      ↓
Validate order
      ↓
Check inventory
      ↓
Process payment
      ↓
Create order
      ↓
Send notification
```

Some operations may be independent.

For example:

```text
Create order
      ↓
      ├── Send email
      └── Publish notification
```

The notification work could potentially be asynchronous.

But don't blindly make everything asynchronous.

Ask:

```text
Does the caller need the result?
Does ordering matter?
Can failure be retried?
Is the operation idempotent?
What happens during shutdown?
How many tasks can run concurrently?
What downstream service limits exist?
```

Concurrency is a design decision, not simply a performance switch.

---

# 100. Interview Mindset

A strong Java backend candidate should be able to reason through:

```text
Shared state
    ↓
Could multiple threads access it?
    ↓
Yes
    ↓
Can we remove the shared state?
    ↓
Yes → Prefer stateless/immutable design
    ↓
No
    ↓
What operation must be atomic?
    ↓
Simple counter → AtomicInteger
Complex critical section → synchronized/Lock
Concurrent map → ConcurrentHashMap
Producer/consumer → BlockingQueue
Async composition → CompletableFuture
Large I/O concurrency → Consider virtual threads
```

The goal is not to memorize every concurrency class.

The goal is to choose the **simplest correct concurrency mechanism for the problem**.

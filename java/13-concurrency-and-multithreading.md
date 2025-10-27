# ⚙️ Java 21 — Concurrency and Multithreading Masterclass

Concurrency in Java has evolved from **manual thread management** to **virtual-thread-based structured concurrency** — making it easier than ever to build scalable, non-blocking, and reactive systems.

This guide provides a **comprehensive, modern, and senior-level understanding** of concurrency in **Java 21**.

---

## 🧭 Table of Contents

1. [Understanding Threads in Java](#1-understanding-threads-in-java)
2. [Thread Lifecycle and States](#2-thread-lifecycle-and-states)
3. [Creating Threads — The Old Way](#3-creating-threads--the-old-way)
4. [Executor Framework and Thread Pools](#4-executor-framework-and-thread-pools)
5. [Virtual Threads (Project Loom - Java 21)](#5-virtual-threads-project-loom---java-21)
6. [Structured Concurrency (Java 21)](#6-structured-concurrency-java-21)
7. [CompletableFuture and Async Pipelines](#7-completablefuture-and-async-pipelines)
8. [Synchronization and Locks](#8-synchronization-and-locks)
9. [Atomic Variables and Concurrent Utilities](#9-atomic-variables-and-concurrent-utilities)
10. [Deadlocks and Thread Safety](#10-deadlocks-and-thread-safety)
11. [Best Practices for Scalable Concurrency](#11-best-practices-for-scalable-concurrency)
12. [Summary](#12-summary)

---

## 1️⃣ Understanding Threads in Java

A **thread** is the smallest unit of CPU execution in Java.  
By default, Java applications start with a **main thread**.

Example:
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName());
    }
}
````

Output:

```
main
```

✅ Every thread runs independently and shares memory with others.
❌ Shared memory can cause **race conditions** if not synchronized properly.

---

## 2️⃣ Thread Lifecycle and States

```
NEW → RUNNABLE → BLOCKED → WAITING → TIMED_WAITING → TERMINATED
```

### Example:

```java
Thread t = new Thread(() -> System.out.println("Running"));
t.start();
```

---

## 3️⃣ Creating Threads — The Old Way

### 🔹 Extending Thread

```java
class Worker extends Thread {
    public void run() {
        System.out.println("Task executed by " + Thread.currentThread().getName());
    }
}

new Worker().start();
```

### 🔹 Implementing Runnable

```java
Runnable task = () -> System.out.println("Runnable task running");
new Thread(task).start();
```

> 💡 Always prefer `Runnable` over subclassing `Thread`.

---

## 4️⃣ Executor Framework and Thread Pools

Introduced in Java 5, the **Executor Framework** manages thread lifecycles automatically.

### Example:

```java
ExecutorService executor = Executors.newFixedThreadPool(3);

for (int i = 0; i < 5; i++) {
    int id = i;
    executor.submit(() -> System.out.println("Task " + id + " on " + Thread.currentThread().getName()));
}

executor.shutdown();
```

✅ Prevents creating too many threads
✅ Manages task queue and lifecycle

> 💡 Use `Executors.newCachedThreadPool()` for dynamic scaling, or `newFixedThreadPool()` for predictable workloads.

---

## 5️⃣ Virtual Threads (Project Loom - Java 21)

**Virtual threads** are lightweight threads managed by the JVM — not the OS.
They allow **millions of concurrent tasks** with minimal overhead.

### Example:

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10).forEach(i -> 
        executor.submit(() -> {
            System.out.println("Task " + i + " executed by " + Thread.currentThread());
        })
    );
}
```

✅ Each task runs in its own virtual thread
✅ Excellent for I/O-bound operations
✅ Near-zero memory footprint

> ⚡ Replace “Reactive boilerplate” (CompletableFuture, WebFlux) with structured, readable concurrency using **virtual threads**.

---

## 6️⃣ Structured Concurrency (Java 21)

**Structured concurrency** manages concurrent tasks as a **single logical unit**.

### Example:

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user = scope.fork(() -> fetchUser());
    Future<String> orders = scope.fork(() -> fetchOrders());

    scope.join(); // Wait for both
    scope.throwIfFailed();

    System.out.println("User: " + user.result() + ", Orders: " + orders.result());
}
```

✅ Automatically cancels dependent tasks if one fails
✅ Improves reliability and observability
✅ Built for **microservices and API composition**

> 🧩 Structured concurrency = “try-with-resources” for threads.

---

## 7️⃣ CompletableFuture and Async Pipelines

### 🔹 Basic Example

```java
CompletableFuture.supplyAsync(() -> "Task")
    .thenApply(str -> str + " completed")
    .thenAccept(System.out::println);
```

### 🔹 Combine Multiple Async Tasks

```java
CompletableFuture<String> user = CompletableFuture.supplyAsync(() -> "User Data");
CompletableFuture<String> orders = CompletableFuture.supplyAsync(() -> "Order Data");

CompletableFuture<Void> combined =
    CompletableFuture.allOf(user, orders)
        .thenRun(() -> System.out.println("Both tasks done"));
combined.join();
```

✅ Non-blocking and chainable
✅ Perfect for service orchestration

> 💡 Combine `CompletableFuture` with **structured concurrency** for the best of both worlds.

---

## 8️⃣ Synchronization and Locks

### 🔹 synchronized Keyword

Prevents race conditions by allowing **only one thread** to access a block at a time.

```java
synchronized void increment() {
    count++;
}
```

### 🔹 ReentrantLock Example

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    shared++;
} finally {
    lock.unlock();
}
```

✅ `ReentrantLock` gives more control (tryLock, fairness).
❌ Use only when needed; prefer immutable objects.

---

## 9️⃣ Atomic Variables and Concurrent Utilities

For fine-grained thread-safe operations without locks.

```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();
```

### Other Atomic Types:

* `AtomicLong`
* `AtomicBoolean`
* `AtomicReference<T>`

### Concurrent Collections:

| Class                  | Description                   |
| ---------------------- | ----------------------------- |
| `ConcurrentHashMap`    | Thread-safe, lock-free map    |
| `CopyOnWriteArrayList` | Safe for frequent reads       |
| `BlockingQueue`        | For producer-consumer systems |

---

## 🔟 Deadlocks and Thread Safety

### 🔹 Deadlock Example

```java
synchronized (lock1) {
    synchronized (lock2) {
        // Deadlock risk if other thread locks in reverse order
    }
}
```

✅ Prevent by:

* Lock ordering
* Avoiding nested locks
* Using high-level concurrency utilities

### 🔹 Thread Safety

Ensure **atomicity, visibility, and ordering**:

* Use `volatile` for visibility
* Use **immutable objects** wherever possible

---

## 11️⃣ Best Practices for Scalable Concurrency

✅ Prefer **virtual threads** for I/O-bound workloads
✅ Avoid shared mutable state
✅ Prefer **immutable data structures** (records, `List.of()`)
✅ Use **structured concurrency** for task groups
✅ Combine **CompletableFuture** for async composition
✅ Use `Executors.newVirtualThreadPerTaskExecutor()` instead of custom thread pools
✅ Never block inside parallel streams
✅ Profile using `jconsole`, `jvisualvm`, or **JDK Flight Recorder**

Example (Real-world microservice):

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<User> user = scope.fork(() -> userService.getUserById(id));
    Future<List<Order>> orders = scope.fork(() -> orderService.getOrders(id));
    scope.join();
    return new UserProfile(user.result(), orders.result());
}
```

✅ Non-blocking, fast, readable, testable.

---

## 12️⃣ Summary

In this chapter, you’ve mastered:

* Classic thread creation and management
* Executor framework and thread pools
* Virtual threads (Project Loom)
* Structured concurrency
* CompletableFuture pipelines
* Locks, atomics, and thread safety

> ⚙️ **Modern concurrency in Java 21** is simple, scalable, and readable — no more callback hell or reactive boilerplate.

> 🧭 **Next Topic:** [`14-memory-management-and-jvm-tuning.md` → Deep dive into **JVM memory model**, **GC tuning**, and **performance profiling** for production-ready systems.](./14-memory-management-and-jvm-tuning.md)
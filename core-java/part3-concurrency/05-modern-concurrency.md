# Modern Concurrency: Futures and Virtual Threads

> **Part 3: Concurrency**  
> **Level:** Principal Engineer  
> **Status:** Completed

---

## 0. Learning Objectives

*   **Developer**: Callback Hell vs `thenApply`.
*   **Senior**: Combining multiple Futures.
*   **Architect**: Migrating from Reactive (WebFlux) to Virtual Threads (Loom).

---

## 1. CompletableFuture (Java 8)

**Pipeline Programming**:
```java
CompletableFuture.supplyAsync(() -> fetchUser(1))
    .thenApply(User::getEmail)
    .thenAccept(this::sendEmail)
    .exceptionally(ex -> { log.error(ex); return null; });
```
*   **Non-Blocking**: The main thread returns immediately.
*   **Pool**: Uses `ForkJoinPool.commonPool()` by default.

---

## 2. Deep Concept: Virtual Threads (Java 21 / Project Loom)

**The Paradigm Shift**:
*   **Old**: 1 Java Thread = 1 OS Thread (Heavy).
*   **New**: 1 Java Virtual Thread = 1 Object in Heap (Light).

### 2.1 M:N Scheduling
*   **Carrier Thread**: The OS thread (Platform Thread).
*   **Mounting**: When a VT runs, it mounts to a Carrier.
*   **Unmounting**: When a VT blocks (IO, Sleep), JVM unmounts it. The Carrier is free to run another VT.
*   **Scale**: You can have **Millions** of Virtual Threads.

### 2.2 Impact on Architecture
*   **Reactive Programming**: No longer strictly needed for scale. You can write simple Blocking Code ("Thread-per-Request") and get the scalability of Non-Blocking.
*   **Pools**: DO NOT Pool Virtual Threads. Create a new one for every task. `Executors.newVirtualThreadPerTaskExecutor()`.

---

## 3. Structured Concurrency (Java 21 Preview)

Treat a group of related tasks as a single unit of work.
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<User> user = scope.fork(() -> findUser());
    Future<Order> order = scope.fork(() -> findOrder());
    scope.join(); // Wait for both
    scope.throwIfFailed(); // Propagate errors
    // Use results
}
```
*   **Benefit**: No orphan threads. If parent cancels, children are cancelled.

---

## 4. Summary & Architect Takeaways

1.  **CompletableFuture**: Great for composing 2-3 async steps.
2.  **Virtual Threads**: The future of high-throughput IO servers. Upgrade to Java 21.
3.  **Simplicity**: Write sync code, scale like async.

---
*End of Part 3. Next: Part 4 - JVM Internals and Performance Engineering.*

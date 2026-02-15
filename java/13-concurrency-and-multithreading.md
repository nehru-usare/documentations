# 🧵 Concurrency & Multithreading: The Scalability Masterclass

> **Document Level:** Architect (15+ years experience)  
> **Focus:** JMM (Java Memory Model), AQS Internals, ForkJoin, and Project Loom (Virtual Threads)

---

## 🏗️ Layer 1: The Foundation - Java Memory Model (JMM)

Concurrency in Java isn't just about `Thread.start()`. It's about **Visibility** and **Ordering**.

### 1. The "Happens-Before" Principle
The JMM (JSR-133) provides a set of rules that guarantee when one thread's write becomes visible to another thread's read.
- **Volatile Variable**: A write to a `volatile` field happens-before every subsequent read of that same field.
- **Locking**: An unlock on a monitor happens-before every subsequent lock on the same monitor.
- **Thread Start**: A call to `Thread.start()` happens-before any action in the started thread.

### 2. Instruction Reordering
CPUs and JIT compilers reorder instructions to improve performance. 
- **Architect Insight**: Without proper synchronization, the JVM might reorder lines of code in a way that breaks a Singleton's "Double-Checked Locking" pattern.

---

## 🛠️ Layer 2: The Core - AQS (AbstractQueuedSynchronizer)

Most of `java.util.concurrent` (ReentrantLock, Semaphore, CountDownLatch) is built on **AQS**.

### How AQS works
1.  **State**: A volatile integer representing the "Lock" status (0 = Free, 1 = Locked).
2.  **CAS (Compare-And-Swap)**: Atomic operations to change the state.
3.  **Wait Queue**: A FIFO queue of threads blocked waiting for the state to change.

### `synchronized` vs. `ReentrantLock`
-   **Synchronized**: Built-in, simpler, supports "Lock Inflation" (Biased -> Thin -> Fat). 
-   **ReentrantLock**: Manual, supports **Fairness**, **Timeouts**, and **Interruptibility**.
-   **Architect Note**: In Java 21, `synchronized` can "pin" a Virtual Thread to its carrier thread, whereas `ReentrantLock` allows the Virtual Thread to yield. **Prefer ReentrantLock for Project Loom.**

---

## 🚀 Layer 3: From Pools to Stealing (ForkJoin)

### ThreadPoolExecutor
Uses a task queue and a fixed/cached set of OS threads. 
- **Limitation**: Not efficient for recursive tasks.

### ForkJoinPool (The Work-Stealing King)
- **Mechanism**: Every worker thread has its own "De-queue" of tasks. If a thread runs out of tasks, it **"Steals"** half the tasks from the back of another thread's queue.
- **Usage**: Foundation of Parallel Streams and `CompletableFuture`.

---

## 🌀 Layer 4: The Revolution - Project Loom (Virtual Threads)

Project Loom solves the "Thread-per-Request" scalability problem.

### Platform Threads vs. Virtual Threads
-   **Platform Threads**: 1:1 mapping to OS threads. 1MB stack. Expensive to context-switch (requires Kernel interrupt).
-   **Virtual Threads**: M:N mapping. Million-to-Few. Stack lives on the Heap. Context-switching is a regular Java object jump.

### Continuity & Scheduling
When a Virtual Thread hits a blocking I/O (e.g., `socket.read()`), the JVM **"Unmounts"** the task from the Carrier thread and stores its state on the heap. When data arrives, the scheduler **"Remounts"** it on any available Carrier thread.

> [!WARNING]
> **Pinning Caveat**: If a Virtual Thread is inside a `synchronized` block or calling a `native` method, it is **Pinned** to the Carrier thread. If the operation blocks, the OS thread is blocked, defeating the purpose of Loom.

---

## 📜 Layer 5: Advanced Patterns - CompletableFuture

For Reactive and Asynchronous flows.
- **Combining**: `thenCompose`, `thenCombine`.
- **Handling Errors**: `exceptionally`, `handle`.
- **Performance**: Always provide a custom `Executor` for `supplyAsync`. Using the default `ForkJoinPool.commonPool()` can lead to thread starvation in high-load production.

---

## 🧭 Interview Prep & Architect Scenarios

### Q: Why is `long` and `double` not atomic?
**A**: Per the JVM spec, a 32-bit JVM might write a 64-bit `long` as two separate 32-bit operations. Without `volatile`, a thread might read "half-written" data. (Most modern 64-bit JVMs make it atomic anyway, but you shouldn't rely on it).

### Q: How do you handle "False Sharing" in Java?
**A**: Use the `@Contended` annotation or manual padding to ensure variables updated by different threads stay on different **CPU Cache Lines** (usually 64 bytes). This prevents the CPU from thrashing the cache.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [12-streams-and-functional-programming.md](./12-streams-and-functional-programming.md) | Streams & FP |
| ⏩ **Next** | [14-memory-management-and-jvm-tuning.md](./14-memory-management-and-jvm-tuning.md) | Memory & GC |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026
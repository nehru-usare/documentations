# Executors: Thread Pools and Futures

> **Part 3: Concurrency**  
> **Level:** Principal Engineer  
> **Status:** Submitted

---

## 0. Learning Objectives

*   **Developer**: Why `newThread().start()` is bad in loops.
*   **Senior**: Configuring `ThreadPoolExecutor` (Core vs Max threads).
*   **Architect**: Using `ForkJoinPool` for recursive tasks.

---

## 1. Deep Concept: The Thread Pool Model

Reuse expensive threads.

### 1.1 `ThreadPoolExecutor` Parameters
1.  **Core Pool Size**: Threads to keep alive always.
2.  **Max Pool Size**: Max threads allowed during bursts.
3.  **Queue**: Holds tasks when Core threads are busy.
    *   `LinkedBlockingQueue`: Unbounded. Risk of OOM.
    *   `SynchronousQueue`: 0 Capacity. Hand-off only. Spites new thread immediately. Used in CachedThreadPool.

### 1.2 Common Pools (`Executors` factory)
*   **FixedThreadPool(N)**: `core=N, max=N, queue=Linked`. Stable for servers.
*   **CachedThreadPool**: `core=0, max=Integer.MAX_VALUE`. Spawns unlimited threads. Risk of crashing OS.
*   **SingleThreadExecutor**: Serializes tasks.

---

## 2. ForkJoinPool (Work Stealing)

*   **Design**: Per-thread Deque.
*   **Work Stealing**: If Thread A finishes its queue, it steals the *tail* of Thread B's queue.
*   **Use Case**: Recursive tasks (Divide and Conquer). Parallel Streams use the Common ForkJoinPool.

---

## 3. Production Debugging Guide

### Thread Leak
*   **Symptom**: Application stops processing. `jstack` shows 1000 threads.
*   **Cause**: Tasks are blocking forever (e.g., IO without timeout). Pool is exhausted.
*   **Fix**: Always use Timeouts. Monitor Queue Depth.

### Graceful Shutdown
```java
pool.shutdown(); // Stop accepting new tasks
if (!pool.awaitTermination(60, SECONDS)) {
    pool.shutdownNow(); // Interrupt running tasks
}
```

---

## 4. Summary & Architect Takeaways

1.  **Custom Pools**: Prefer `new ThreadPoolExecutor(...)` over `Executors.newFixedThreadPool` to control Queue capacity and RejectionPolicy.
2.  **Size Matters**: CPU-bound = N_CORES + 1. IO-bound = N_CORES * (1 + Wait/Compute).
3.  **Rejection**: Handle `RejectedExecutionException`. Drop? Log? Caller-Runs?

---
*Next Chapter: CompletableFuture and Virtual Threads (Loom).*

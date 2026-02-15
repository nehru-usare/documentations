# Synchronization: Monitors, Locks, and Wait Sets

> **Part 3: Concurrency**  
> **Level:** Principal Engineer  
> **Status:** Locked

---

## 0. Learning Objectives

*   **Developer**: Why `synchronized` is safer than `Lock`.
*   **Senior**: How Biased Locking optimized un-contended locks (and why it's deprecated).
*   **Architect**: Choosing between `ReentrantLock` (fairness) and `synchronized` (simplicity).

---

## 1. Deep Concept: The Monitor (Intrinsic Lock)

Every Object in Java has an associated **Monitor**.
*   **Owner**: Only one thread can own the Monitor at a time.
*   **Wait Set**: Threads waiting for a condition (`wait()`).
*   **Entry Set**: Threads blocked waiting to acquire the lock.

### 1.1 `synchronized` Implementation
*   **Bytecode**: `monitorenter` and `monitorexit`.
*   **Mechanism**: Changes the **Mark Word** in the Object Header to point to a standard Monitor Record.

### 1.2 Lock Inflation (The Cost)
1.  **Biased Lock**: (Deprecated in Java 15). Assumed only one thread ever locks it. Zero CAS.
2.  **Thin Lock (Lightweight)**: CAS operation on the Stack. Fast.
3.  **Fat Lock (Heavyweight)**: Contention detected. Inflates to a full OS Mutex. Puts threads to sleep (Context Switch). Slow.

---

## 2. ReentrantLock (JUC)

*   **API**: `lock.lock()`, `lock.unlock()`.
*   **Features**:
    *   **Fairness**: `new ReentrantLock(true)`. FIFO order (Slow).
    *   **TryLock**: `lock.tryLock()`. Non-blocking attempt.
    *   **Conditions**: `lock.newCondition()`. Multiple wait-sets per lock (unlike `synchronized`).

---

## 3. Production Debugging Guide

### Deadlock Analysis
*   **Scenario**: Thread A holds Lock 1, wants Lock 2. Thread B holds Lock 2, wants Lock 1.
*   **Detection**: `jstack <pid>` or VisualVM.
*   **Output**: "Found one Java-level deadlock".

---

## 4. Summary & Architect Takeaways

1.  **Prefer `synchronized`**: JVM optimizes it better (Lock Elision, Escpae Analysis) and it's exception-safe (auto-release).
2.  **Use `ReentrantLock` only if**: You need fairness, tryLock, or interruptible lock acquisition.
3.  **Minimize Scope**: `synchronized(lock) { fast_op(); }`. Don't wrap huge I/O blocks.

---
*Next Chapter: The Java Memory Model (JMM) and Volatile.*

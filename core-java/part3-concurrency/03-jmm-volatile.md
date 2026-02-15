# Java Memory Model (JMM): Visibility and Ordering

> **Part 3: Concurrency**  
> **Level:** Principal Engineer  
> **Status:** Visible

---

## 0. Learning Objectives

*   **Developer**: Why `volatile` is not a Lock.
*   **Senior**: The definition of "Happens-Before".
*   **Architect**: Designing lock-free algorithms using `AtomicReference` and CAS.

---

## 1. Deep Concept: Visibility vs Atomicity

### 1.1 CPU Caches and Shared Memory
*   Thread A runs on Core 1. Writes `x = 1`. Value is in L1 Cache.
*   Thread B runs on Core 2. Reads `x`. Value is stale (0).
*   **The JMM**: Defines when Core 1 MUST flush to Main Memory and Core 2 MUST invalidate cache.

### 1.2 The `volatile` Keyword
*   **Visibility Guarantee**: A write to `volatile` x is visible to all threads immediately.
*   **Ordering Guarantee**: Prevents Instruction Reordering.
*   **Atomicity**: NO. `x++` is NOT atomic even if `x` is volatile. (It's Read-Modify-Write).

---

## 2. Deep Technical Explanation: Happens-Before

The JMM is defined by a partial order called **Happens-Before (HB)**.
*   If Action A HB Action B, memory writes in A are visible to B.
*   **Rules**:
    1.  **Program Order**: Statements in a thread HB subsequent statements.
    2.  **Volatile Write**: Write to volatile HB subsequent Read of same volatile.
    3.  **Monitor Lock**: `unlock` HB `lock`.
    4.  **Thread Start**: `thread.start()` HB actions in the new thread.

---

## 3. Safe Publication

How to safely share an object?
1.  **Static Initializer**: `static final MyObj o = new MyObj();` (ClassLoad lock handles it).
2.  **Volatile Field**: `volatile MyObj o = ...`.
3.  **Final Fields**: If an object has `final` fields, they are guaranteed visible after constructor finishes. (Explains why Immutable Objects are thread-safe).

---

## 4. Production Debugging: Double-Checked Locking

**The Broken Pattern**:
```java
if (instance == null) {
    synchronized(this) {
        if (instance == null) instance = new Singleton(); // Unsafe!
    }
}
```
**Why?**: `new Singleton()` is 3 steps: (1) Allocate Memory, (2) Call Constructor, (3) Assign Reference.
The CPU can reorder to 1 -> 3 -> 2. Another thread sees non-null `instance` but uninitialized fields.
**Fix**: `private volatile static Singleton instance;`.

---

## 5. Summary & Architect Takeaways

1.  **Volatile is expensive**: Flushing caches is slower than register access. Use sparingly.
2.  **HB is the law**: If you can't establish a Happens-Before link, your code is racy.
3.  **Final Fields**: Use `final` everywhere. It gives free thread-safety guarantees.

---
*Next Chapter: Executors and Thread Pools.*

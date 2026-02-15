# JVM Memory Model: Heap, Stack, and Metaspace

> **Part 4: JVM Internals**  
> **Level:** Principal Engineer  
> **Status:** Allocated

---

## 0. Learning Objectives

*   **Developer**: Why `StackOverflowError` is different from `OutOfMemoryError`.
*   **Senior**: Tuning `-Xmx` and `-Xms`.
*   **Architect**: Understanding Native Memory usage (Direct Buffers, Metaspace).

---

## 1. Runtime Data Areas

### 1.1 The Heap (Shared)
*   **Young Generation (Eden + Survivor S0/S1)**: Where new objects are born. High churn.
*   **Old Generation (Tenured)**: Where long-lived objects survive.
*   **Hypothesis**: Weak Generational Hypothesis (Most objects die young).

### 1.2 The Stack (Thread Local)
*   Frames: Local Variables, Operand Stack, Return Address.
*   **Error**: Recursion too deep -> `StackOverflowError`.

### 1.3 Metaspace (Native Memory)
*   Replaced PermGen (Java 8).
*   Stores: Class Metadata, Static Variables, Bytecode.
*   **Limit**: `-XX:MaxMetaspaceSize`. If unlimited, can eat all OS RAM.

### 1.4 Code Cache
*   Stores JIT Compiled Native Code (Assembly).
*   If full, JIT stops optimizing.

---

## 2. Production Debugging Guide

### Calculating Total RAM
`Total RAM = Heap + Metaspace + CodeCache + (Stack Size * Thread Count) + Direct Buffers`.
*   **Mistake**: Setting Heap = Physical RAM. OS will kill the process (OOM Killer) when it tries to allocate Stack or Metaspace.

---

## 3. Summary & Architect Takeaways

1.  **Always Set Xmx and Xms**: To the same value in Prod (Avoid resizing jitter).
2.  **Monitor Metaspace**: ClassLoader leaks (common in hot-reload containers) fill Metaspace.
3.  **Direct Memory**: Netty/NIO uses off-heap memory. Monitor `-XX:MaxDirectMemorySize`.

---
*Next Chapter: Garbage Collection Algorithms.*

# Garbage Collection: Algorithms and Tuning

> **Part 4: JVM Internals**  
> **Level:** Principal Engineer  
> **Status:** Cleanned

---

## 0. Learning Objectives

*   **Developer**: Why Full GC pauses the world.
*   **Senior**: Tuning G1GC Pause Goals (`-XX:MaxGCPauseMillis`).
*   **Architect**: Choosing ZGC for large heaps (>100GB).

---

## 1. The Classics (Stop-The-World)

### 1.1 Mark and Sweep
1.  **Mark**: Traverse Object Graph from GC Roots. Mark live objects.
2.  **Sweep**: Reclaim memory of unmarked objects.
3.  **Compact**: Move live objects together to defragment heap. (Expensive).

### 1.2 Collector Types
*   **Serial**: Single-threaded. For CLI apps.
*   **Parallel**: High Throughput. Long Pauses. Default in Java 8.
*   **CMS**: Concurrent Mark Sweep. Low Latency. Fragmentation issues. Deprecated.

---

## 2. G1GC (Garbage First) - Default since Java 9

Region-based collector.
*   **Heap Partition**: Splits Heap into 2048 regions (1MB-32MB).
*   **Logic**: Tracks "Garbage Value" of each region. Collects regions with *most garbage first*.
*   **Pause Goal**: You set `MaxGCPauseMillis=200`. G1 tries to meet it by collecting fewer regions.
*   **String Deduplication**: Enabled by default.

---

## 3. ZGC (The Future) - Production ready in Java 15

Scalable Low Latency GC.
*   **Goal**: Max Pause < 10ms. Heap size 8MB - 16TB.
*   **Mechanism**: **Colored Pointers** and **Load Barriers**. It does almost everything concurrently with the application threads.
*   **Trade-off**: Higher CPU usage (throughput slightly lower than Parallel).

---

## 4. Production Debugging Guide

### GC Logs
*   Enable: `-Xlog:gc*:file=gc.log:time,tags`.
*   Analyze: Use **GCViewer** or **GCEasy**.
*   **Red Flag**: "Full GC (Allocation Failure)". Means your heap is too small or you are leaking memory.

---

## 5. Summary & Architect Takeaways

1.  **Use G1**: It's the best balance for general Microservices.
2.  **Use ZGC**: If you have huge heaps (32GB+) and need low latency.
3.  **Avoid System.gc()**: Disable it with `-XX:+DisableExplicitGC`.

---
*Next Chapter: JIT Compilation and Warmup.*

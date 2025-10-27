# 🧠 Java 21 — Memory Management and JVM Tuning

To build high-performance and scalable applications, every senior Java developer must understand **how the JVM allocates memory, reclaims it (GC), and executes bytecode efficiently**.

This guide provides a **practical, production-oriented overview** of the **Java Virtual Machine (JVM)** — covering **memory structure**, **garbage collection**, and **JVM tuning techniques**.

---

## 🧭 Table of Contents

1. [The JVM Architecture Overview](#1-the-jvm-architecture-overview)
2. [JVM Runtime Memory Areas](#2-jvm-runtime-memory-areas)
3. [Java Memory Model (JMM)](#3-java-memory-model-jmm)
4. [Garbage Collection in Java](#4-garbage-collection-in-java)
5. [Major Garbage Collectors in Java 21](#5-major-garbage-collectors-in-java-21)
6. [GC Tuning Parameters](#6-gc-tuning-parameters)
7. [JIT Compilation and HotSpot Optimizations](#7-jit-compilation-and-hotspot-optimizations)
8. [Performance Monitoring and Profiling Tools](#8-performance-monitoring-and-profiling-tools)
9. [Heap Dump and Thread Dump Analysis](#9-heap-dump-and-thread-dump-analysis)
10. [Best Practices for JVM Performance](#10-best-practices-for-jvm-performance)
11. [Summary](#11-summary)

---

## 1️⃣ The JVM Architecture Overview

The **Java Virtual Machine (JVM)** is the engine that executes Java bytecode.  

It consists of:
- **Class Loader Subsystem** → Loads `.class` files  
- **Runtime Data Areas** → Stores data during execution  
- **Execution Engine** → Interprets / JIT compiles bytecode  
- **Garbage Collector** → Manages memory  
- **Native Interface (JNI)** → Interacts with native code  

```
          ┌─────────────────────────────┐
          │        Java Source          │
          │        .java files          │
          └─────────────┬───────────────┘
                        │
                        ▼
               ┌─────────────────┐
               │   javac (.class)│
               └─────────────────┘
                        │
                        ▼
              ┌──────────────────┐
              │     JVM Runtime   │
              ├──────────────────┤
              │ Class Loader      │
              │ Runtime Data Area │
              │ Execution Engine  │
              │ Garbage Collector │
              └──────────────────┘
```

---

## 2️⃣ JVM Runtime Memory Areas

| Memory Area | Description |
|--------------|--------------|
| **Method Area** | Stores class metadata, static variables, and bytecode |
| **Heap** | Stores all objects and arrays |
| **Stack** | Stores method frames, local variables, and partial results |
| **PC Register** | Keeps track of the current instruction |
| **Native Method Stack** | For native (C/C++) calls |

### 🔹 Heap Structure (Simplified)
```
+-------------------------------+
| Young Generation              |
|   ├── Eden                    |
|   ├── Survivor Space 1 (S0)   |
|   ├── Survivor Space 2 (S1)   |
+-------------------------------+
| Old Generation (Tenured)      |
+-------------------------------+
| Metaspace (Native Memory)     |
+-------------------------------+
```

> 💡 Java 8+ replaced *PermGen* with *Metaspace* (allocated in native memory).

---

## 3️⃣ Java Memory Model (JMM)

The **Java Memory Model** defines how threads interact with shared memory.

- Variables are stored in **main memory**.  
- Threads maintain **local caches** (working memory).  
- Synchronization ensures **visibility** and **ordering** of operations.

### 🔹 Key JMM Keywords:
| Keyword | Description |
|----------|--------------|
| `volatile` | Ensures visibility across threads |
| `synchronized` | Ensures atomicity and mutual exclusion |
| `final` | Guarantees immutability after construction |

> 💡 Always use `volatile` for shared read/write variables, or better — prefer **immutable objects**.

---

## 4️⃣ Garbage Collection in Java

The Garbage Collector (GC) automatically removes objects that are no longer reachable.

### 🔹 GC Process
1. **Mark:** Identify reachable objects.  
2. **Sweep:** Remove unreferenced objects.  
3. **Compact:** Reorganize memory to reduce fragmentation.

### 🔹 Generational Hypothesis:
- Most objects die young → Collect young generation frequently.  
- Long-lived objects move to old generation.

> ⚙️ Tuning GC = balancing throughput (speed) vs pause time (latency).

---

## 5️⃣ Major Garbage Collectors in Java 21

| GC Name | Introduced | Best For | Description |
|----------|-------------|----------|--------------|
| **Serial GC** | Legacy | Small apps | Single-threaded, simple |
| **Parallel GC** | Java 5 | High throughput | Multi-threaded stop-the-world |
| **G1 GC (Default)** | Java 9 | Balanced workloads | Region-based, concurrent |
| **ZGC** | Java 15 | Low-latency systems | Pause time < 10ms, scalable |
| **Shenandoah** | Java 12 | Ultra-low latency | Concurrent compaction, <10ms pause |

✅ **Java 21 Default GC:** G1 (high performance, concurrent, region-based).

### Example: Enable ZGC
```bash
java -XX:+UseZGC -Xmx8g -Xms8g MyApp
```

---

## 6️⃣ GC Tuning Parameters

| Parameter | Description | Example |
|------------|--------------|----------|
| `-Xms` | Initial heap size | `-Xms512m` |
| `-Xmx` | Maximum heap size | `-Xmx2g` |
| `-XX:NewRatio` | Ratio of young to old generation | `-XX:NewRatio=2` |
| `-XX:+UseG1GC` | Enable G1 GC | Default in Java 21 |
| `-XX:+UseZGC` | Enable Z Garbage Collector | `ZGC` |
| `-XX:+PrintGCDetails` | Prints GC logs | For tuning |
| `-Xlog:gc*` | Unified GC logging | `-Xlog:gc*,gc+heap=debug:file=gc.log` |

> 💡 Always start with **G1GC**. For ultra-low-latency apps, try **ZGC** or **Shenandoah**.

---

## 7️⃣ JIT Compilation and HotSpot Optimizations

The JVM uses **Just-In-Time (JIT)** compilation to optimize bytecode at runtime.

### 🔹 JIT Flow:
1. Bytecode interpreted by JVM.  
2. Frequently executed methods (“hot code”) are **compiled to native code**.  
3. Code is optimized dynamically based on runtime profiling.

### 🔹 Two Compilers:
| Compiler | Description |
|-----------|--------------|
| **C1 (Client)** | Quick startup, used in dev |
| **C2 (Server)** | Aggressive optimization |
| **Graal JIT (Java 11+)** | Modern, plugin-based compiler for performance-critical workloads |

✅ Java 21 integrates **Graal JIT** and **AOT (Ahead-of-Time)** compilation for better startup in microservices.

---

## 8️⃣ Performance Monitoring and Profiling Tools

| Tool | Description |
|------|--------------|
| **jconsole** | GUI-based JVM monitor |
| **jvisualvm** | Heap, thread, CPU profiler |
| **jcmd** | Command-line diagnostics tool |
| **jmap / jstack** | Dump heap and thread states |
| **Java Flight Recorder (JFR)** | Low-overhead profiling |
| **Mission Control (JMC)** | Visual analysis of JFR data |

### 🔹 Example Commands
```bash
jcmd <pid> GC.heap_info
jcmd <pid> VM.flags
jmap -heap <pid>
jstack <pid> > thread-dump.txt
```

> 🧩 Combine **JFR + Mission Control** for production-safe performance monitoring.

---

## 9️⃣ Heap Dump and Thread Dump Analysis

### 🔹 Create Heap Dump
```bash
jmap -dump:live,format=b,file=heapdump.hprof <pid>
```

Analyze with:
- **Eclipse MAT (Memory Analyzer Tool)**
- **VisualVM Heap Dump Viewer**

### 🔹 Thread Dump Example
```bash
jstack <pid> > threads.log
```

Use to:
- Detect deadlocks
- Identify blocked or waiting threads
- Analyze contention points

---

## 🔟 Best Practices for JVM Performance

✅ Choose GC based on **application profile**  
✅ Right-size your heap (`Xms` and `Xmx`)  
✅ Avoid excessive object creation (especially in loops)  
✅ Use **object pooling** only if GC pressure is high  
✅ Reuse immutable objects  
✅ Prefer **records** for DTOs (faster, compact)  
✅ Enable GC logs in production  
✅ Profile before optimizing — don’t guess  
✅ For microservices: Use **container-aware flags**  
```bash
-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0
```

✅ Combine **Virtual Threads + ZGC** for high concurrency and minimal GC pause time  

---

## 11️⃣ Summary

In this chapter, you’ve mastered:
- JVM architecture and memory areas  
- Java Memory Model and visibility  
- GC algorithms: G1, ZGC, Shenandoah  
- GC tuning and monitoring  
- JIT compilation and HotSpot optimizations  
- JVM profiling tools (JFR, VisualVM, Mission Control)  

> 🧩 JVM tuning is an iterative process — **measure, analyze, then adjust**.  
> The best developers don’t guess memory usage; they *profile and optimize*.

> 🧭 **Next Topic (Optional Extension):**  
[15-performance-and-optimization-patterns.md → Advanced Performance Patterns in Java 21 — Caching, Pooling, Benchmarking, and Profiling](./15-performance-and-optimization-patterns.md)
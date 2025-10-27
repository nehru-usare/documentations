# ⚙️ Java Architecture and JVM Internals (Java 21)

To write efficient Java applications — especially in large, scalable systems — understanding the **JVM (Java Virtual Machine)** and **how Java executes code** is essential.  
This document explains **how Java works internally**, focusing on the **Java 21 architecture** and its runtime performance optimizations.

---

## 🧭 Table of Contents

1. [Java Architecture Overview](#1-java-architecture-overview)
2. [How Java Code Executes](#2-how-java-code-executes)
3. [JVM Architecture](#3-jvm-architecture)
4. [Class Loader Subsystem](#4-class-loader-subsystem)
5. [JVM Runtime Data Areas](#5-jvm-runtime-data-areas)
6. [Execution Engine](#6-execution-engine)
7. [JIT Compiler (Just-In-Time)](#7-jit-compiler-just-in-time)
8. [Garbage Collection in Java 21](#8-garbage-collection-in-java-21)
9. [Key JVM Parameters and Monitoring](#9-key-jvm-parameters-and-monitoring)
10. [Summary](#10-summary)

---

## 1️⃣ Java Architecture Overview

Java’s architecture is built on the concept of **"Write Once, Run Anywhere"**, powered by the **JVM**.

### 🔹 Key Components

| Component | Description |
|------------|-------------|
| **JDK** | Java Development Kit – full development environment (compiler, debugger, JRE) |
| **JRE** | Java Runtime Environment – includes JVM and libraries for running programs |
| **JVM** | Java Virtual Machine – executes compiled bytecode |
| **Java Libraries (API)** | Pre-built classes (Collections, IO, Networking, etc.) |
| **Java Compiler (`javac`)** | Converts `.java` files into `.class` (bytecode) |

---

## 2️⃣ How Java Code Executes

Java follows a **multi-stage execution model** that ensures portability and performance.

```

Source Code (.java)
↓ (javac compiler)
Bytecode (.class)
↓ (Class Loader)
JVM Execution Engine
↓
Native Machine Code

```

### 🔹 Step-by-Step Flow

1. **Write code** – Developer writes `.java` files.
2. **Compile** – `javac` compiles code into platform-independent `.class` bytecode.
3. **Load** – ClassLoader loads `.class` files into JVM memory.
4. **Verify** – Bytecode verifier checks validity and security.
5. **Execute** – JVM’s Execution Engine runs bytecode using JIT (Just-In-Time) compilation.

> 🔸 Java bytecode is the same across platforms — only the JVM implementation changes.

---

## 3️⃣ JVM Architecture

The **Java Virtual Machine (JVM)** is the runtime engine that drives Java’s cross-platform capabilities.

### 🧩 Major Components

1. **Class Loader Subsystem**
2. **Runtime Data Areas (Memory)**
3. **Execution Engine**
4. **Garbage Collector**
5. **Native Interface (JNI)**
6. **Native Method Libraries**

```

```
        ┌─────────────────────────────┐
        │         Class Loader        │
        └────────────┬────────────────┘
                     │
```

┌────────────────────────┼──────────────────────────┐
│                    JVM MEMORY                     │
│ ┌─────────────┐ ┌──────────────┐ ┌──────────────┐ │
│ │ Method Area │ │ Heap Memory  │ │ Stack Memory │ │
│ └─────────────┘ └──────────────┘ └──────────────┘ │
│ │ PC Register │ │ Native Stack │ │               │ │
└───────────────────────────────────────────────────┘
│
┌───────────────┐
│ Execution Eng │
│  (Interpreter │
│     + JIT)    │
└───────────────┘

```

---

## 4️⃣ Class Loader Subsystem

The **Class Loader Subsystem** is responsible for dynamically loading classes at runtime.

### 🔹 Class Loading Process

| Phase | Description |
|--------|-------------|
| **Loading** | `.class` files are read by ClassLoader |
| **Linking** | Classes are verified and prepared |
| **Initialization** | Static variables initialized, static blocks executed |

### 🔹 Class Loader Hierarchy

| Loader | Purpose |
|---------|----------|
| **Bootstrap ClassLoader** | Loads core Java libraries (`rt.jar`, now `modules`) |
| **Platform ClassLoader** | Loads JDK platform modules |
| **Application ClassLoader** | Loads user-defined classes (from classpath) |

> Each class is loaded **once per ClassLoader**, ensuring memory safety and consistency.

---

## 5️⃣ JVM Runtime Data Areas

JVM divides its memory into several runtime areas:

| Area | Description |
|------|--------------|
| **Method Area** | Stores class metadata, method code, constants |
| **Heap** | Stores all objects and instance variables |
| **Stack** | Each thread has its own stack storing method frames |
| **Program Counter (PC)** | Holds address of current instruction being executed |
| **Native Method Stack** | Used for native (non-Java) methods |

### 🧠 Memory Snapshot

```

+--------------------------+
| Method Area              | ← Class-level data
+--------------------------+
| Heap                     | ← All objects & instances
+--------------------------+
| Java Stacks (per thread) | ← Method calls & local vars
+--------------------------+
| PC Registers             | ← Instruction addresses
+--------------------------+
| Native Stack             | ← JNI calls (C/C++)
+--------------------------+

````

---

## 6️⃣ Execution Engine

The **Execution Engine** is the heart of JVM — it converts bytecode into native code.

### 🔹 Components
| Component | Description |
|------------|-------------|
| **Interpreter** | Reads and executes bytecode line-by-line |
| **JIT Compiler** | Converts frequently used code into optimized native machine code |
| **Garbage Collector** | Frees memory occupied by unreachable objects |

> Modern JVMs like **HotSpot** use both interpreter and JIT compiler for hybrid performance.

---

## 7️⃣ JIT Compiler (Just-In-Time)

The **JIT Compiler** enhances performance by compiling bytecode **at runtime** into native machine code.

### 🔹 Types of JIT Compilers

| Type | Description |
|------|--------------|
| **Client Compiler (C1)** | Optimized for startup speed |
| **Server Compiler (C2)** | Optimized for long-running apps |
| **Graal JIT (Java 21+)** | Next-gen JIT compiler written in Java itself, highly optimized |

### 🔹 Process
1. JVM identifies “hot” code paths.
2. Converts them into native machine instructions.
3. Caches the compiled code for reuse.

> ⚡ With Java 21, the **Graal JIT** provides advanced optimizations like **escape analysis** and **vectorization** for modern CPUs.

---

## 8️⃣ Garbage Collection in Java 21

Garbage Collection (GC) automatically manages memory, preventing leaks and freeing unused objects.

### 🔹 Key GC Algorithms (Java 21)
| GC | Description | Use Case |
|----|-------------|----------|
| **G1 GC (Default)** | Parallel, incremental GC for low pause times | General-purpose |
| **ZGC** | Ultra-low pause GC (<1 ms) | Large heaps, real-time apps |
| **Shenandoah GC** | Concurrent GC from Red Hat | High throughput servers |
| **Serial GC** | Single-threaded, simple GC | Small apps, embedded systems |

> Java 21 allows switching GCs using runtime flags:
```bash
java -XX:+UseZGC MyApp
````

### 🔹 GC Phases

1. **Mark** – Identify reachable objects
2. **Sweep** – Reclaim memory from unreachable objects
3. **Compact** – Rearrange live objects to prevent fragmentation

---

## 9️⃣ Key JVM Parameters and Monitoring

### 🔹 Common JVM Options

```bash
java -Xms512m -Xmx4g -XX:+UseG1GC -XX:+PrintGCDetails -jar app.jar
```

| Option                | Description              |
| --------------------- | ------------------------ |
| `-Xms`                | Initial heap size        |
| `-Xmx`                | Maximum heap size        |
| `-XX:+UseG1GC`        | Use G1 garbage collector |
| `-XX:+PrintGCDetails` | Print detailed GC logs   |

### 🔹 Monitoring Tools

| Tool                           | Purpose                                |
| ------------------------------ | -------------------------------------- |
| **jconsole**                   | Basic GUI-based monitoring             |
| **jvisualvm**                  | Advanced memory, thread, CPU analysis  |
| **jcmd / jmap / jstack**       | Command-line diagnostic tools          |
| **JFR (Java Flight Recorder)** | Real-time profiling built into Java 21 |

Example:

```bash
jcmd <pid> GC.heap_info
```

---

## 🔟 Summary

By now, you understand how the **JVM powers Java’s portability and performance**.
In Java 21, with features like **Graal JIT**, **ZGC**, and **improved class loading**, the JVM is faster and more memory-efficient than ever.

✅ Key Takeaways:

* JVM executes bytecode through **class loading → verification → JIT compilation**
* Memory is managed via **heap**, **stack**, and **method areas**
* Garbage collection automates memory cleanup
* Monitoring tools help diagnose performance issues

---

> 🧭 **Next Topic:** [03-setup-and-environment.md → Setting up Java 21 Development Environment](./03-setup-and-environment.md)

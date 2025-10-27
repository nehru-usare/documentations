# ⚡ Java 21 — Performance and Optimization Patterns

Building fast, memory-efficient, and scalable applications requires more than knowing the language — you must **understand performance trade-offs across the JVM, data structures, I/O, and concurrency**.

This guide explores **real-world performance engineering patterns** for Java 21 — practical for developers building enterprise, low-latency, or high-throughput systems.

---

## 🧭 Table of Contents

1. [Why Performance Optimization Matters](#1-why-performance-optimization-matters)
2. [Performance Mindset for Developers](#2-performance-mindset-for-developers)
3. [Microbenchmarking with JMH (Java Microbenchmark Harness)](#3-microbenchmarking-with-jmh-java-microbenchmark-harness)
4. [Data Structure Selection and Memory Efficiency](#4-data-structure-selection-and-memory-efficiency)
5. [String, Object, and Record Optimization](#5-string-object-and-record-optimization)
6. [Concurrency and Parallelism Optimization](#6-concurrency-and-parallelism-optimization)
7. [I/O and Network Performance](#7-io-and-network-performance)
8. [JIT and AOT Compilation Tuning](#8-jit-and-aot-compilation-tuning)
9. [Profiling and Monitoring Tools](#9-profiling-and-monitoring-tools)
10. [Practical Performance Patterns](#10-practical-performance-patterns)
11. [Best Practices Summary](#11-best-practices-summary)
12. [Official Resources and Reference Links](#12-official-resources-and-reference-links)

---

## 1️⃣ Why Performance Optimization Matters

In enterprise applications, **poor performance = business loss**.  

Optimizing Java applications ensures:
- **Lower latency** for API responses  
- **Better scalability** under load  
- **Reduced cloud cost** and footprint  
- **Higher throughput and concurrency**

> 🧩 Goal: “Do more with less” — fewer CPU cycles, smaller memory footprint, faster GC.

---

## 2️⃣ Performance Mindset for Developers

✅ Always **measure first**, never guess  
✅ Profile real workloads, not synthetic tests  
✅ Optimize based on **bottlenecks**, not code size  
✅ Keep **code maintainability** higher than micro-optimizations  

> 💡 The 80/20 Rule: 80% of execution time is spent in 20% of the code.

---

## 3️⃣ Microbenchmarking with JMH (Java Microbenchmark Harness)

[JMH (OpenJDK)](https://openjdk.org/projects/code-tools/jmh/) is the official framework for **accurate microbenchmarks**.

### Example:
```java
import org.openjdk.jmh.annotations.*;

@State(Scope.Benchmark)
public class StringConcatBenchmark {

    @Benchmark
    public String testPlusOperator() {
        return "Hello " + "World";
    }

    @Benchmark
    public String testStringBuilder() {
        return new StringBuilder("Hello ").append("World").toString();
    }
}
````

### Run Benchmark:

```bash
java -jar target/benchmarks.jar
```

✅ JMH handles **JIT warm-up**, **GC effects**, and **CPU jitter**.
✅ Use for algorithm comparison, serialization tests, caching efficiency, etc.

---

## 4️⃣ Data Structure Selection and Memory Efficiency

| Goal                   | Recommended Data Structure                  |
| ---------------------- | ------------------------------------------- |
| Fast lookup            | `HashMap`, `ConcurrentHashMap`              |
| Order + fast traversal | `ArrayList`                                 |
| Frequent insert/delete | `LinkedList` (use carefully)                |
| Sorted elements        | `TreeMap`, `TreeSet`                        |
| Thread-safe list       | `CopyOnWriteArrayList`                      |
| High throughput queues | `ArrayBlockingQueue`, `LinkedBlockingQueue` |

> 💡 Choose **ArrayList over LinkedList** unless random insert/delete dominates.

### Example:

```java
List<String> list = new ArrayList<>(1000); // Pre-size to avoid resizing
```

---

## 5️⃣ String, Object, and Record Optimization

### 🔹 Use StringBuilder for concatenation in loops:

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) sb.append(i);
```

### 🔹 Prefer Records for DTOs:

```java
public record UserDTO(String name, int age) {}
```

✅ Immutable
✅ Compact memory layout
✅ Faster serialization

### 🔹 Avoid unnecessary object creation:

* Use `String.intern()` cautiously.
* Reuse frequently used objects.
* Prefer **primitive streams** (`IntStream`, `DoubleStream`) for numeric processing.

---

## 6️⃣ Concurrency and Parallelism Optimization

### 🔹 Virtual Threads for I/O-bound tasks

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10).forEach(i ->
        executor.submit(() -> {
            Thread.sleep(500);
            return i;
        })
    );
}
```

✅ Millions of lightweight threads
✅ No blocking overhead
✅ Best for microservices

---

### 🔹 CompletableFuture for Asynchronous Pipelining

```java
CompletableFuture
    .supplyAsync(() -> service.fetchData())
    .thenApply(data -> process(data))
    .thenAccept(System.out::println);
```

✅ Non-blocking pipelines
✅ Thread reusability

---

## 7️⃣ I/O and Network Performance

| Task                 | Optimization                                           |
| -------------------- | ------------------------------------------------------ |
| File I/O             | Use `Files.newBufferedReader()` or NIO2                |
| Network I/O          | Use async libraries (Netty, HTTPClient)                |
| JSON serialization   | Use Jackson with `ObjectReader` / `ObjectWriter` reuse |
| Database connections | Use Connection Pooling (HikariCP)                      |
| Caching              | Use in-memory caches (Caffeine, EHCache, Redis)        |

### Example: Efficient Buffered File Reading

```java
try (BufferedReader reader = Files.newBufferedReader(Path.of("data.txt"))) {
    reader.lines().forEach(System.out::println);
}
```

> 💡 Always close resources or use try-with-resources.

---

## 8️⃣ JIT and AOT Compilation Tuning

JIT (Just-In-Time) compiles hot methods into native code.
Java 21 supports **Graal JIT** and **AOT** for faster startup.

### Tuning Flags:

```bash
-XX:+TieredCompilation
-XX:+UseG1GC
-XX:+PrintCompilation
-XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler
```

### Ahead-of-Time Compilation:

```bash
jaotc --output libApp.so MyApp.jar
```

✅ Ideal for microservices and CLI tools with startup constraints.

---

## 9️⃣ Profiling and Monitoring Tools

| Tool                           | Purpose                          |
| ------------------------------ | -------------------------------- |
| **Java Flight Recorder (JFR)** | Continuous profiling             |
| **JVisualVM**                  | Memory, CPU profiling            |
| **Mission Control**            | Visual JFR analysis              |
| **async-profiler**             | Flame graphs for CPU/memory      |
| **JProfiler**                  | Commercial all-in-one tool       |
| **Arthas (Alibaba)**           | Runtime inspection in production |

### Example Command:

```bash
java -XX:StartFlightRecording=name=app_profile,filename=recording.jfr,duration=60s MyApp
```

---

## 🔟 Practical Performance Patterns

✅ **Cache frequently used computations**
✅ **Batch operations** instead of frequent small calls
✅ **Avoid reflection** in performance-critical paths
✅ **Reduce GC pressure** by reusing immutable objects
✅ **Profile before optimizing**
✅ **Measure memory footprint** using `-Xlog:gc*`

### Example: Caching Pattern with Caffeine

```java
Cache<String, User> cache = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .maximumSize(10_000)
    .build();

User user = cache.get("nehru", id -> userService.loadUser(id));
```

✅ Lightweight and high-performance alternative to EHCache.

---

## 11️⃣ Best Practices Summary

✅ Measure → Profile → Optimize → Measure again
✅ Tune GC only after profiling
✅ Use **ZGC or G1GC** for long-running apps
✅ Keep heap < 80% of available RAM
✅ Use **records** and **virtual threads** for modern design
✅ Prefer **immutable collections**
✅ Limit logging on hot paths
✅ Leverage **structured concurrency + JFR** for async observability

---

## 12️⃣ Official Resources and Reference Links

📘 **Official Documentation**

* [Java 21 Documentation – Oracle](https://docs.oracle.com/en/java/javase/21/)
* [JVM Tuning Guide (Oracle)](https://docs.oracle.com/en/java/javase/21/gctuning/)
* [OpenJDK Project Loom](https://openjdk.org/projects/loom/)
* [OpenJDK ZGC Docs](https://wiki.openjdk.org/display/zgc/Main)
* [JMH (Microbenchmark Harness)](https://openjdk.org/projects/code-tools/jmh/)
* [Java Flight Recorder and Mission Control](https://www.oracle.com/java/technologies/jdk-mission-control.html)

📊 **Tools**

* [VisualVM](https://visualvm.github.io/)
* [Async Profiler](https://github.com/jvm-profiling-tools/async-profiler)
* [Caffeine Cache](https://github.com/ben-manes/caffeine)
* [Arthas Diagnostic Tool](https://github.com/alibaba/arthas)

> ⚙️ Optimize intentionally — modern Java (21+) provides powerful runtime and monitoring tools.
> The best developers **don’t guess performance — they measure it**.

---

## 🧭 Next Topic

[16-jvm-monitoring-and-production-tuning.md → Monitoring Java 21 Applications in Production (JFR, Metrics, and Observability)](./16-jvm-monitoring-and-production-tuning.md)
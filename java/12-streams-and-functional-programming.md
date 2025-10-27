# 🔁 Java 21 — Streams and Functional Programming

Functional programming in Java has evolved from being “syntactic sugar” to a core paradigm that powers modern backend systems.  
The **Streams API** and **lambda expressions** enable concise, expressive, and parallelizable code — all while maintaining type safety and immutability.

This chapter dives deep into **functional patterns**, **stream pipelines**, and **Java 21 enhancements** designed for experienced Java developers.

---

## 🧭 Table of Contents

1. [Functional Programming in Java](#1-functional-programming-in-java)
2. [Lambdas and Method References](#2-lambdas-and-method-references)
3. [Functional Interfaces and the `java.util.function` Package](#3-functional-interfaces-and-the-javautilfunction-package)
4. [Stream Basics — What and Why](#4-stream-basics--what-and-why)
5. [Stream Creation](#5-stream-creation)
6. [Intermediate Operations (map, filter, flatMap)](#6-intermediate-operations-map-filter-flatmap)
7. [Terminal Operations (collect, reduce, forEach)](#7-terminal-operations-collect-reduce-foreach)
8. [Collectors and Grouping](#8-collectors-and-grouping)
9. [Parallel Streams and Performance](#9-parallel-streams-and-performance)
10. [Pattern Matching, Records, and Streams (Java 21)](#10-pattern-matching-records-and-streams-java-21)
11. [Streams with Virtual Threads (Java 21)](#11-streams-with-virtual-threads-java-21)
12. [Best Practices](#12-best-practices)
13. [Summary](#13-summary)

---

## 1️⃣ Functional Programming in Java

Functional programming (FP) emphasizes **immutable data**, **declarative transformations**, and **pure functions**.

Java introduced FP gradually:
- **Java 8** → Lambdas, Streams, Optional  
- **Java 11–17** → var, improved type inference  
- **Java 21** → Pattern matching, records, virtual threads for FP pipelines  

Key advantages:
✅ Concise code  
✅ No shared mutable state  
✅ Thread-safe, parallelizable design  

Example (imperative vs functional):

```java
// Imperative
List<String> names = new ArrayList<>();
for (String user : users)
    if (user.startsWith("A"))
        names.add(user.toUpperCase());

// Functional
List<String> names = users.stream()
    .filter(u -> u.startsWith("A"))
    .map(String::toUpperCase)
    .toList();
````

---

## 2️⃣ Lambdas and Method References

### 🔹 Lambda Syntax

```java
(parameter) -> expression
```

Example:

```java
Runnable r = () -> System.out.println("Running...");
```

### 🔹 Method References

Simplify calling existing methods:

```java
List<String> names = List.of("Nehru", "John", "Alice");
names.forEach(System.out::println);
```

### 🔹 Four Common Forms:

| Type                              | Example                 |
| --------------------------------- | ----------------------- |
| Static method                     | `ClassName::methodName` |
| Instance method of object         | `instance::method`      |
| Instance method of arbitrary type | `String::toLowerCase`   |
| Constructor                       | `MyClass::new`          |

---

## 3️⃣ Functional Interfaces and the `java.util.function` Package

A **functional interface** has exactly one abstract method — usable in lambdas.

### Common Functional Interfaces:

| Interface          | Method              | Example                                         |
| ------------------ | ------------------- | ----------------------------------------------- |
| `Predicate<T>`     | `boolean test(T t)` | `Predicate<String> isEmpty = s -> s.isEmpty();` |
| `Function<T,R>`    | `R apply(T t)`      | `Function<Integer,String> f = n -> "Num:" + n;` |
| `Consumer<T>`      | `void accept(T t)`  | `Consumer<String> c = System.out::println;`     |
| `Supplier<T>`      | `T get()`           | `Supplier<Double> rand = Math::random;`         |
| `UnaryOperator<T>` | `T apply(T t)`      | `UnaryOperator<Integer> square = n -> n * n;`   |

✅ Use **composition**:

```java
Function<Integer, Integer> doubleIt = n -> n * 2;
Function<Integer, Integer> squareIt = n -> n * n;
Function<Integer, Integer> pipeline = doubleIt.andThen(squareIt);
System.out.println(pipeline.apply(3)); // (3*2)^2 = 36
```

---

## 4️⃣ Stream Basics — What and Why

A **Stream** is a pipeline of operations on a data source.
It processes elements **lazily** and **immutably**.

Key features:

* Declarative processing
* Supports functional transformations
* Parallel execution support

> ⚙️ Streams **do not store data** — they operate on the source (collections, arrays, I/O, etc.)

---

## 5️⃣ Stream Creation

### 🔹 From Collections

```java
Stream<String> stream = List.of("A", "B", "C").stream();
```

### 🔹 From Arrays

```java
Stream<Integer> s = Arrays.stream(new Integer[]{1, 2, 3});
```

### 🔹 Using Static Methods

```java
Stream<Double> randoms = Stream.generate(Math::random).limit(5);
Stream<Integer> numbers = Stream.iterate(1, n -> n + 1).limit(5);
```

### 🔹 From Files

```java
try (Stream<String> lines = Files.lines(Path.of("data.txt"))) {
    lines.forEach(System.out::println);
}
```

---

## 6️⃣ Intermediate Operations (map, filter, flatMap)

| Operation    | Description               | Example                              |
| ------------ | ------------------------- | ------------------------------------ |
| `filter()`   | Select elements           | `.filter(n -> n > 10)`               |
| `map()`      | Transform elements        | `.map(String::toUpperCase)`          |
| `flatMap()`  | Flatten nested structures | `.flatMap(List::stream)`             |
| `distinct()` | Remove duplicates         | `.distinct()`                        |
| `sorted()`   | Sort elements             | `.sorted(Comparator.reverseOrder())` |
| `peek()`     | Debug or inspect          | `.peek(System.out::println)`         |

Example:

```java
List<String> names = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .map(Employee::getName)
    .sorted()
    .toList();
```

---

## 7️⃣ Terminal Operations (collect, reduce, forEach)

| Operation   | Description                         |
| ----------- | ----------------------------------- |
| `collect()` | Aggregate results into a collection |
| `reduce()`  | Combine into a single result        |
| `forEach()` | Consume the stream (void)           |

### Example: `collect()`

```java
List<String> highEarners = employees.stream()
    .filter(e -> e.getSalary() > 100000)
    .map(Employee::getName)
    .collect(Collectors.toList());
```

### Example: `reduce()`

```java
int total = Stream.of(10, 20, 30)
    .reduce(0, Integer::sum);
```

### Example: `forEach()`

```java
Stream.of("A", "B", "C").forEach(System.out::println);
```

---

## 8️⃣ Collectors and Grouping

```java
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.counting()));
```

### 🔹 Joining

```java
String names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", "));
```

### 🔹 Partitioning

```java
Map<Boolean, List<Employee>> partition = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 80000));
```

---

## 9️⃣ Parallel Streams and Performance

Parallel streams divide work across multiple threads.

```java
int total = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();
```

✅ Use when:

* Heavy CPU-bound computations
* Data source is large and thread-safe

❌ Avoid when:

* Stream involves I/O
* Small collections (overhead > benefit)

> 💡 For fine-grained parallelism, combine **virtual threads (Java 21)** with stream processing.

---

## 🔟 Pattern Matching, Records, and Streams (Java 21)

Combine records and pattern matching for **type-safe, concise transformations**.

```java
sealed interface Payment permits CreditCard, UPI {}
record CreditCard(String id, double amount) implements Payment {}
record UPI(String upiId, double amount) implements Payment {}

List<Payment> payments = List.of(new CreditCard("123", 5000), new UPI("xyz@upi", 2000));

double total = payments.stream()
    .mapToDouble(p -> switch (p) {
        case CreditCard c -> c.amount();
        case UPI u -> u.amount();
    })
    .sum();
```

✅ Clean, functional pattern matching
✅ Eliminates `instanceof` checks
✅ Perfect for event-driven and financial pipelines

---

## 11️⃣ Streams with Virtual Threads (Java 21)

Virtual threads (Project Loom) integrate naturally with functional patterns.

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Future<String>> results = Stream.of("Task1", "Task2", "Task3")
        .map(task -> executor.submit(() -> {
            Thread.sleep(1000);
            return task + " done by " + Thread.currentThread();
        }))
        .toList();

    results.forEach(r -> {
        try { System.out.println(r.get()); } catch (Exception e) { e.printStackTrace(); }
    });
}
```

✅ 1000s of lightweight threads
✅ Non-blocking stream pipelines
✅ Ideal for **concurrent stream-based workflows**

---

## 12️⃣ Best Practices

✅ Avoid modifying state inside streams
✅ Prefer **method references** for clarity
✅ Use `.parallel()` only for CPU-intensive workloads
✅ Keep **pipelines pure and short**
✅ Use `Collectors.toUnmodifiableList()` for immutability
✅ Handle exceptions using wrappers like:

```java
Function<T, R> safe(Function<T, R> f) {
    return t -> {
        try { return f.apply(t); }
        catch (Exception e) { throw new RuntimeException(e); }
    };
}
```

✅ Combine **Streams + Records + Sealed Types** for clean domain logic

---

## 13️⃣ Summary

In this chapter, you’ve learned:

* How to build **functional pipelines** with Streams
* Use of **lambdas**, **collectors**, and **pattern matching**
* Integration of **virtual threads** for scalable concurrency
* Modern **record + sealed type** functional workflows

> 🧭 **Next Topic:** [13-concurrency-and-multithreading.md → Advanced Concurrency in Java 21 (Virtual Threads, Executors, CompletableFuture)](./13-concurrency-and-multithreading.md)
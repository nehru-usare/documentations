# 🧠 Java 21 — Modern Best Practices and Design Principles

Java 21 brings together a decade of language evolution — from lambdas and streams to pattern matching, records, sealed classes, and virtual threads.

This guide summarizes **industry-grade best practices**, **modern patterns**, and **clean code principles** for writing maintainable, secure, and high-performance Java systems.

---

## 🧭 Table of Contents

1. [Language and Syntax Best Practices](#1-language-and-syntax-best-practices)
2. [Object-Oriented Design (OOP) Patterns](#2-object-oriented-design-oop-patterns)
3. [Functional Programming and Streams](#3-functional-programming-and-streams)
4. [Exception Handling and Error Design](#4-exception-handling-and-error-design)
5. [Records, Sealed Classes, and Pattern Matching](#5-records-sealed-classes-and-pattern-matching)
6. [Concurrency and Virtual Threads](#6-concurrency-and-virtual-threads)
7. [Memory and Resource Management](#7-memory-and-resource-management)
8. [Logging, Observability, and Monitoring](#8-logging-observability-and-monitoring)
9. [API and Service Design](#9-api-and-service-design)
10. [Testing and Code Quality](#10-testing-and-code-quality)
11. [Security and Defensive Programming](#11-security-and-defensive-programming)
12. [Performance and Deployment Practices](#12-performance-and-deployment-practices)
13. [Official References and Resources](#13-official-references-and-resources)
14. [Summary](#14-summary)

---

## 1️⃣ Language and Syntax Best Practices

✅ Use **`var`** for local type inference (readability, not laziness):
```java
var users = List.of("Alice", "Bob");
````

✅ Prefer **text blocks** for readability:

```java
String json = """
    {
      "name": "Nehru",
      "role": "Developer"
    }
    """;
```

✅ Use **`switch` expressions** with pattern matching:

```java
String role = switch (user) {
    case Admin a -> "Full Access";
    case Guest g -> "Limited Access";
    default -> "Unknown";
};
```

✅ Prefer **immutable collections**:

```java
List<String> names = List.of("A", "B", "C");
```

✅ Always use `Optional` for return values that may be empty, not `null`.

---

## 2️⃣ Object-Oriented Design (OOP) Patterns

✅ Follow **SOLID principles**:

* **S**ingle Responsibility
* **O**pen/Closed
* **L**iskov Substitution
* **I**nterface Segregation
* **D**ependency Inversion

✅ Use **composition over inheritance**:

```java
class Engine { void start() {} }
class Car { private final Engine engine = new Engine(); }
```

✅ Leverage **sealed classes** for controlled inheritance:

```java
sealed interface Shape permits Circle, Square {}
record Circle(double r) implements Shape {}
record Square(double s) implements Shape {}
```

✅ Avoid “God classes” — one class per core responsibility.

---

## 3️⃣ Functional Programming and Streams

✅ Keep stream pipelines **short and pure**:

```java
List<String> emails = users.stream()
    .filter(u -> u.isActive())
    .map(User::getEmail)
    .toList();
```

✅ Avoid side-effects inside `map()` or `filter()`.
✅ Use `Collectors.toUnmodifiableList()` for immutability.
✅ For complex transformations, extract lambdas into named functions.

> 💡 Stream != Loop replacement. Use only when transformations are **declarative**.

---

## 4️⃣ Exception Handling and Error Design

✅ Use **checked exceptions** for recoverable errors and **unchecked** for programming faults.
✅ Wrap third-party exceptions using custom domain types:

```java
class PaymentException extends RuntimeException {
    PaymentException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

✅ Never swallow exceptions — always log or rethrow.
✅ Add context to errors (`userId`, `operation`, etc.).
✅ Use `CompletableFuture.exceptionally()` for async errors.

---

## 5️⃣ Records, Sealed Classes, and Pattern Matching

### 🔹 Records

Use records for **DTOs and immutable data**:

```java
public record User(String name, int age) {}
```

✅ Auto-generates equals(), hashCode(), toString()
✅ Perfect for REST payloads and DB projections

### 🔹 Pattern Matching

```java
if (obj instanceof User u && u.age() > 18) {
    System.out.println(u.name());
}
```

### 🔹 Sealed Classes

Control class hierarchies explicitly:

```java
sealed interface Payment permits UPI, Card {}
record UPI(String id) implements Payment {}
record Card(String number) implements Payment {}
```

> 💡 Combine these for **type-safe domain models**.

---

## 6️⃣ Concurrency and Virtual Threads

✅ Use **Virtual Threads** for I/O-heavy workloads:

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> service.callExternalAPI());
}
```

✅ Combine with **Structured Concurrency**:

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var u = scope.fork(() -> fetchUser());
    var o = scope.fork(() -> fetchOrders());
    scope.join();
}
```

✅ Avoid shared mutable state.
✅ Use `ConcurrentHashMap`, `AtomicInteger`, or `Lock` where necessary.
✅ Always test under load (JMeter, Gatling).

---

## 7️⃣ Memory and Resource Management

✅ Use try-with-resources for all I/O:

```java
try (var reader = Files.newBufferedReader(Path.of("file.txt"))) {
    reader.lines().forEach(System.out::println);
}
```

✅ Avoid object churn inside loops.
✅ Reuse immutable objects.
✅ Tune GC only after profiling (not before).
✅ Monitor memory with JFR or VisualVM.

---

## 8️⃣ Logging, Observability, and Monitoring

✅ Use **SLF4J** with async appenders.
✅ Never log sensitive data.
✅ Use **structured JSON logs** for ELK or Grafana:

```json
{
  "timestamp": "2025-10-26T12:00:00Z",
  "service": "auth-service",
  "level": "INFO",
  "traceId": "abc-123",
  "message": "User login successful"
}
```

✅ Expose `/health`, `/metrics`, and `/prometheus` endpoints.
✅ Use **OpenTelemetry** for distributed tracing.

---

## 9️⃣ API and Service Design

✅ REST API principles:

* Use nouns, not verbs (`/users`, `/orders/{id}`)
* Use appropriate HTTP codes (200, 400, 404, 500)
* Support pagination & filtering
* Version your APIs (`/v1/users`)

✅ Prefer **record-based request/response models**.
✅ Validate inputs with Bean Validation:

```java
public record User(@NotBlank String name, @Email String email) {}
```

✅ Use **DTOs** — never expose entities directly.

---

## 🔟 Testing and Code Quality

✅ Use **JUnit 5** + **Mockito** / **AssertJ**.
✅ Write **unit**, **integration**, and **contract** tests.
✅ Maintain > 70% meaningful coverage.
✅ Use `@TestInstance(Lifecycle.PER_CLASS)` to optimize setup.
✅ Test concurrency with `@RepeatedTest`.

✅ Example:

```java
@Test
void shouldReturnValidUser() {
    var user = service.getUser("123");
    assertThat(user.name()).isEqualTo("Alice");
}
```

---

## 11️⃣ Security and Defensive Programming

✅ Always **sanitize inputs** (especially from web forms or APIs).
✅ Use parameterized SQL (never string concatenation).
✅ Secure sensitive data with environment variables.
✅ Hash passwords with **bcrypt / Argon2**.
✅ Implement rate limiting for APIs.
✅ Use **CORS**, **CSRF** protection, and **JWT** for auth.

> 💡 Security is not a feature — it’s a foundation.

---

## 12️⃣ Performance and Deployment Practices

✅ Use **G1GC** or **ZGC** in production.
✅ Enable container flags:

```bash
-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0
```

✅ Use **CDS (Class Data Sharing)** for faster startup.
✅ For cloud workloads:

* Prefer **Native Image (GraalVM)** for small services
* Use **Virtual Threads** for concurrency

✅ Profile regularly with:

* **JFR**
* **Mission Control**
* **Async Profiler**

> 🧩 Measure before optimizing. Always back changes with metrics.

---

## 13️⃣ Official References and Resources

📘 **Official Docs**

* [Java 21 Language Documentation](https://docs.oracle.com/en/java/javase/21/)
* [Java 21 Performance Guide](https://docs.oracle.com/en/java/javase/21/performance/)
* [Project Loom](https://openjdk.org/projects/loom/)
* [Project Amber (Pattern Matching, Records)](https://openjdk.org/projects/amber/)
* [Project Panama (Native Interop)](https://openjdk.org/projects/panama/)
* [GraalVM & Native Image Docs](https://www.graalvm.org/)

🧠 **Code Quality & Testing**

* [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
* [AssertJ Fluent Assertions](https://assertj.github.io/doc/)
* [Mockito Documentation](https://site.mockito.org/)

📊 **Observability & Tools**

* [Micrometer Metrics](https://micrometer.io/)
* [OpenTelemetry Java](https://opentelemetry.io/docs/instrumentation/java/)
* [JFR & Mission Control](https://www.oracle.com/java/technologies/jdk-mission-control.html)

---

## 14️⃣ Summary

You now have a **complete Java 21 engineering playbook**:

* Write **clean, immutable, and modular code**
* Handle concurrency with **Virtual Threads**
* Tune memory with **GC + JFR insight**
* Observe everything with **Prometheus + OpenTelemetry**
* Deploy cloud-native services using **GraalVM or JVM**

> 🧩 Modern Java 21 empowers you to build **scalable, fast, and clean** systems — not by magic, but by mastering the fundamentals.

> 🧭 **Next (optional advanced):**
> [21-java-architecture-and-design-patterns.md → Enterprise Java Design Patterns and System Architecture](./21-java-architecture-and-design-patterns.md)
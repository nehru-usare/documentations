# ⚠️ Java 21 — Exception Handling and Best Practices

Exception handling is one of the most misunderstood yet vital parts of Java.  
For developers with 4+ years of experience, mastering **exception design**, **error propagation**, and **fault isolation** is key to building reliable enterprise systems.

This guide explores both foundational and advanced exception-handling techniques in **Java 21** — designed for production-grade backend applications.

---

## 🧭 Table of Contents

1. [What Is an Exception?](#1-what-is-an-exception)
2. [Types of Exceptions](#2-types-of-exceptions)
3. [Checked vs Unchecked Exceptions](#3-checked-vs-unchecked-exceptions)
4. [Exception Hierarchy in Java](#4-exception-hierarchy-in-java)
5. [Try-Catch-Finally Structure](#5-try-catch-finally-structure)
6. [Try-With-Resources (Java 7+)](#6-try-with-resources-java-7)
7. [Multi-Catch and Rethrow (Java 21 Syntax)](#7-multi-catch-and-rethrow-java-21-syntax)
8. [Custom Exception Classes](#8-custom-exception-classes)
9. [Sealed and Record-Based Exceptions (Java 21)](#9-sealed-and-record-based-exceptions-java-21)
10. [Exception Translation Pattern (Spring Style)](#10-exception-translation-pattern-spring-style)
11. [Best Practices for Enterprise Exception Handling](#11-best-practices-for-enterprise-exception-handling)
12. [Summary](#12-summary)

---

## 1️⃣ What Is an Exception?

An **exception** is an event that disrupts the normal flow of a program during execution.  
It represents an **error condition or unexpected behavior** — handled gracefully using **try-catch** mechanisms.

Example:
```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.err.println("Cannot divide by zero");
}
````

> 💡 In Java, exceptions are objects — they inherit from `java.lang.Throwable`.

---

## 2️⃣ Types of Exceptions

| Category                | Example                                            | Description                 |
| ----------------------- | -------------------------------------------------- | --------------------------- |
| **Checked**             | `IOException`, `SQLException`                      | Checked at compile-time     |
| **Unchecked (Runtime)** | `NullPointerException`, `IllegalArgumentException` | Checked at runtime          |
| **Errors**              | `OutOfMemoryError`, `StackOverflowError`           | Critical JVM-level failures |

---

## 3️⃣ Checked vs Unchecked Exceptions

### 🔹 Checked Exceptions

* Must be **declared or handled** using `throws`
* Represent **recoverable** errors

Example:

```java
public void readFile() throws IOException {
    Files.readAllLines(Path.of("data.txt"));
}
```

✅ Use when:

* The caller **can recover** (e.g., retry a failed operation)
* You need to **force awareness** of the issue

---

### 🔹 Unchecked Exceptions

* Extend `RuntimeException`
* Not required to declare
* Represent **programming bugs or invalid state**

Example:

```java
if (user == null) throw new IllegalArgumentException("User cannot be null");
```

✅ Use for:

* Validation failures
* Illegal method arguments
* Programming logic errors

---

## 4️⃣ Exception Hierarchy in Java

```
Throwable
 ├── Exception
 │    ├── IOException
 │    ├── SQLException
 │    └── RuntimeException
 │         ├── NullPointerException
 │         ├── IllegalArgumentException
 │         └── IndexOutOfBoundsException
 └── Error
      ├── OutOfMemoryError
      └── StackOverflowError
```

> ⚙️ Never catch `Error` — those are **JVM-level issues** (e.g., memory, system crashes).

---

## 5️⃣ Try-Catch-Finally Structure

### 🔹 Basic Example

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.err.println("Divide by zero: " + e.getMessage());
} finally {
    System.out.println("Cleanup executed");
}
```

✅ **Finally block** is always executed (even after `return`), except during JVM shutdown.

---

## 6️⃣ Try-With-Resources (Java 7+)

Automatically closes resources implementing `AutoCloseable`.

### Example:

```java
try (BufferedReader br = Files.newBufferedReader(Path.of("data.txt"))) {
    System.out.println(br.readLine());
} catch (IOException e) {
    e.printStackTrace();
}
```

✅ No need for manual `finally` cleanup
✅ Closes resources in reverse order of opening

> 💡 As of **Java 9+**, resources can be declared outside `try` and still auto-closed.

---

## 7️⃣ Multi-Catch and Rethrow (Java 21 Syntax)

### 🔹 Multi-Catch

```java
try {
    process();
} catch (IOException | SQLException e) {
    System.err.println("I/O or SQL error: " + e.getMessage());
}
```

### 🔹 Rethrow with Type Inference (Java 7+)

The compiler now tracks rethrown exceptions precisely:

```java
try {
    riskyOperation();
} catch (IOException e) {
    throw e; // type-safe rethrow
}
```

---

## 8️⃣ Custom Exception Classes

Custom exceptions improve **readability**, **maintainability**, and **error context**.

### Example: Business Exception

```java
public class InvalidOrderException extends RuntimeException {
    public InvalidOrderException(String message) {
        super(message);
    }
}
```

Usage:

```java
if (order.getItems().isEmpty()) {
    throw new InvalidOrderException("Order must have at least one item");
}
```

✅ Best Practices:

* Extend `RuntimeException` unless the caller must handle it.
* Always include **meaningful error messages**.
* Avoid overly deep exception hierarchies.

---

## 9️⃣ Sealed and Record-Based Exceptions (Java 21)

### 🔹 Sealed Exceptions

Encapsulate a fixed hierarchy of exception types.

```java
public sealed class PaymentException extends Exception
        permits InsufficientFundsException, PaymentTimeoutException {}

public final class InsufficientFundsException extends PaymentException {}
public final class PaymentTimeoutException extends PaymentException {}
```

✅ Helps prevent uncontrolled subclassing
✅ Works perfectly with **pattern matching** in `switch`

---

### 🔹 Record-Based Exception (Java 21)

Records can be used for **immutable, data-rich exception payloads**.

```java
public record ValidationException(String field, String message)
        implements Supplier<RuntimeException> {
    @Override
    public RuntimeException get() {
        return new IllegalArgumentException("Invalid " + field + ": " + message);
    }
}
```

Usage:

```java
throw new ValidationException("email", "Invalid format").get();
```

✅ Compact, immutable, and expressive exception structure
✅ Ideal for validation and service layers

---

## 🔟 Exception Translation Pattern (Spring Style)

This is a critical **enterprise design pattern** — converting low-level exceptions into domain-specific ones.

Example:

```java
try {
    jdbcTemplate.query("SELECT * FROM users");
} catch (SQLException e) {
    throw new DataAccessException("Database error", e);
}
```

> 🧩 Spring uses this extensively (`@Repository` annotated classes).
> It decouples persistence frameworks from business logic layers.

---

## 11️⃣ Best Practices for Enterprise Exception Handling

✅ **Never swallow exceptions** (e.g., empty catch blocks)
✅ **Log meaningful context**, not stack traces blindly
✅ **Wrap external exceptions** in custom domain exceptions
✅ **Don’t overuse checked exceptions** — prefer runtime for business logic
✅ **Include cause** in chained exceptions (`new Exception(msg, cause)`)
✅ **Centralize exception handling** in frameworks like Spring Boot’s `@ControllerAdvice`
✅ **Differentiate client vs server errors** in APIs
✅ For concurrency, handle exceptions in **CompletableFuture.exceptionally()**

Example (Spring Controller Advice):

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(InvalidOrderException.class)
    public ResponseEntity<String> handleInvalidOrder(InvalidOrderException ex) {
        return ResponseEntity.badRequest().body(ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneric(Exception ex) {
        return ResponseEntity.internalServerError().body("Unexpected error");
    }
}
```

---

## 12️⃣ Summary

✅ You’ve learned:

* How to design a **robust exception hierarchy**
* Checked vs unchecked trade-offs
* Try-with-resources, multi-catch, and rethrow patterns
* Modern **sealed and record-based exceptions**
* Enterprise best practices for **layered architecture handling**

💡 **Mastering exception design** transforms your Java applications from *functional* to *production-ready and maintainable*.

> 🧭 **Next Topic:** [11-generics-and-enums.md → Java 21 Generics and Enums (Type Safety, Records, and Advanced Usage)](./11-generics-and-enums.md)
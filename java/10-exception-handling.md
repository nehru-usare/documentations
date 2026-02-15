# ⚠️ Exception Handling: Building Resilient Enterprise Systems

> **Document Level:** Architect (15+ years experience)  
> **Focus:** Stack Trace Costs, Checked vs. Unchecked Trade-offs, and Exception Translation

---

## 🏗️ Layer 1: The Core - Performance Costs

Most developers use exceptions for control flow. **This is an anti-pattern.**

### 1. The Cost of `fillInStackTrace()`
Creating an exception object is cheap. However, the `Throwable` constructor calls `fillInStackTrace()`, which is a **Native JVM call**.
- **The Task**: The JVM must walk the execution stack, find the method names, file lines, and class pointers.
- **Architect Insight**: In high-throughput systems, throwing 10,000 exceptions per second can consume 50% of your CPU just on stack-walking.

### 2. Optimization: Pre-allocated Exceptions
If you MUST use an exception for a frequent, expected logic jump, pre-allocate a static exception and override `fillInStackTrace()` to return `this` (or do nothing).

---

## 🛠️ Layer 2: Structural Strategy - Checked vs. Unchecked

This is the most debated topic in Java history.

### 1. Checked Exceptions (The "Good Intent")
- **Rationale**: Forces the developer to handle external failures (Network, I/O).
- **Reality**: Leads to "Catch and Ignore" or wrapping in `RuntimeException` anyway.

### 2. Unchecked Exceptions (The "Modern Standard")
Modern frameworks (Spring, Hibernate) convert all checked exceptions into unchecked ones.
- **Architect Design**: Use **Unchecked Exceptions** for business logic and flow-control. Use **Checked Exceptions** only when the caller **REALLY** has a specific, probable way to recover from the error.

---

## 🚀 Layer 3: The Exception Translation Pattern

Don't let internal implementation details leak out of your service layers.

### 1. Decoupling the Tiers
If your `UserRepository` throws a `SQLException`, your `UserService` should NOT re-throw it.
- **Pattern**: Catch `SQLException` and throw a `UserStorageException` (Domain Exception).
- **Result**: If you swap SQL for MongoDB later, your business logic layer remains unchanged.

### 2. Chaining
Always preserve the **Root Cause**.
```java
try { ... } 
catch (SpecificException e) {
    throw new DomainException("Error processing user", e); // Pass 'e' as cause
}
```

---

## 📜 Layer 4: Try-With-Resources Internals

Introduced in Java 7, it's more than just syntactic sugar.

### 1. Suppressed Exceptions
If the `try` block throws an exception, and the `close()` method also throws one, what happens? 
- **Legacy**: The `close()` exception would mask (hide) the original error.
- **Java 7+**: The original exception is thrown, and the `close()` error is added as a **"Suppressed Exception"**. You can retrieve it via `e.getSuppressed()`.

---

## 🚦 Layer 5: Modern Patterns - Sealed Hierarchies

Use Java 17+ **Sealed Classes** to model your error hierarchy.
```java
public sealed class PaymentError extends RuntimeException 
    permits InsufficientFunds, Timeout, ProviderDown {}
```
**Architect Benefit**: When handling these, the compiler can check if you've missed any specific error type in your `switch` or `if-else` blocks.

---

## 🧭 Interview Prep & Architect Scenarios

### Q: Why shouldn't you catch `Throwable` or `Error`?
**A**: `Error` represents critical JVM failures (OMM, StackOverflow). If you catch them, you might prevent the JVM from shutting down safely or leave the application in a "Zombie" state. Only catch `Exception`.

### Q: How do you handle exceptions in an Asynchronous `CompletableFuture`?
**A**: Use `.exceptionally()` or `.handle()`. If an exception is not handled, it will be "swallowed" until the future is joined, making it notoriously hard to debug.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [09-packages-and-access-modifiers.md](./09-packages-and-access-modifiers.md) | Packages |
| ⏩ **Next** | [11-generics-and-enums.md](./11-generics-and-enums.md) | Generics & Enums |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026
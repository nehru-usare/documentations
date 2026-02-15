# 🧠 Java 21 Best Practices: The Modern Engineering Standard

> **Document Level:** Architect (15+ years experience)  
> **Focus:** Data-Oriented Programming, Record Evolution, Pattern Matching, and Modern Clean Code

---

## 🏗️ Layer 1: The Tactical Shift - Data-Oriented Programming

Java 21 has finalized a new paradigm: **Data-Oriented Programming (DOP)**. 

### 1. The Core Principles of DOP
1.  **Separate Data from Logic**: Use Records for data, and static methods/pattern matching for logic.
2.  **Immutability is Default**: Records are natively shallowly-immutable.
3.  **Model Logic as Exhaustive Handlers**: Use `switch` over `sealed` interfaces.

### 2. Architect Scenario: The "Entity" vs. the "DTO"
- **Entity (Mutable)**: Use standard classes for JPA/Hibernate entities where identity is tracked over time.
- **DTO (Immutable)**: Use **Records** for every message that travels across a service boundary (API, Kafka, Queue).

---

## 🛠️ Layer 2: API Design & Naming for Senior Leads

### 1. Fluent APIs vs. Verbose APIs
- **Modern Rule**: Favor **Fluent Builders** for complex configuration objects.
- **Naming**: Use "Contextual" naming. `user.activate()` is better than `userService.activateUser(user)`.

### 2. The `Optional` Architecture
- **Rule**: `Optional` is for **Return Types only**.
- **The Why**: It alerts the caller that they MUST handle the missing case. Using it in fields or parameters causes unnecessary heap bloat and breaks serialization.

---

## 🚀 Layer 3: Modern Concurrency Best Practices

### 1. Virtual Threads (Project Loom)
- **Rule**: **Don't pool Virtual Threads**. They are cheap and abundant. If you need to limit concurrency to a database, use a `Semaphore` instead of a small thread pool.
- **Locking**: Replace `synchronized` with `ReentrantLock` where possible to avoid the "Carrier Thread Pinning" issue.

### 2. Structured Concurrency (Preview)
- **Concept**: If you spawn 3 background tasks, they should all be cancelled if one fails. No more "Orphaned Threads".

---

## 📜 Layer 4: Exception & Error Handling Strategy

### 1. Sealed Error Hierarchies
Instead of broad `Exception` catches, use sealed types to define precise error states.
```java
public sealed interface OrderError 
    permits PaymentFailed, OutOfStock, ShippingAddressInvalid {}
```
- **Architect Benefit**: This forces the UI developer to handle all three cases explicitly.

### 2. The Performance Cost of Stack Traces
- **Recommendation**: For "Business Exceptions" that happen frequently (e.g., `InvalidCredentials`), override `fillInStackTrace()` to be empty to save CPU cycles.

---

## 🏁 Layer 5: Clean Code in the record/switch era

### 1. Keep Methods Small
- **The JIT Reason**: Small methods (< 325 bytes) are more likely to be inlined, leading to an "Inlining Cascade" that makes your app exponentially faster.

### 2. Favor `var` for Technical Boilerplate
- **Good**: `var connection = database.getConnection();`
- **Bad**: `var result = service.process();` (What is result? A boolean? A list? A DTO?)

---

## 🧭 Interview Prep & Architect Scenarios

### Q: Why is "Composition over Inheritance" even more important in Java 21?
**A**: Because of **Sealed Classes**. Designers can now explicitly prevent random inheritance, forcing developers to use composition or a very specific, controlled hierarchy. This makes the system more predictable and easier to refactor.

### Q: How do you enforce "Clean Architecture" boundaries in a Java project?
**A**: I use **ArchUnit**. It allows me to write tests like: `classes().that().resideInAPackage("..domain..").should().notDependOnAnyClassesThat().resideInAPackage("..infrastructure..")`. This automates the enforcement of my architectural vision.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [19-cloud-native-java-performance.md](./19-cloud-native-java-performance.md) | Cloud Native |
| ⏩ **Next** | [21-java-architecture-and-design-patterns.md](./21-java-architecture-and-design-patterns.md) | Design Patterns |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026
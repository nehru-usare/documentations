# New Language Features: Java 17 to 21+

> **Part 5: Modern Java**  
> **Level:** Principal Engineer  
> **Status:** Recorded

---

## 0. Learning Objectives

*   **Developer**: Converting POJOs to Records.
*   **Senior**: Using Switch Expressions for cleaner logic.
*   **Architect**: Modeling Domain Types with Sealed Classes.

---

## 1. Records (Java 16)

**Data Carrier**. Immutable. Transparent.
```java
public record User(String name, int age) {}
```
*   **Generates**: Constructor, getters (`name()`, not `getName()`), `equals`, `hashCode`, `toString`.
*   **Constraint**: Cannot extend other classes (implicitly extends `Record`). Can implement interfaces.
*   **Use Case**: DTOs, Map Keys.

---

## 2. Sealed Classes (Java 17)

**Restricted Hierarchy**.
```java
public sealed interface Shape permits Circle, Square {}
public final class Circle implements Shape {}
public final class Square implements Shape {}
```
*   **Goal**: The compiler knows exactly *all* possible subtypes.
*   **Safety**: Allows exhaustive Pattern Matching.

---

## 3. Pattern Matching (Java 21)

### 3.1 instanceof
```java
if (obj instanceof String s) { // checks type AND casts to 's'
    System.out.println(s.length());
}
```

### 3.2 Switch Pattern Matching
```java
String result = switch (shape) {
    case Circle c -> "Area: " + (Math.PI * c.radius() * c.radius());
    case Square s -> "Area: " + (s.side() * s.side());
    // No default needed! Compiler knows Shape is sealed.
};
```

---

## 4. Summary & Architect Takeaways

1.  **Upgrade to 21**: It is the new LTS. Records + Pattern Matching = Less Boilerplate, More Safety.
2.  **Switch Expressions**: Use `var x = switch(...)`. It forces you to cover all cases.
3.  **Sealed Classes**: Use them for Domain Modeling (State enumerations).

---
*End of Part 5. The Core Java Handbook is Complete.*

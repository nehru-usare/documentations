# Builder Pattern

> **Part 2: Creational Patterns**  
> **Difficulty:** ⭐⭐ (Junior)  
> **Status:** Ubiquitous (Thanks to Lombok)

---

## 0. Learning Objectives

*   **Beginner**: Fix the "Telescoping Constructor" problem.
*   **Developer**: Use Lombok `@Builder` effectively.
*   **Architect**: Enforce immutability using the Builder pattern.

---

## 1. Problem Statement

### The Telescoping Constructor
Imagine a `User` class with 10 fields (First Name, Last Name, Age, Phone, Address...).
*   **Constructor 1**: `new User(first, last)`
*   **Constructor 2**: `new User(first, last, age)`
*   **Constructor 3**: `new User(first, last, age, phone)` ...
*   **Usage**: `new User("John", "Doe", 0, null, "123 Main St", null)` -> **Unreadable**. "What is `0`? What is `null`?"

### What happens if we don't use it?
1.  **Bug Risk**: Swapping two `String` parameters (e.g., passing Phone into Address) is a common compile-time-safe bug.
2.  **Setter Spaghetti**: You create an object, then call 10 setters. The object is in an *inconsistent state* until the last setter is called.

---

## 2. Real-World Analogy

**Subway Sandwich**
*   You don't just say "Give me a sandwich".
*   You say: "Start with Italian Bread" -> "Add Turkey" -> "Add Cheese" -> "No Onions" -> "Add Mayo".
*   You construct it step-by-step. The result is a `Sandwich`.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Separate the construction of a complex object from its representation so that the same construction process can create different representations.

### Key Feature: Fluent Interface
Method Chaining: `.setName("Gui").setAge(20).build()`.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. Manual Implementation (The "GoF" way)
```java
public class User {
    // All final! Immutable.
    private final String firstName;
    private final String lastName;
    private final int age;

    // Private Constructor: Only Builder can call it.
    private User(UserBuilder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
    }

    // Static Inner Class
    public static class UserBuilder {
        private final String firstName; // Required
        private final String lastName; // Required
        private int age; // Optional

        public UserBuilder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this; // Return self for chaining
        }

        public User build() {
            // Validation Logic here!
            if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
            return new User(this);
        }
    }
}
```

### 2. Client Usage
```java
User user = new User.UserBuilder("James", "Bond")
                .age(40)
                .build();
```

---

## 6. Spring Boot / Modern Implementation (Lombok)

In 2025, **NOBODY** writes manual Builders. We use Lombok.

```java
import lombok.Builder;
import lombok.Getter;

@Getter
@Builder
public class User {
    private String firstName;
    private String lastName;
    private int age;
}
```

**Usage**:
```java
User user = User.builder()
                .firstName("James")
                .age(40)
                .build();
```

---

## 7. Internal Mechanics (Architect Level 🔴)

### Memory Overhead
*   Builder creates **two** objects: The `Builder` object and the `Product` object.
*   For high-performance loops (Millions of objects), this Double-Allocation *might* matter (GC Pressure).
*   **Optimization**: For simple DTOs, a Constructor is faster. But readability > micro-optimization.

### Thread Safety
*   The `Builder` instance is **NOT** thread-safe (it has mutable state).
*   The `Product` (User) **IS** thread-safe (if designed immutable).
*   **Rule**: Do not share a Builder across threads. Create, Build, Discard.

---

## 8. Advantages

1.  **Readability**: Named parameters (`.age(10)`) is better than positional arguments (`10`).
2.  **Immutability**: You can force the Object to be immutable (no Setters).
3.  **Validation**: The `build()` method is the perfect place to check invariants ("If Age < 18, Phone is required").

---

## 9. Disadvantages

1.  **Verbosity**: Requires a lot of boilerplate (if not using Lombok).
2.  **Duplication**: Fields are repeated in Class and Builder.

---

## 10. When NOT to Use

1.  **Simple Objects**: `new Point(x, y)` is better than `Point.builder().x(10).y(20).build()`.
2.  **Mapping Libraries**: Frameworks like MapStruct or Jackson work better with Setters or Constructors.

---

## 14. Interview Questions

### Basic
1.  **What problem does Builder solve?** (Telescoping Constructor / Too many parameters).
2.  **Can we use Builder for immutable classes?** (Yes, it's the *best* way to create immutable classes).

### Intermediate
3.  **What is a "Director" in Builder pattern?** (An optional class that executes the building steps in a specific order).
4.  **Difference between Builder and Factory?** (Factory returns an object in one shot. Builder constructs it step-by-step).

### Advanced
5.  **How to enforce "Required" parameters in Builder?** (Put them in the Builder's Constructor, or use a multi-stage builder interface chain).
6.  **Does Lombok `@Builder` support validation?** (Not natively. You have to manually override the `build()` method in a stub builder to add validation).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: SQL Query Builder.
    *   *Design*: `query.select("*").from("users").where("id = 1").build()`.
    *   *Why*: SQL syntax is complex. Builder prevents syntax errors.

2.  **Scenario**: HTTP Request (Rest Template / WebClient).
    *   *Design*: `client.post().uri("/api").header("Auth", token).body(data).retrieve()`.

3.  **Scenario**: Configuration Object with 50 options (Timeouts, URLs, Retries).
    *   *Design*: Default all options in Builder. Let user override only what they need.

---

## 16. Summary & Architect Takeaways

*   **Lombok is standard**. Use it.
*   **Immutability**: The biggest win is that you don't expose `setters` on your Domain Entities. 
*   **Validation**: Always validate inside `build()`. Make it impossible to create an invalid object.

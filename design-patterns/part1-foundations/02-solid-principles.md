# SOLID Principles

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐ (Junior)  
> **Status:** The Bible of OOP

---

## 0. Learning Objectives

*   **Beginner**: Memorize the acronym.
*   **Developer**: Identify violations in code review ("This class does too much").
*   **Architect**: Design systems that adhere to OCP (Plugin Architecture).

---

## 1. Problem Statement

### The Rotting Code
Code starts clean. Then:
1.  **Rigidity**: Hard to change. Change one thing, break 10 others.
2.  **Fragility**: Breaks in unexpected places.
3.  **Immobility**: Can't reuse code because it drags 10 dependencies with it.

**SOLID** prevents rot.

---

## 2. Real-World Analogy

*   **SRP**: A Swiss Army Knife (does everything poorly) vs A Chef's Knife (does one thing perfectly).
*   **OCP**: A charging socket. You can plug in *any* device (Open for extension) without breaking the wall (Closed for modification).

---

## 3. Core Concepts (🟢 Beginner Level)

1.  **S - Single Responsibility Principle (SRP)**: A class should have one, and only one, reason to change.
2.  **O - Open/Closed Principle (OCP)**: Open for extension, Closed for modification.
3.  **L - Liskov Substitution Principle (LSP)**: Subtypes must be substitutable for their base types.
4.  **I - Interface Segregation Principle (ISP)**: Many client-specific interfaces are better than one general-purpose interface.
5.  **D - Dependency Inversion Principle (DIP)**: Depend upon abstractions, not concretions.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. SRP Violation (Bad)
```java
class UserManager {
    void addUser(String user) { ... } // Logic
    void log(String msg) { ... } // Logging
    void sendEmail(String user) { ... } // Notification
}
// If Email logic changes, UserManager changes.
// If Log format changes, UserManager changes.
```

### 1. SRP Fixed (Good)
```java
class UserManager {
    private final EmailService emailer;
    private final Logger logger;
    // Logic only. Delegates others.
}
```

### 2. OCP Violation (Bad)
```java
class Payment {
    void pay(String type) {
        if (type.equals("PAYPAL")) { ... }
        else if (type.equals("CC")) { ... }
    }
}
// Adding CRYPTO requires modifying 'pay' method. Risk of breaking existing code.
```

### 2. OCP Fixed (Good)
```java
interface PaymentStrategy { void pay(); }
class PayPal implements PaymentStrategy { ... }
class Crypto implements PaymentStrategy { ... }
// Adding CRYPTO = creating new class. No existing code touched.
```

---

## 6. Spring Boot Implementation

*   **DIP**: The core of Spring `IOC`.
    *   `@Autowired Service service` -> Spring injects the *Implementation*. The Controller only knows the Interface (ideally).
*   **ISP**: Spring Data Repositories.
    *   `CrudRepository` (Basic).
    *   `PagingAndSortingRepository` (Extended).
    *   You pick what you need.

---

## 7. Internal Mechanics (Architect Level 🔴)

### The Cost of SOLID
*   **More Classes**: Splitting 1 God Class into 10 small classes increases file count.
*   **Indirection**: Debugging OCP code requires jumping through Interfaces.

### Performance
*   **Virtual Calls**: Java `invokevirtual` (Interface calls) is slightly slower than `invokestatic`. In 99% of apps, this is irrelevant compared to I/O latency.

---

## 8. Advantages

1.  **Testability**: Small classes are easy to Unit Test.
2.  **Maintainability**: Bugs are isolated.
3.  **Parallel Work**: 5 devs can work on 5 different Strategy implementations without conflicting.

---

## 10. When NOT to Use

1.  **DIP**: If a class *only* ever has one implementation (e.g., `StringHelper`), don't make an Interface for it.
2.  **SRP**: Don't split a class if the logic is tightly coupled. "Cohesion" is as important as "Decoupling".

---

## 12. Refactoring Example

### The "God Interface" (ISP Violation)

```java
interface Worker {
    void code();
    void test();
    void eat();
    void sleep();
}

class Robot implements Worker {
    void code() { ... }
    void eat() { throw new RuntimeException("Robots don't eat"); } // LSP Violation mostly
}
```

### Refactored

```java
interface Coder { void code(); }
interface Eater { void eat(); }

class Robot implements Coder { ... } // Clean.
class Human implements Coder, Eater { ... }
```

---

## 14. Interview Questions

### Basic
1.  **What does SOLID stand for?** (Acronym).
2.  **Why is SRP important?** (Reduces merge conflicts, limits impact of changes).
3.  **What is the difference between Interface and Abstract Class in OCP terms?** (Interface allows multiple inheritance of type, Abstract allows sharing code).

### Intermediate
4.  **If I throw `NotImplementedException` in an overridden method, which principle did I break?** (LSP - Liskov Substitution).
5.  **How does Dependency Injection relate to DIP?** (DI is the mechanism to achieve DIP).

### Advanced
6.  **Can adhering to OCP lead to overengineering?** (Yes, creating factories/strategies for simple `if-else` is bad).
7.  **Is SRP about "One Method" or "One Concept"?** (One Concept/Actor. A User class can have `getName` and `changeName`, that's fine).
8.  **How to refactor a switch-statement based legacy code to SOLID?** (Replace Conditional with Polymorphism - Strategy Pattern).

---

## 16. Summary & Architect Takeaways

*   **SOLID is a spectrum**. You aim for it, but heavily optimized code (Game Engine core loop) might violate it for performance. Start SOLID, optimize later.
*   **DIP is the most powerful**: It decouples your business logic from the Framework/Database.

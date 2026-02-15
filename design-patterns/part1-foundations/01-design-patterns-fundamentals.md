# Design Patterns Fundamentals

> **Part 1: Foundations**  
> **Difficulty:** ⭐ (Beginner)  
> **Status:** Mandatory Reading

---

## 0. Learning Objectives

*   **Beginner**: Understand what a "Design Pattern" actually is (it's not code, it's a concept).
*   **Developer**: Learn the 3 categories (Creational, Structural, Behavioral).
*   **Architect**: Understand that Patterns are **Solutions to Repeating Problems** in a specific **Context**.

---

## 1. Problem Statement

### The Communication Gap
*   **Junior Dev**: "I created a class that manages the distinct instance of the database connection and ensures no other instance can be created." (20 words).
*   **Senior Dev**: "I used a Singleton." (3 words).

Without Design Patterns, developers waste time explaining *basic structures* instead of focusing on *business logic*. Code becomes ad-hoc, with 5 different ways to instantiate objects in the same project.

### What happens if we don't use them?
1.  **Spaghetti Code**: Zero structure.
2.  **Reinventing the Wheel**: Solving solved problems (poorly).
3.  **High Cognitive Load**: New developers take weeks to understand the custom architecture.

---

## 2. Real-World Analogy

**Tailoring vs. Templates**
*   **Ad-Hoc Code**: A tailor measuring you from scratch every time you need a shirt. Expensive, slow, unique.
*   **Design Pattern**: Buying a "Size M" shirt. It fits 90% of people perfectly. It's a proven template.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
A **Design Pattern** is a general, reusable solution to a commonly occurring problem within a given context.

### The Formula
**Pattern = Problem + Solution + Context**
*   *Problem*: "I need to notify multiple objects when state changes."
*   *Context*: "The objects are decoupled."
*   *Solution*: **Observer Pattern**.

### Classification (Gang of Four)
1.  **Creational**: How objects are created. (e.g., Singleton, Factory).
2.  **Structural**: How objects are composed. (e.g., Adapter, Decorator).
3.  **Behavioral**: How objects communicate. (e.g., Strategy, Observer).

---

## 4. UML-Style Structure

*   Not applicable for this intro chapter.
*   *Concept*: Patterns rely on **Polymorphism** (Interfaces) and **Composition** ("Has-a") over Inheritance ("Is-a").

---

## 5. Java Implementation (Developer Level 🟡)

### The "Pattern" of coding without Patterns (Bad)

```java
public class Service {
    public void doWork() {
        // Hard dependency
        Database db = new PostgresDatabase(); 
        db.save();
    }
}
```

### The "Pattern" approach (Dependency Injection - Strategy)

```java
public class Service {
    private final Database db;

    // We don't care *which* DB it is. That's the pattern.
    public Service(Database db) {
        this.db = db;
    }
}
```

---

## 6. Spring Boot Implementation

Spring Framework is essentially **Design Patterns: The Framework**.
1.  **Singleton**: All Beans are Singletons by default.
2.  **Factory**: `BeanFactory` creates beans.
3.  **Proxy**: `@Transactional`, AOP.
4.  **Template**: `JdbcTemplate`, `RestTemplate`.
5.  **Observer**: `ApplicationEventPublisher`.

---

## 7. Internal Mechanics (Architect Level 🔴)

### Pattern vs Library
*   **Library**: Code you use (Log4j).
*   **Pattern**: A mental model you implement.

### The "Golden Hammer" Risk
"If the only tool you have is a hammer, everything looks like a nail."
*   **Junior Architects** often overuse patterns.
*   **Result**: Overengineering. ("Hello World" using Abstract Factory and Bridge).

---

## 8. Advantages

1.  **Common Vocabulary**: "Use Strategy here" is instant communication.
2.  **Best Practices**: These solutions have survived 30 years of engineering. They work.
3.  **Refactoring**: Patterns make code easier to change (Open/Closed Principle).

---

## 9. Disadvantages

1.  **Complexity**: Patterns add classes (Factory, Interface, Impl).
2.  **Learning Curve**: Requires abstract thinking.
3.  **Performance Check**: More indirection *can* mean slower execution (usually negligible, but exists).

---

## 10. When NOT to Use

1.  **Simple logic**: Don't use Strategy for a simple `if (x) else (y)`.
2.  **Prototyping**: Just get it working. Refactor to patterns later.
3.  **YAGNI (You Ain't Gonna Need It)**: Don't implement Visitor pattern "just in case" you need to traverse a graph next year.

---

## 11. Common Mistakes

1.  **Pattern Obsession**: Trying to fit 5 patterns into one class.
2.  **Wrong Context**: Using Singleton for a Database Connection (good) vs State (bad - concurrency issues).
3.  **Copy-Paste**: Implementing the robust "Gang of Four" C++ version in Java, ignoring Java's native features (Streams, Lambdas).

---

## 14. Interview Questions

### Basic
1.  **What are the 3 types of Design Patterns?** (Creational, Structural, Behavioral).
2.  **What is the Gang of Four (GoF)?** (The authors who codified the 23 original patterns).
3.  **Why favor Composition over Inheritance?** (Inheritance is rigid/fragile. Composition is flexible).
4.  **Give an example of a Creational Pattern.** (Factory, Builder).
5.  **Give an example of a Behavioral Pattern.** (Observer, Strategy).

### Intermediate
6.  **How does Spring use the Proxy pattern?** (For AOP, Transactions, and Lazy Loading).
7.  **What is the difference between Strategy and State?** (Intent. Strategy = How to do something. State = What state I am in).
8.  **Why is Singleton considered an Anti-Pattern sometimes?** (Global state, hard to test, hides dependencies).
9.  **What is Dependency Injection?** (An implementation of Inversion of Control - Strategy Pattern).
10. **Explain Actuator in terms of patterns.** (Observer/Visitor/Command mix).

### Advanced
11. **How do Patterns affect CPU caching?** (More indirection/pointers = more cache misses. Monolith code is sometimes faster).
12. **Compare Reactor Pattern (Netty) vs Thread-per-Request.** (Async Event Loop vs Blocking).
13. **How does the Flightweight pattern help with memory?** (Sharing common state).
14. **Is MVC a design pattern?** (It's an Architectural Pattern, a compound of Observer, Strategy, and Composite).
15. **How to refactor a "God Class"?** (Facade to hide it, then extract Strategies/States).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: You have a `PaymentProcessor` with a huge `switch(paymentType)` statement.
    *   *Solution*: **Strategy Pattern**. (Replace `switch` with `Map<Type, PaymentStrategy>`).

2.  **Scenario**: You need to process a heavy object, but only if the user actually asks for its data.
    *   *Solution*: **Proxy Pattern** (Virtual Proxy / Lazy Loading).

3.  **Scenario**: You need to copy an object but it has private fields.
    *   *Solution*: **Prototype Pattern** (or Copy Constructor).

4.  **Scenario**: You call a 3rd party API that is flaky.
    *   *Solution*: **Circuit Breaker / Retry Pattern**.

5.  **Scenario**: You need to parse a complex XML tree structure.
    *   *Solution*: **Composite Pattern** (Treat Leaf and Node same way) + **Visitor Pattern**.

6.  **Scenario**: You want to decouple the sender of a request from the receiver.
    *   *Solution*: **Command Pattern** (or Message Queue).

---

## 16. Summary & Architect Takeaways

*   **Patterns are Tools, not Rules**.
*   **Context is King**. The same pattern can be a lifesaver or a disaster depending on the problem.
*   **Start Simple**. Refactor TO patterns, don't start WITH them (unless it's a known problem like Config -> Builder).

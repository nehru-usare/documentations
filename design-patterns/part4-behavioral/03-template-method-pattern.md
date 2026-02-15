# Template Method Pattern

> **Part 4: Behavioral Patterns**  
> **Difficulty:** ⭐ (Beginner)  
> **Status:** The Abstract Framework

---

## 0. Learning Objectives

*   **Beginner**: Understand why `Abstract Class` exists.
*   **Developer**: Stop copy-pasting code in Service classes.
*   **Architect**: Define a rigid process (Step 1 -> Step 2 -> Step 3) while allowing flexibility in Step 2.

---

## 1. Problem Statement

### The Copy-Paste Problem
You have `CsvReportGenerator` and `PdfReportGenerator`.
1.  Connect to DB.
2.  Run Query.
3.  Format Data (Different for CSV/PDF).
4.  Save File.
5.  Close DB.

*   **Issue**: Steps 1, 2, 4, 5 are identical. Only Step 3 changes.
*   **Solution**: Put steps 1, 2, 4, 5 in a Parent Class. Make Step 3 abstract.

---

## 2. Real-World Analogy

**House Construction**
1.  Build Foundation (Same).
2.  Build Walls (Brick vs Wood).
3.  Build Roof (Tiles vs Metal).
4.  Install Windows (Same).

The **Process** (Foundation -> Walls -> Roof) is fixed. The **Materials** vary.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.

### Participants
1.  **AbstractClass**: Defines the `templateMethod()` (final) and `abstract` primitive operations.
2.  **ConcreteClass**: Implements the primitive operations.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Abstract Class (The Framework)
```java
public abstract class DataProcessor {
    
    // This is the Template Method. It is FINAL so subclasses can't change the flow.
    public final void process() {
        readData();
        processData(); // Abstract hook
        saveData();
    }

    private void readData() {
        System.out.println("Reading data from DB...");
    }

    private void saveData() {
        System.out.println("Saving data to File...");
    }

    // The Hook: Subclasses MUST implement this.
    protected abstract void processData();
}
```

### 2. Concrete Classes
```java
public class JSONProcessor extends DataProcessor {
    @Override
    protected void processData() {
        System.out.println("Converting Data to JSON format.");
    }
}

public class XMLProcessor extends DataProcessor {
    @Override
    protected void processData() {
        System.out.println("Converting Data to XML format.");
    }
}
```

### 3. Client
```java
DataProcessor p = new JSONProcessor();
p.process(); // Calls Read -> JSON -> Save
```

---

## 6. Spring Boot Implementation

Spring uses this heavily in `JdbcTemplate` (historically, via callback interfaces effectively functioning as a Template Method).

But a better example is `WebSecurityConfigurerAdapter` (Old Spring Security).
*   `configure(HttpSecurity http)`: This was a hook you overrode to customize security. The Framework called it at the right time.

---

## 8. Advantages

1.  **Code Reuse**: Common code (Steps 1, 2, 4, 5) lives in one place.
2.  **Inversion of Control**: The Parent calls the Child. (Hollywood Principle: "Don't call us, we'll call you").

---

## 9. Disadvantages

1.  **Rigidity**: You are stuck with the structure defined by the Parent.
2.  **Liskov Violation**: If a subclass implementing the hook violates the expected behavior (e.g., throwing a checked exception the parent didn't declare).

---

## 13. Comparison

| Template Method | Strategy |
|:---|:---|
| Uses **Inheritance**. | Uses **Composition**. |
| Modifies part of an algorithm. | Replaces the *entire* algorithm. |
| Structure is fixed. | Structure is replaceable. |

---

## 14. Interview Questions

### Basic
1.  **Why make the template method `final`?** (To prevent subclasses from changing the sequence of execution. Security/Consistency).
2.  **What is a Hook?** (A method that does nothing by default, but *can* be overridden if needed).

### Intermediate
3.  **Difference between Abstract Method and Hook?** (Abstract must be overridden. Hook is optional).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Game Level Loader.
    *   *Design*: `loadAssets()`, `setupEnemies()`, `startMusic()`.

2.  **Scenario**: Testing Framework (JUnit).
    *   *Design*: `setUp()`, `test()`, `tearDown()`. The runner executes them in that order.

---

## 16. Summary & Architect Takeaways

*   **Frameworks**: If you are writing a library for others, use Template Method to give them extension points.
*   **Don't Overuse**: Prefer Composition (Strategy) if possible. Inheritance is brittle.

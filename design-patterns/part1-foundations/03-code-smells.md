# Code Smells and Refactoring

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐ (Senior Dev)  
> **Status:** Critical Skill

---

## 0. Learning Objectives

*   **Beginner**: Recognize when a method is too long.
*   **Developer**: Apply "Extract Method" safely in IntelliJ.
*   **Architect**: Prioritize Technical Debt repayment based on "Smell Density".

---

## 1. Problem Statement

### The Silent Killer
Code doesn't break overnight. It rots.
*   *Day 1*: `calculateTax()` is 10 lines.
*   *Day 100*: `calculateTax()` is 500 lines with 5 nested loops.
*   **Result**: Fear. Nobody wants to touch it.

### Code Smell
A surface indication that usually corresponds to a deeper problem in the system.
"If it stinks, check for rot."

---

## 2. Real-World Analogy

*   **Kitchen**: You walk in and smell something weird.
    *   *Smell*: The symptom.
    *   *Root Cause*: Expired milk in the fridge.
    *   *Refactoring*: Throwing out the milk (and cleaning the fridge).

---

## 3. Core Concepts (🟢 Beginner Level)

### The Taxonomy of Smells

1.  **Bloaters**: Code, methods, and classes that have grown to such gargantuan proportions that they are hard to work with.
    *   *Long Method*: > 20 lines.
    *   *Large Class*: > 500 lines.
    *   *Long Parameter List*: > 3 parameters.

2.  **OO Abusers**: Misuse of Object-Oriented programming principles.
    *   *Switch Statements*: `switch(type)` usually means missing Polymorphism (Strategy).
    *   *Refused Bequest*: Subclass inherits methods it doesn't need (LSP violation).

3.  **Change Preventers**: If you need to change something in one place, you have to verify diverse changes in other places.
    *   *Shotgun Surgery*: One change requires edits to 10 classes.

4.  **Couplers**: Excessive coupling between classes.
    *   *Feature Envy*: A method accesses the data of another object more than its own.
    *   *Inappropriate Intimacy*: Classes knowing too much about each others' privates.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. Long Method -> Extract Method

**Before (Bad)**
```java
void printOwing() {
    printBanner();
    // Calculate outstanding
    double outstanding = 0.0;
    for (Order order : _orders) {
        outstanding += order.getAmount();
    }
    // Print details
    System.out.println("name: " + _name);
    System.out.println("amount: " + outstanding);
}
```

**After (Good)**
```java
void printOwing() {
    printBanner();
    double outstanding = calculateOutstanding();
    printDetails(outstanding);
}

double calculateOutstanding() {
    return _orders.stream().mapToDouble(Order::getAmount).sum();
}
```

### 2. Feature Envy -> Move Method

**Before (Bad)**
```java
class Order {
    // ...
}

class Report {
    void printReport(Order order) {
        // Report is envious of Order's data
        double total = order.getPrice() * order.getQuantity();
        System.out.println("Total: " + total);
    }
}
```

**After (Good)**
```java
class Order {
    double getTotal() {
        return getPrice() * getQuantity();
    }
}

class Report {
    void printReport(Order order) {
        System.out.println("Total: " + order.getTotal());
    }
}
```

---

## 7. Internal Mechanics (Architect Level 🔴)

### Refactoring vs Rewriting
*   **Refactoring**: Changing the structure of code *without* changing its behavior. Small steps. Tests pass at every step.
*   **Rewriting**: Throwing code away and starting over. High risk.

### The Boy Scout Rule
"Always leave the code cleaner than you found it."
*   If you touch a file to add a feature, fix one smell.

---

## 10. When NOT to Use

1.  **Strict Deadlines**: If the release is tomorrow, fix the bug, ignore the smell. (Log a Tech Debt ticket).
2.  **Stable Code**: If a 5000-line class works perfectly and nobody touches it, leave it alone. Refactoring carries regression risk.

---

## 14. Interview Questions

### Basic
1.  **What is Refactoring?** (Improving structure without changing behavior).
2.  **Naming is hard. Why is `d` a bad variable name?** (Ambiguous. Use `daysSinceCreation`).
3.  **What is a "Magic Number"?** (Hardcoded `3.14` in code. Use `Math.PI` or constant).

### Intermediate
4.  **How do you fix a "Long Parameter List"?** (Introduce Parameter Object / Builder Pattern).
5.  **What is "Primitive Obsession"?** (Using `String` for ZipCode or Phone. Make a Value Object).
6.  **Explain the "Shotgun Surgery" smell.** (Modify one feature -> Must edit 5 files).

### Advanced
7.  **How do you refactor Legacy Code with no tests?** (Write Characterization Tests first. Then refactor).
8.  **Is "Comments" a code smell?** (Yes. Code should explain *what* and *how*. Comments explain *why*. If code needs comments to explain *what*, it's complex).

---

## 16. Summary & Architect Takeaways

*   **Smells are Heuristics**: Not laws. Sometimes a long method is fine (e.g., initialization logic).
*   **Tooling**: Use SonarQube / Checkstyle to auto-detect smells.
*   **Culture**: A team that tolerates smells will eventually drown in mud.

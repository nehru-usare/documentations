# Strategy Pattern

> **Part 4: Behavioral Patterns**  
> **Difficulty:** ⭐ (Beginner)  
> **Status:** The Variable Logic

---

## 0. Learning Objectives

*   **Beginner**: Kill `switch` statements.
*   **Developer**: Switch algorithms at runtime (e.g., File Saver: PDF, CSV, JSON).
*   **Architect**: Design systems where new logic handles can be dropped in via plugins (OCP).

---

## 1. Problem Statement

### The "Switch" Statement from Hell
You have a `NavigationService`.
```java
if (transport == "CAR") { ... calculate car route ... }
else if (transport == "WALK") { ... calculate walk route ... }
else if (transport == "BIKE") { ... calculate bike route ... }
```
*   **Problem**: If you add "BUS", you modify the class. 
*   **Problem**: The class becomes huge.
*   **Problem**: You can't change behavior at runtime easily.

### The Solution
Extract the algorithms into separate classes (`CarStrategy`, `WalkStrategy`). Pass the strategy to the `NavigationService`.

---

## 2. Real-World Analogy

**Getting to the Airport**
*   **Context**: You.
*   **Strategy**: "How do I get there?"
    *   Strategy A: Take a Taxi.
    *   Strategy B: Take a Bus.
    *   Strategy C: Walk.
*   The **Destination** is the same. The **Implementation** varies. You can swap strategies instantly based on traffic (Context).

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

### Participants
1.  **Strategy**: The Interface (`RouteStrategy`).
2.  **ConcreteStrategy**: `CarRoute`, `WalkRoute`.
3.  **Context**: The class that uses the strategy (`Navigator`).

---

## 4. UML-Style Structure

*   `Context` -> *has a* -> `Strategy`
*   `ConcreteStrategyA` implements `Strategy`
*   `ConcreteStrategyB` implements `Strategy`

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Strategy Interface
```java
public interface PaymentStrategy {
    void pay(int amount);
}
```

### 2. Concrete Strategies
```java
public class CreditCardStrategy implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " via Credit Card.");
    }
}

public class PayPalStrategy implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " via PayPal.");
    }
}
```

### 3. The Context
```java
public class Checkout {
    private PaymentStrategy strategy;

    // Setter Injection allows runtime switching!
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.strategy = strategy;
    }

    public void processOrder(int amount) {
        // Delegate to the strategy
        strategy.pay(amount);
    }
}
```

### 4. Client
```java
Checkout cart = new Checkout();

// Pay by Card
cart.setPaymentStrategy(new CreditCardStrategy());
cart.processOrder(100);

// Pay by PayPal
cart.setPaymentStrategy(new PayPalStrategy());
cart.processOrder(100);
```

---

## 6. Spring Boot Implementation

Spring makes Strategy Pattern trivial with **Dependency Injection**.

```java
public interface StorageStrategy { void save(File f); }

@Service("aws")
class S3Storage implements StorageStrategy { ... }

@Service("local")
class LocalStorage implements StorageStrategy { ... }

@Service
class FileService {
    private final Map<String, StorageStrategy> strategies;

    // Spring injects ALL implementations into this Map
    public FileService(Map<String, StorageStrategy> strategies) {
        this.strategies = strategies;
    }

    public void upload(String type, File f) {
        strategies.get(type).save(f); // Dynamic Strategy Selection!
    }
}
```

---

## 8. Advantages

1.  **Open/Closed Principle**: Add new strategies (e.g., `BitCoinStrategy`) without changing the `Checkout` class.
2.  **Cleaner Code**: Removes giant conditional blocks (`if-else`).
3.  **Testing**: You can inject a `MockStrategy` during Unit Testing.

---

## 9. Disadvantages

1.  **Client Awareness**: The client needs to know *which* strategy to pick (or use a Factory).
2.  **Class Explosion**: Instead of 1 method with 10 if-else, you have 10 classes.

---

## 10. When NOT to Use

1.  **Simple Logic**: If the `if-else` is just 2 lines (`if (x) return 1 else return 2`), creating 2 classes is Overengineering.

---

## 14. Interview Questions

### Basic
1.  **How does Strategy differ from State?** (Strategy = How I do something. State = What state I am in. Structurally identical, structurally different intent).
2.  **How to select a strategy at runtime?** (Use a Map or Factory).

### Intermediate
3.  **Can we use Lambda expressions for Strategy?** (Yes! If the Strategy interface has only 1 method, it is a `@FunctionalInterface`).
    ```java
    setStrategy(amount -> System.out.println("Paid " + amount));
    ```

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Sorting.
    *   *Design*: `Collections.sort(list, Comparator)`. `Comparator` IS a Strategy.

2.  **Scenario**: Compression (Zip, Rar, 7z).
    *   *Design*: `CompressionStrategy`.

3.  **Scenario**: Validations.
    *   *Design*: `ValidatorStrategy` (EmailValidator, PhoneValidator).

---

## 16. Summary & Architect Takeaways

*   **Polymorphism is King**: Strategy is just "Composition + Polymorphism".
*   **Refactor Tool**: Whenever you see a `switch` statement on a Type code, replace it with Strategy.

# Adapter Pattern

> **Part 3: Structural Patterns**  
> **Difficulty:** ⭐ (Beginner)  
> **Status:** The Application Glue

---

## 0. Learning Objectives

*   **Beginner**: Understand how to connect a defined Interface to a Legacy Class.
*   **Developer**: Implement an Object Adapter using Composition.
*   **Architect**: Use Adapters to keep the Core Domain pure (Hexagonal Architecture).

---

## 1. Problem Statement

### The "Square Peg in a Round Hole" Problem
You have a new system that expects a `PaymentGateway` interface.
You have an old Legacy Library (`OldBankClient`) that does payments, but the method names are different.
*   New Interface: `pay(amount, currency)`
*   Old Class: `doTransaction(cents, countryCode)`
*   **Conflict**: You can't change the Old Class (it's a 3rd party JAR). You can't change the New Interface (it's used by other modules).

### What happens if we don't use it?
1.  **Pollution**: You start adding `if (isOldBank)` logic inside your business core to handle the weird method signatures.
2.  **Coupling**: Your clean code becomes tightly coupled to the legacy library's implementation details.

---

## 2. Real-World Analogy

**Travel Power Adapter**
*   **Problem**: Your Laptop has a US Plug (Flat pins). The Wall Socket is EU (Round holes).
*   **Solution**: You don't rebuild the laptop. You don't rebuild the house. You buy an **Adapter**.
*   The Adapter *translates* the shape of the plug.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

### Participants
1.  **Target**: The interface your app expects (`PaymentGateway`).
2.  **Adaptee**: The class you want to use (`OldBankClient`).
3.  **Adapter**: The wrapper class that implements Target and delegates to Adaptee.

---

## 4. UML-Style Structure

*   `Client` -> calls `Target.request()`
*   `Adapter` implements `Target`
    *   has reference to `Adaptee`
    *   `request()` calls `adaptee.specificRequest()`

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Target Interface (What we want)
```java
// Our system expects this.
public interface PaymentGateway {
    void processPayment(double amount);
}
```

### 2. The Adaptee (What we have)
```java
// 3rd party library. Cannot change this.
public class StripeLegacy {
    public void makeCharge(int amountInCents) { // Note: Cents, not Dollars
        System.out.println("Charged " + amountInCents + " cents via Stripe.");
    }
}
```

### 3. The Adapter (The Bridge)
```java
public class StripeAdapter implements PaymentGateway {
    private final StripeLegacy stripe;

    public StripeAdapter(StripeLegacy stripe) {
        this.stripe = stripe;
    }

    @Override
    public void processPayment(double amount) {
        // Translate Logic!
        // 1. Convert Dollars to Cents
        int cents = (int) (amount * 100);
        // 2. Delegate
        stripe.makeCharge(cents);
    }
}
```

### 4. Client
```java
StripeLegacy legacy = new StripeLegacy();
PaymentGateway gateway = new StripeAdapter(legacy); // Client sees PaymentGateway
gateway.processPayment(10.50);
```

---

## 6. Spring Boot Implementation

Spring MVC uses **HandlerAdapter** heavily.
*   **Problem**: A Controller method can possess any signature (`@RequestMapping`, `@GetMapping`, etc.).
*   **Solution**: Spring has a `HandlerAdapter`.
    *   `DispatcherServlet` calls `handlerAdapter.handle(request)`.
    *   The specific Adapter knows how to invoke your method (using Reflection).

---

## 7. Internal Mechanics (Architect Level 🔴)

### Object Adapter vs Class Adapter
1.  **Object Adapter**: Uses Composition (`public class XAdapter implements Target { private Adaptee a; }`). **Preferred**.
2.  **Class Adapter**: Uses Inheritance (`public class XAdapter extends Adaptee implements Target`). **Avoid** (Java doesn't support multiple inheritance of classes, and it exposes Adaptee methods to Client).

### Memory Impact
*   Minimal. Just one extra object wrapper.

---

## 8. Advantages

1.  **Single Responsibility**: The data translation logic (Dollars to Cents) lives in the Adapter, not the Business Logic.
2.  **Open/Closed**: You can introduce new Adapters for new 3rd party libs without changing the Core.

---

## 9. Disadvantages

1.  **Complexity**: If you have many Adapters, the code path becomes harder to trace ("Where is the actual logic?").

---

## 10. When NOT to Use

1.  **If you own the code**: If you can change the `OldBankClient`, just change it! Don't wrap it if you don't have to.
2.  **Simple Mapping**: If the difference is just a method name, sometimes a simple forwarding method is enough.

---

## 12. Refactoring Example

### The Bad Code
```java
class ShoppingCart {
    void checkout(String type) {
        if (type.equals("STRIPE")) {
            Stripe s = new Stripe();
            s.charge(100);
        } else if (type.equals("PAYPAL")) {
            PayPal p = new PayPal();
            p.sendMoney(1.00);
        }
    }
}
```

### Refactored to Adapter
```java
class ShoppingCart {
    private PaymentGateway gateway; // Interface

    void checkout() {
        gateway.pay(100); // Standardized
    }
}
```

---

## 14. Interview Questions

### Basic
1.  **What acts as the "Glue" in Adapter pattern?** (The Adapter class).
2.  **Difference between Adapter and Decorator?** (Adapter changes interface. Decorator keeps interface but adds behavior).

### Intermediate
3.  **Why favor Object Adapter over Class Adapter?** (Composition > Inheritance. Class Adapter is rigid).

### Advanced
4.  **How does Adapter relate to Facade?** (Adapter wraps ONE class to change interface. Facade wraps MANY classes to simplify interface).
5.  **What is a "Two-Way Adapter"?** (An adapter that implements both Target and Adaptee interfaces, so it can be used by both systems. Rare).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: XML to JSON Migration.
    *   *Design*: Old system expects XML. New API returns JSON. Create `JsonToXmlAdapter` that converts data format on the fly.

2.  **Scenario**: Logging (Slf4j).
    *   *Design*: Slf4j is an interface. Log4jAdapter, LogbackAdapter allow you to swap underlying loggers.

3.  **Scenario**: Unit Testing Legacy Code.
    *   *Design*: Create a `MockAdapter` for a Static Util class so you can test code that depends on it.

---

## 16. Summary & Architect Takeaways

*   **Anti-Corruption Layer**: Adapter is the building block of ACL. It protects your domain from external chaos.
*   **Don't leak the Adaptee**: The Client should NEVER know it's talking to `Stripe`. It deals only with `PaymentGateway`.

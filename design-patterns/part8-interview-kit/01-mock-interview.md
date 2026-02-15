# Mock Interview: Design Patterns

> **Part 8: Interview Kit**  
> **Status:** Critical Prep Material  
> **Role:** Senior / Staff Engineer

---

## 1. Creational Patterns

### Q1: When would you use the Builder Pattern over the Factory Pattern?
*   **Hint**: Think about complexity and optional parameters.
*   **Answer**: Use **Factory** when the object creation is simple or polymorphic (subclasses). Use **Builder** when the object has **many optional parameters** (Telescoping Constructor problem) or complex construction steps that need to be readable.

### Q2: Is Singleton thread-safe by default? How do you make it robust?
*   **Hint**: `synchronized` vs `enum`.
*   **Answer**: No. Lazy initialization is prone to race conditions.
    *   **Double-Checked Locking**: Complex and error-prone (volatile).
    *   **Enum Singleton**: The best way ([Effective Java]). Handles serialization and thread-safety automatically.

---

## 2. Structural Patterns

### Q3: What is the difference between Proxy and Decorator? Structure looks the same.
*   **Hint**: Intent.
*   **Answer**:
    *   **Decorator**: Adds **Behavior** (UI Borders, Input Streams). Used to enhance an object.
    *   **Proxy**: Controls **Access** (Security, Lazy Loading, Transactions). Used to wrap an object.

### Q4: How does Spring use the Adapter Pattern?
*   **Hint**: `HandlerAdapter`.
*   **Answer**: Spring MVC uses `HandlerAdapter` to run differnet types of controllers (Servlet, `@Controller`, `HttpRequestHandler`) using a single interface. Also `JpaVendorAdapter` adapts different JPA implementations.

---

## 3. Behavioral Patterns

### Q5: Explain the Strategy Pattern in the context of Spring Dependency Injection.
*   **Hint**: `@Autowired` interface.
*   **Answer**: DI is the Strategy Pattern. By injecting an Interface (`PaymentService`), we can swap the implementation (`StripeService` vs `PayPalService`) at startup (using Profiles) or runtime, without changing the client code.

### Q6: Why is the Template Method Pattern considered "invasive"?
*   **Hint**: Inheritance.
*   **Answer**: It forces you to extend a specific Parent Class. In Java, you can only extend one class. This ties your code permanently to the framework. Strategy (Composition) is usually preferred over Template Method (Inheritance).

---

## 4. Microservices Patterns

### Q7: What is the "Saga Pattern" and why do we need it?
*   **Hint**: ACID in Distributed Systems.
*   **Answer**: In Microservices, we cannot have a single ACID transaction across databases. Saga breaks the transaction into local steps. If one fails, we execute **Compensating Transactions** (Undo) to restore consistency (Eventual Consistency).

### Q8: Explain "Circuit Breaker" states.
*   **Hint**: Closed, Open, Half-Open.
*   **Answer**:
    *   **Closed**: Normal.
    *   **Open**: Failing. Request rejected immediately.
    *   **Half-Open**: Trying to recover. Allows limited requests.

---

## 5. Anti-Patterns

### Q9: What is an "Anemic Domain Model"?
*   **Hint**: Getters/Setters only.
*   **Answer**: Entities that contain data but NO logic. All validation and business rules are in "Service" classes. This violates encapsulation and is closer to Procedural programming than OOP.

### Q10: How do you fix a "God Object"?
*   **Hint**: SRP.
*   **Answer**: Identify responsibilities. Group related fields and methods. Extract them into new classes (`Order` -> `OrderRepository`, `OrderEmailer`, `OrderValidator`).

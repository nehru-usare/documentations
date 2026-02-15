# Factory Pattern

> **Part 2: Creational Patterns**  
> **Difficulty:** ⭐⭐ (Junior)  
> **Status:** Essential

---

## 0. Learning Objectives

*   **Beginner**: Stop using `new Dog()` or `new Cat()` directly in your main logic.
*   **Developer**: Implement a `PaymentFactory` that returns `CreditCard` or `PayPal` objects.
*   **Architect**: Understand Open/Closed Principle benefits of Factories.

---

## 1. Problem Statement

### The "New" Problem
Using the `new` keyword is a hard dependency.
```java
// Bad: Tightly coupled
Duck duck = new MallardDuck(); 
```
If you want to change `MallardDuck` to `RubberDuck`, you have to edit code.
If you have logic (e.g., `if (env == "prod")`), you duplicate it everywhere.

### What happens if we don't use it?
1.  **Violation of OCP**: Adding a new Duck type requires modifying every class that instantiates Ducks.
2.  **Complex Setup**: If `Duck` requires 5 dependencies to be created, you copy-paste that setup code 50 times.

---

## 2. Real-World Analogy

**The Pizza Store**
*   **Without Factory**: You (the customer) go into the kitchen, find the dough, put cheese on it, bake it, and eat it.
*   **With Factory**: You order "One Pepperoni". The Factory (Kitchen) handles the complexity of creation. You just get the object.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Define an interface for creating an object, but let subclasses decide which class to instantiate.

### Simple Factory (Idiom) vs Factory Method (Pattern)
*   **Simple Factory**: A class `DuckFactory` with a static method `createDuck(type)`.
*   **Factory Method**: An abstract method `abstract Duck createDuck()` that subclasses implement.

---

## 4. UML-Style Structure

*   **Product** (Interface): `Notification`
*   **Concrete Product**: `EmailNotification`, `SMSNotification`
*   **Creator** (Factory): `NotificationFactory`
    *   `+ createNotification(type): Notification`

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Interface
```java
public interface Notification {
    void notifyUser();
}
```

### 2. Concrete Classes
```java
public class EmailNotification implements Notification {
    public void notifyUser() { System.out.println("Sending Email"); }
}

public class SMSNotification implements Notification {
    public void notifyUser() { System.out.println("Sending SMS"); }
}
```

### 3. The Factory
```java
public class NotificationFactory {
    public Notification createNotification(String channel) {
        if (channel == null || channel.isEmpty()) return null;
        switch (channel) {
            case "SMS": return new SMSNotification();
            case "EMAIL": return new EmailNotification();
            default: throw new IllegalArgumentException("Unknown channel " + channel);
        }
    }
}
```

### 4. Client Usage
```java
// Client doesn't know about 'EmailNotification' class.
// Just asks for "EMAIL".
Notification notification = factory.createNotification("EMAIL");
notification.notifyUser();
```

---

## 6. Spring Boot Implementation

Spring makes Factories even cleaner using Map Injection (Strategy + Factory).

```java
@Service
public class PaymentFactory {
    
    // Spring automatically injects all beans implementing PaymentService
    // Key = Bean Name ("payPalService"), Value = The Bean
    private final Map<String, PaymentService> services;

    public PaymentFactory(Map<String, PaymentService> services) {
        this.services = services;
    }

    public PaymentService getService(String type) {
        return services.get(type);
    }
}
```
*   **Benefits**: No `switch` statement! To add a new payment type, just create a new class `@Service("crypto")`. Zero changes to the Factory.

---

## 7. Internal Mechanics (Architect Level 🔴)

### Simplicity vs Flexibility
*   A `Simple Factory` (static method) is usually enough.
*   The `Factory Method` (inheritance) is powerful but adds complexity (requires a Factory class for each Product class).

### Performance reflection
*   Some generic factories use `Class.forName(type).newInstance()`.
*   **Warning**: Reflection is slower and breaks compile-time safety. Avoid if simple `switch` or `Map` works.

---

## 8. Advantages

1.  **Decoupling**: Client code depends on Interface, not Implementation.
2.  **Single Responsibility**: Creation logic is moved to one specific place.
3.  **Open/Closed**: New types can be introduced without breaking existing client code (especially with the Spring Map approach).

---

## 9. Disadvantages

1.  **Code Volume**: More classes (Interfaces, Factory class).
2.  **Indirection**: Reading code requires jumping to the Factory to see what is actually created.

---

## 10. When NOT to Use

1.  **Simple Instantiation**: `List names = new ArrayList();` is fine. Don't make `ListFactory`.
2.  **Value Objects**: Don't make a factory for `UserDTO`. Just `new` it.

---

## 12. Refactoring Example

### The Bad Code
```java
class Client {
    void doSomething(String type) {
        Database db;
        if (type.equals("oracle")) {
            db = new OracleDB();
        } else if (type.equals("mysql")) {
            db = new MySQLDB();
        }
        db.connect();
    }
}
```

### Refactored to Factory
```java
class DatabaseFactory {
    static Database getDB(String type) {
        // Switch logic here...
    }
}

class Client {
    void doSomething(String type) {
        Database db = DatabaseFactory.getDB(type); // Clean!
        db.connect();
    }
}
```

---

## 14. Interview Questions

### Basic
1.  **What is the main goal of Factory Pattern?** (Decouple creation from usage).
2.  **Can a Factory return null?** (Yes, if the type is unknown, or throw Exception // Optional).

### Intermediate
3.  **Difference between Factory and Singleton?** (Singleton = One instance. Factory = Creates instances).
4.  **Difference between Simple Factory and Factory Method?** (Simple Factory is a helper class. Factory Method relies on Inheritance).

### Advanced
5.  **How does Spring `FactoryBean` interface work?** (Allows complex initialization logic for beans that can't be created with `new`).
6.  **Does Factory pattern violate Dependency Inversion?** (No, it enables it. Client depends on abstraction, Factory handles concretion).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: You are building a Report Generator (PDF, HTML, CSV).
    *   *Design*: `ReportFactory.create("PDF")`. Returns `Report` interface.

2.  **Scenario**: You handle login for Google, Facebook, Apple.
    *   *Design*: `AuthProviderFactory` returns `AuthProvider`.

3.  **Scenario**: You need to parse config files (JSON, XML, YAML).
    *   *Design*: `ConfigParserFactory` determines parser based on file extension.

4.  **Scenario**: Bank Application. Different account types (Savings, Checking).
    *   *Design*: `AccountFactory`.

5.  **Scenario**: Conditional Bean creation in Spring.
    *   *Design*: Use `@ConditionalOnProperty` (Framework-level factory) or `@Bean` methods.

6.  **Scenario**: Dependency loops in Factory.
    *   *Design*: Use `ObjectProvider<T>` or `@Lazy` in Spring to break the loop.

---

## 16. Summary & Architect Takeaways

*   **Factories are the glue** of clean architecture.
*   **Static Factory Method**: Prefer `User.createWithEmail()` over `new User(email)`. It has a name!
*   **Spring Power**: Use the Map Injection pattern to implement OCP-compliant Factories.

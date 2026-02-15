# Factory Method Pattern

> **Part 2: Creational Patterns**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Architecture Foundation

---

## 0. Learning Objectives

*   **Beginner**: Distinguish this from "Simple Factory".
*   **Developer**: Use inheritance to allow libraries to create your objects.
*   **Architect**: Build extensible frameworks where users can plug in their own classes.

---

## 1. Problem Statement

### The Framework Problem
Imagine you are building a **Logistics Framework**.
core works with `Truck`.
Now, a user wants to use `Ship`.
*   **Simple Factory**: You have to edit your framework code to add `case "SHIP": return new Ship()`. (Bad: You don't know what users will need).
*   **Factory Method**: You tell the user: "Extend my `Logistics` class and override `createTransport()`."

---

## 2. Real-World Analogy

**Franchise Business**
*   **Burger King (Headquarters)**: Defines the standard `makeBurger()`.
*   **Burger King India**: Overrides `makeBurger()` to create `PaneerKing`.
*   **Burger King USA**: Overrides `makeBurger()` to create `BeefKing`.
The process (bundling, selling) is same. The *product creation* is localized (subclassed).

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
"Defines an interface for creating an object, but lets subclasses decide which class to instantiate."

### Participants
1.  **Creator** (Abstract Class): Declares `factoryMethod()`.
2.  **Concrete Creator**: Overrides `factoryMethod()` to return a specific Product.
3.  **Product**: The interface.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Logistics Framework (The "Creator")
```java
abstract class Logistics {
    // This is the core logic. It uses the product, but doesn't know concrete class.
    public void planDelivery() {
        Transport t = createTransport(); // The Factory Method!
        t.deliver();
    }

    // Abstract method: Keep it protected.
    protected abstract Transport createTransport();
}
```

### 2. Road Logistics (Concrete Creator)
```java
class RoadLogistics extends Logistics {
    @Override
    protected Transport createTransport() {
        return new Truck();
    }
}
```

### 3. Sea Logistics (Concrete Creator)
```java
class SeaLogistics extends Logistics {
    @Override
    protected Transport createTransport() {
        return new Ship();
    }
}
```

### 4. Client
```java
Logistics log = new SeaLogistics();
log.planDelivery(); // internally creates Ship and calls deliver()
```

---

## 6. Spring Boot Implementation

Spring uses this heavily in `FactoryBean`.

```java
public interface FactoryBean<T> {
    T getObject() throws Exception;
    Class<?> getObjectType();
}
```
If you implement this, Spring will call `getObject()` and inject the *result*, not the factory itself.
*   Example: `LocalContainerEntityManagerFactoryBean` creates the JPA `EntityManagerFactory`.

---

## 8. Advantages

1.  **Extensibility**: Users can extend your library without touching your code.
2.  **Connecting Parallel Hierarchies**: `Document` class creates `Page` class. `Resume` subclass creates `SkillPage`.

---

## 9. Disadvantages

1.  **Class Explosion**: For every new Product, you need a new Creator subclass.
    *   Truck -> TruckLogistics
    *   Ship -> SeaLogistics
    *   Drone -> AirLogistics
2.  **Complexity**: Deeper inheritance tree.

---

## 13. Comparison: Simple Factory vs Factory Method

| Simple Factory | Factory Method |
|:---|:---|
| One class with a giant `switch`. | Abstract class with subclasses. |
| Hard to extend (Modify code). | Easy to extend (Add subclass). |
| Good for fixed set of products. | Good for frameworks / unknown products. |

---

## 14. Interview Questions

### Basic
1.  **Is `Calendar.getInstance()` a Factory Method?** (Yes, technically a Static Factory Method that can return different subclasses like `BuddhistCalendar`).
2.  **Why abstract method?** (To force subclasses to provide an implementation).

### Intermediate
3.  **Can Factory Method be static?** (No. It relies on Polymorphism/Overriding. Static methods cannot be overridden).
4.  **What problem does it solve that Simple Factory doesn't?** (Open/Closed Principle for the *Creator*).

### Advanced
5.  **Explain "Hook Methods" in this context.** (A method in the base class that provides a default, but can be overridden. Factory Method is often a hook).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Cross-Platform UI Framework (Windows vs Mac).
    *   *Design*: `Dialog` (Creator). `WindowsDialog` creates `WindowsButton`. `MacDialog` creates `MacButton`.

2.  **Scenario**: Processing different file formats (CSV, XML) with strict validation rules.
    *   *Design*: `Parser` (Creator). `CSVParser` creates `CSVValidator`.

3.  **Scenario**: Testing.
    *   *Design*: `MockDatabaseLogistics` overrides `createTransport()` to return a `MockTransport`. Excellent for Unit Testing logic without real dependencies.

---

## 16. Summary & Architect Takeaways

*   **Framework writers love this pattern**. Application developers use it less often (Simple Factory + Map is usually enough).
*   **Inheritance is the key**. If you aren't using inheritance to switch behavior, it's not the GoF Factory Method.

# Decorator Pattern

> **Part 3: Structural Patterns**  
> **Difficulty:** ⭐⭐ (Intermediate)  
> **Status:** The Extender

---

## 0. Learning Objectives

*   **Beginner**: Understand why `java.io` has so many nested classes (`new BufferedInputStream(new FileInputStream...)`).
*   **Developer**: Add features (Logging, Caching) to a class without modifying it.
*   **Architect**: Avoid "Class Explosion" (e.g., `EncryptedFileStream`, `CompressedFileStream`, `EncryptedCompressedFileStream`).

---

## 1. Problem Statement

### The Inheritance Explosion
Imagine a `Coffee` class.
You want `MilkCoffee`. -> Extends Coffee.
You want `SugarCoffee`. -> Extends Coffee.
You want `MilkSugarCoffee`. -> ??
*   **Problem**: You cannot extend multiple classes in Java.
*   **Result**: You end up creating a combinatorial explosion of subclasses.

### What happens if we don't use it?
1.  **Rigidity**: To add a new feature (e.g., "WhippedCream"), you have to create combinations for every existing coffee (`MilkWhipped`, `SugarWhipped`, `MilkSugarWhipped`...).

---

## 2. Real-World Analogy

**Pizza Toppings**
*   Base: **Pizza Dough**.
*   Decorator 1: **Tomato Sauce**. (Wraps Dough).
*   Decorator 2: **Cheese**. (Wraps Sauce).
*   Decorator 3: **Olives**. (Wraps Cheese).
The result is still a "Pizza", but it has layers of flavor added.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

### Key Mechanism
**Recursive Composition**: The Decorator *is a* Component, and it *has a* Component.

---

## 4. UML-Style Structure

*   `Component` (Interface): `Coffee`
*   `ConcreteComponent`: `SimpleCoffee`
*   `Decorator` implements `Coffee`
    *   has `Coffee wrappedObj`
*   `MilkDecorator` extends `Decorator`

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Component Interface
```java
public interface Coffee {
    double getCost();
    String getDescription();
}
```

### 2. The Concrete Component (Base)
```java
public class SimpleCoffee implements Coffee {
    public double getCost() { return 10.0; }
    public String getDescription() { return "Simple Coffee"; }
}
```

### 3. The Abstract Decorator
```java
public abstract class CoffeeDecorator implements Coffee {
    protected final Coffee decoratedCoffee; // Reference to inner layer

    public CoffeeDecorator(Coffee c) {
        this.decoratedCoffee = c;
    }

    public double getCost() { return decoratedCoffee.getCost(); }
    public String getDescription() { return decoratedCoffee.getDescription(); }
}
```

### 4. Concrete Decorators
```java
public class Milk extends CoffeeDecorator {
    public Milk(Coffee c) { super(c); }

    @Override
    public double getCost() { return super.getCost() + 2.0; }

    @Override
    public String getDescription() { return super.getDescription() + ", Milk"; }
}

public class Sugar extends CoffeeDecorator {
    public Sugar(Coffee c) { super(c); }

    @Override
    public double getCost() { return super.getCost() + 1.0; }

    @Override
    public String getDescription() { return super.getDescription() + ", Sugar"; }
}
```

### 5. Client Usage
```java
// Start with base
Coffee c = new SimpleCoffee(); 
// Wrap with Milk
c = new Milk(c); 
// Wrap with Sugar
c = new Sugar(c);

// Usage
System.out.println(c.getCost()); // 13.0
System.out.println(c.getDescription()); // Simple Coffee, Milk, Sugar
```

---

## 6. Spring Boot Implementation

Spring mostly uses **AOP** (Proxy Pattern) for this, but Decorator exists in:
*   `HttpServletRequestWrapper`: Allows you to decorate the HTTP Request to read the body multiple times or sanitize inputs.

```java
public class SanitizedRequest extends HttpServletRequestWrapper {
    public SanitizedRequest(HttpServletRequest request) {
        super(request);
    }
    
    @Override
    public String getParameter(String name) {
        String original = super.getParameter(name);
        return sanitize(original); // Add behavior
    }
}
```

---

## 7. Internal Mechanics (Architect Level 🔴)

### Stack Depth
*   Every layer adds a stack frame call.
*   `Milk -> Sugar -> Simple`. Calling `getCost()` invokes 3 methods.
*   **Performance**: Negligible for 10 layers. Bad for 1000 layers.

### Identity Crisis
*   `c` is a `Sugar` object in the example above.
*   `c instanceof SimpleCoffee` is FALSE (if it's wrapped).
*   **Warning**: Do not rely on object identity (`==`) when using Decorators.

---

## 8. Advantages

1.  **Extensibility**: Add behavior at runtime.
2.  **SRP**: Splitting "Core Logic" from "Decoration Logic" (e.g., Logging, Validation).
3.  **Composition > Inheritance**: No class explosion.

---

## 9. Disadvantages

1.  **Complexity**: Many small classes.
2.  **Debugging**: "Where is this data coming from?" requires stepping through 10 wrappers.
3.  **Ordering**: Does `Encrypt(Compress(Data))` produce the same result as `Compress(Encrypt(Data))`? No. The order of decorators matters.

---

## 12. Refactoring Example

### The Bad Code
```java
class Notifier {
    void send(String msg) {
        // Core logic
    }
}

class NotifierWithLog extends Notifier { ... }
class NotifierWithLogAndEnc extends Notifier { ... }
```

### Refactored
```java
Notifier n = new LogDecorator(new EncryptDecorator(new Notifier()));
```

---

## 14. Interview Questions

### Basic
1.  **Example of Decorator in Java Standard Lib?** (`java.io` classes: `BufferedReader`, `GZIPOutputStream`).
2.  **Difference between Decorator and Inheritance?** (Inheritance is static/compile-time. Decorator is dynamic/runtime).

### Intermediate
3.  **Why pass the Component in Constructor?** (To establish the link for delegation).
4.  **Can a Decorator generally remove behavior?** (No, usually it adds. Proxy can remove/block).

### Advanced
5.  **How does Decorator differ from Proxy?** (Intent. Proxy controls access. Decorator adds features. Structurally they are almost identical).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Caching Service.
    *   *Design*: `CacheDecorator` wraps `DatabaseService`. Checks `Map` before calling `DatabaseService`.

2.  **Scenario**: Input Streams (Compression).
    *   *Design*: `GZIPInputStream` wraps `FileInputStream`.

3.  **Scenario**: UI Scrollbars.
    *   *Design*: `Window` object. `ScrollbarDecorator` adds the UI element and logic to `Window` interface.

---

## 16. Summary & Architect Takeaways

*   **Recursive Composition** is powerful.
*   **Use Interface**: Wrapper and Wrapped must share an interface.
*   **Wrappers**: This is the basis for "Middleware" patterns in web frameworks (Express.js, etc).

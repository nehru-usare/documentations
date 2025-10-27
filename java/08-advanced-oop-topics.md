# 🧠 Advanced OOP Topics in Java 21

As a 4+ years experienced Java developer, you’ve likely used classes, interfaces, and inheritance daily.  
However, building **robust enterprise systems** requires a deeper understanding of **how to design with OOP principles effectively** — balancing **abstraction**, **composition**, and **inheritance** without overcomplicating architecture.

This document covers advanced OOP concepts and their application in **modern Java 21**.

---

## 🧭 Table of Contents

1. [Abstract Classes vs Interfaces](#1-abstract-classes-vs-interfaces)
2. [Composition vs Inheritance](#2-composition-vs-inheritance)
3. [Object Relationships — HAS-A and IS-A](#3-object-relationships--has-a-and-is-a)
4. [Inner, Nested, and Anonymous Classes](#4-inner-nested-and-anonymous-classes)
5. [Sealed Hierarchies and Design Control (Java 17+)](#5-sealed-hierarchies-and-design-control-java-17)
6. [Designing with Records and Immutability (Java 16+)](#6-designing-with-records-and-immutability-java-16)
7. [Method Overriding, Hiding, and Covariant Returns](#7-method-overriding-hiding-and-covariant-returns)
8. [Abstract Factory and Strategy Patterns (Practical)](#8-abstract-factory-and-strategy-patterns-practical)
9. [Best Practices for OOP in Java 21](#9-best-practices-for-oop-in-java-21)
10. [Summary](#10-summary)

---

## 1️⃣ Abstract Classes vs Interfaces

Both **abstract classes** and **interfaces** are used to define abstraction — but they serve different design purposes.

| Feature | Abstract Class | Interface |
|----------|----------------|------------|
| Inheritance | Single | Multiple |
| Members | Can have state (fields) | Cannot have instance fields |
| Methods | Abstract + Concrete | Abstract, Default, Static, Private |
| Constructors | Allowed | Not allowed |
| Access Modifiers | Can have protected | All methods are implicitly public |
| Use Case | Base class with shared code | Contract for behavior |

### 🔹 Abstract Class Example
```java
abstract class PaymentProcessor {
    abstract void process(double amount);
    void audit() {
        System.out.println("Audit completed");
    }
}

class UpiPayment extends PaymentProcessor {
    @Override
    void process(double amount) {
        System.out.println("Processing ₹" + amount + " via UPI");
    }
}
````

### 🔹 Interface Example

```java
interface Payment {
    void pay(double amount);
    default void log(String method) {
        System.out.println("Payment method: " + method);
    }
}

class CreditCardPayment implements Payment {
    public void pay(double amount) {
        log("Credit Card");
        System.out.println("Paid ₹" + amount);
    }
}
```

> 💡 Use **interfaces** for capabilities, **abstract classes** for shared base behavior.

---

## 2️⃣ Composition vs Inheritance

### 🔹 Inheritance (IS-A relationship)

Inheritance expresses a hierarchy where one class **is a** specialized form of another.

```java
class Vehicle {}
class Car extends Vehicle {}
```

✅ Pros:

* Code reuse
* Simpler polymorphism

❌ Cons:

* Tight coupling
* Fragile base class problem
* Violates encapsulation if misused

---

### 🔹 Composition (HAS-A relationship)

Composition means **one class contains another** as part of its behavior.

```java
class Engine {
    void start() { System.out.println("Engine started"); }
}

class Car {
    private final Engine engine = new Engine();
    void drive() { engine.start(); }
}
```

✅ Pros:

* Loose coupling
* Flexible and testable
* Encouraged by SOLID’s “Composition over Inheritance” principle

> 💡 Use composition when behavior can be reused **without inheriting implementation**.

---

## 3️⃣ Object Relationships — HAS-A and IS-A

### 🔹 IS-A Relationship

Defined via **inheritance**.

```java
class Dog extends Animal {}
```

### 🔹 HAS-A Relationship

Defined via **composition**.

```java
class Car {
    Engine engine; // Car HAS-A Engine
}
```

### 🔹 Example in Enterprise Context

```java
class Order {
    private Customer customer;   // HAS-A
    private List<Item> items;    // HAS-A
}
class Customer extends User {}    // IS-A
```

---

## 4️⃣ Inner, Nested, and Anonymous Classes

### 🔹 Non-static Inner Class

Can access members of the outer class.

```java
class Outer {
    private String message = "Hello";

    class Inner {
        void print() {
            System.out.println(message);
        }
    }
}
```

Usage:

```java
Outer.Inner obj = new Outer().new Inner();
obj.print();
```

---

### 🔹 Static Nested Class

Does **not** hold reference to outer class.

```java
class Database {
    static class Config {
        static void print() {
            System.out.println("Connected to DB");
        }
    }
}
```

---

### 🔹 Anonymous Class

For **one-time use** — typically implementing an interface or abstract class inline.

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running in thread");
    }
};
new Thread(r).start();
```

> 💡 Replace with **lambda expressions** where possible:

```java
new Thread(() -> System.out.println("Running in thread")).start();
```

---

## 5️⃣ Sealed Hierarchies and Design Control (Java 17+)

Sealed classes/interfaces restrict subclassing to known implementations.

### Example:

```java
public sealed interface Notification permits Email, SMS {}

final class Email implements Notification {}
final class SMS implements Notification {}
```

✅ Benefits:

* Explicit control over class hierarchies
* Enables **pattern matching**
* Improves **type safety** for domain models

### Real-world Use Case:

```java
sealed interface Payment permits CreditCard, UPI {}
final class CreditCard implements Payment {}
final class UPI implements Payment {}
```

Now you can use:

```java
static String handle(Payment p) {
    return switch (p) {
        case CreditCard c -> "CreditCard payment";
        case UPI u -> "UPI payment";
    };
}
```

---

## 6️⃣ Designing with Records and Immutability (Java 16+)

Records are **final, immutable, and concise** — perfect for value objects.

```java
public record Product(String id, String name, double price) {}
```

✅ Key Advantages:

* Auto-generated constructor, accessors, and methods
* Thread-safe by design
* Ideal for DTOs, events, and configuration classes

> 💡 Combine **sealed hierarchies + records** for strong domain models.

Example:

```java
sealed interface Shape permits Circle, Rectangle {}
record Circle(double radius) implements Shape {}
record Rectangle(double width, double height) implements Shape {}
```

---

## 7️⃣ Method Overriding, Hiding, and Covariant Returns

### 🔹 Overriding

Occurs when a subclass redefines a parent method.

```java
@Override
void run() { ... }
```

### 🔹 Hiding

Static methods cannot be overridden — they are **hidden**.

```java
class Parent {
    static void greet() {}
}
class Child extends Parent {
    static void greet() {} // hides Parent.greet()
}
```

### 🔹 Covariant Return

Java allows overriding methods to return **subtypes**.

```java
class Animal {}
class Dog extends Animal {}

class AnimalFactory {
    Animal create() { return new Animal(); }
}

class DogFactory extends AnimalFactory {
    @Override
    Dog create() { return new Dog(); } // covariant return
}
```

---

## 8️⃣ Abstract Factory and Strategy Patterns (Practical)

### 🔹 Abstract Factory Pattern

Creates families of related objects without specifying concrete classes.

```java
interface PaymentFactory {
    Payment createPayment();
}

class CreditCardFactory implements PaymentFactory {
    public Payment createPayment() { return new CreditCardPayment(); }
}
```

---

### 🔹 Strategy Pattern

Encapsulates algorithms within interchangeable classes.

```java
interface CompressionStrategy {
    void compress(String file);
}

class ZipCompression implements CompressionStrategy {
    public void compress(String file) { System.out.println("ZIP compressing " + file); }
}

class Compressor {
    private final CompressionStrategy strategy;
    Compressor(CompressionStrategy strategy) { this.strategy = strategy; }
    void compress(String file) { strategy.compress(file); }
}
```

Usage:

```java
Compressor compressor = new Compressor(new ZipCompression());
compressor.compress("report.txt");
```

> ⚙️ These patterns make use of **polymorphism and composition** — crucial for enterprise-level flexibility.

---

## 9️⃣ Best Practices for OOP in Java 21

✅ Follow **SOLID principles** (especially *O* and *D*)
✅ Prefer **composition** over inheritance
✅ Keep **constructors light**, move heavy logic to factories
✅ Use **interfaces with default methods** for version-safe APIs
✅ Combine **sealed classes + records** for modeling fixed domain data
✅ Use **dependency injection (Spring)** for loose coupling
✅ Keep **object state immutable** unless absolutely necessary
✅ Write **unit tests** for behavior, not structure

---

## 🔟 Summary

You’ve now explored:

* How to design **modern Java 21 class hierarchies**
* When to use **inheritance vs composition**
* How **sealed classes, records, and pattern matching** redefine OOP
* Real-world design patterns with best practices

This is the bridge between being a *Java developer* and a *Java architect*.

> 🧭 **Next Topic:** [09-packages-and-access-modifiers.md → Java 21 Packaging, Access Control, and Modular Design](./09-packages-and-access-modifiers.md)
# 🧩 Java 21 — Object-Oriented Programming (OOP) Concepts

Java is built entirely on the principles of **Object-Oriented Programming (OOP)**.  
Every Java developer — especially at a professional level — must not only know these concepts but also understand how they interact at runtime and how they evolve in **modern Java 21**.

This document explains each **OOP pillar**, **modern enhancements**, and **best practices** for writing clean, maintainable, and extensible code.

---

## 🧭 Table of Contents

1. [Introduction to OOP](#1-introduction-to-oop)
2. [Encapsulation](#2-encapsulation)
3. [Inheritance](#3-inheritance)
4. [Polymorphism](#4-polymorphism)
5. [Abstraction](#5-abstraction)
6. [Interfaces and Default Methods](#6-interfaces-and-default-methods)
7. [Sealed Classes (Java 17+)](#7-sealed-classes-java-17)
8. [Records and Data Abstraction (Java 16+)](#8-records-and-data-abstraction-java-16)
9. [Pattern Matching and Type Safety (Java 21)](#9-pattern-matching-and-type-safety-java-21)
10. [Design Principles and Best Practices](#10-design-principles-and-best-practices)
11. [OOP in Modern Frameworks (Spring, Microservices)](#11-oop-in-modern-frameworks-spring-microservices)
12. [Summary](#12-summary)

---

## 1️⃣ Introduction to OOP

**OOP** (Object-Oriented Programming) models real-world entities using **objects**.  
It promotes **modularity**, **code reuse**, and **maintainability**.

### 🔹 Four Pillars of OOP
| Concept | Description |
|----------|--------------|
| **Encapsulation** | Binding data and behavior into one unit (class) |
| **Inheritance** | Deriving new classes from existing ones |
| **Polymorphism** | One interface, many implementations |
| **Abstraction** | Hiding complexity, exposing only essentials |

---

## 2️⃣ Encapsulation

Encapsulation is the process of **wrapping variables (state)** and **methods (behavior)** into a single entity — the **class**.

### 🔹 Example:
```java
public class Account {
    private double balance;

    public Account(double balance) {
        this.balance = balance;
    }

    public void deposit(double amount) {
        if (amount > 0) balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
````

✅ **Best Practices:**

* Always keep fields `private`
* Use **getters and setters**
* Validate data in setters (not in constructors)
* Make classes immutable wherever possible

---

## 3️⃣ Inheritance

Inheritance allows one class to acquire properties and behaviors from another class using the `extends` keyword.

### 🔹 Example:

```java
class Employee {
    String name;
    double salary;

    void work() {
        System.out.println(name + " is working");
    }
}

class Manager extends Employee {
    void manageTeam() {
        System.out.println(name + " is managing the team");
    }
}

public class Demo {
    public static void main(String[] args) {
        Manager m = new Manager();
        m.name = "Alice";
        m.work();
        m.manageTeam();
    }
}
```

✅ **Key Points:**

* Java supports **single inheritance** only.
* Use **interfaces** for multiple-type behavior.
* The `super` keyword refers to the parent class.

---

### 🔹 Constructor Chaining

Child constructors implicitly call the parent’s no-arg constructor, or explicitly via `super()`.

```java
class Person {
    Person() { System.out.println("Person created"); }
}

class Student extends Person {
    Student() { super(); System.out.println("Student created"); }
}
```

---

## 4️⃣ Polymorphism

**Polymorphism** means “many forms.”
In Java, it allows one interface to represent multiple underlying forms (implementations).

### 🔹 Compile-Time Polymorphism (Method Overloading)

```java
class MathUtils {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
}
```

### 🔹 Runtime Polymorphism (Method Overriding)

```java
class Animal {
    void makeSound() { System.out.println("Some sound"); }
}
class Dog extends Animal {
    @Override
    void makeSound() { System.out.println("Bark"); }
}

Animal a = new Dog();
a.makeSound(); // "Bark"
```

✅ JVM decides at runtime which method to invoke — **dynamic dispatch**.

---

## 5️⃣ Abstraction

Abstraction means **hiding complex details** and exposing only essential behavior.

You can achieve abstraction using:

1. **Abstract Classes**
2. **Interfaces**

### 🔹 Abstract Class Example

```java
abstract class Shape {
    abstract double area();
    void display() { System.out.println("Calculating area..."); }
}

class Circle extends Shape {
    double radius;
    Circle(double r) { this.radius = r; }

    @Override
    double area() { return Math.PI * radius * radius; }
}
```

✅ Abstract classes can have both **abstract** and **concrete** methods.

---

## 6️⃣ Interfaces and Default Methods

Interfaces define **contracts** — what a class must do, not how.

```java
interface Payment {
    void pay(double amount);
}

class UpiPayment implements Payment {
    public void pay(double amount) {
        System.out.println("Paid ₹" + amount + " via UPI");
    }
}
```

### 🔹 Default and Private Methods (Java 8+ / 9+)

```java
interface Logger {
    default void log(String message) {
        format(message);
        System.out.println("[LOG] " + message);
    }

    private void format(String msg) {
        System.out.println("Formatting: " + msg);
    }
}
```

✅ **Java 21 interfaces** can now:

* Have **default**, **static**, and **private** methods
* Be used with **sealed interfaces**

---

## 7️⃣ Sealed Classes (Java 17+)

**Sealed classes** restrict which other classes can extend or implement them.

### 🔹 Example

```java
public sealed class Shape permits Circle, Rectangle {}

final class Circle extends Shape {}
final class Rectangle extends Shape {}
```

✅ Benefits:

* Provides **controlled inheritance**
* Enhances **pattern matching** and **type safety**
* Useful for **domain modeling**

---

## 8️⃣ Records and Data Abstraction (Java 16+)

Records simplify **data-carrier classes** (immutable POJOs).

```java
public record Employee(String name, double salary) {}
```

Auto-generates:

* `constructor`
* `toString()`
* `equals()`
* `hashCode()`
* `getters` (via component methods)

✅ Best for:

* DTOs
* Immutable data models
* API responses

> 💡 Records complement abstraction — they hide implementation details while exposing data cleanly.

---

## 9️⃣ Pattern Matching and Type Safety (Java 21)

Pattern matching simplifies type checks and casts.

### 🔹 Before Java 16:

```java
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.toUpperCase());
}
```

### 🔹 Java 21:

```java
if (obj instanceof String s) {
    System.out.println(s.toUpperCase());
}
```

### 🔹 Pattern Matching in Switch (Java 21)

```java
static String format(Object o) {
    return switch (o) {
        case Integer i -> "Integer: " + i;
        case String s when s.length() > 5 -> "Long String: " + s;
        case String s -> "Short String: " + s;
        case null -> "Null value";
        default -> "Unknown type";
    };
}
```

✅ Advantages:

* Eliminates unnecessary casting
* Enables **type-safe** polymorphism
* Works seamlessly with **sealed hierarchies**

---

## 🔟 Design Principles and Best Practices

### 🔹 SOLID Principles

| Principle | Description                                                    |
| --------- | -------------------------------------------------------------- |
| **S**     | Single Responsibility — each class has one job                 |
| **O**     | Open/Closed — open for extension, closed for modification      |
| **L**     | Liskov Substitution — subclass must behave like parent         |
| **I**     | Interface Segregation — fine-grained interfaces preferred      |
| **D**     | Dependency Inversion — depend on abstractions, not concretions |

### 🔹 Tips

✅ Prefer **composition over inheritance**
✅ Favor **interfaces** for contracts
✅ Make objects **immutable** where possible
✅ Use **sealed hierarchies** for domain constraints
✅ Avoid deep inheritance chains (>2 levels)

---

## 11️⃣ OOP in Modern Frameworks (Spring, Microservices)

### 🔹 Spring Framework

* Uses OOP heavily for **Dependency Injection (DI)**.
* Beans are objects managed by the Spring container.
* Interfaces define contracts for services.

Example:

```java
public interface PaymentService {
    void processPayment(double amount);
}

@Service
public class CreditCardPaymentService implements PaymentService {
    @Override
    public void processPayment(double amount) {
        System.out.println("Paid ₹" + amount + " via Credit Card");
    }
}
```

### 🔹 Microservices

OOP concepts help define **modular, loosely coupled components**.

---

## 12️⃣ Summary

✅ You’ve learned:

* The **4 OOP pillars** (Encapsulation, Inheritance, Polymorphism, Abstraction)
* Modern OOP enhancements: **sealed classes**, **records**, **pattern matching**
* How OOP integrates with **Java 21** and modern frameworks
* SOLID principles and design patterns for cleaner architecture

> 🧭 **Next Topic:** [08-advanced-oop-topics.md → Advanced OOP in Java 21: Inner Classes, Abstract Classes, and Composition](./08-advanced-oop-topics.md)
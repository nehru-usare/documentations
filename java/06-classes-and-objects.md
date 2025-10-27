# 🧱 Java 21 — Classes and Objects

At the heart of Java lies **Object-Oriented Programming (OOP)** — everything in Java revolves around **classes and objects**.  
Understanding how classes work, how objects are created and managed, and how memory is allocated in the JVM is key to writing scalable, maintainable Java code.

This guide dives deep into **Java 21 classes, object lifecycle, static behavior, and design conventions** for real-world development.

---

## 🧭 Table of Contents

1. [What Are Classes and Objects?](#1-what-are-classes-and-objects)
2. [Class Declaration and Structure](#2-class-declaration-and-structure)
3. [Creating and Using Objects](#3-creating-and-using-objects)
4. [Constructors in Java 21](#4-constructors-in-java-21)
5. [Static Members and Initialization Blocks](#5-static-members-and-initialization-blocks)
6. [The `this` Keyword](#6-the-this-keyword)
7. [Memory Allocation and Object Lifecycle](#7-memory-allocation-and-object-lifecycle)
8. [Object Reference and Comparison](#8-object-reference-and-comparison)
9. [Immutability and Best Practices](#9-immutability-and-best-practices)
10. [Anonymous and Inner Classes](#10-anonymous-and-inner-classes)
11. [Records in Java 21 (Data-Carrying Classes)](#11-records-in-java-21-data-carrying-classes)
12. [Summary](#12-summary)

---

## 1️⃣ What Are Classes and Objects?

A **class** is a blueprint or template for creating **objects**.  
An **object** is an instance of a class, representing real-world entities with **state (fields)** and **behavior (methods)**.

Example:
```java
class Car {
    String brand;
    int speed;

    void accelerate() {
        speed += 10;
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car(); // Object creation
        car.brand = "Tesla";
        car.accelerate();
        System.out.println(car.brand + " speed: " + car.speed);
    }
}
````

---

## 2️⃣ Class Declaration and Structure

A well-defined class consists of:

* Fields (instance variables)
* Methods (behavior)
* Constructors
* Optional: static blocks, inner classes, and initialization blocks

```java
public class Employee {
    // Fields
    private String name;
    private double salary;

    // Constructor
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    // Methods
    public void raiseSalary(double percent) {
        salary += salary * percent / 100;
    }

    public String getDetails() {
        return name + " earns $" + salary;
    }
}
```

---

## 3️⃣ Creating and Using Objects

Objects are created using the `new` keyword, which:

1. Allocates memory in the heap.
2. Calls the constructor to initialize the object.

```java
Employee emp = new Employee("Nehru", 85000);
System.out.println(emp.getDetails());
```

You can create multiple instances from the same class blueprint:

```java
Employee emp1 = new Employee("Alice", 75000);
Employee emp2 = new Employee("Bob", 90000);
```

---

## 4️⃣ Constructors in Java 21

Constructors are **special methods** used to initialize objects.

### 🔹 Types of Constructors

| Type                          | Description                                               |
| ----------------------------- | --------------------------------------------------------- |
| **Default Constructor**       | Provided automatically by Java if none defined            |
| **Parameterized Constructor** | Accepts arguments for initialization                      |
| **Copy Constructor**          | Custom constructor that copies fields from another object |

```java
class Student {
    String name;
    int age;

    Student() { // Default
        this("Unknown", 0);
    }

    Student(String name, int age) { // Parameterized
        this.name = name;
        this.age = age;
    }

    Student(Student s) { // Copy
        this.name = s.name;
        this.age = s.age;
    }
}
```

> ⚙️ Constructors are **not inherited**, but can be **chained** using `this()`.

---

## 5️⃣ Static Members and Initialization Blocks

### 🔹 Static Fields and Methods

Static members belong to the **class**, not to any instance.

```java
class MathUtil {
    static final double PI = 3.14159;

    static int square(int x) {
        return x * x;
    }
}

System.out.println(MathUtil.square(5));
```

✅ Best Practices:

* Use static members for **shared utilities**.
* Avoid using them for **stateful logic** (causes thread-safety issues).

---

### 🔹 Static Blocks

Used for one-time class-level initialization.

```java
class Config {
    static String ENV;
    static {
        ENV = System.getenv("APP_ENV");
        System.out.println("Configuration loaded: " + ENV);
    }
}
```

### 🔹 Instance Initializer Block

Runs **before constructors** when creating an object.

```java
{
    System.out.println("Instance initialized");
}
```

---

## 6️⃣ The `this` Keyword

`this` is a reference to the current object — used to differentiate between instance variables and parameters.

```java
class Account {
    private double balance;

    Account(double balance) {
        this.balance = balance; // referring to instance variable
    }

    void deposit(double balance) {
        this.balance += balance;
    }
}
```

`this()` can also be used for **constructor chaining**:

```java
Account() {
    this(0.0); // calls parameterized constructor
}
```

---

## 7️⃣ Memory Allocation and Object Lifecycle

Java manages memory automatically using the **Heap and Stack** areas.

| Memory Area     | Purpose                                  |
| --------------- | ---------------------------------------- |
| **Stack**       | Stores local variables and references    |
| **Heap**        | Stores objects created by `new`          |
| **Method Area** | Stores class metadata and static members |

### 🔹 Object Lifecycle

1. **Creation:** via `new` → heap allocation
2. **Usage:** accessed via references
3. **Dereferencing:** reference goes out of scope
4. **Garbage Collection:** JVM reclaims memory

> 💡 Java 21 uses **ZGC** or **G1 GC** for efficient object lifecycle management.

---

## 8️⃣ Object Reference and Comparison

### 🔹 Reference vs Content Comparison

* `==` → compares **references** (memory addresses)
* `.equals()` → compares **object content**

Example:

```java
String s1 = new String("Java");
String s2 = new String("Java");
System.out.println(s1 == s2);        // false
System.out.println(s1.equals(s2));   // true
```

✅ Always override `equals()` and `hashCode()` for custom objects:

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (!(obj instanceof Employee e)) return false;
    return Objects.equals(name, e.name) && salary == e.salary;
}

@Override
public int hashCode() {
    return Objects.hash(name, salary);
}
```

---

## 9️⃣ Immutability and Best Practices

Immutable classes are **thread-safe** and **easier to reason about**.

### 🔹 Creating an Immutable Class

```java
public final class User {
    private final String name;
    private final int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String name() { return name; }
    public int age() { return age; }
}
```

> 💡 Immutability is heavily promoted in **functional programming** and frameworks like **Spring Boot**.

---

## 🔟 Anonymous and Inner Classes

### 🔹 Inner Class

```java
class Outer {
    class Inner {
        void show() {
            System.out.println("Inner class");
        }
    }
}
```

### 🔹 Static Nested Class

```java
static class Logger {
    static void log(String msg) {
        System.out.println("[LOG] " + msg);
    }
}
```

### 🔹 Anonymous Class

Used for **one-time implementations**.

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running thread...");
    }
};
new Thread(r).start();
```

> ✅ Use **lambda expressions** instead of anonymous classes when possible.

---

## 11️⃣ Records in Java 21 (Data-Carrying Classes)

Introduced in Java 16 and refined in Java 21 — **records** are a concise way to define immutable data classes.

```java
public record Point(int x, int y) {}
```

Auto-generates:

* Constructor
* Getters
* `toString()`, `equals()`, `hashCode()`

Example:

```java
Point p = new Point(10, 20);
System.out.println(p.x()); // 10
System.out.println(p);     // Point[x=10, y=20]
```

✅ Benefits:

* Immutability by design
* Compact, readable code
* Ideal for DTOs and value objects

---

## 12️⃣ Summary

✅ You learned:

* How to create and manage classes & objects
* Constructors, static members, and initialization blocks
* Memory allocation, references, and garbage collection
* Immutability and record classes
* Best OOP practices for Java 21

> 🧭 **Next Topic:** [07-oop-concepts.md → Java 21 OOP Concepts: Abstraction, Inheritance, Polymorphism, Encapsulation](./07-oop-concepts.md)
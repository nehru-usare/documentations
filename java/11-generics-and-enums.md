# 🧬 Java 21 — Generics and Enums Deep Dive

Generics and Enums are two of Java’s most powerful language constructs.  
They help you write **type-safe**, **reusable**, and **extensible** code — essential in modern, enterprise-grade Java development.

In this guide, we’ll explore how **Java 21** uses generics and enums to promote **cleaner abstractions**, **compile-time safety**, and **functional-style programming**.

---

## 🧭 Table of Contents

1. [Introduction to Generics](#1-introduction-to-generics)
2. [Generic Classes and Methods](#2-generic-classes-and-methods)
3. [Bounded Type Parameters (`extends`, `super`)](#3-bounded-type-parameters-extends-super)
4. [Wildcard Usage and PECS Principle](#4-wildcard-usage-and-pecs-principle)
5. [Type Inference (`var`, `<>` Diamond Operator)](#5-type-inference-var--diamond-operator)
6. [Generic Records and Type-Safe Models (Java 21)](#6-generic-records-and-type-safe-models-java-21)
7. [Enum Basics and Advanced Features](#7-enum-basics-and-advanced-features)
8. [Enum with Methods and Fields](#8-enum-with-methods-and-fields)
9. [Enum Design Patterns (Strategy & Singleton)](#9-enum-design-patterns-strategy--singleton)
10. [Enum and Sealed Interfaces (Java 21)](#10-enum-and-sealed-interfaces-java-21)
11. [Best Practices for Generics and Enums](#11-best-practices-for-generics-and-enums)
12. [Summary](#12-summary)

---

## 1️⃣ Introduction to Generics

Generics enable **type-safe programming** — allowing you to write code that works for any data type **without losing compile-time type checking**.

### Without Generics:
```java
List list = new ArrayList();
list.add("Hello");
Integer num = (Integer) list.get(0); // ClassCastException
````

### With Generics:

```java
List<String> list = new ArrayList<>();
list.add("Hello");
// list.add(123); // Compile-time error
String str = list.get(0);
```

✅ Benefits:

* Compile-time type safety
* No explicit casting
* Cleaner, reusable code

---

## 2️⃣ Generic Classes and Methods

### 🔹 Generic Class Example

```java
public class Box<T> {
    private T value;

    public void set(T value) { this.value = value; }
    public T get() { return value; }
}
```

Usage:

```java
Box<String> box = new Box<>();
box.set("Java 21");
System.out.println(box.get());
```

---

### 🔹 Generic Method Example

```java
public static <T> void printArray(T[] array) {
    for (T item : array) System.out.print(item + " ");
}
```

Usage:

```java
String[] names = {"Nehru", "John", "Alice"};
printArray(names);
```

> 💡 `<T>` before return type declares a **generic method**, not just a method using generics.

---

## 3️⃣ Bounded Type Parameters (`extends`, `super`)

Bounds restrict the types that can be passed to a generic.

### 🔹 Upper Bound (`extends`)

```java
public static <T extends Number> double sum(T a, T b) {
    return a.doubleValue() + b.doubleValue();
}
```

Usage:

```java
System.out.println(sum(10, 20));       // int
System.out.println(sum(3.5, 2.5));     // double
```

### 🔹 Lower Bound (`super`)

Used primarily with wildcards:

```java
List<? super Integer> list = new ArrayList<Number>();
list.add(100); // allowed
```

✅ Ensures type safety in producer/consumer scenarios.

---

## 4️⃣ Wildcard Usage and PECS Principle

Wildcard generics allow flexibility when passing parameterized types.

| Wildcard      | Meaning                           |
| ------------- | --------------------------------- |
| `?`           | Unknown type                      |
| `? extends T` | Accepts T or subtype (producer)   |
| `? super T`   | Accepts T or supertype (consumer) |

### 🔹 Example:

```java
public static void printNumbers(List<? extends Number> list) {
    for (Number n : list) System.out.println(n);
}
```

### 🔹 PECS Rule:

> **Producer Extends, Consumer Super**

| Scenario                         | Use         |
| -------------------------------- | ----------- |
| You only read (produce) from it  | `? extends` |
| You only write (consume) into it | `? super`   |

---

## 5️⃣ Type Inference (`var`, `<>` Diamond Operator)

### 🔹 Diamond Operator (Java 7+)

```java
List<String> names = new ArrayList<>();
```

### 🔹 Type Inference with `var` (Java 10+)

```java
var map = new HashMap<String, Integer>();
map.put("Java", 21);
```

### 🔹 Combine Both:

```java
var box = new Box<Double>();
box.set(3.14);
```

> 💡 Type inference doesn’t make Java dynamically typed — types are still resolved at **compile-time**.

---

## 6️⃣ Generic Records and Type-Safe Models (Java 21)

Java 21 allows **generic records** — perfect for reusable DTOs and responses.

### Example:

```java
public record Response<T>(int status, String message, T data) {}
```

Usage:

```java
Response<String> res = new Response<>(200, "OK", "Success");
Response<List<Integer>> res2 = new Response<>(200, "OK", List.of(1, 2, 3));
```

✅ Ideal for REST responses, cache entries, and typed data containers.

> 💡 Combine with **sealed interfaces** for powerful, type-safe API designs.

---

## 7️⃣ Enum Basics and Advanced Features

Enums represent **fixed sets of constants**, but in Java they are **full-fledged classes**.

```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

Usage:

```java
Day today = Day.MONDAY;
switch (today) {
    case MONDAY -> System.out.println("Start of week");
    case FRIDAY -> System.out.println("Weekend soon!");
    default -> System.out.println("Midweek");
}
```

---

## 8️⃣ Enum with Methods and Fields

Enums can have **fields**, **constructors**, and **methods**.

```java
public enum Status {
    NEW("N"), IN_PROGRESS("P"), COMPLETED("C");

    private final String code;

    Status(String code) { this.code = code; }

    public String getCode() { return code; }
}
```

Usage:

```java
System.out.println(Status.NEW.getCode()); // N
```

✅ Each enum constant is an **instance** of the enum class.

---

## 9️⃣ Enum Design Patterns (Strategy & Singleton)

### 🔹 Strategy Pattern with Enum

```java
public enum Operation {
    ADD {
        public double apply(double x, double y) { return x + y; }
    },
    MULTIPLY {
        public double apply(double x, double y) { return x * y; }
    };

    public abstract double apply(double x, double y);
}
```

Usage:

```java
System.out.println(Operation.ADD.apply(2, 3)); // 5
```

---

### 🔹 Singleton Pattern with Enum

```java
public enum ConfigManager {
    INSTANCE;

    private final Properties props = new Properties();

    ConfigManager() {
        // Load config from file
    }

    public Properties getProperties() {
        return props;
    }
}
```

✅ Simplest thread-safe singleton (used internally in Spring, Hibernate).

---

## 🔟 Enum and Sealed Interfaces (Java 21)

Enums can now implement **sealed interfaces** for closed polymorphic hierarchies.

```java
sealed interface NotificationType permits Email, SMS {}

enum Email implements NotificationType { INSTANCE }
enum SMS implements NotificationType { INSTANCE }
```

✅ Benefits:

* Restricts extension
* Enables **switch pattern matching**
* Perfect for **business-rule enums** (PaymentType, RoleType, etc.)

---

## 11️⃣ Best Practices for Generics and Enums

✅ Prefer **composition with generics** over inheritance
✅ Use **bounded wildcards** carefully — avoid overuse of `?`
✅ Never use **raw types** (e.g., `List list`)
✅ Combine **records + generics** for immutable data models
✅ Prefer **enum singletons** over manually synchronized instances
✅ Avoid storing **mutable state** in enums
✅ Use **switch with pattern matching** for sealed-enum hierarchies
✅ Keep **generic type names short and meaningful** (`T`, `E`, `K`, `V`, `R`)

---

## 12️⃣ Summary

In this chapter, you’ve learned:

* Advanced generics with bounded types, wildcards, and inference
* Modern Java 21 features: generic records and sealed enums
* Enum patterns for real-world design (Strategy, Singleton)
* Type-safe API and DTO design for enterprise applications

> 🧭 **Next Topic:** [12-streams-and-functional-programming.md → Java 21 Streams, Lambdas, and Functional Paradigms](./12-streams-and-functional-programming.md)
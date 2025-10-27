# ☕ Java 21 — Language Basics and Syntax Guide

Understanding Java’s syntax and structure is the foundation of mastering the language.  
While Java’s core syntax has remained stable for decades, **Java 21** brings several modern enhancements that improve code readability, conciseness, and maintainability.

This guide provides a deep dive into **Java 21’s core syntax**, **data types**, **control structures**, and **modern conventions** every developer should master.

---

## 🧭 Table of Contents

1. [Java Program Structure](#1-java-program-structure)
2. [Identifiers and Keywords](#2-identifiers-and-keywords)
3. [Data Types in Java 21](#3-data-types-in-java-21)
4. [Variables and `var` Keyword](#4-variables-and-var-keyword)
5. [Constants and `final` Keyword](#5-constants-and-final-keyword)
6. [Operators and Expressions](#6-operators-and-expressions)
7. [Control Flow Statements](#7-control-flow-statements)
8. [Enhanced Switch Expression (Java 21)](#8-enhanced-switch-expression-java-21)
9. [Code Blocks, Scope, and Comments](#9-code-blocks-scope-and-comments)
10. [Naming Conventions and Best Practices](#10-naming-conventions-and-best-practices)
11. [Modern Syntax Additions in Java 21](#11-modern-syntax-additions-in-java-21)
12. [Summary](#12-summary)

---

## 1️⃣ Java Program Structure

A typical Java program follows a **class-based structure**:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Java 21!");
    }
}
````

### 🔹 Breakdown

* `public class HelloWorld` → Class declaration
* `main(String[] args)` → Program entry point
* `System.out.println()` → Prints output to console

Every Java program must have **at least one class**, and **main()** is the starting point of execution.

---

## 2️⃣ Identifiers and Keywords

### 🔹 Identifiers

Identifiers are names for classes, methods, variables, etc.
Rules:

* Must start with a **letter, $, or _**
* Cannot start with a digit
* Are **case-sensitive**
* Cannot be a Java keyword

✅ Examples:

```java
int age = 25;
String userName = "Nehru";
double _price = 299.99;
```

---

### 🔹 Keywords (Reserved Words)

Java has **67 reserved keywords** (as of Java 21).
Some important categories:

| Category                         | Keywords                                                    |
| -------------------------------- | ----------------------------------------------------------- |
| **Access Control**               | `public`, `private`, `protected`                            |
| **Class/Object**                 | `class`, `interface`, `extends`, `implements`, `new`        |
| **Modifiers**                    | `static`, `final`, `abstract`, `synchronized`, `volatile`   |
| **Control Flow**                 | `if`, `else`, `switch`, `while`, `for`, `break`, `continue` |
| **Error Handling**               | `try`, `catch`, `finally`, `throw`, `throws`                |
| **OOP**                          | `this`, `super`, `return`, `instanceof`                     |
| **Modern Keywords (Java 17–21)** | `record`, `sealed`, `permits`, `var`, `yield`               |

---

## 3️⃣ Data Types in Java 21

Java is **statically typed**, meaning every variable type is known at compile time.

### 🔹 Primitive Data Types

| Type      | Size   | Example                    | Default  | Range             |
| --------- | ------ | -------------------------- | -------- | ----------------- |
| `byte`    | 8-bit  | `byte b = 10;`             | 0        | -128 to 127       |
| `short`   | 16-bit | `short s = 1000;`          | 0        | -32,768 to 32,767 |
| `int`     | 32-bit | `int x = 100;`             | 0        | -2³¹ to 2³¹-1     |
| `long`    | 64-bit | `long id = 999999L;`       | 0L       | -2⁶³ to 2⁶³-1     |
| `float`   | 32-bit | `float rate = 3.5f;`       | 0.0f     | ±3.4e38           |
| `double`  | 64-bit | `double pi = 3.14159;`     | 0.0d     | ±1.7e308          |
| `char`    | 16-bit | `char grade = 'A';`        | '\u0000' | 0–65535           |
| `boolean` | 1-bit  | `boolean isActive = true;` | false    | true/false        |

### 🔹 Reference Types

* **Classes** → `String`, `Scanner`, `ArrayList`, etc.
* **Interfaces** → `List`, `Map`
* **Arrays** → `int[]`, `String[]`
* **Enums**, **Records**, **Objects**

> Java 21 continues to use UTF-16 internally for `char` and `String`.

---

## 4️⃣ Variables and `var` Keyword

### 🔹 Traditional Declaration

```java
int count = 10;
String name = "Alice";
```

### 🔹 Using `var` (Java 10+)

Introduced in **Java 10**, `var` allows **local variable type inference**:

```java
var list = new ArrayList<String>();
var message = "Hello, World!";
```

✅ Rules:

* Only for **local variables**
* Must be **initialized** when declared
* Type is **inferred** at compile time (not dynamic typing)

```java
var num = 10;      // int
var text = "Java"; // String
```

> 💡 `var` is *syntactic sugar* — it doesn’t make Java dynamically typed.

---

## 5️⃣ Constants and `final` Keyword

The `final` keyword makes a variable **immutable**.

```java
final double PI = 3.14159;
final String APP_NAME = "JavaDocs";
```

* `final` variables cannot be reassigned.
* Best practice: use **UPPERCASE naming** for constants.

```java
final int MAX_THREADS = 10;
```

---

## 6️⃣ Operators and Expressions

### 🔹 Arithmetic Operators

`+`, `-`, `*`, `/`, `%`

```java
int result = (10 + 5) * 3;  // 45
```

### 🔹 Comparison Operators

`==`, `!=`, `<`, `>`, `<=`, `>=`

### 🔹 Logical Operators

`&&`, `||`, `!`

### 🔹 Assignment Operators

`=`, `+=`, `-=`, `*=`, `/=`, `%=`

```java
count += 5; // equivalent to count = count + 5
```

### 🔹 Ternary Operator

```java
String status = (age >= 18) ? "Adult" : "Minor";
```

---

## 7️⃣ Control Flow Statements

Java offers **deterministic control flow constructs** for conditional and iterative operations.

### 🔹 Conditional

```java
if (score >= 90) {
    System.out.println("Excellent");
} else if (score >= 75) {
    System.out.println("Good");
} else {
    System.out.println("Try Again");
}
```

### 🔹 Loops

**For Loop:**

```java
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}
```

**Enhanced For (For-Each):**

```java
for (String item : list) {
    System.out.println(item);
}
```

**While Loop:**

```java
while (count < 10) {
    count++;
}
```

**Do-While:**

```java
do {
    count++;
} while (count < 10);
```

---

## 8️⃣ Enhanced Switch Expression (Java 21)

Java 14 introduced **switch expressions**, and Java 21 enhances them further with **pattern matching**.

### 🔹 Traditional Switch (Old)

```java
switch (day) {
    case 1: System.out.println("Monday"); break;
    default: System.out.println("Other day");
}
```

### 🔹 Modern Switch Expression

```java
String result = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    default -> "Weekend";
};
```

### 🔹 Pattern Matching with Switch (Java 21)

```java
static String formatObject(Object obj) {
    return switch (obj) {
        case Integer i -> "Integer: " + i;
        case String s -> "String: " + s.toUpperCase();
        case null -> "Null value";
        default -> "Unknown type";
    };
}
```

> 💡 Java 21 allows **null handling**, **type matching**, and **sealed class compatibility** directly in switch statements.

---

## 9️⃣ Code Blocks, Scope, and Comments

### 🔹 Code Blocks

Curly braces `{}` define scope for variables and logic.

### 🔹 Comments

```java
// Single line comment
/* Multi-line
   comment */
/**
 * Javadoc comment for methods or classes
 */
```

> 🧩 Use Javadoc comments to generate documentation:

```bash
javadoc MyClass.java
```

---

## 🔟 Naming Conventions and Best Practices

| Element       | Convention       | Example               |
| ------------- | ---------------- | --------------------- |
| **Classes**   | PascalCase       | `EmployeeDetails`     |
| **Methods**   | camelCase        | `calculateSalary()`   |
| **Variables** | camelCase        | `employeeCount`       |
| **Constants** | UPPER_SNAKE_CASE | `MAX_LIMIT`           |
| **Packages**  | lowercase        | `com.company.project` |

✅ Follow **Google Java Style** or **Oracle Code Conventions**.
✅ Use meaningful variable names.
✅ Avoid abbreviations (e.g., `numOfEmp` → `numberOfEmployees`).

---

## 11️⃣ Modern Syntax Additions in Java 21

| Feature                               | Introduced In | Example                         |
| ------------------------------------- | ------------- | ------------------------------- |
| **Text Blocks**                       | Java 15       | `"""Multiline String"""`        |
| **Records**                           | Java 16       | `record Point(int x, int y) {}` |
| **Sealed Classes**                    | Java 17       | Restrict inheritance hierarchy  |
| **Pattern Matching for `instanceof`** | Java 16       | `if (obj instanceof String s)`  |
| **Switch Pattern Matching**           | Java 21       | Advanced type-safe `switch`     |
| **Unnamed Variables**                 | Java 21       | `_` to ignore unused variables  |

### Example:

```java
record User(String name, int age) {}

var u = new User("Nehru", 28);
System.out.println(u.name()); // Compact record accessor
```

---

## 12️⃣ Summary

✅ You’ve learned:

* Java 21’s syntax and structure
* Data types and variables (`var`, `final`)
* Control flow, operators, and expressions
* Enhanced switch with pattern matching
* Modern language constructs like `record`, `text blocks`, `sealed classes`

> 🧭 **Next Topic:** [05-operators-and-control-flow.md → Java 21 Control Flow and Operators Deep Dive](./05-operators-and-control-flow.md)
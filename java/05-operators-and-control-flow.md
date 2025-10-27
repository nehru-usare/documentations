# ⚙️ Java 21 Operators and Control Flow Masterclass

Java’s power lies not only in its syntax but in its **expressive control flow and operator system**.  
As a professional Java developer, understanding how Java evaluates expressions, manages branching, and optimizes loops is essential for writing efficient, maintainable code.

This chapter provides an in-depth exploration of **Java 21 operators**, **expressions**, and **control structures**.

---

## 🧭 Table of Contents

1. [Understanding Expressions and Evaluation](#1-understanding-expressions-and-evaluation)
2. [Java Operator Categories](#2-java-operator-categories)
3. [Operator Precedence and Associativity](#3-operator-precedence-and-associativity)
4. [Assignment and Compound Operators](#4-assignment-and-compound-operators)
5. [Conditional and Logical Operators](#5-conditional-and-logical-operators)
6. [Bitwise and Shift Operators](#6-bitwise-and-shift-operators)
7. [Control Flow Statements](#7-control-flow-statements)
8. [Branching (if-else, switch, yield)](#8-branching-if-else-switch-yield)
9. [Looping Constructs](#9-looping-constructs)
10. [Break, Continue, and Label Statements](#10-break-continue-and-label-statements)
11. [Modern Switch (Pattern Matching)](#11-modern-switch-pattern-matching)
12. [Best Practices](#12-best-practices)
13. [Summary](#13-summary)

---

## 1️⃣ Understanding Expressions and Evaluation

An **expression** is a combination of variables, literals, and operators that produces a value.

Example:
```java
int result = (a + b) * c / 2;
````

* Java evaluates expressions **left-to-right** following **operator precedence**.
* Parentheses `()` can override default precedence.

> 💡 Always use parentheses for clarity in complex mathematical or logical expressions.

---

## 2️⃣ Java Operator Categories

| Category           | Operators            | Example                        |          |               |
| ------------------ | -------------------- | ------------------------------ | -------- | ------------- |
| **Arithmetic**     | `+ - * / %`          | `int sum = a + b;`             |          |               |
| **Unary**          | `++ -- + - !`        | `count++; !flag`               |          |               |
| **Assignment**     | `= += -= *= /= %=`   | `a += 5;`                      |          |               |
| **Comparison**     | `== != > < >= <=`    | `if (x == y)`                  |          |               |
| **Logical**        | `&&                  |                                | !`       | `if (a && b)` |
| **Bitwise**        | `&                   | ^ ~ << >> >>>`                 | `n << 2` |               |
| **Ternary**        | `?:`                 | `max = (a > b) ? a : b;`       |          |               |
| **Type**           | `instanceof`, `cast` | `obj instanceof String`        |          |               |
| **New (Java 16+)** | `yield`              | Used inside switch expressions |          |               |

---

## 3️⃣ Operator Precedence and Associativity

Operator **precedence** defines evaluation order; **associativity** defines tie-breaking when precedence is equal.

| Precedence  | Operator                | Associativity |               |               |
| ----------- | ----------------------- | ------------- | ------------- | ------------- |
| 1 (Highest) | `()` `[]` `.`           | Left to Right |               |               |
| 2           | `++ --`                 | Right to Left |               |               |
| 3           | `* / %`                 | Left to Right |               |               |
| 4           | `+ -`                   | Left to Right |               |               |
| 5           | `<< >> >>>`             | Left to Right |               |               |
| 6           | `< > <= >= instanceof`  | Left to Right |               |               |
| 7           | `== !=`                 | Left to Right |               |               |
| 8           | `&`                     | Left to Right |               |               |
| 9           | `^`                     | Left to Right |               |               |
| 10          | `                       | `             | Left to Right |               |
| 11          | `&&`                    | Left to Right |               |               |
| 12          | `                       |               | `             | Left to Right |
| 13          | `?:`                    | Right to Left |               |               |
| 14 (Lowest) | `=` `+=` `-=` `*=` `/=` | Right to Left |               |               |

> 🧩 Always remember: Arithmetic → Comparison → Logical → Assignment

---

## 4️⃣ Assignment and Compound Operators

Basic assignment:

```java
int x = 5;
```

Compound assignment:

```java
x += 10;  // same as x = x + 10
x *= 2;   // same as x = x * 2
```

Works with all numeric and compatible types:

```java
double result = 10.5;
result += 2; // implicit cast to double
```

---

## 5️⃣ Conditional and Logical Operators

### 🔹 Comparison Operators

Used to compare primitive values.

```java
if (a != b) { ... }
if (score >= 90) { ... }
```

### 🔹 Logical Operators

| Operator | Meaning     | Example               |            |            |   |         |
| -------- | ----------- | --------------------- | ---------- | ---------- | - | ------- |
| `&&`     | Logical AND | `if (x > 0 && y > 0)` |            |            |   |         |
| `        |             | `                     | Logical OR | `if (x > 0 |   | y > 0)` |
| `!`      | Logical NOT | `if (!isValid)`       |            |            |   |         |

### 🔹 Short-Circuiting

Java uses **short-circuit evaluation**:

```java
if (obj != null && obj.isActive()) // avoids NullPointerException
```

---

## 6️⃣ Bitwise and Shift Operators

Bitwise operators operate directly on **binary bits**.

| Operator | Description          | Example   |    |    |
| -------- | -------------------- | --------- | -- | -- |
| `&`      | AND                  | `a & b`   |    |    |
| `        | `                    | OR        | `a | b` |
| `^`      | XOR                  | `a ^ b`   |    |    |
| `~`      | NOT                  | `~a`      |    |    |
| `<<`     | Left shift           | `x << 2`  |    |    |
| `>>`     | Right shift          | `x >> 1`  |    |    |
| `>>>`    | Unsigned right shift | `x >>> 1` |    |    |

Example:

```java
int n = 5; // 0101
int shifted = n << 1; // 1010 = 10
```

> ⚙️ Bitwise operators are commonly used in low-level operations, enums, and flag systems.

---

## 7️⃣ Control Flow Statements

Control flow defines **how program execution proceeds**.

### Categories:

* **Decision Making:** `if`, `switch`
* **Looping:** `for`, `while`, `do-while`, `for-each`
* **Branching:** `break`, `continue`, `return`

---

## 8️⃣ Branching (if-else, switch, yield)

### 🔹 If-Else Example

```java
if (balance > 0) {
    System.out.println("Active account");
} else {
    System.out.println("Inactive account");
}
```

### 🔹 Nested Conditions

```java
if (score >= 90) {
    grade = "A";
} else if (score >= 75) {
    grade = "B";
} else {
    grade = "C";
}
```

### 🔹 Switch Expression with `yield`

In **Java 14+**, switch became an **expression** (returns a value).

```java
String category = switch (day) {
    case 1, 2, 3, 4, 5 -> "Weekday";
    case 6, 7 -> "Weekend";
    default -> {
        yield "Invalid";
    }
};
```

> 💡 Use `yield` inside block cases to return a value.

---

## 9️⃣ Looping Constructs

### 🔹 For Loop

```java
for (int i = 0; i < 5; i++) {
    System.out.println("Iteration " + i);
}
```

### 🔹 Enhanced For Loop

For collections or arrays:

```java
for (String name : names) {
    System.out.println(name);
}
```

### 🔹 While Loop

```java
while (count < 10) {
    count++;
}
```

### 🔹 Do-While Loop

```java
do {
    System.out.println("Executed at least once");
} while (condition);
```

---

## 🔟 Break, Continue, and Label Statements

### 🔹 break

Terminates current loop or switch:

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) break;
}
```

### 🔹 continue

Skips the current iteration:

```java
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue; // skips even numbers
}
```

### 🔹 Labeled Loops

Used in nested loops for fine-grained control:

```java
outer:
for (int i = 1; i <= 3; i++) {
    for (int j = 1; j <= 3; j++) {
        if (i == j) break outer;
    }
}
```

---

## 11️⃣ Modern Switch (Pattern Matching)

Pattern Matching in **Java 21** allows type-safe, concise branching.

### Example:

```java
static String describe(Object obj) {
    return switch (obj) {
        case Integer i -> "Integer: " + i;
        case String s when s.length() > 5 -> "Long String: " + s;
        case String s -> "Short String: " + s;
        case null -> "Null value";
        default -> "Unknown type";
    };
}
```

✅ Features:

* Supports `when` guards (conditional filters)
* Handles `null` safely
* Integrates with **records** and **sealed classes**

---

## 12️⃣ Best Practices

✅ Prefer **switch expressions** for readability and immutability
✅ Use **pattern matching** for cleaner type checks
✅ Avoid deep nested if-else; use guard clauses
✅ Always handle `null` safely
✅ For loops — prefer `for-each` for collections
✅ Use `final` for loop variables where possible

> 🧩 Clean control flow improves testability, readability, and performance.

---

## 13️⃣ Summary

✅ You learned about:

* All major Java operators and their precedence
* Conditional and logical expressions
* Modern switch and `yield`
* Looping and branching structures
* Java 21 pattern-matching enhancements

> 🧭 **Next Topic:** [06-classes-and-objects.md → Java 21 Classes, Objects, and Object Lifecycle](./06-classes-and-objects.md)
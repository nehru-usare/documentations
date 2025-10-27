# 🧭 Java 21 — Packages, Access Modifiers, and Modular Design

In large-scale enterprise systems, code organization and access control are as important as the code itself.  
This document provides an advanced-level understanding of **packages**, **access modifiers**, and **module system (JPMS)** in Java 21 — helping experienced developers build **clean, modular, and maintainable architectures**.

---

## 🧾 Table of Contents

1. [Introduction to Packages](#1-introduction-to-packages)
2. [Creating and Using Packages](#2-creating-and-using-packages)
3. [Importing Classes and Static Imports](#3-importing-classes-and-static-imports)
4. [Access Modifiers in Java](#4-access-modifiers-in-java)
5. [Encapsulation and Access Boundaries](#5-encapsulation-and-access-boundaries)
6. [Java Platform Module System (JPMS)](#6-java-platform-module-system-jpms)
7. [module-info.java Explained](#7-module-infojava-explained)
8. [Practical Modular Example](#8-practical-modular-example)
9. [Package-by-Feature vs Package-by-Layer Design](#9-package-by-feature-vs-package-by-layer-design)
10. [Best Practices for Enterprise Projects](#10-best-practices-for-enterprise-projects)
11. [Summary](#11-summary)

---

## 1️⃣ Introduction to Packages

A **package** in Java is a **namespace** that groups related classes and interfaces together.  
They help prevent **naming conflicts**, control **access**, and improve **code maintainability**.

Think of packages as **folders** in your file system:
```

com.companyname.projectname.module

````

Example:
```java
package com.techdocs.employee;

public class Employee {
    private String name;
}
````

---

## 2️⃣ Creating and Using Packages

### 🔹 Defining a Package

At the top of a `.java` file:

```java
package com.company.app.models;
```

### 🔹 Compiling with Packages

```bash
javac -d . Employee.java
```

### 🔹 Directory Structure

```
src/
 └── com/
     └── company/
         └── app/
             └── models/
                 └── Employee.java
```

---

## 3️⃣ Importing Classes and Static Imports

### 🔹 Regular Import

```java
import java.util.List;
```

### 🔹 Wildcard Import

```java
import java.util.*;
```

### 🔹 Static Import (Java 5+)

Lets you import static members directly:

```java
import static java.lang.Math.*;
System.out.println(PI); // instead of Math.PI
```

> 💡 Static imports improve readability in utility-heavy code (like testing or math operations).

---

## 4️⃣ Access Modifiers in Java

Access modifiers define **who can see or use** a class, method, or field.

| Modifier                      | Within Class | Same Package | Subclass (diff pkg) | Other Packages |
| ----------------------------- | ------------ | ------------ | ------------------- | -------------- |
| **private**                   | ✅            | ❌            | ❌                   | ❌              |
| **default (package-private)** | ✅            | ✅            | ❌                   | ❌              |
| **protected**                 | ✅            | ✅            | ✅                   | ❌              |
| **public**                    | ✅            | ✅            | ✅                   | ✅              |

---

### 🔹 Class-Level Access

* **Top-level classes** can only be `public` or *default (package-private)*.
* `private` and `protected` are **not allowed** for top-level classes.

### 🔹 Member-Level Access

```java
public class User {
    private String name;      // accessible only within this class
    protected int age;        // accessible to subclasses
    String email;             // package-private
    public void showDetails() {}
}
```

✅ **Encapsulation principle:**
Keep variables **private**, expose through **getters/setters** or **records**.

---

## 5️⃣ Encapsulation and Access Boundaries

Encapsulation ensures data integrity by restricting access to internal implementation.

### Example:

```java
public class Account {
    private double balance;

    public void deposit(double amount) {
        if (amount > 0) balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

> ✅ Data hiding + controlled access → better maintainability and security.

### Enterprise Use Case:

Use **package-private classes** for internal logic (repositories, mappers) that should not leak outside the module.

---

## 6️⃣ Java Platform Module System (JPMS)

Introduced in **Java 9**, **JPMS** modularized the JDK itself.
It allows developers to:

* **Divide applications into modules**
* **Control inter-module dependencies**
* **Encapsulate internal packages**
* **Enable compile-time safety**

### 🔹 Benefits:

* Improved startup performance
* Reduced memory footprint
* Clear API boundaries
* Easier dependency management

---

## 7️⃣ `module-info.java` Explained

Each module defines a `module-info.java` file at its root.

Example:

```java
module com.company.app {
    exports com.company.app.service;
    requires com.company.app.model;
}
```

### 🔹 Keywords

| Keyword             | Meaning                                                       |
| ------------------- | ------------------------------------------------------------- |
| `exports`           | Makes a package accessible to other modules                   |
| `requires`          | Declares dependency on another module                         |
| `opens`             | Allows reflection (used by frameworks like Spring, Hibernate) |
| `uses`              | Declares a service interface for ServiceLoader                |
| `provides ... with` | Defines a service implementation                              |

---

### 🔹 Example Project Structure

```
src/
 ├── module-info.java
 ├── com.company.app.service/
 │   └── PaymentService.java
 └── com.company.app.model/
     └── Payment.java
```

`module-info.java`

```java
module com.company.app {
    exports com.company.app.service;
    requires com.company.app.model;
}
```

---

## 8️⃣ Practical Modular Example

### Module 1 — Model

```java
// module-info.java
module com.techdocs.model {
    exports com.techdocs.model;
}

// com/techdocs/model/Employee.java
package com.techdocs.model;
public record Employee(String id, String name) {}
```

### Module 2 — Service

```java
// module-info.java
module com.techdocs.service {
    requires com.techdocs.model;
    exports com.techdocs.service;
}

// com/techdocs/service/EmployeeService.java
package com.techdocs.service;
import com.techdocs.model.Employee;

public class EmployeeService {
    public void print(Employee e) {
        System.out.println("Employee: " + e.name());
    }
}
```

### Run:

```bash
java --module-path out --module com.techdocs.service/com.techdocs.service.EmployeeService
```

✅ Clear module boundaries
✅ No circular dependencies
✅ Compile-time modular validation

---

## 9️⃣ Package-by-Feature vs Package-by-Layer Design

| Approach               | Description                                               | Example                           |
| ---------------------- | --------------------------------------------------------- | --------------------------------- |
| **Package-by-Layer**   | Organize by architecture layer (controller, service, dao) | `com.app.controller`              |
| **Package-by-Feature** | Organize by business capability                           | `com.app.orders`, `com.app.users` |

> 💡 For microservices or modular monoliths, **package-by-feature** is recommended.

Example:

```
com.app.orders
 ├── OrderController.java
 ├── OrderService.java
 ├── OrderRepository.java
```

✅ Promotes modularity
✅ Easier feature ownership
✅ Simplifies dependency management

---

## 🔟 Best Practices for Enterprise Projects

✅ Use **JPMS modules** for large systems
✅ Keep **internal APIs package-private**
✅ Use **sealed classes** for domain modeling
✅ Use **package-by-feature** for modern architectures
✅ Export only **public-facing packages**
✅ Avoid cyclic dependencies between modules
✅ Use **dependency injection frameworks** (Spring, Micronaut) for modular decoupling
✅ Document module boundaries with `module-info.java`

---

## 11️⃣ Summary

In this chapter, you learned how to:

* Design clean **package structures**
* Apply proper **access control**
* Use **JPMS** for modular architecture
* Understand the difference between **package vs module**
* Organize enterprise-level projects efficiently

✅ Mastering packages and access boundaries leads to **scalable**, **maintainable**, and **testable** applications — a key skill for any senior Java developer.

> 🧭 **Next Topic:** [10-exception-handling.md → Advanced Exception Handling in Java 21 (Checked, Unchecked, and Best Practices)](./10-exception-handling.md)
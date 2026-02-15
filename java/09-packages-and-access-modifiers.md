# 🧭 Packages & Modularity: Architecting for Scale

> **Document Level:** Architect (15+ years experience)  
> **Focus:** JPMS Internals (Jigsaw), Encapsulation Patterns, and Hexagonal Architecture

---

## 🏗️ Layer 1: The Modular Evolution - JPMS (Project Jigsaw)

Java 9 introduced the most significant architectural change in Java history: **The Module System**.

### 1. Why Jigsaw?
Before Java 9, the "Classpath" was a flat mess. Any class could access any public class in any other JAR. This led to **"Jar Hell"** and security vulnerabilities (e.g., people using internal classes like `sun.misc.Unsafe`).

### 2. `module-info.java` Mechanics
-   **`requires`**: Declares a dependency.
-   **`exports`**: Makes a package available to other modules at **compile-time**.
-   **`opens`**: Makes a package available for **deep reflection** at runtime (Crucial for Spring/Hibernate).
-   **`provides / uses`**: Implements the Service Loader pattern for decoupling.

---

## 🛠️ Layer 2: Access Control as an Architectural Shield

### 1. The Power of "Package-Private" (Default)
Most developers make everything `public`. This is a mistake.
- **Architect Rule**: Keep classes **Package-Private** unless they are intended to be entry points for your module. This allows you to refactor internal logic without breaking external consumers.

### 2. The `protected` Trap
`protected` is often misused for "Ease of access". 
- **The Reality**: It creates tight coupling via inheritance. Architects prefer **Composition** and `public` interfaces over `protected` hooks.

---

## 🚀 Layer 3: Design Patterns - Hexagonal & Layered

### 1. Package-by-Layer vs. Package-by-Feature
-   **By-Layer**: `com.app.controller`, `com.app.service`, `com.app.repository`. (Hard to maintain as the app grows).
-   **By-Feature**: `com.app.orders`, `com.app.billing`, `com.app.users`. (Each package is a mini-contained world).

### 2. Hexagonal (Ports and Adapters)
-   **Domain Core**: In the center (The purest code).
-   **Ports**: Interfaces defining what the core needs.
-   **Adapters**: External implementations (SQL, Kafka, REST).
-   **Architect Benefit**: You can swap your database or messaging system without touching a single line of business logic.

---

## 📜 Layer 4: ClassLoader Isolation

In large systems (like Application Servers or IDEs), we need multiple versions of the same library (e.g., Log4j 1.x and 2.x) running simultaneously.
- **The Solution**: **Custom ClassLoaders**. By creating a hierarchy where each module has its own ClassLoader, we can isolate dependencies and prevent "LinkageErrors".

---

## 🏁 Layer 5: The "Unnamed Module" & Backward Compatibility

If you don't define a `module-info.java`, your code lives in the **Unnamed Module**.
- **Automatic Modules**: JARs on the module-path without a module-info are treated as automatic modules, with their name derived from the JAR filename.
- **Architect Impact**: This allows a gradual migration from Classpath to Module-path.

---

## 🧭 Interview Prep & Architect Scenarios

### Q: What is the difference between `exports` and `exports... to`?
**A**: `exports` makes a package public to *everyone*. `exports... to` is a **Qualified Export**, allowing you to restrict access to a specific "Friend" module (e.g., internal testing or monitoring tools).

### Q: Why can't a module see its own `requires transitive` dependencies?
**A**: It can. `requires transitive` means that if Module A requires Module B, any module that requires A will **automatically** require B as well. It is used to simplify the dependency graph for consumers.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [08-advanced-oop-topics.md](./08-advanced-oop-topics.md) | Advanced OOP |
| ⏩ **Next** | [10-exception-handling.md](./10-exception-handling.md) | Exceptions |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026
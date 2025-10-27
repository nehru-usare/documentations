# 🏗️ Java 21 — Architecture and Design Patterns

> “Good architecture is not about frameworks — it’s about discipline.”  
> — Uncle Bob Martin

Modern Java architecture emphasizes **modularity**, **clean design**, and **testability**.  
This document serves as a **practical guide for system architects and senior developers** to design robust, scalable Java systems using Java 21 features.

---

## 🧭 Table of Contents

1. [Architectural Overview](#1-architectural-overview)
2. [Layered Architecture](#2-layered-architecture)
3. [Hexagonal (Ports & Adapters) Architecture](#3-hexagonal-ports--adapters-architecture)
4. [Clean Architecture Principles](#4-clean-architecture-principles)
5. [Domain-Driven Design (DDD)](#5-domain-driven-design-ddd)
6. [Microservice Design Best Practices](#6-microservice-design-best-practices)
7. [Modularization and Java Platform Module System (JPMS)](#7-modularization-and-java-platform-module-system-jpms)
8. [Creational Design Patterns](#8-creational-design-patterns)
9. [Structural Design Patterns](#9-structural-design-patterns)
10. [Behavioral Design Patterns](#10-behavioral-design-patterns)
11. [Reactive and Asynchronous Design Patterns](#11-reactive-and-asynchronous-design-patterns)
12. [Best Practices Summary](#12-best-practices-summary)
13. [Official Resources and References](#13-official-resources-and-references)

---

## 1️⃣ Architectural Overview

Modern enterprise Java systems typically follow:
- **Layered architecture** → clear separation between layers
- **Hexagonal/Clean architecture** → independence from frameworks
- **Domain-driven design** → business logic at the core
- **Microservice patterns** → modular and autonomous services

✅ **Goals of good architecture:**
- Maintainability  
- Testability  
- Scalability  
- Separation of concerns  
- Technology independence  

---

## 2️⃣ Layered Architecture

The **traditional architecture** most enterprise apps start with.

### Layers:
1. **Presentation Layer (Web/UI)**  
   - Handles REST controllers or UI interactions.
2. **Service Layer (Business Logic)**  
   - Encapsulates domain logic and rules.
3. **Repository Layer (Persistence)**  
   - Handles database operations.
4. **Domain Layer (Entities + Value Objects)**  
   - Represents core business data.

```text
Controller → Service → Repository → Database
````

✅ Easy to understand
✅ Works well for CRUD apps
❌ Harder to scale for complex domains

### Example:

```java
@RestController
@RequestMapping("/users")
class UserController {
    private final UserService service;
    UserController(UserService service) { this.service = service; }

    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getUser(@PathVariable String id) {
        return ResponseEntity.ok(service.getUser(id));
    }
}
```

---

## 3️⃣ Hexagonal (Ports & Adapters) Architecture

Introduced by **Alistair Cockburn**, this architecture isolates **core logic** from **external systems**.

### 🔹 Key Concepts:

* **Ports** — Interfaces for input/output boundaries
* **Adapters** — Implement those interfaces for technologies (DB, API, UI)
* **Domain** — The pure business logic

```
         +-----------------------+
         |   Application Core    |
         |  (Ports + Domain)     |
         +-----------+-----------+
                     |
        +------------+------------+
        | Adapters: REST, DB, MQ  |
        +-------------------------+
```

✅ Technology-agnostic
✅ Highly testable
✅ Easier refactoring (swap DB or API)

### Example:

```java
interface UserRepositoryPort {
    User findById(String id);
}

class JpaUserRepositoryAdapter implements UserRepositoryPort {
    private final UserJpaRepository repo;
    public User findById(String id) {
        return repo.findById(id).orElseThrow();
    }
}
```

---

## 4️⃣ Clean Architecture Principles

Promoted by **Robert C. Martin (Uncle Bob)**.

* **Entities (Core domain)** – independent business logic
* **Use cases** – interact with domain logic
* **Interface adapters** – convert between domain and framework
* **Frameworks/drivers** – external tools (Spring, DB, Web)

```text
Frameworks → Adapters → Use Cases → Entities
```

✅ Framework independence
✅ Testable use cases
✅ Business rules don’t depend on infrastructure

> 💡 Core rule: *“Dependencies always point inward.”*

---

## 5️⃣ Domain-Driven Design (DDD)

DDD focuses on modeling **complex business domains** using **ubiquitous language**.

### Building Blocks:

* **Entity** — Identity + Lifecycle
* **Value Object** — Immutable, no identity
* **Aggregate** — Cluster of entities treated as one
* **Repository** — Retrieve aggregates
* **Service** — Domain operation

### Example:

```java
record Money(BigDecimal amount, String currency) {}
class Account {
    private final String id;
    private Money balance;

    void deposit(Money money) {
        balance = new Money(balance.amount().add(money.amount()), balance.currency());
    }
}
```

> 🧩 Use DDD for **complex**, business-heavy systems, not CRUD APIs.

---

## 6️⃣ Microservice Design Best Practices

✅ One service = one bounded context
✅ Use asynchronous communication (Kafka, RabbitMQ)
✅ Maintain database per service
✅ Keep APIs versioned
✅ Implement distributed tracing
✅ Deploy independently

> ⚙️ Use lightweight frameworks — **Spring Boot, Quarkus, Micronaut**
> 💡 Prefer **GraalVM Native Image** for fast startup microservices.

---

## 7️⃣ Modularization and Java Platform Module System (JPMS)

Introduced in Java 9, modules enforce boundaries inside monoliths.

### Example: `module-info.java`

```java
module com.nehrudev.user {
    exports com.nehrudev.user.api;
    requires com.nehrudev.common;
}
```

✅ Enforces encapsulation
✅ Improves maintainability and security
✅ Enables better startup and footprint control

> 💡 Use JPMS for large codebases to avoid cyclic dependencies.

---

## 8️⃣ Creational Design Patterns

| Pattern              | Purpose                     | Java 21 Example      |
| -------------------- | --------------------------- | -------------------- |
| **Singleton**        | One instance only           | Enum-based singleton |
| **Factory Method**   | Centralized object creation | Static factory       |
| **Builder**          | Stepwise construction       | Fluent Builder       |
| **Prototype**        | Clone existing objects      | `clone()`            |
| **Abstract Factory** | Family of related objects   | Dependency injection |

### Example (Builder Pattern):

```java
record User(String name, int age) {
    static class Builder {
        private String name;
        private int age;
        Builder name(String n) { this.name = n; return this; }
        Builder age(int a) { this.age = a; return this; }
        User build() { return new User(name, age); }
    }
}
```

---

## 9️⃣ Structural Design Patterns

| Pattern       | Use Case                               | Example            |
| ------------- | -------------------------------------- | ------------------ |
| **Adapter**   | Bridge between incompatible interfaces | Legacy integration |
| **Decorator** | Add features dynamically               | Logging decorator  |
| **Composite** | Hierarchical structures                | Filesystem nodes   |
| **Facade**    | Simplify subsystem access              | Service Facade     |
| **Proxy**     | Control access                         | Security proxy     |

### Example (Decorator Pattern):

```java
interface Notifier { void send(String message); }

class EmailNotifier implements Notifier {
    public void send(String message) { System.out.println("Email: " + message); }
}

class SlackNotifier implements Notifier {
    private final Notifier notifier;
    SlackNotifier(Notifier n) { this.notifier = n; }
    public void send(String message) {
        notifier.send(message);
        System.out.println("Slack: " + message);
    }
}
```

---

## 🔟 Behavioral Design Patterns

| Pattern                     | Description                   |
| --------------------------- | ----------------------------- |
| **Strategy**                | Select algorithm dynamically  |
| **Observer**                | Event-driven communication    |
| **Command**                 | Encapsulate requests          |
| **Template Method**         | Skeleton algorithm with steps |
| **Chain of Responsibility** | Pass request through handlers |

### Example (Strategy Pattern):

```java
interface PaymentStrategy { void pay(double amount); }
class CreditCardPayment implements PaymentStrategy {
    public void pay(double amount) { System.out.println("Paid via Card: " + amount); }
}
class UpiPayment implements PaymentStrategy {
    public void pay(double amount) { System.out.println("Paid via UPI: " + amount); }
}
class PaymentProcessor {
    private final PaymentStrategy strategy;
    PaymentProcessor(PaymentStrategy s) { this.strategy = s; }
    void process(double amount) { strategy.pay(amount); }
}
```

---

## 11️⃣ Reactive and Asynchronous Design Patterns

✅ Use **CompletableFuture**, **Virtual Threads**, or **Project Reactor** for async flows.
✅ Avoid blocking calls; prefer non-blocking I/O.
✅ Implement backpressure and circuit breakers.

Example using **CompletableFuture**:

```java
CompletableFuture.supplyAsync(() -> service.fetchData())
    .thenApply(data -> process(data))
    .thenAccept(System.out::println);
```

✅ Combine with **Structured Concurrency** for better control in Java 21.

---

## 12️⃣ Best Practices Summary

✅ Keep business logic framework-agnostic
✅ Use **sealed interfaces + records** for clean domains
✅ Avoid God classes and circular dependencies
✅ Make services stateless where possible
✅ Prefer **dependency injection (DI)** over singletons
✅ Always **log, monitor, and profile** before optimizing
✅ Use **architectural fitness functions** in CI/CD pipelines

> 🧠 Simplicity scales better than cleverness.

---

## 13️⃣ Official Resources and References

📘 **Architecture**

* [Clean Architecture – Uncle Bob Martin](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)
* [Hexagonal Architecture – Alistair Cockburn](https://alistair.cockburn.us/hexagonal-architecture/)
* [Domain-Driven Design – Eric Evans](https://domainlanguage.com/)
* [Building Evolutionary Architectures – Neal Ford, ThoughtWorks](https://evolutionaryarchitecture.com/)

📊 **Design Patterns**

* [Refactoring.Guru Patterns Catalog](https://refactoring.guru/design-patterns)
* [Java Design Patterns Repository](https://github.com/iluwatar/java-design-patterns)

💻 **Java 21**

* [Java 21 Docs – Oracle](https://docs.oracle.com/en/java/javase/21/)
* [Project Loom (Virtual Threads)](https://openjdk.org/projects/loom/)
* [Project Amber (Records, Sealed Classes, Pattern Matching)](https://openjdk.org/projects/amber/)

---

## 🧩 Summary

You now understand how to:

* Architect clean, modular, and scalable Java systems
* Apply modern design patterns effectively
* Use Java 21 features (records, sealed classes, virtual threads) to simplify architecture
* Build frameworks-agnostic, maintainable applications

> ⚙️ **Architecture is about boundaries and clarity — not complexity.**
> Modern Java gives you the tools to make systems both elegant and practical.

> 🧭 **Next (optional advanced)**:
> [22-enterprise-java-case-studies.md → Real-World Architecture Case Studies and Best Practices in Java 21](./22-enterprise-java-case-studies.md)
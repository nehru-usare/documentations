# 🏛️ Java Architecture & Design Patterns: Solving Complexity

> **Document Level:** Architect (15+ years experience)  
> **Focus:** SOLID at Scale, Modern Strategy/Template, Hexagonal Deep Dive, and Event-Driven Patterns

---

## 🏗️ Layer 1: The Modern Strategy & Template Patterns

Java 8-21 has changed how we implement classic Gang of Four (GoF) patterns.

### 1. Strategy Pattern (The Lambda Way)
- **Old Way**: Creating 5 classes implementing an interface.
- **Modern Way**: Using a `Map<Type, Function<In, Out>>` or a **Sealed Interface with Pattern Matching**.
- **Architect Benefit**: Logic is centralized and declarative, reducing "Class Explosion".

### 2. Template Method Pattern (Default Interfaces)
Instead of abstract classes (which limit inheritance), use interfaces with `default` methods to provide the template. 
- **Advantage**: Classes can now inherit from a base `Entity` while also implementing multiple "Template" interfaces for different behaviors (like `Auditable`, `Versioned`).

---

## 🛠️ Layer 2: SOLID Principles at Scale

Most developers understand the acronym. Architects understand the **Trade-offs**.

### 1. Open/Closed Principle (O/C)
- **Tactical**: Adding a new feature without code changes.
- **Architectural**: Using **Plugins** or **SPI (Service Provider Interface)** to allow external teams to add functionality to your core without modifying your JAR.

### 2. Dependency Inversion Principle (D)
- **Architect Reality**: This is the foundation of **Hexagonal Architecture**. Depend on the "Port" (Interface in the Core), not the "Adapter" (Implementations like SQL or Kafka).

---

## 🚀 Layer 3: High-Scale Resilience Patterns

In a distributed Java system, patterns move from "Class-level" to "System-level".

### 1. The Circuit Breaker (Resilience4j)
Prevents cascading failures by "Tripping" the circuit when a downstream service is slow.
- **Architect Insight**: Use **Bulkheading** to isolate thread pools so that one slow service doesn't starve the whole JVM's thread count.

### 2. The Saga Pattern (Distributed Transactions)
Since 2PC (Two-Phase Commit) is slow and doesn't scale in the cloud, we use Sagas.
- **Choreography**: Each service emits an event that triggers the next service.
- **Orchestration**: A central "Saga Manager" coordinates the calls.

---

## 📜 Layer 4: Event-Driven Patterns in Java

### 1. Event Sourcing
Instead of storing the "Current State", store the "Full History" of events.
- **Java Tooling**: Use **Axon Framework** or native Kafka Streams.
- **Benefit**: Auditability and the ability to rebuild the state (Projection) at any point in time.

### 2. CQRS (Command Query Responsibility Segregation)
Separating the "Read" model from the "Write" model.
- **Architect Note**: Only use this for highly unbalanced workloads (e.g., millions of reads, few writes). It adds massive complexity.

---

## 🏁 Layer 5: Design Patterns - Hexagonal (Ports & Adapters)

### 1. The Directory Structure for Architects
```text
com.company.module
├── core          (Business Logic & Domain Objects)
│   ├── model
│   ├── service
│   └── port      (Interfaces for DB/Messaging)
├── infrastructure (Implementation of Ports)
│   ├── db        (SQL/Mongo)
│   └── messaging (Kafka/Rabbit)
└── api           (Controllers/GraphQL/CLI)
```
- **Architect Rule**: Classes in `core` must NEVER import anything from `infrastructure`.

---

## 🧭 Interview Prep & Architect Scenarios

### Q: How do you implement the "Singleton" pattern in a way that is resistant to Reflection attacks?
**A**: Use an **Enum**. The JVM guarantees that Enum constructors cannot be called via Reflection, and it handles serialization natively, preventing duplicate instances from being created during deserialization.

### Q: What is the "Bridge" pattern and why use it?
**A**: It decouples an Abstraction from its Implementation so that both can vary independently. In Java, this often looks like an API (JDBC Interface) and its Drivers (MySQL/Postgres Implementations). It's critical for building "Platform" software.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [20-java-21-best-practices.md](./20-java-21-best-practices.md) | Best Practices |
| ⏩ **Next** | [22-enterprise-java-case-studies.md](./22-enterprise-java-case-studies.md) | Case Studies |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026
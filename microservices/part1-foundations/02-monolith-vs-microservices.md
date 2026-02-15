# 02. Monolith vs Microservices

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐ (Architect)  
> **Status:** Production-Grade

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand that "Monolith" is not an insult. |
| **Developer** | Learn how to build a "Modular Monolith" (Modulith). |
| **Architect** | Master the decision matrix for choosing architecture based on Team Size and Domain Complexity. |

---

## 1. Why This Topic Exists

### The Hype Cycle
Every junior developer wants to build Microservices because "Netflix does it".
**Architect's Job**: To say "No" when it's not necessary.

### The Trade-off
*   **Monolith**: Cheap to start, expensive to maintain at scale.
*   **Microservices**: Expensive to start (Infra tax), cheaper to scale (Organizational agility).

---

## 2. Big Picture Architecture View

### The Evolution

1.  **Monolith**: `UI + Logic + DB` in one box.
2.  **SOA (Service Oriented Architecture)**: Enterprise Service Bus (ESB), XML, SOAP. "Smart Pipes, Dumb Endpoints".
3.  **Microservices**: Lightweight REST/gRPC. "Smart Endpoints, Dumb Pipes".

---

## 3. Core Concepts (🟢 Beginner Level)

### The Monolith
*   **Definition**: All code in one repository, deployed as one artifact (`app.war`).
*   **Analogy**: A Swiss Army Knife. It does everything, but if the scissors break, you might have to throw the whole knife away (or send it for repair).

### The Modular Monolith (Modulith)
*   **Definition**: A Monolith with strict package boundaries. `com.app.order` cannot import `com.app.inventory`.
*   **Analogy**: A toolset where tools are in the same box but clearly separated.

### Microservices
*   **Definition**: Separate processes communicating over network.
*   **Analogy**: A team of specialists (Plumber, Electrician), each with their own van and tools.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Code Structure Comparison

**Monolith (Layered)**
```java
// Bad: Layered Architecture leaks logic
controller/
  OrderController.java
  InventoryController.java
service/
  OrderService.java (Calls InventoryRepository directly - BAD!)
```

**Modular Monolith (Package by Feature)**
```java
// Good: Strict Encapsulation
modules/
  order/
    OrderService.public.java (Interface)
    internal/ (Hidden)
      OrderRepository.java
  inventory/
    InventoryService.public.java
```
*   *Tool*: **Spring Modulith** or **ArchUnit** tests to enforce "No cycles" and "No internal access".

---

## 5. Internal Mechanics (🔴 Architect Level)

### The Hidden Tax of Microservices
In a Monolith, a function call is a memory jump (`JMP`).
In Microservices, a function call is:
1.  **Serialization** (Object -> JSON).
2.  **Network Stack** (TCP/IP packets).
3.  **Switch/Router** (Physical wires).
4.  **Deserialization** (JSON -> Object).

**Impact**:
*   **Latency**: 100ns -> 10ms (100,000x increase).
*   **Reliability**: `JMP` never fails. Network fails often.
*   **Consistency**: `JMP` is transactional. Network is eventual.

---

## 6. Production & Failure Scenarios

### Scenario: The Distributed Monolith
*   **Definition**: You split the code, but not the dependencies.
*   **Symptom**: "We have to deploy Order Service and Inventory Service at the exact same time because they share a DTO library."
*   **Fix**: **Consumer Driven Contracts**. Decouple libraries.

### Scenario: The Reporting Nightmare
*   **Monolith**: `SELECT * FROM Orders JOIN Users JOIN Products`. (Easy).
*   **Microservices**: Data is in 3 differents DBs. FAILS.
*   **Fix**: **Data Warehouse** (ETL) or **CQRS**.

---

## 7. Performance & Scalability Considerations

| Feature | Monolith | Microservices |
|:---|:---|:---|
| **Startup Time** | Slow (Huge Classpath) | Fast (Small Classpath) |
| **Scaling** | All or Nothing (Scale X-axis) | Targeted (Scale only "Checkout") |
| **Memory** | Efficient (Shared Pools) | High Overhead (N * JVMs) |

---

## 8. Security Considerations

*   **Monolith**: Security Context is shared via ThreadLocal (`SecurityContextHolder`). easy.
*   **Microservices**: Security Context must be propagated.
    *   **JWT**: Pass the token.
    *   **Opaque Token**: Validate at Gateway, pass User ID.

---

## 9. Architect-Level Best Practices

### When to Stick to Monolith
1.  **Startups**: < 10 Developers. Focus on Product-Market Fit.
2.  **Simple Domain**: CRUD app, CMS.
3.  **High Performance**: High-Frequency Trading (Network hops are too slow).

### When to Migrate to Microservices
1.  **Team Scaling**: > 20 Developers. Teams are blocking each other.
2.  **Independent Release Cycles**: "Search team" wants to deploy daily, "Checkout team" wants weekly.
3.  **Polyglot Requirements**: ML team needs Python.

---

## 10. Anti-Patterns & Common Mistakes

### 1. "Microservices will make it faster"
**FALSE**. They usually make it slower (Network latency). They make *development* faster (for large teams), not execution.

### 2. Shared Database (The Root of All Evil)
Allowing Service A and Service B to talk to the same PostgreSQL schema.
*   **Result**: Zero isolation. Tighter coupling than a monolith.

---

## 11. Debugging & Troubleshooting Guide

### Debugging a Monolith
1.  Check Application Log.
2.  Check Stack Trace.
3.  Fix.

### Debugging Microservices
1.  Check Application Log A. "Error calling B".
2.  Check Application Log B. "Success".
3.  What??
4.  Check Network/Load Balancer logs. "Timeout".
5.  Check Trace (Zipkin).
6.  Realize a firewall rule blocked the packet.

---

## 12. Interview Questions

### Basic
1.  What is a Monolith?
2.  Does Microservices architecture guarantee better performance? (No).
3.  What is a "Modular Monolith"?

### Intermediate
1.  Explain Conway's Law. ("Systems design implies organizational structure").
2.  How do you handle Reporting in Microservices?
3.  What are the 3 main drawbacks of Microservices? (Complexity, Network, Consistency).

### Advanced
1.  Compare SOA vs Microservices. (ESB vs Smart Endpoints).
2.  How do you prevent a Distributed Monolith?
3.  Explain "Database per Service" pattern.

### Architect-Level
1.  Design a migration strategy for a FinTech Monolith that handles $1B/day. Zero downtime allowed.
2.  Evaluate the cost impact (Cloud Bill) of moving from Monolith to Microservices. (Infrastructure overhead).

---

## 13. Scenario-Based Architecture Questions

1.  **Startup Scenario**: "We are a team of 3. Expected users: 10k. Budget: Low." (Answer: Monolith).
2.  **Hyper-Growth**: "We have 100 devs. Deployment takes 4 hours. Tests are flaky." (Answer: Microservices).
3.  **Legacy Modernization**: "COBOL mainframe. No API." (Answer: Facade / Anti-Corruption Layer).

---

## 14. Summary & Architect Takeaways

*   **Default to Monolith**. Prove you need Microservices.
*   **Complexity Conservation**: You don't eliminate complexity; you move it from Code (Class coupling) to Infrastructure (Network coupling).
*   **Organizational Alignment**: Microservices are about People/Teams first, Technology second.

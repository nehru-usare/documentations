# 03. Domain-Driven Design (DDD) & Bounded Contexts

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Architect)  
> **Status:** Production-Grade

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand that software should model the real world. |
| **Developer** | Learn to implement Entities, Value Objects, and Repositories. |
| **Architect** | Master Bounded Contexts to define Microservice Boundaries. |

---

## 1. Why This Topic Exists

### The Boundary Problem
How do you know where one microservice ends and another begins?
If you get this wrong, you build a **Distributed Monolith** (tightly coupled services).

### The Solution
**DDD (Domain-Driven Design)** provides the strategic tools to draw these lines based on **Business Semantics**, not technical layers.

---

## 2. Big Picture Architecture View

```mermaid
graph TD
    subgraph "Sales Context"
        Customer[Customer (Lead)]
        Opportunity
    end
    
    subgraph "Support Context"
        Ticket
        Requester[Customer (User)]
    end
    
    SalesContext --"Events (CustomerWon)"--> SupportContext
```

*   **Context Map**: Shows how different domains interact.
*   **Upstream/Downstream**: Who depends on whom.

---

## 3. Core Concepts (🟢 Beginner Level)

### Ubiquitous Language
A language shared by Developers and Domain Experts.
*   *Bad*: Dev says "Row inserted", Expert says "Order Booked".
*   *Good*: Everyone says "Order Booked". Code class is `OrderBooked`.

### Bounded Context
A linguistic boundary.
*   **Example**: The word **"Flight"**.
    *   In *Booking Context*: It has a Price, Seat Class.
    *   In *Maintenance Context*: It has Mechanic, Fuel Level, Repair Schedule.
    *   *Architect Decision*: Do NOT make one giant `Flight` object. Make two services: `BookingService` and `MaintenanceService`.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Tactical DDD Patterns

1.  **Entity**: Defined by Identity (ID). Mutable.
    *   `User (id=1, name="John")`. If name changes, it's still John.
2.  **Value Object**: Defined by Value. Immutable.
    *   `Address (Street, City)`. If street changes, it's a new Address.
    *   *Java*: `record Address(String street, String city) {}`.
3.  **Aggregate**: A cluster of objects treated as a unit.
    *   `Order` is the **Aggregate Root**. `OrderItem` is part of it.
    *   Rule: You cannot delete an Item directly; you must ask the Order to remove it.

---

## 5. Internal Mechanics (🔴 Architect Level)

### Context Mapping Patterns

1.  **Partnership**: Two teams work together on two contexts. Frequent sync.
2.  **Shared Kernel**: A common library shared by both (e.g., `CoreTypes.jar`). *Dangerous coupling*.
3.  **Customer-Supplier**: Upstream (Supplier) feeds Downstream (Customer). Downstream is helpless if Upstream breaks.
4.  **Anti-Corruption Layer (ACL)**:
    *   *Scenario*: Legacy system speaks "XML Hell". Modern service speaks "clean JSON".
    *   *Solution*: Create an Adapter layer to translate. DO NOT let legacy concepts leak into your new domain.
5.  **Open Host Service (OHS)**:
    *   A service provides a public, stable API (e.g., Google Maps API).

---

## 6. Production & Failure Scenarios

### Scenario: The God Object
*   **Symptom**: A single `User` service with 500 fields (Address, Preferences, Billing, History, Social Graph).
*   **Failure**: Every team needs to change `User` service. Release queue is 3 weeks long.
*   **Fix**: Split by Bounded Context.
    *   `IdentityContext`: Login, Password.
    *   `BillingContext`: Credit Cards.
    *   `SocialContext`: Friends.

---

## 9. Architect-Level Best Practices

1.  **One Bounded Context = One Microservice** (Roughly).
2.  **Don't share Aggregates**. Microservice A should send an ID to Microservice B, not the whole Object.
3.  **Eventual Consistency Between Contexts**.
    *   When `SalesContext` updates a Customer address, `ShippingContext` is updated via a Domain Event (Kafka). It doesn't happen instantly.

---

## 10. Anti-Patterns & Common Mistakes

### 1. Anemic Domain Model
*   **Symptom**: Entities are just Getters/Setters. All logic is in `Service` classes.
*   **Fix**: Put logic in the Entity. `order.addItem(item)` should validate rules, not `orderService.addItem(order, item)`.

### 2. Generic Subdomains as Core
*   **Symptom**: Writing your own "Authentication System" or "Logging Framework".
*   **Fix**: Buy/Use off-the-shelf for Generic Subdomains (Keycloak, ELK). Focus effort on **Core Domain** (Your secret sauce).

---

## 12. Interview Questions

### Basic
1.  What is DDD?
2.  Difference between Entity and Value Object?

### Intermediate
1.  Explain Ubiquitous Language.
2.  What is an Aggregate Root?

### Advanced
1.  What is an Anti-Corruption Layer? When would you use it?
2.  How does DDD help in Microservices decomposition?

### Architect-Level
1.  Design the Bounded Contexts for an E-commerce system (Catalog, Order, Payment, Shipping). Where do they overlap?
2.  How do you handle ID generation across boundaries?

---

## 14. Summary & Architect Takeaways

*   **Logic over Tech**: Focus on business rules first, technology second.
*   **Boundaries are King**: Good boundaries = Loose Coupling = Agility.
*   **Context Map**: Always draw it. know your dependencies.

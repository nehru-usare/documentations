# 02. CQRS (Command Query Responsibility Segregation)

> **Part 4: Data & Consistency**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Architect)  
> **Status:** High Performance

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand that Reading and Writing are different problems. |
| **Developer** | Implement a basic CQRS flow with Read/Write Replicas. |
| **Architect** | Design an Async CQRS system with Eventual Consistency. |

---

## 1. Why This Topic Exists

### The Mismatched Load
*   **Writes**: Complex validation, ACID transactions, Normalized (3NF).
*   **Reads**: Aggregation, Filtering, De-normalized (Flat), High concurrency.
*   **Problem**: Optimizing a single DB for both is impossible. Adding indexes helps Reads but slows Writes.

### The Solution: CQRS
Split the models.
*   **Command Model**: Optimized for Writes (PostgreSQL).
*   **Query Model**: Optimized for Reads (Elasticsearch / Redis / MongoDB).

---

## 2. Big Picture Architecture View

```mermaid
graph TD
    Client -->|1. Command (Create Order)| CMD_API[Command API]
    CMD_API -->|2. Write| DB_Write[(PostgreSQL - 3NF)]
    
    DB_Write -->|3. Event (OrderCreated)| Kafka
    
    Kafka -->|4. Consume| Projector[Read Projector]
    Projector -->|5. Update View| DB_Read[(Elasticsearch - Flat)]
    
    Client -->|6. Query (Search Orders)| QRY_API[Query API]
    QRY_API -->|7. Read| DB_Read
```

---

## 3. Core Concepts (🟢 Beginner Level)

### Command
An intent to mutate state. `CreateOrder`, `CancelOrder`.
*   Imperative ("Do this").
*   Has side effects.
*   Returns void (or ID).

### Query
An intent to read state. `GetOrder(id)`.
*   Functional ("Get this").
*   No side effects.
*   Returns Data DTO.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Materialized Views
The Read DB is essentially a **Materialized View** of the Write DB.
*   **Write DB**: Tables `Orders`, `Items`, `Products`.
*   **Read DB**: Document `OrderView { id, total, product_names: [], customer_name }`.
    *   *Benefit*: No Joins. Fast reads.

### The Lag (Eventual Consistency)
Since we use Kafka to sync, the Read DB is milliseconds (or seconds) behind.
*   *UI Trick*: When user clicks "Save", show a spinner or optimistic update. Don't immediately `GET`.

---

## 6. Production & Failure Scenarios

### Scenario: The Event Replay
*   **Goal**: We want to add a new screen showing "Orders by City".
*   **Problem**: We don't have that view.
*   **Fix**: **Replay Events**. Read all `OrderCreated` events from Day 1 and populate the new `CityView` table.

---

## 9. Architect-Level Best Practices

1.  **Don't overdo it**: CQRS adds massive complexity (2 DBs, Sync lag). Use it only for specific Bounded Contexts (e.g., Search, Catalog).
2.  **Versioning**: If Command schema changes, Projectors must handle old and new events.
3.  **Commands should be behavioral**: `UpgradeCustomer` (Logic) is better than `UpdateCustomer` (CRUD).

---

## 12. Interview Questions

### Basic
1.  What does CQRS stand for?
2.  Why separate Reads and Writes?

### Intermediate
1.  What is a Materialized View?
2.  How do you handle the "Lag" between write and read?

### Advanced
1.  Explain the relationship between CQRS and Event Sourcing. (They are best friends).
2.  How do you rebuild a corrupt Read Database? (Replay events).

### Architect-Level
1.  Design a Search system for an E-commerce site. Write to SQL, Search from Elastic.
2.  Discuss the trade-offs of logical CQRS (Code split) vs physical CQRS (DB split).

---

## 14. Summary & Architect Takeaways

*   **Optimization**: CQRS allows you to scale Reads (x1000) independently of Writes (x10).
*   **Complexity Cost**: You are now a Distributed Systems Engineer. You have to deal with Lag, Replays, and Sync issues.

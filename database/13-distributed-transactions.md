# Distributed Transactions (2PC, Saga)

> **Part 3: Scaling**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Advanced)  
> **Status:** Preparing...

---

## 0. Learning Objectives
*   **Beginner**: Why we can't just `@Transactional` across microservices.
*   **Developer**: Implementing the Saga Pattern.
*   **Architect**: Choosing between Strong Consistency (2PC) and Eventual Consistency (Saga).

---

## 1. Context
**Monolith**: `Begin Tx -> Update Order -> Update Inventory -> Commit`. Easy.
**Microservices**: Order Service and Inventory Service are different DBs. Network can fail between them.

---

## 2. Core Concepts (🟢 Beginner Level)

### The Problem
*   Order Service commits.
*   Inventory Service fails.
*   **Result**: Order placed, but no stock reserved. Data corruption.

---

## 3. Architecture Breakdown (Solutions)

### 1. Two-Phase Commit (2PC / XA)
*   **Coordinator**: "Can everyone commit?" (Prepare Phase).
*   **Participants**: "Yes/No".
*   **Coordinator**: "Commit!" (Commit Phase).
*   *Blocking Protocol*: If Coordinator dies, everyone waits (Locks held). **Not scalable**.

### 2. Saga Pattern (Long Running Transaction)
*   Sequence of local transactions.
*   T1 -> T2 -> T3.
*   **Failure**: If T3 fails, execute **Compensating Transactions** (Undo) backwards. C2 -> C1.
*   *Eventual Consistency*.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Choreography (Events)
*   Order Service publishes `OrderCreated`.
*   Inventory Service listens, reserves stock, publishes `StockReserved`.
*   *Pros**: Decoupled.
*   *Cons**: Cyclical dependencies. Hard to track state.

### 2. Orchestration (Command)
*   **Saga Orchestrator** (State Machine): Central service.
*   Sends Command `ReserveStock` -> Awaits Reply.
*   Sends Command `ChargeCard` -> Awaits Reply.
*   *Pros**: Central visibility (Camunda/Temporal).

---

## 5. Trade-Off Analysis

| Strategy | Consistency | Availability | Latency |
| :--- | :--- | :--- | :--- |
| **2PC** | Strong | Low (Blocking) | High |
| **Saga** | Eventual | High | Low |

---

## 6. Scaling Considerations

### The Outbox Pattern
*   **Problem**: "Save to DB" and "Publish to Kafka" is not atomic.
*   **Fix**:
    1.  Transaction: Write to `Outbox` table (in same DB).
    2.  Commit DB.
    3.  Background Process (Debezium): Reads `Outbox` table -> Pushes to Kafka.
    4.  Guarantees "At Least Once" delivery.

---

## 7. Failure Scenarios & Recovery

### 1. The Lost Compensation
*   T1 succeeds. T2 fails. C1 (Undo T1) fails.
*   **Result**: System is inconsistent. (Order exists, Stock not reserved).
*   **Fix**: Human Intervention (Admin Dashboard) or Infinite Retry on Compensation.

---

## 8. Security Considerations

### 1. Idempotency
*   Saga messages might be delivered twice.
*   Compensating transaction `RefundUser` must check "Is user already refunded?".

---

## 9. Performance Considerations

*   **Latency**: Sagas are slow. User clicks "Buy", sees "Order Pending".
*   System confirms via Email 5 seconds later.

---

## 10. Real Production Lessons

### Uber
*   Uses **Cadence/Temporal** for Orchestration.
*   Code looks synchronous, but engine handles retries, timeouts, and state persistence.

---

## 11. Interview Questions

### Basic
1.  Why is 2PC bad for microservices?
2.  What is a Compensating Transaction?
3.  Choreography vs Orchestration.

### Intermediate
1.  Explain the Outbox Pattern.
2.  What happens if the Orchestrator crashes? (Persist state).
3.  Is Saga ACID? (No, it's ACD. Lacks Isolation).

### Advanced
1.  Handle "Pivot Transaction" in Saga.
2.  Design a Distributed Transaction using only Kafka.
3.  Explain TCC (Try-Confirm-Cancel) pattern.

---

## 12. Summary & Architect Takeaways

1.  **Avoid Distributed Tx**: Align Service Boundaries so transactions stay local (DDD).
2.  **Saga is Standard**: If you must split, use Saga.
3.  **Outbox is Mandatory**: Never dual-write to DB and Queue without Outbox.

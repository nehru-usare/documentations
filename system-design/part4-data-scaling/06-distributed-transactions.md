# Distributed Transactions

> **Part 4: Data Scaling**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The Atomic Myth

---

## 0. Learning Objectives
*   **Beginner**: Why "Transaction across two databases" is hard.
*   **Developer**: Implementing the Saga Pattern.
*   **Architect**: Negotiating Consistency requirements to avoid 2-Phase Commit.

---

## 1. Problem Context
**Why does this exist?**
Monolith: `BEGIN TX; Insert User; Insert Audit; COMMIT;`. Simple.
Microservices:
*   Service A (Postgres): Insert User.
*   Service B (Mongo): Insert Audit.
*   If B fails, A is already committed. **Inconsistency**.
**Distributed Transactions** attempt to link these disparate commit operations.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. The ACID illusion
*   In distributed systems, ACID across boundaries is impossibly slow (CAP theorem says choose CP).
*   We usually settle for **BASE** (Eventual Consistency).

### 2. Two-Phase Commit (2PC)
*   **Coordinator**: "Are you ready?"
*   **Participants**: "Yes."
*   **Coordinator**: "Commit!"
*   *Problem*: If Coordinator dies in the middle, everyone locks forever.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Saga Pattern
A sequence of local transactions. Each step has a **Compensating Transaction** (Undo).
*   **T1**: Book Flight.
*   **T2**: Book Hotel. (Fails).
*   **C1**: Cancel Flight (Compensate T1).

### Types of Sagas
1.  **Choreography**: Events. Service A publishes `FlightBooked`. Service B listens.
2.  **Orchestration**: Central Manager (Camunda/Temporal) calls A, then B.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. 2PC Failure Modes
*   **Coordinator Crash**: Participants hold locks. DB grinds to halt.
*   **Network Partition**: Some commit, some don't.
*   *Verdict*: 2PC is flawed for high-scale systems.

### 2. Saga State Machine
*   The Orchestrator persists the state (`STEP_1_DONE`, `STEP_2_FAILED`, `COMPENSATING`).
*   Durability is key. If Orchestrator crashes, it must resume.

---

## 5. Trade-Off Analysis

| Strategy | Consistency | Availability | Latency |
| :--- | :--- | :--- | :--- |
| **2 Phase Commit** | Strong | Low (Locking) | High |
| **Saga (Choreography)** | Eventual | High | Low |
| **Saga (Orchestration)** | Eventual | Medium | Medium |

---

## 6. Scaling Considerations

### Lock Duration
*   **2PC**: Locks held for network RTT. 100ms.
*   **Saga**: Locks held for local TX only. 5ms.
*   *Result*: Sagas scale. 2PC does not.

---

## 7. Failure Scenarios & Recovery

### 1. Compensation Failure
*   **Scenario**: Book Flight (Success). Book Hotel (Fail). Cancel Flight (Fail - API down).
*   **Event**: "Zombie Reservation".
*   **Fix**: **Idempotent Retries**. The system must retry the Compensation forever ... or alert a human.

---

## 8. Security Considerations

### 1. Partial State Visibility
*   In Sagas, the "Flight" is booked *before* the "Hotel" is confirmed.
*   A User might see "Flight Confirmed" and then 5 seconds later "Flight Cancelled".
*   **Isolation**: This violates ACID Isolation. (Dirty Reads).
*   *Fix*: "Pending" states in UI.

---

## 9. Performance Considerations

*   **Throughput**: 2PC reduces DB throughput by 10x due to lock contention.
*   **Latency**: Sagas add latency because the client must wait for async events (or poll).

---

## 10. Real Production Lessons

### Uber's Payment Flows
*   **Design**: Distributed Sagas.
*   **Reality**: Complex failure graphs. "If credit card fails, do we cancel the ride or retry?"
*   **Lesson**: Use a Workflow Engine (Cadence/Temporal). Hand-coding Sagas is bug-prone.

---

## 11. Interview Questions

### Basic
1.  What is a Distributed Transaction?
2.  Why is 2PC bad for performance?
3.  What is a Saga?
4.  What is a Compensating Transaction?
5.  What happens if the Compensation fails?

### Intermediate
1.  Choreography vs Orchestration Sagas.
2.  What is XA Standard? (The API for 2PC).
3.  How do you handle Lack of Isolation in Sagas? (Semantic Locking).
4.  Difference between "At Least Once" and "Exactly Once" delivery.
5.  How does Idempotency help error handling?

### Advanced
1.  Design a TCC (Try-Confirm-Cancel) pattern.
2.  Analyze Three-Phase Commit (3PC). Does it solve the blocking problem? (Mostly, but adds latency).
3.  How does Google Spanner achieve Distributed ACID? (TrueTime API + Paxos).
4.  Critique the "Dual Write" problem (Writing to DB and Kafka). (Use Outbox Pattern).
5.  Architect a method to handle "Pivot Transactions" in a Saga.

### Architect-Level
1.  "We need financial accuracy across 3 Microservices." Architect the solution. (Saga with Reconciliation jobs).
2.  Evaluate Temporal.io vs AWS Step Functions.
3.  Design the "Transactional Outbox" pattern for reliable event publishing.

---

## 12. Scenario-Based System Design Problems

### 1. Design Booking.com Checkout
*   **Steps**: Reserve Room, Charge Card, Email User.
*   **Pattern**: Orchestration Saga.
*   **Failure**: If Email fails, do NOT roll back Room. Just retry Email. (Non-Critical step).

### 2. Design Inventory Transfer
*   **Step**: Debit Warehouse A, Credit Warehouse B.
*   **Pattern**: TCC or Saga.
*   **Risk**: Warehouse A reserved, B full. Rollback A.

---

## 13. Summary & Architect Takeaways

1.  **Avoid Distributed Transactions**: Merge the microservices if they are that tightly coupled.
2.  **Sagas > 2PC**: Always prefer Sagas for scale.
3.  **Think about Failure**: The "Happy Path" is easy. The code dealing with "Compensation Failed" is the hard part.

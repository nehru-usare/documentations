# Event Sourcing Pattern

> **Part 6: Enterprise & Microservices Patterns**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The Time Machine

---

## 1. Core Concept
Store the **Events** that caused a change, not the **Current State**.

*   **Traditional**:
    *   Row: `Account { balance: 100 }`
    *   Update: `UPDATE Account SET balance = 150`
    *   *Result*: `Account { balance: 150 }`. (History is lost).

*   **Event Sourcing**:
    *   Event 1: `AccountCreated { id: 1, balance: 0 }`
    *   Event 2: `Deposited { amount: 100 }`
    *   Event 3: `Deposited { amount: 50 }`
    *   *Current State*: `0 + 100 + 50 = 150` (Calculated on the fly).

---

## 2. Benefits

### 1. Audit Trail
"How did the user get to -$500 balance?"
In CRUD: "I don't know, a bug maybe?"
In Event Sourcing: "They deposited 100, Withdrew 200, then Withdrew 400." You have proof.

### 2. Time Travel
You can rebuild the database as it looked on **Dec 31st 2024** by replaying events only up to that date.

---

## 3. Implementation Challenges

### Snapshots
If an Account has 1,000,000 events, calculating balance is slow.
**Solution**: Every 100 events, save a **Snapshot** (`Balance: 5000`).
Replay: Load Snapshot + Replay subsequent events.

### Schema Evolution
What if `DepositEvent` structure changes?
You effectively need to version your events (`DepositEventV1`, `DepositEventV2`).

---

## 4. Architect Takeaway
*   **Event Store**: You need a specialized DB (Axon Server, EventStoreDB) or Append-Only Kafka/Postgres table.
*   **CQRS Required**: You effectively **MUST** use CQRS with Event Sourcing. You cannot Query an Event Stream efficiently. You need a Read Model.

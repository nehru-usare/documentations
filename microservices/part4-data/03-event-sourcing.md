# 03. Event Sourcing Pattern

> **Part 4: Data & Consistency**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Architect)  
> **Status:** Niche but Powerful

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand the Bank Account analogy. |
| **Developer** | Implement an Aggregate that reconstitutes state from events. |
| **Architect** | Decide if the complexity is worth the Audit/Replay benefits. |

---

## 1. Why This Topic Exists

### The Audit Problem
In a CRUD DB, if you update `Status` from `PENDING` to `APPROVED`, the history is lost. You only know the *current* state.
**Event Sourcing** persists *every* change.

### The Analogy: Accounting
Bank DBs don't just store "Current Balance: $100".
They store:
1.  Deposit $50.
2.  Deposit $60.
3.  Withdraw $10.
4.  Current Balance = Sum(Events) = $100.

---

## 2. Big Picture Architecture View

```mermaid
graph LR
    Command[Cmd: Withdraw $10] --> Aggregate
    Aggregate[Account Aggregate] -->|1. Validate logic| Aggregate
    Aggregate -->|2. Emit Event| EventStore[(Event Store)]
    
    EventStore -->|3. Saved: Withdrawn($10)| EventBus
    
    subgraph "Rehydration (Loading)"
        EventStore -->|Load Events| Aggregate
        Aggregate -->|Replay| AggregateState
    end
```

---

## 3. Core Concepts (🟢 Beginner Level)

### Event Store
A database optimized for appending immutable events. (e.g., EventStoreDB, Axon Server).
*   **Immutable**: You can never change history. To fix a bug, you publish a *Correction Event*.

### Rehydration (Replay)
Loading an object by creating a blank instance and applying all its past events in order.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Snapshotting
Replaying 10,000 events to load one User is slow.
*   **Optimization**: Save a "Snapshot" every 100 events.
*   *Load*: Load Snapshot (v100) + Events (101...105).

### Code Example (Conceptual)
```java
class Account {
    int balance = 0;
    
    // Command Handler
    void handle(WithdrawCmd cmd) {
        if (balance < cmd.amount) throw new Error();
        apply(new WithdrawnEvent(cmd.amount));
    }
    
    // Event Sourcing Handler (Replay)
    void on(WithdrawnEvent e) {
        this.balance -= e.amount;
    }
}
```

---

## 9. Architect-Level Best Practices

1.  **Don't use Kafka as Event Store**: Kafka has retention policies (deletes old data). Event Store must keep data forever.
2.  **Schema Evolution**: Changing event structure is hard. (Upcasting).
3.  **CQRS is mandatory**: You cannot query an Event Store efficiently ("Find users with balance > 100"). You need a Projection (Read DB).

---

## 12. Interview Questions

1.  What is Event Sourcing?
2.  Why do you need Snapshots?
3.  How do you fix a bad event? (Compensating Event).

---

## 14. Summary & Architect Takeaways

*   **Ultimate Source of Truth**: The log of events is the truth. The DB is just a cache.
*   **Temporal Queries**: "What was the state of the system last Friday?" -> Easy (Replay until Friday).
*   **Warning**: Extremely high learning curve. Don't use for simple CRUD.

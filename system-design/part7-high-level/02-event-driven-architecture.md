# Event-Driven Architecture (EDA)

> **Part 7: High-Level Architecture**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** Don't call us, we'll call you

---

## 0. Learning Objectives
*   **Beginner**: The difference between "Asking for Data" and "Reacting to Events".
*   **Developer**: Implementing an Event Listener that updates a projection.
*   **Architect**: Designing a system that can replay history to fix bugs (Event Sourcing).

---

## 1. Problem Context
**Why does this exist?**
Order Svc calls Payment Svc. Payment Svc calls Inventory Svc. Inventory calls Shipping.
*   **Blocking**: User waits 5 seconds.
*   **Coupling**: If Shipping is down, User can't buy.
**EDA**: Order Svc emits `OrderPlaced`. Returns "Success" to user.
Payment, Inventory, Shipping listen to the event and do their job async.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Command vs Event
*   **Command**: "Create Order". (Intent. Can be rejected).
*   **Event**: "Order Created". (Fact. Already happened. Cannot be rejected).

### 2. Producer / Consumer
*   Components are decoupled. Producer doesn't know who listens.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Patterns

1.  **Pub/Sub**: Message Broker (Kafka) routes events.
2.  **Event Notification**: "Object 5 changed". Consumers call back to get details. (Thin Event).
3.  **Event Carried State Transfer**: "Object 5 changed to {name: Bob}". Consumers update local cache. No callback. (Fat Event).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Event Sourcing
*   Store the **Events**, not the State.
*   DB: `[AccountCreated, Depost(100), Withdraw(20)]`.
*   Current Balance: Replay events = 80.
*   **Pros**: Perfect Audit trail. Time Travel.
*   **Cons**: Hard to query "Who has balance > 100?"

### 2. CQRS (Command Query Responsibility Segregation)
*   **Write Model**: Optimized for Validation/Events (Event Store).
*   **Read Model**: Optimized for Queries (SQL/Elasticsearch).
*   **Sync**: Projector listens to Events and updates Read Model.

---

## 5. Trade-Off Analysis

| Feature | Request/Response | Event-Driven |
| :--- | :--- | :--- |
| **Coupling** | Tight | Loose |
| **Latency** | Low (if simple) | Higher (Async) |
| **Consistency** | Strong | Eventual |
| **Debuggability** | Easy (Stack Trace) | Hard (Distributed Trace) |

---

## 6. Scaling Considerations

### The Thundering Herd
*   If an event `PriceChanged` triggers 1 Million users to be emailed, and the Email Service listens directly...
*   **Fix**: Buffering. Parallel processing (Consumer Groups).

---

## 7. Failure Scenarios & Recovery

### 1. Out of Order Events
*   `UserUpdated` (v2) arrives before `UserCreated` (v1).
*   System crashes.
*   **Fix**: **Versioning** or Sequence Numbers. Reject/Park events that are "from the future".

---

## 8. Security Considerations

### 1. PII in Events
*   If `UserUpdated` contains email/phone and is written to Kafka (Immutable Log).
*   **GDPR**: User demands "Right to be Forgotten". You can't delete from Kafka.
*   **Fix**: **Crypto-Shredding**. Encrypt PII in the event. Determine key ID by UserID. Throw away the Key to "Delete" the data.

---

## 9. Performance Considerations

*   **Throughput**: EDA is king of throughput (Buffering).
*   **Latency**: EDA is poor for "Read your own write" scenarios.

---

## 10. Real Production Lessons

### Amazon.com
*   Everything is an event.
*   "Order Placed" -> Triggers Billing, Warehouse, Recommendation Engine, Fraud, Email using SNS/SQS.
*   Decoupling allowed Amazon to scale teams independently.

---

## 11. Interview Questions

### Basic
1.  What is Event-Driven Architecture?
2.  Difference between Queue and Topic.
3.  What is Eventual Consistency?
4.  Advantage of Decoupling.
5.  What is a Dead Letter Queue?

### Intermediate
1.  Explain Event Carried State Transfer.
2.  What is CQRS? Why use it?
3.  How do you handle duplicate events? (Idempotency).
4.  What is Event Sourcing?
5.  How to upgrade an Event Schema?

### Advanced
1.  Design an Event Sourcing system for a Bank Ledger.
2.  Analyze Snapshotting strategies to speed up Replay.
3.  How to handle "Compensating Actions" in EDA? (Saga).
4.  Explain the "Dual Write" problem. (Writing to DB and Kafka atomically).
5.  Critique the complexity of CQRS. (Is it worth it for CRUD?).

### Architect-Level
1.  "We moved to EDA and now nobody knows the flow of the business." (The Choreography problem). Architect a solution (Process Manager / Orchestration).
2.  Design a system that complies with GDPR while using immutable Event Logs.
3.  Evaluate the use of Change Data Capture (CDC) to drive EDA from a legacy monolith.

---

## 12. Scenario-Based System Design Problems

### 1. Design Notification System
*   **Req**: Email, SMS, Push.
*   **Arch**: EDA. Service emits `NotificationRequired`. Handlers pick up.

### 2. Design Stock Exchange
*   **Req**: Audit everything. High speed.
*   **Arch**: Event Sourcing (LMAX Architecture). In-memory processing of event stream.

---

## 13. Summary & Architect Takeaways

1.  **Async by Default**: Makes systems robust.
2.  **Events are Facts**: Name them in past tense (`UserCreated`).
3.  **Complexity**: EDA is harder to debug. Invest in Tracing (OpenTelemetry) early.

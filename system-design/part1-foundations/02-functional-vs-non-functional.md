# Functional vs Non-Functional Requirements

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐ (Basic)  
> **Status:** Critical Interview Step

---

## 0. Learning Objectives
*   **Beginner**: Distinguish between "What" (Functional) and "How" (Non-Functional).
*   **Developer**: Translate NFRs into technical constraints (e.g., "Fast" -> "Latency < 200ms").
*   **Architect**: Negotiate NFRs. Explain why "100% Availability" is impossible and expensive.

---

## 1. Problem Context
**Why does this exist?**
In a System Design Interview (or real project), you get a vague prompt: "Design YouTube."
If you just start drawing boxes, you fail.
You must clarify **Constraints**.
*   How many videos? (Storage)
*   How many views? (Throughput)
*   Can it be slow? (Latency)
*   Can we lose data? (Reliability)

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Functional Requirements (FR)
*   **Definition**: The specific behaviors or functions. "What the system does."
*   **Example**: "User should be able to upload a video." "User should be able to like a comment."
*   **Test**: Can I write a user story for this?

### 2. Non-Functional Requirements (NFR)
*   **Definition**: The criteria that judge the operation of a system. "How the system behaves."
*   **Example**: "The video upload must finish in < 5 seconds." "The system must support 1M users."
*   **Test**: Does this end in "-ility"? (Scalability, Availability, Reliability).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Mapping NFRs to Components

| Requirement | Technical Solution |
| :--- | :--- |
| **Scalability** | Load Balancers, Horizontal Scaling, Sharding. |
| **Availability** | Replication, Failover, Redundancy. |
| **Latency** | Caching (Redis), CDN, Edge Computing. |
| **Consistency** | SQL Transactions (ACID), Distributed Locking. |
| **Durability** | RAID, Backups, WAL (Write Ahead Log). |

---

## 4. Internal Mechanics (🔴 Architect Level)

### The CAP Theorem Connection
NFRs are often contradictory.
*   **High Consistency** (NFR) creates **High Latency** (NFR).
    *   *Why?* To be Consistent, data must be replicated to all nodes before returning success. This takes time (Network RTT).
*   **High Availability** (NFR) compromises **Consistency** (NFR).
    *   *Why?* To be Available during a partition, nodes must accept writes independently, leading to drift.

---

## 5. Trade-Off Analysis

| NFR Choice | Trade-Off |
| :--- | :--- |
| **Strong Consistency** | Higher Latency. Lower Availability (CAP). |
| **High Availability** | Eventual Consistency. Risk of Stale Reads. |
| **Low Latency** | Expensive RAM usage (Caching). Stale Data. |
| **High Security** | Reduced Usability (MFA, 2FA). Slower performace (Encryption overhead). |

---

## 6. Scaling Considerations

### NFRs Change with Scale
*   **Startup Phase**: Functional Requirements are King. "Does it work?"
*   **Growth Phase**: Scalability becomes King. "Can it handle 10k users?"
*   **Enterprise Phase**: Security & Compliance become King. "Is it GDPR compliant?"

---

## 7. Failure Scenarios & Recovery

### 1. Missing NFRs
*   **Scenario**: Architect designs for "High Functionality" but ignored "Data Durability".
*   **Event**: Server crashes.
*   **Result**: All user data is lost. Product is dead.

### 2. Unrealistic NFRs
*   **Scenario**: "We need 100% Availability."
*   **Reality**: Verified 100% requires infinite money.
*   **Result**: Project over-budget and delayed.

---

## 8. Security Considerations
*   **NFR**: Confidentiality.
*   **Implementation**: TLS 1.3 everywhere. Encryption at REST.
*   **Trade-off**: CPU usage increases by ~10-20% for encryption/decryption.

---

## 9. Performance Considerations
*   **NFR**: Throughput vs Latency.
*   **Optimization**:
    *   For **Throughput**, use Kafka (Batching).
    *   For **Latency**, use UDP or gRPC (Streaming).

---

## 10. Real Production Lessons

### The "Real-Time" Dashboard
*   **Requirement**: "Dashboard must update in real-time."
*   **Implementation**: WebSockets with every database update.
*   **Outcome**: DB overloaded by 10k WebSocket connections triggering queries.
*   **Fix**: Relax the NFR. "Real-time" -> "Near Real-time (5s delay)". Use Polling or Batched Events.

---

## 11. Interview Questions

### Basic
1.  Define Functional vs Non-Functional requirements.
2.  Give 3 examples of NFRs.
3.  Why are NFRs important?
4.  What is "Availability" measured in? (9s).
5.  What is "Latency"?

### Intermediate
1.  How does Consistency affect Latency?
2.  How do you design for High Availability?
3.  What is the difference between Scalability and Elasticity?
4.  Translate "Fast" into a metric. (p99 < 100ms).
5.  What NFRs are critical for a Banking App? (Consistency, Security, Durability).

### Advanced
1.  How do you handle the trade-off between Reliability and Cost?
2.  Explain the "Error Budget" concept in SRE.
3.  Design a system where Availability is sacrificed for Consistency.
4.  How do NFRs influence Database selection?
5.  What are the NFRs for a self-driving car system? (Extremely Low Latency, Safety).

### Architect-Level
1.  You have a budget of $10k/month. Negotiate the NFRs for a video streaming service.
2.  Prove why 100% Availability is mathematically impossible in a distributed system.
3.  Define the SLA for a service that depends on 3 downstream services with 99.9% SLA each.
4.  How do you measure "Maintainability" as an NFR?
5.  Critique a design that claims "Strong Consistency" and "Low Latency" over a WAN.

---

## 12. Scenario-Based System Design Problems

### 1. Design a Flash Sale System
*   **Critical NFRs**: High Scalability (Spike traffic), Fairness (First come first serve).
*   **Dropped NFRs**: Personalization (Too slow).

### 2. Design a Medical Record System
*   **Critical NFRs**: Confidentiality (HIPAA), Durability (Cannot lose history), Consistency.
*   **Dropped NFRs**: Low Latency (Doctors can wait 1s for a record).

---

## 13. Summary & Architect Takeaways

1.  **Clarify First**: Never start designing until you know the NFRs.
2.  **Numbers Matter**: "Fast" means nothing. "100ms" means something.
3.  **Everything is a Negotiation**: You cannot have it all (CAP). Pick the NFRs that drive business value.

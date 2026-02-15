# PACELC Theorem

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** CAP 2.0

---

## 0. Learning Objectives
*   **Beginner**: Realize that CAP Theorem is incomplete because partitions are rare.
*   **Developer**: Learn that even without failures, you have to trade Latency for Consistency.
*   **Architect**: Use PACELC to categorize modern databases (DynamoDB vs Voltdb).

---

## 1. Problem Context
**Why does this exist?**
CAP only tells you what happens during a **Failure** (Partition).
But systems function normally 99.9% of the time.
CAP doesn't explain the trade-offs during **Normal Operation**.
Daniel Abadi proposed PACELC in 2012 to fix this.

---

## 2. Core Concepts (🟢 Beginner Level)

### The Acronym
If **P**artition (P):
*   Choose **A**vailability (A) or **C**onsistency (C). (This is standard CAP).

**E**lse (E = Normal Operation):
*   Choose **L**atency (L) or **C**onsistency (C).

**Formula**:
`P -> (A or C) || E -> (L or C)`

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Normal Operation Trade-off
Why do we have to choose between Latency and Consistency when everything is fine?
*   **Scenario**: Master-Slave Replication.
*   **Consistent Choice**: Write to Master -> Wait for Slave to Confirm -> Return Success. -> **High Latency**.
*   **Low Latency Choice**: Write to Master -> Return Success. (Slave replicates async). -> **Low Consistency** (Slave is stale).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Synchronous Replication (EC - PC)
*   Used by systems prioritizing Consistency.
*   Every commit is a 2-Phase Commit (2PC) or Quorum wait.
*   **Latency Penalty**: Network RTT is added to every write.

### 2. Asynchronous Replication (EL - PA)
*   Used by systems prioritizing Latency.
*   Write returns as soon as it hits memory/WAL on one node.
*   **Consistency Penalty**: Replication lag (ms to seconds).

---

## 5. Trade-Off Analysis

| Strategy | Normal Mode (Else) | Partition Mode (Partition) | Database Example |
| :--- | :--- | :--- | :--- |
| **PC/EC** | Consistent | Consistent | **HBase**, **BigTable** (Sacrifices Avail & Latency) |
| **PA/EL** | Low Latency | Available | **DynamoDB**, **Cassandra**, **Riak** (Sacrifices Consistency) |
| **PC/EL** | Low Latency | Consistent | **PNUTS** (Yahoo) - Rare combination |
| **PA/EC** | Consistent | Available | **MongoDB** (Tunable) |

---

## 6. Scaling Considerations

### Latency Sensitivity
*   If your SLA is <10ms, you **MUST** choose **EL** (Else Latency).
*   You physically cannot achieve Strong Consistency (EC) across regions in <10ms due to speed of light.

---

## 7. Failure Scenarios & Recovery

### 1. Replication Lag Spike
*   **System**: EL (Asynchronous).
*   **Event**: Network congestion.
*   **Result**: Replication lag grows from 10ms to 5 minutes.
*   **Impact**: Users see "Old Data" for 5 minutes. (e.g., I just bought the item, but order history is empty).
*   **Mitigation**: Sticky Sessions (Route user to the same Master they wrote to).

---

## 8. Security Considerations

*   **Consistency Exploits**: In an EL system, an attacker can exploit "Time of Check vs Time of Use" (TOCTOU).
*   *Example*: Withdraw $100. Balance checks out. Quick withdraw $100 again before replica updates.
*   *Defense*: Critical actions must force EC (Consistency) read.

---

## 9. Performance Considerations

*   **EC**: Write Throughput is limited by the slowest link in the cluster.
*   **EL**: Write Throughput is limited only by the local disk speed.

---

## 10. Real Production Lessons

### Amazon's Shopping Cart
*   **Choice**: PA/EL.
*   **Reason**: It is better to let a user add an item (Low Latency / High Avail) and risk a "Ghost Item" (Inconsistency) than to block the user.
*   **Revenue**: "Lateny kills sales." EL is a business requirement.

### Stock Trading
*   **Choice**: PC/EC.
*   **Reason**: Showing an old price (EL) causes financial loss. Traders tolerate slightly higher latency for accurate data.

---

## 11. Interview Questions

### Basic
1.  What does PACELC stand for?
2.  How does PACELC differ from CAP?
3.  Why does consistency hurt latency?
4.  Give an example of an EL database.
5.  Give an example of an EC database.

### Intermediate
1.  Can you have both Low Latency and Strong Consistency? (No).
2.  How does DynamoDB fit into PACELC?
3.  Explain the "E" in PACELC.
4.  How does Async Replication relate to PACELC?
5.  Why is MongoDB considered PA/EC?

### Advanced
1.  Analyze the architecture of Spanner using PACELC. (Highly Optimized PC/EC).
2.  Design a Toggle System that switches between EC and EL.
3.  How do Conflict-Free Replicated Data Types (CRDTs) help AP/EL systems?
4.  Calculate the theoretical minimum latency for a confirmed EC write between NY and London.
5.  Critique the claim that "Quorums solve consistency without latency." (False, Quorums add latency).

### Architect-Level
1.  "Latency is the new outage." Discuss this with respect to PACELC.
2.  Architect a global user session store. Justify your PACELC choice.
3.  How do business requirements (Revenue vs Risk) dictate the PACELC choice?
4.  Evaluate the impact of Network Topology (Star vs Mesh) on EC systems.
5.  Design a Hybrid system: EL for Reads, EC for Writes.

---

## 12. Scenario-Based System Design Problems

### 1. Design a Global Leaderboard
*   **Requirement**: Fast updates.
*   **Choice**: PA/EL.
*   **Reason**: It's okay if the score is 5 seconds old. Speed matters.

### 2. Design an Inventory System
*   **Requirement**: Never oversell.
*   **Choice**: PC/EC.
*   **Reason**: Overselling is a business failure. We accept wait times.

---

## 13. Summary & Architect Takeaways

1.  **CAP is for Failures**: PACELC is for Every Day.
2.  **Latency is a cost**: You pay Latency to buy Consistency.
3.  **Know your Database**: Don't use Cassandra (EL) if you need Transactions (EC).

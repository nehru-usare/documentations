# 04. CAP Theorem & Distributed Systems Basics

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐⭐ (Architect)  
> **Status:** Theory & Practice

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand the acronym CAP (Consistency, Availability, Partition Tolerance). |
| **Developer** | Know which database to pick (CP vs AP). |
| **Architect** | Master the PACELC theorem and designing for Network Partitions. |

---

## 1. Why This Topic Exists

### The Physics of Data
In a distributed system, you cannot cheat the speed of light. Data takes time to travel.
If a network cable is cut, you have a hard choice: **Block the user** (Safety) or **Show old data** (Uptime).

---

## 3. Core Concepts (🟢 Beginner Level)

### CAP Theorem (Eric Brewer)
In a distributed data store, you can only guarantee **2 out of 3** guarantees:

1.  **Consistency (C)**: Every read receives the most recent write or an error. (All nodes see same data).
2.  **Availability (A)**: Every request receives a (non-error) response, without the guarantee that it contains the most recent write. (System stays up).
3.  **Partition Tolerance (P)**: The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network.

### The "Pick Two" Tricky Rule
Since networks **WILL** fail (Partition is guaranteed), you cannot forfeit **P**.
So the real choice implies:
*   **CP (Consistency + Partition Tolerance)**: If network breaks, shut down 1 side. (Safe).
*   **AP (Availability + Partition Tolerance)**: If network breaks, keep serving both sides. (Fast/Uptime, but data diverges).

---

## 5. Internal Mechanics (🔴 Architect Level)

### CP Systems (e.g., MongoDB, HBase, Redis Cluster)
*   **Partition happens**: Node A cannot talk to Node B.
*   **Action**: System elects a Leader. Minorities go into "Read Only" or "Offline" mode.
*   **Trade-off**: Writes fail during election. Availability drops.

### AP Systems (e.g., Cassandra, DynamoDB, CouchDB)
*   **Partition happens**: Node A cannot talk to Node B.
*   **Action**: Node A accepts writes. Node B accepts writes.
*   **Result**: "Split Brain". Two different versions of data exist.
*   **Reconciliation**: When network heals, system merges data (Last Write Wins, or Vector Clocks).

### PACELC Theorem
CAP is only during failures. What about normal times?
**PACELC**:
*   If **P**artition (P): Choose **A** or **C**.
*   **EL**se (E): Choose **L**atency or **C**onsistency.
*   *Meaning*: Even without failures, if you want strong Consistency, you must pay Latency penalty (waiting for replication).

---

## 6. Production & Failure Scenarios

### Scenario: The Split Brain
*   **System**: A CP Database (Redis Cluster).
*   **Event**: Network cut between US-East and US-West.
*   **Result**: Each side thinks the other is dead. Both elect a master?
    *   No, Quorum prevents this. One side will not have > 50% votes and will stop.

---

## 9. Architect-Level Best Practices

1.  **Financial Data**: Always **CP** (RDBMS, ACID). You can't have "Eventual Consistency" on a bank balance.
2.  **User Profiles / Feeds**: Always **AP** (Cassandra). Better to show an old profile pic than a 500 Error.
3.  **Shopping Cart**: **AP**. Never block a user from adding to cart. Merge conflicts later.

---

## 12. Interview Questions

### Basic
1.  What does CAP stand for?
2.  Can you have CA? (No, because networks assume Partitions).

### Intermediate
1.  Is MySQL CP or AP? (CP by default with replication).
2.  What is Eventual Consistency?

### Architect-Level
1.  Explain how Quorums (R+W > N) allow you to tune Consistency in Cassandra.
2.  Design a system that is AP for writing but CP for reading? (Possible?).

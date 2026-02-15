# Consistency Models

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The Spectrum of Truth

---

## 0. Learning Objectives
*   **Beginner**: Understand "Strong" vs "Eventual".
*   **Developer**: Differentiate "Monotonic Reads" from "Read-your-writes".
*   **Architect**: Master "Linearizability" (CAP) vs "Serializability" (ACID).

---

## 1. Problem Context
**Why does this exist?**
"Consistency" is not a boolean (Yes/No). It is a spectrum.
*   **Strong**: Everybody sees the same thing instantly. (Expensive).
*   **Weak**: Everybody sees... something... eventually. (Cheap).
Architects pick the point on the spectrum that balances **Correctness** vs **Cost/Latency**.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Strong Consistency
*   **Definition**: Once a write is confirmed, ALL subsequent reads see it.
*   **Analogy**: A bank vault.

### 2. Eventual Consistency
*   **Definition**: If no new updates are made, all reads will *eventually* return the last updated value.
*   **Analogy**: Updating a Facebook Profile Picture. (Friends might see the old one for 5 mins).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Consistency Hierarchy

1.  **Strict Consistency** (Theoretical, Impossible).
2.  **Linearizability** (Strongest Real-world). Atomic. Real-time constraint.
3.  **Serializability** (ACID). Transactions allow reordering but result is correct.
4.  **Causal Consistency**. (Comments appear after Posts).
5.  **Eventual Consistency**. (Anything goes).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Tunable Consistency (Quorums)
Formula: $R + W > N$
*   $N$: Number of Replicas (e.g., 3).
*   $W$: Write Quorum (How many must ack write).
*   $R$: Read Quorum (How many must respond to read).

**Configurations**:
*   **Strong**: $N=3, W=2, R=2$. ($2+2 > 3$). Overlap guarantees you see the latest write.
*   **Fast Write**: $N=3, W=1, R=3$. (Risk of data loss).
*   **Fast Read**: $N=3, W=3, R=1$. (Slow write, fast read).

### 2. Linearizability vs Serializability
*   **Linearizability**: Single object. "C" in CAP. Real-time ordering.
*   **Serializability**: Multiple objects (Transactions). "I" in ACID. Execution order.
*   **Strict Serializability**: Both. (Spanner).

---

## 5. Trade-Off Analysis

| Model | Application | Trade-Off |
| :--- | :--- | :--- |
| **Strong** | Banking, Ledger. | High Latency. Low Availability. |
| **Causal** | Chat Apps. | Tracking "Happened-Before" is complex (Vector Clocks). |
| **Eventual** | Social Feed, Analytics. | Stale reads. Confusing UX. |

---

## 6. Scaling Considerations

### The Cost of Synchronization
*   Strong Consistency requires **Chatty Protocols** (Paxos/Raft).
*   Raft is limited to ~5-7 nodes before performance degrades.
*   Eventual Consistency scales linearly to 1000s of nodes (Gossip Protocol).

---

## 7. Failure Scenarios & Recovery

### 1. Read Change in Eventual System
*   **Scenario**: User updates profile. Refreshes page. Sees old profile.
*   **User Reaction**: "It didn't save!" (Clicks save again).
*   **Fix**: **Read-Your-Writes Consistency**. (Sticky Session ensuring user reads from the replica they wrote to).

---

## 8. Security Considerations

*   **Replay Attacks**: In weak consistency models, replaying an old message might be accepted as valid state.
*   **Defense**: Nonces and Timestamps (Lamport Timestamps).

---

## 9. Performance Considerations

*   **Latency**: Causal Consistency provides the best balance of "Usable Consistency" without the cost of Global Locking.
*   **Throughput**: Eventual Consistency allows "Blind Writes" (High throughput).

---

## 10. Real Production Lessons

### The "Comment before Post" Bug
*   **Scenario**: User A posts. User B comments.
*   **DB**: Sharded. Post goes to Shard 1. Comment goes to Shard 2.
*   **Replication**: Shard 2 replicates faster.
*   **Result**: User C sees the Comment, but the Post is missing ("Orphaned Comment").
*   **Fix**: **Causal Consistency**. Ensure Comment carries a vector clock dependency on Post.

---

## 11. Interview Questions

### Basic
1.  Define Eventual Consistency.
2.  Which is faster: Strong or Eventual?
3.  What is a Quorum?
4.  What is Read-your-writes consistency?
5.  Give an example of a system needing Strong Consistency.

### Intermediate
1.  Explain $R + W > N$.
2.  What is the difference between Linearizability and Serializability?
3.  How do Vector Clocks detect conflicts?
4.  What is Conflict Resolution in DynamoDB?
5.  Why is Causal Consistency better than Eventual?

### Advanced
1.  Design a distributed counter with Strong Consistency.
2.  How does Google Spanner achieve External Consistency?
3.  Explain "Monotonic Reads".
4.  Derive the latency cost of Raft during a leader election.
5.  Compare Last-Write-Wins (LWW) vs CRDTs for conflict resolution.

### Architect-Level
1.  "Strong Consistency implies high latency." Is this always true? (Network speed limit).
2.  Architect a collab editing tool (Google Docs). Which model? (Operational Transformation / CRDT).
3.  Evaluate the consistency guarantees of Kafka (Partition ordering).
4.  Design a mechanism to upgrade Eventual Consistency to Strong Consistency on-demand.
5.  Critique the consistency of Blockchain (Nakamoto Consensus). (It is Probabilistic).

---

## 12. Scenario-Based System Design Problems

### 1. Design Google Docs
*   **Model**: Strong Eventual Consistency (CRDTs).
*   **Goal**: Everyone eventually sees the same doc. No conflicts.

### 2. Design a Real-Time Strategy Game
*   **Model**: Lockstep (Strong).
*   **Goal**: All players must agree on unit position. Cost: Latency (Input lag).

---

## 13. Summary & Architect Takeaways

1.  **Don't pay for what you don't need**: Strong Consistency is expensive. Don't use it for Likes.
2.  **User Perception Matters**: "Read-your-writes" is usually enough to make a user happy.
3.  **Conflict Resolution**: If you choose Eventual, you **MUST** have a plan for conflicts (LWW or CRDT).

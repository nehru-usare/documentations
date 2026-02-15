# CAP Theorem

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** The Law of Distributed Systems

---

## 0. Learning Objectives
*   **Beginner**: Understand that you cannot have it all.
*   **Developer**: Know which database categories (SQL vs NoSQL) fit into CP vs AP.
*   **Architect**: Explain why "CA" systems do not strictly exist in distributed computing.

---

## 1. Problem Context
**Why does this exist?**
In a perfect world, our distributed database would be:
1.  **Consistent**: Every node sees the same data at the same time.
2.  **Available**: Every request gets a success response.
3.  **Partition Tolerant**: System works even if the network cable between nodes is cut.
**Eric Brewer** proved in 2000 that you can only have **2 out of 3**.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Consistency (C)
*   **Definition**: Linearizability. Every read receives the most recent write or an error.
*   **Analogy**: If I write "X=10" on a blackboard, the next person to look MUST see "10".

### 2. Availability (A)
*   **Definition**: Every request receives a (non-error) response, without the guarantee that it contains the most recent write.
*   **Analogy**: If I ask a question, I MUST get an answer, even if the answer is "I don't know" or outdated.

### 3. Partition Tolerance (P)
*   **Definition**: The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network between nodes.
*   **Analogy**: The phone line between London and New York is dead.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The "Pick Two" Dilemma

Since networks **WILL** fail (P is non-negotiable in distributed systems), your choice is actually binary:
*   **CP (Consistency + Partition Tolerance)**: Wait for the partition to heal. Return Error/Timeout in the meantime.
*   **AP (Availability + Partition Tolerance)**: Return stale data. Keep serving requests.

| Category | Database Examples | Behavior during Network Cut |
| :--- | :--- | :--- |
| **CP** | Redis, HBase, MongoDB (default), Zookeeper | Writes FAIL. Reads might FAIL (if leader unreachable). |
| **AP** | Cassandra, DynamoDB, CouchDB | Writes SUCCESS (to available nodes). Reads SUCCESS (might be old). |
| **CA** | MySQL (Single Node), Oracle (Single Node) | System FAILS if network partitions (which shouldn't happen inside one server). |

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. CP Mechanics (The Stopper)
*   **Protocol**: Active-Passive or Consensus (Raft/Paxos).
*   **Failure**: If Leader cannot contact Quorum (Majority), it steps down. System becomes Read-Only or Unavailable.
*   **Cost**: Latency. Every write must be acknowledged by multiple nodes.

### 2. AP Mechanics (The Optimist)
*   **Protocol**: Active-Active (Master-Master).
*   **Failure**: Partitioned nodes accept writes independently.
*   **Conflict**: When partition heals, you have two versions of data (`v1` and `v2`).
*   **Resolution**: Last-Write-Wins (LWW) or Vector Clocks.

---

## 5. Trade-Off Analysis

| Strategy | When to Choose | When NOT to Choose |
| :--- | :--- | :--- |
| **CP** | Banking, Inventory, Booking Systems. (Double spending is fatal). | Social Feeds, Likes, Analytics. (Downtime is fatal). |
| **AP** | Social Media, IoT Sensor Logs, Shopping Carts. | Financial Ledger. (Incorrect balance is illegal). |

---

## 6. Scaling Considerations

### Horizontal Scaling Impact
*   **CP**: Harder to scale writes. Writing to 100 nodes with Strong Consistency is slow.
*   **AP**: Easy to scale writes. Add more nodes, they accept writes instantly.

---

## 7. Failure Scenarios & Recovery

### 1. Network Split (Split Brain)
*   **Scenario**: NYC and London datacenters lose connection.
*   **CP System**: One side (Minority) shuts down. Other side (Majority) keeps running.
*   **AP System**: Both sides keep accepting writes.
    *   *Recovery*: When linked, database performs "Read Repair" or "Anti-Entropy" to merge data.

---

## 8. Security Considerations

### 1. Availability Attacks
*   An attacker can induce a Partition (DDoS on network links).
*   If you are **CP**, the attacker effectively shuts you down (Denial of Service).
*   If you are **AP**, you survive, but data drifts.

---

## 9. Performance Considerations

*   **CP Latency**: $WriteLatency = Max(Node1, Node2, ... NodeQuorum)$. Slowest node dictates speed.
*   **AP Latency**: $WriteLatency = Min(AnyNode)$. Fastest node dictates speed.

---

## 10. Real Production Lessons

### MongoDB Default Confusion
*   **Mistake**: Devs think MongoDB is CP.
*   **Reality**: By default, Mongo allows reading from Secondaries (Eventual Consistency).
*   **Lesson**: Explicitly configure `ReadConcern` and `WriteConcern` to achieve true CP.

---

## 11. Interview Questions

### Basic
1.  What does CAP stand for?
2.  Why can't you have CA in a distributed system?
3.  Give an example of a CP database.
4.  Give an example of an AP database.
5.  What is a Network Partition?

### Intermediate
1.  Explain "Consistency" in CAP vs "Consistency" in ACID. (Hint: CAP=Linearizability, ACID=Rules).
2.  How does Cassandra handle CAP?
3.  How does a CP system behave during a partition?
4.  What is Quorum?
5.  Can a database change from CP to AP dynamically? (Yes, Tunable Consistency).

### Advanced
1.  Prove that CA is impossible over an asynchronous network.
2.  Analyze the CAP properties of a Blockchain (Bitcoin). (Hint: Probabilistic Consistency).
3.  How does Raft Consensus relate to CAP?
4.  Design a system that switches between CP and AP based on business hours.
5.  Explain Spanner's claim of being "CA". (Hint: It uses TrueTime/Atomic Clocks to minimize P probability).

### Architect-Level
1.  "The network is reliable nowadays." Refute this statement with data.
2.  Design a shopping cart. AP or CP? justify the revenue impact.
3.  How does the "Two Generals Problem" relate to CAP?
4.  Critique the "PACELC" theorem as an extension of CAP.
5.  Evaluate the trade-off of using Zookeeper (CP) for Service Discovery vs Eureka (AP).

---

## 12. Scenario-Based System Design Problems

### 1. Design an ATM Network
*   **Choice**: CP.
*   **Reason**: You cannot withdraw money if you have $0.
*   **Failure**: If network is down, ATM says "Out of Service". Better than giving free money.

### 2. Design a "Like" Button
*   **Choice**: AP.
*   **Reason**: It doesn't matter if you see 99 or 100 likes. It matters that the user can click the button.

---

## 13. Summary & Architect Takeaways

1.  **P is Inevitable**: You don't choose P. You have P. You choose between C and A.
2.  **Business Drives CAP**: Ask the Product Manager: "Is it worse to show wrong data (AP) or error message (CP)?"
3.  **Tunable Consistency**: Modern DBs (Cassandra, Dynamo) let you choose CAP per query.

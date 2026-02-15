# Replication Strategies

> **Part 4: Data Scaling**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Keeping Copies

---

## 0. Learning Objectives
*   **Beginner**: Know not to put all eggs in one basket (Backups vs Replica).
*   **Developer**: Configure Postgres Streaming Replication.
*   **Architect**: Choose between Multi-Leader and Leaderless replication based on "Write Availability".

---

## 1. Problem Context
**Why does this exist?**
1.  **High Availability**: If Server A crashes, Server B takes over.
2.  **Latency**: Server A is in US. Server B is in Asia. Users connect to the closest one.
3.  **Throughput**: Server A handles Writes. Servers B, C, D handle Reads.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Synchronous Replication
*   Master writes to Disk -> Sends to Slave -> Slave Writes -> Slave Acks -> Master Acks to User.
*   *Pros*: Zero Data Loss.
*   *Cons*: Slow. If Slave dies, Write fails (or hangs).

### 2. Asynchronous Replication
*   Master writes to Disk -> Acks to User -> Sends to Slave later.
*   *Pros*: Fast. Master not blocked by Slave.
*   *Cons*: **Data Loss** if Master crashes before sending.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Topologies

1.  **Single Leader (Master-Slave)**:
    *   All Writes -> Master.
    *   Reads -> Master or Slaves.
    *   *Standard for*: SQL (MySQL, Postgres).

2.  **Multi-Leader (Master-Master)**:
    *   Writes -> Master A OR Master B.
    *   Replication streams both ways.
    *   *Use*: Multi-Datacenter survivability.
    *   *Risk*: **Write Conflicts** (User A changes Title in US. User B changes Title in EU).

3.  **Leaderless (Dynamo-style)**:
    *   Client writes to ANY node.
    *   No Master.
    *   *Use*: Cassandra, DynamoDB.
    *   *Mech*: Quorums & Entropy Repair.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Replication Lag
*   In Async systems, the Slave is always "behind".
*   **Statement-Based Replication**: Forward specific SQL (`INSERT INTO...`). Fast but dangerous with `NOW()` or `RAND()`.
*   **WAL (Write Ahead Log) Shipping**: Forward the byte changes. (Standard).

### 2. Quorums (Leaderless)
*   You don't trust one node. You ask 3.
*   $N=3$ (Replicas).
*   $W=2$ (Must write to 2).
*   $R=2$ (Must read from 2).
*   Since $W + R > N$, you are guaranteed to read the latest data (Pigeonhole Principle).

---

## 5. Trade-Off Analysis

| Strategy | Write Availability | Read Consistency | Complexity |
| :--- | :--- | :--- | :--- |
| **Single Leader** | Low (Master is SPOF) | High | Low |
| **Multi-Leader** | High | Low (Conflicts) | High |
| **Leaderless** | Very High | Tunable (Eventual) | Medium |

---

## 6. Scaling Considerations

### "Read Your Writes" Consistency
*   User updates profile -> Redirected to Profile Page.
*   Profile Page reads from Slave (Lagging).
*   User sees old profile. "It's broken!"
*   **Fix**: Sticky Routing (Read from Master for 1 minute after write) or Versioning.

---

## 7. Failure Scenarios & Recovery

### 1. Split Brain (Multi-Master)
*   Network cut between US and EU.
*   Both sides accept writes.
*   Network heals.
*   **Conflict Hell**: US says "A=1". EU says "A=2".
*   **Fix**: Last Write Wins (LWW) or Manual Merge.

---

## 8. Security Considerations

### 1. Replication Stream Encryption
*   Data travels between datacenters.
*   **Must** use SSL/TLS for the replication channel.
*   Often overlooked in old MySQL setups.

---

## 9. Performance Considerations

*   **Bandwidth**: Replication uses network.
*   **Cross-Region**: US -> Asia replication adds 200ms lag *minimum*.
*   **Throttling**: Don't let replication traffic kill your user traffic. Use separate NICs.

---

## 10. Real Production Lessons

### GitHub's Outage
*   **Event**: Network partition.
*   **Auto-Failover**: Promoted a Slave that was 2 minutes behind.
*   **Result**: 2 minutes of code commits vanished (Orphaned).
*   **Lesson**: Sometimes "Down" is better than "Data Loss".

---

## 11. Interview Questions

### Basic
1.  Difference between Backup and Replication. (Backup is cold/snapshot. Replication is hot/stream).
2.  What is Master-Slave?
3.  What happens if Master dies?
4.  What is Replication Lag?
5.  What is a Heartbeat?

### Intermediate
1.  Explain Async vs Sync replication trade-offs.
2.  How do you scale Writes? (Sharding, not Replication).
3.  Why does Multi-Leader have conflicts?
4.  What is a Quorum?
5.  How does Cassandra handle deleted data? (Tombstones).

### Advanced
1.  Design a conflict resolution strategy for a collaborative text editor (OT / CRDT).
2.  Derive the probability of data loss in a Quorum system with N=3.
3.  Analyze the impact of "Cascading Replication" (Master -> Slave A -> Slave B).
4.  How does chain replication work (MongoDB)?
5.  Explain "Read Repair" vs "Anti-Entropy" (Merkle Trees).

### Architect-Level
1.  "We need 5-nines availability." Architect the replication topology. (Multi-Region Active-Active).
2.  Critique the use of Auto-Failover scripts vs Consensus (Raft/Paxos).
3.  Design a topology where Views are eventually consistent but Payments are strongly consistent.

---

## 12. Scenario-Based System Design Problems

### 1. Design Global Counter
*   **Req**: Count Facebook Likes.
*   **Choice**: Leaderless (Cassandra). Counter CRDTs.

### 2. Design Stock Trading Engine
*   **Req**: Zero Data Loss.
*   **Choice**: Single Leader. Synchronous Replication to at least 1 slave.

---

## 13. Summary & Architect Takeaways

1.  **Async by Default**: The internet is unreliable. Sync replication halts the system too often.
2.  **Reads Scale, Writes Don't**: Adding replicas only helps Read QPS.
3.  **The Lag is Real**: Always design your UI to handle "stale" data gracefully.

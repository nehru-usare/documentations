# ACID Properties Deep Dive

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐ (Core Concept)  
> **Status:** Committed

---

## 0. Learning Objectives
*   **Beginner**: Why money doesn't disappear during a bank transfer.
*   **Developer**: Understanding why your app reads old data (Isolation Levels).
*   **Architect**: Choosing between ACID (SQL) and BASE (NoSQL).

---

## 1. Core Concepts (🟢 Beginner Level)

### The Acronym
1.  **A - Atomicity**: "All or Nothing". If 10 steps succeed and 11th fails, Rollback everything.
2.  **C - Consistency**: Database constraints (Foreign Keys, Unique) are never violated.
3.  **I - Isolation**: Concurrent transactions don't interfere with each other.
4.  **D - Durability**: Once committed, it survives a power outage.

---

## 2. Architecture Breakdown (The Internals)

### 1. Atomicity & Durability (The WAL)
*   **Write-Ahead Log (WAL)**: Before writing to the actual table (heap file), the DB writes the change to an append-only log on disk.
*   **Crash Recovery**: On reboot, DB replays the WAL to reconstruct state.
*   **Rollback**: DB reads the WAL backwards to undo changes.

### 2. Isolation (The Hardest Part)
*   Implemented using **Locks** (2PL) or **MVCC** (Multi-Version Concurrency Control).

---

## 3. Isolation Levels (🔴 Architect Level)

| Level | Dirty Read? | Non-Repeatable Read? | Phantom Read? | Performance |
| :--- | :--- | :--- | :--- | :--- |
| **Read Uncommitted** | Yes | Yes | Yes | Fastest |
| **Read Committed** | No | Yes | Yes | Default (Postgres/Oracle) |
| **Repeatable Read** | No | No | Yes | Default (MySQL) |
| **Serializable** | No | No | No | Slowest (Strict serialization) |

### Real World Consequences
*   **Dirty Read**: User A sees uncommitted data from User B. (B rolls back, A acted on false info).
*   **Non-Repeatable Read**: User A queries `balance=100`. User B updates to `200`. User A queries again: `200`. (Data changed mid-transaction).
*   **Phantom Read**: User A: `SELECT count(*) WHERE age > 18` -> Returns 10. User B inserts a new 19yo. User A repeats -> Returns 11.

---

## 4. Trade-Off Analysis

### ACID vs Performance
*   **CAP Theorem**: ACID provides **Consistency**.
*   In distributed systems, maintaining ACID across nodes is expensive (2 Phase Commit, Latency).
*   NoSQL drops ACID for **BASE** (Basically Available, Soft state, Eventual consistency).

---

## 5. Scaling Considerations

### Distributed ACID
*   **The Problem**: Commit across Shard 1 and Shard 2.
*   **Solutions**:
    *   **2PC (Two Phase Commit)**: Coordinator asks "Prepare?", then "Commit?". Blocking. Slow.
    *   **NewSQL (Spanner/CockroachDB)**: Uses TrueTime (Atomic Clocks) to achieve distributed ACID efficiently.

---

## 6. Failure Scenarios & Recovery

### 1. Torn Writes
*   Writing a 16KB page to disk. Power fails after 8KB.
*   **Fix**: Double Write Buffer (MySQL). Writes page to a safe area first, then to main area.

---

## 7. Security Considerations

### 1. Lost Updates
*   User A reads Row (Val=10). User B reads Row (Val=10).
*   A writes 11. B writes 12.
*   **Result**: A's update is lost.
*   **Fix**: Optimistic Locking (Version column). `UPDATE ... WHERE version=1`.

---

## 8. Performance Considerations

*   **Lock Contention**:
    *   If many transactions try to update the same row (Hot Row), they serialize. Performance tanks.
    *   **Fix**: Batch updates or use partitioning.

---

## 9. Real Production Lessons

### "The Default Trap"
*   Developers assume `Serializable` behavior but get `Read Committed` (Postgres Default).
*   Bug: "I checked if stock > 0, then deducted. But stock went negative!".
*   **Fix**: `SELECT ... FOR UPDATE` (Explicit Locking) or `Repeatable Read`.

---

## 10. Interview Questions

### Basic
1.  Explain ACID.
2.  What is a Dirty Read?
3.  Difference between Commit and Rollback.

### Intermediate
1.  Explain Isolation Levels. Which is default in MySQL vs Postgres?
2.  How does WAL ensure Durability?
3.  What is MVCC?

### Advanced
1.  Design a system that needs ACID across microservices. (Saga Pattern vs Distributed Transaction).
2.  How does Postgres handle "Vacuum" in relation to MVCC?
3.  Explain "Write Skew" in Snapshot Isolation.

---

## 11. Summary & Architect Takeaways

1.  **Defaults Matter**: Know your DB's default isolation level.
2.  **Concurrency is Hard**: Don't rely on intuition. Reproduce race conditions with tests.
3.  **Disk is Truth**: The WAL is the only thing that matters.

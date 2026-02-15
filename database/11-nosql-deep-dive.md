# NoSQL Deep Dive (LSM Trees, SSTables)

> **Part 2: Internals**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** Compacting...

---

## 0. Learning Objectives
*   **Beginner**: Why Cassandra writes are faster than MySQL writes.
*   **Developer**: Understanding Compaction strategies.
*   **Architect**: Choosing the right storage engine for the workload.

---

## 1. Core Concepts (🟢 Beginner Level)

### B-Tree vs LSM Tree
*   **B-Tree (SQL)**: Updates happen *in place*. Random I/O. Best for Reads.
*   **LSM Tree (NoSQL)**: Updates are *appended* to a log. Sequential I/O. Best for Writes.
*   **Examples**: Cassandra, RocksDB, LevelDB, HBase.

---

## 2. Architecture Breakdown (LSM Tree)

### 1. MemTable (Memory)
*   Incoming write goes to **MemTable** (In-RAM Red-Black Tree). Sorted.
*   Also written to **WAL** (Append-only Log) for durability.
*   *Speed*: Instant (RAM).

### 2. SSTable (Disk)
*   When MemTable is full (e.g., 64MB), it is flushed to disk as an **SSTable** (Sorted String Table).
*   **Immutable**: SSTables are never modified.
*   *Result*: Disk has many SSTable files: `[A-C]`, `[K-M]`, etc.

### 3. Compaction
*   Background process merges old SSTables into new, larger ones.
*   Discards overwritten/deleted keys (Garbage Collection).

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. The Read Path (Complexity)
*   To find Key "X":
    1.  Check MemTable.
    2.  Check SSTable L0 (Most recent).
    3.  Check SSTable L1...
*   *Problem*: Reads are slow (Scan many files).
*   *Optimization*: **Bloom Filters**. Each SSTable has a filter. "Might contains X?" -> No -> Skip file.

---

## 4. Scenarios

| Feature | LSM Tree (Cassandra) | B-Tree (MySQL) |
| :--- | :--- | :--- |
| **Write Pattern** | Append (Sequential) | Update Page (Random) |
| **Write Speed** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Read Speed** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Space Amp** | Low (Compressed) | High (Fragmentation) |

---

## 5. Scaling Considerations

### Write Amplification
*   In Leveled Compaction, data is rewritten many times.
*   Pays write cost asynchronously to gain read performance later.

---

## 6. Real Production Lessons

### "The Tombstone Issue"
*   **Deleting** in NoSQL writes a special marker called a "Tombstone".
*   If you delete 50% of data, read speed *slows down* (DB must read tombstones to know data is gone).
*   **Lesson**: Avoid frequent deletes in LSM stores. Use TTL (Time To Live).

---

## 7. Interview Questions

### Basic
1.  What is LSM Tree?
2.  Why are Writes fast? (Sequential I/O).
3.  What is an SSTable?

### Intermediate
1.  Explain the role of Bloom Filters in NoSQL.
2.  What is Compaction? Why is it needed?
3.  MemTable vs SSTable.

### Advanced
1.  Compare Leveled Compaction (LevelDB) vs Size-Tiered Compaction (Cassandra).
2.  Impact of Tombstones on query performance.
3.  Design a Key-Value store using a Log + Index (Bitcask model).

---

## 8. Summary & Architect Takeaways

1.  **Write Heavy?** Use LSM (Cassandra/RocksDB).
2.  **Read Heavy?** Use B-Tree (SQL/MongoDB).
3.  **Immutable**: The secret to speed is never modifying data in place.

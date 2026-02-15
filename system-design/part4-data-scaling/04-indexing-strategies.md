# Indexing Strategies

> **Part 4: Data Scaling**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** O(log N) or Die

---

## 0. Learning Objectives
*   **Beginner**: Why `CREATE INDEX` makes reads fast but writes slow.
*   **Developer**: Use "Covering Indexes" to avoid disk seeks.
*   **Architect**: Choosing the right storage engine (B-Tree for SQL, LSM for NoSQL).

---

## 1. Problem Context
**Why does this exist?**
Table has 1 Billion rows.
`SELECT * FROM Users WHERE email = 'bob@gmail.com'`.
*   **Without Index**: Read 1 Billion rows. (Full Table Scan). 1 Hour.
*   **With Index**: Read 5 nodes. (Binary Search). 10ms.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. B-Tree (Balanced Tree)
*   Standard for SQL (MySQL, Postgres).
*   Optimized for **Read** heavy workloads.
*   Updates are "in-place". Random I/O.

### 2. LSM Tree (Log Structured Merge Tree)
*   Standard for NoSQL (Cassandra, RocksDB).
*   Optimized for **Write** heavy workloads.
*   Updates are "Append Only" (Sequential I/O).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Index Types

1.  **Clustered Index**:
    *   The Index *IS* the Table. The leaf nodes contain the actual row data.
    *   Sorted by Primary Key.
    *   Only 1 per table.

2.  **Non-Clustered (Secondary) Index**:
    *   Contains the Indexed Column + Pointer to Primary Key.
    *   **Double Lookup**: Index -> PK -> Clustered Index -> Data.

3.  **Covering Index**:
    *   `CREATE INDEX idx_a_b ON table (a, b)`.
    *   `SELECT b FROM table WHERE a = 5`.
    *   Database reads *only* index. Never touches the table heap. Extremely fast.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. LSM Tree Internals
*   **MemTable**: Writes go to RAM (Sorted Tree).
*   **SSTable**: When RAM is full, flush to Disk (Immutable Sorted Strings).
*   **Compaction**: Background process merges SSTables to remove duplicates/deleted items.
*   **Bloom Filter**: Checks if key exists before scanning SSTables.

### 2. Geospatial Indexes
*   **Quadtree**: Divide world into 4 squares. Recursively divide.
*   **Geohash**: Base32 string representation of lat/long. "dr5r" is New York. Prefix matching = Proximity search.

---

## 5. Trade-Off Analysis

| Strategy | Read Speed | Write Speed | Space |
| :--- | :--- | :--- | :--- |
| **B-Tree** | ⭐⭐⭐⭐⭐ | ⭐⭐ | Compact |
| **LSM Tree** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Needs overhead (Compaction) |
| **Hash Index** | ⭐⭐⭐⭐⭐ (O(1)) | ⭐⭐⭐ | No Range Queries |

---

## 6. Scaling Considerations

### Write Amplification
*   In Index-heavy tables, 1 `INSERT` might trigger 5 Disk Writes (1 for Data + 4 for Indexes).
*   *Rule*: Don't index everything. Only index patterns used in `WHERE`, `JOIN`, `ORDER BY`.

---

## 7. Failure Scenarios & Recovery

### 1. Index Fragmentation
*   **Scenario**: Heavy DELETEs/UPDATEs on a B-Tree.
*   **Result**: Pages are half-empty. Disk usage doubles. Scans take 2x time.
*   **Fix**: `REINDEX` or `OPTIMIZE TABLE`. (Careful: Locks table in some DBs).

---

## 8. Security Considerations

### 1. Indexing Sensitive Data
*   If you index `SSN`, the logic is potentially readable in the Index file even if Table is encrypted (depending on DB implementation).

---

## 9. Performance Considerations

*   **Cardinality**: Low cardinality columns (Gender: M/F) are bad candidates for B-Tree Indexes.
*   **Composite Indexes**: Order matters!
    *   Index `(A, B)`.
    *   Query `WHERE A=1 AND B=1` -> **Fast**.
    *   Query `WHERE B=1` -> **Slow** (Index not used).

---

## 10. Real Production Lessons

### Uber's move to Schemaless (LSM)
*   **Problem**: Updating Indexes on MySQL (B-Tree) locked tables for hours.
*   **Solution**: Moved to a bespoke sharded store on top of MySQL using Append-Only logs (LSM-like behavior).

---

## 11. Interview Questions

### Basic
1.  What is a Primary Key?
2.  Difference between Clustered and Secondary Index.
3.  Why is `SELECT *` bad for Indexes?
4.  What data structure does MySQL use?
5.  What is Cardinality?

### Intermediate
1.  Explain the "Leftmost Prefix" rule in Composite Indexes.
2.  Why are LSM trees faster for writes?
3.  What is a Bloom Filter?
4.  How does an Index slow down Writes?
5.  What is a Full Text Index?

### Advanced
1.  Design a Geospatial Index using Geohashing.
2.  Compare Hash Map vs B-Tree for database indexing.
3.  Analyze the worst-case scenario for an LSM Tree read.
4.  How does Postgres `GIN` index work (for JSONB)?
5.  Explain "Index Skip Scan".

### Architect-Level
1.  "Our write throughput is bottlenecked by disk IOPS." Optimize the indexing strategy. (Drop unused indexes, Switch to LSM).
2.  Design a Multi-Dimensional Index (e.g., Search for Price + Location + Time). (R-Tree / Space Filling Curves).
3.  Evaluate the trade-off of "Bitmap Indexes" in data warehousing.

---

## 12. Scenario-Based System Design Problems

### 1. Design Uber Geo-Lookup
*   **Req**: Find drivers near user.
*   **Naive**: SQL `WHERE dist < 5`. (Math on every row. Slow).
*   **Smart**: Geohash. `WHERE geohash LIKE 'dr5r%'`. (String Prefix match. Fast B-Tree lookup).

### 2. Design Analytics Event Store
*   **Req**: 50k Writes/sec.
*   **Choice**: Cassandra (LSM Tree).
*   **Why**: Sequential disk writes can saturate SSD. B-Tree random writes would kill the disk.

---

## 13. Summary & Architect Takeaways

1.  **Reads vs Writes**: The fundamental database trade-off. B-Tree for Read. LSM for Write.
2.  **Covering Indexes**: The single best optimization for SQL queries.
3.  **Maintenance**: Indexes fragment. They need care and feeding (Reindexing).

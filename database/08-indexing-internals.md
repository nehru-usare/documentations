# Database Indexing Internals

> **Part 2: Internals**  
> **Difficulty:** ⭐⭐⭐⭐ (Deep Dive)  
> **Status:** Optimized

---

## 0. Learning Objectives
*   **Beginner**: Why `CREATE INDEX` makes queries fast but inserts slow.
*   **Developer**: Choosing between B-Tree, Hash, and GIN.
*   **Architect**: Designing generic indexing strategies for high-throughput apps.

---

## 1. Core Concepts (🟢 Beginner Level)

### The Phonebook Analogy
*   **Table Scan**: Reading every name in the book to find "Smith". (O(N)).
*   **Index**: Jumping to "S", then "Sm". (O(log N)).

### Cost of Indexes
*   **Reads**: Faster.
*   **Writes**: Slower. (Must update Table + Update Index B-Tree).
*   **Storage**: Index takes disk space.

---

## 2. Architecture Breakdown (The Data Structures)

### 1. B-Tree (Balanced Tree) - *The Standard*
*   **Best For**: `<`, `>`, `=`, `BETWEEN`, `ORDER BY`.
*   **Structure**: Root -> Branch -> Leaf Pages.
*   **Leaf Pages**: Contain (`Key`, `Row Pointer`). Sorted Linked List.
*   **Depth**: Typically 3-4 levels (even for millions of rows). Access is constant.

### 2. Hash Index
*   **Best For**: `=` (Equality) only.
*   **Structure**: Hash Map. O(1) Access.
*   **Limitation**: Cannot do Range scan (`WHERE age > 18`). Storage inefficient for sparse data.

### 3. GIN (Generalized Inverted Index)
*   **Best For**: Arrays, JSONB, Full Text Search.
*   **Structure**: `Token -> List of Row IDs`.
*   Example: Tagging system. `Post -> Tags`. Index allows `WHERE 'java' = ANY(tags)`.

### 4. GiST (Generalized Search Tree)
*   **Best For**: Geometric (PostGIS), Nearest Neighbor.
*   **Structure**: R-Tree variants.

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. Clustered vs Non-Clustered (Secondary)
*   **Clustered Index**: The Table *is* the Index. (e.g., MySQL InnoDB Primary Key). Leaf nodes contain *actual row data*.
    *   *Limit*: Only 1 per table.
*   **Non-Clustered (Secondary)**: Separate structure. Leaf nodes contain *Pointer to Row* (or PK in InnoDB).
    *   *Double Lookup*: Index Scan -> Get PK -> Primary Key Scan -> Get Row.

### 2. Covering Index
*   Query: `SELECT email FROM users WHERE id = 5`.
*   Index: `(id, email)`.
*   **Optimization**: DB finds answer *in the index* (Leaf node). Does NOT need to look up the Heap Table.
*   *Massive Performance Win*.

---

## 4. Trade-Off Analysis

| Index Type | Read Perf | Write Perf | Range Query? |
| :--- | :--- | :--- | :--- |
| **B-Tree** | ⭐⭐⭐⭐ | ⭐⭐⭐ | Yes |
| **Hash** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | No |
| **LSM (NoSQL)**| ⭐⭐ | ⭐⭐⭐⭐⭐ | Yes |

---

## 5. Scaling Considerations

### Write Amplification
*   Adding 5 indexes on a table = 1 INSERT calls 6 writes (1 Table + 5 B-Trees).
*   **Rule of Thumb**: Don't index everything. Index what you query.

---

## 6. Failure Scenarios & Recovery

### 1. Index Fragmentation
*   Frequent UPDATE/DELETE leaves holes in B-Tree pages.
*   **Result**: Index bloats. Cache hit ratio drops.
*   **Fix**: `REINDEX` (Postgres) or `OPTIMIZE TABLE` (MySQL). Locks table! Use `CONCURRENTLY`.

---

## 7. Security Considerations

### 1. Information Leakage
*   If you index an encrypted column (Deterministic Encryption), frequency analysis on the index can break encryption.

---

## 8. Performance Considerations

*   **Composite Indexes (Order Matters)**:
    *   Index on `(Last_Name, First_Name)`.
    *   Query `WHERE First_Name = 'Bob'` -> **Index NOT used**. (Prefix Rule).
    *   Query `WHERE Last_Name = 'Smith'` -> **Index Used**.

---

## 9. Real Production Lessons

### "The Null Trap"
*   `WHERE column IS NULL` often cannot use B-Tree indexes efficiently (Depends on DB).
*   **Partial Index**: `CREATE INDEX ... WHERE status != 'ARCHIVED'`. Saves index size.

---

## 10. Interview Questions

### Basic
1.  Why not index every column?
2.  Difference between Clustered and Non-Clustered index.
3.  What is a Composite Index?

### Intermediate
1.  Explain B-Tree structure. Why is it used over Binary Search Tree? (Disk page alignment).
2.  How does `LIKE 'abc%'` use index vs `LIKE '%abc'`? (Prefix vs Suffix).
3.  What is a Covering Index?

### Advanced
1.  Implement a search for "Nearest Coffee Shop" (Geo-spatial indexing).
2.  Compare LSM Tree (Write heavy) vs B-Tree (Read heavy).
3.  How does Postgres `HOT` (Heap Only Tuples) optimization work?

---

## 11. Summary & Architect Takeaways

1.  **Explain Plan**: Always run `EXPLAIN` to verify index usage.
2.  **Leftmost Prefix**: Remember column order in composite indexes.
3.  **Delete Unused**: Unused indexes are pure tech debt (Slow down writes, consume RAM).

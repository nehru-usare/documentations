# Databases (SQL vs NoSQL)

> **Part 3: Core Components**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** The Holy War

---

## 0. Learning Objectives
*   **Beginner**: Know the difference between a Table (SQL) and a JSON Doc (NoSQL).
*   **Developer**: Choose the right DB for the job (Don't put Graph data in Postgres).
*   **Architect**: Master the Migration patterns and Polyglot Persistence.

---

## 1. Problem Context
**Why does this exist?**
For 30 years, RDBMS (SQL) was the only choice.
Then Google/Amazon hit a wall.
*   SQL scales **Vertically** (Bigger Server). Expensive. Hard Limit.
*   NoSQL scales **Horizontally** (More Servers). Cheap. Infinite Limit.
Now we have 100 choices. Choosing wrong kills your startup.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. SQL (Relational)
*   **Interface**: Structured Query Language.
*   **Structure**: Tables, Rows, Foreign Keys.
*   **Philosophy**: ACID (Safety First).
*   **Examples**: PostgreSQL, MySQL, Oracle.

### 2. NoSQL (Non-Relational)
*   **Interface**: API / JSON.
*   **Structure**: Flexible (Documents, Key-Value).
*   **Philosophy**: BASE (Scale First).
*   **Examples**: MongoDB, Redis, Cassandra, Neo4j.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The 4 Types of NoSQL

1.  **Key-Value**: `Map<String, Object>`.
    *   *Use*: Caching, Sessions, Profiles.
    *   *Ex*: Redis, DynamoDB.
2.  **Document**: JSON storage.
    *   *Use*: CMS, Catalogs, Rapid Prototyping.
    *   *Ex*: MongoDB, Couchbase.
3.  **Wide-Column**: 2D Key-Value.
    *   *Use*: Time-series, IoT, Massive Writes.
    *   *Ex*: Cassandra, HBase.
4.  **Graph**: Nodes and Edges.
    *   *Use*: Social Networks, Fraud Detection.
    *   *Ex*: Neo4j, Neptune.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. ACID (SQL)
*   **Atomicity**: All or Nothing.
*   **Consistency**: Data obeys rules (Constraints).
*   **Isolation**: Transactions don't interfere.
*   **Durability**: Written to disk.

### 2. BASE (NoSQL)
*   **Basic Availability**: System works mostly.
*   **Soft State**: State might change without input (Eventual Consistency).
*   **Eventual Consistency**: It will be correct... later.

---

## 5. Trade-Off Analysis

| Feature | SQL | NoSQL |
| :--- | :--- | :--- |
| **Schema** | Rigid (Alter Table is hard) | Flexible (schemaless) |
| **Scaling** | Vertical (Read Replicas help) | Horizontal (Native Sharding) |
| **Joins** | Powerful | Non-existent (App side joins) |
| **Transactions** | Strong (ACID) | Weak (or Single Doc only) |

---

## 6. Scaling Considerations

### The Sharding Problem
*   SQL databases *can* be sharded (Vitess, Citus). But it breaks Joins and Foreign Keys.
*   NoSQL databases are *built* to be sharded. They don't have Joins to break.

---

## 7. Failure Scenarios & Recovery

### 1. Schema Migration Lock
*   **Scenario**: `ALTER TABLE Users ADD COLUMN age INT`.
*   **SQL**: On a 1TB table, this locks the table for Hours. Downtime.
*   **NoSQL**: Instant. Just start writing the field. Old docs just don't have it.

---

## 8. Security Considerations

### 1. SQL Injection
*   The #1 Hack.
*   NoSQL is generally immune to *SQL* injection, but vulnerable to *NoSQL* Injection (passing JSON query operators).

---

## 9. Performance Considerations

*   **Read**: SQL B-Trees are fast for range queries. NoSQL Hash Maps are O(1) for point lookups.
*   **Write**:
    *   SQL: Slow (ACID, Fsync, Constraints).
    *   NoSQL (Cassandra): Fast (Append Only LSM Tree).

---

## 10. Real Production Lessons

### The "Migration Back to SQL"
*   **Trend**: many startups began with Mongo (easy).
*   **Reality**: Data is relational. App side joins became slow/buggy. Data integrity issues arose.
*   **Result**: Migrated back to Postgres.
*   **Lesson**: Use NoSQL for *Volume*, not for *Laziness*.

---

## 11. Interview Questions

### Basic
1.  What does NoSQL stand for? (Not Only SQL).
2.  Difference between Vertical and Horizontal scaling.
3.  Name the 4 types of NoSQL.
4.  What is ACID?
5.  What is a Foreign Key?

### Intermediate
1.  Why is MongoDB faster for writes than MySQL? (Safety trade-off).
2.  Explain Eventual Consistency in Cassandra.
3.  When would you choose a Graph DB?
4.  How do you do JOINs in NoSQL? (Denormalization).
5.  What is the CAP theorem status of MySQL vs Cassandra?

### Advanced
1.  Analyze LSM Trees (Log Structured Merge) vs B-Trees.
2.  Design a Schema for a Chat App in Cassandra. (Partition Key is crucial).
3.  Explain NewSQL (Spanner/CockroachDB) - How do they get ACID + Scale?
4.  How does Consistent Hashing work in DynamoDB?
5.  Critique Polyglot Persistence (Operational complexity).

### Architect-Level
1.  "We need strict transactions but also 100k writes/sec." Architect the data layer. (Sharded SQL or NewSQL).
2.  Design the Migration plan from Oracle to MongoDB with 0 downtime. (Dual Write).
3.  Evaluate the TCO (Total Cost of Ownership) of DynamoDB (On-Demand) vs Provisioned Cassandra.

---

## 12. Scenario-Based System Design Problems

### 1. Design User Profile Service
*   **Data**: JSON blob, rarely changes ID.
*   **Choice**: Document DB (MongoDB). Or Key-Value (DynamoDB).

### 2. Design Banking Ledger
*   **Data**: Transactions, Balance.
*   **Req**: Strong ACID.
*   **Choice**: SQL (Postgres). Do not use Eventual Consistency for Money.

---

## 13. Summary & Architect Takeaways

1.  **Default to SQL**: Unless you have a specific reason (Scale, Graph, Flexibility), SQL is the safe bet.
2.  **NoSQL is for specific problems**: Use Redis for Speed. Use Cassandra for Write Volume. Use Neo4j for Connections.
3.  **Data Shape dictates DB**: Relational Data -> SQL. Unstructured Data -> NoSQL.

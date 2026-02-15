# Search Systems

> **Part 3: Core Components**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** Finding the Needle

---

## 0. Learning Objectives
*   **Beginner**: Understand why `SELECT * FROM Table WHERE text LIKE '%term%'` is slow.
*   **Developer**: Master Inverted Indexes and Tokenization.
*   **Architect**: Scale an Elasticsearch cluster for 1 Billion documents.

---

## 1. Problem Context
**Why does this exist?**
SQL Databases are optimized for **Exact Match** (`ID = 5`).
They are terrible at **Full-Text Search**.
*   Query: "Find shoes that are red and under $50".
*   SQL: Full Table Scan. O(N). Slow.
*   Search Engine: Inverted Index Lookup. O(1). Fast.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Inverted Index
*   **Definition**: A map of `Word -> List[Document IDs]`.
*   **Analogy**: The Index at the back of a book.
    *   "Apple": Page 5, 10, 20.
    *   "Banana": Page 3.

### 2. Tokenization
*   Splitting text into words (Tokens).
*   "Hello, World!" -> `["hello", "world"]`.

### 3. Analysis (Stemming/Lemmatization)
*   Standardizing words.
*   "Running", "Run", "Ran" -> `run`.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The ELK Stack (Elasticsearch, Logstash, Kibana)
1.  **Elasticsearch**: The Search Engine (Database).
2.  **Logstash**: The Ingestion Pipeline (ETL).
3.  **Kibana**: The Visualization UI (Dashboard).

### Query Types
*   **Term Query**: Exact match (`status: "active"`).
*   **Match Query**: Full text (`message: "error in payment"`).
*   **Fuzzy Query**: Approximate match (`user: "jhon"` finds "john").

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Lucene Segments
*   Elasticsearch is built on **Apache Lucene**.
*   Data is stored in immutable **Segments**.
*   **Merge**: Background process merges small segments into bigger ones (Costly CPU/IO).

### 2. Scattering and Gathering
*   **Query Phase**: The Coordinator Node sends the query to ALL shards.
*   **Fetch Phase**: Each shard returns Top 10 results. Coordinator merges them (Top 10 of Top 10s) and fetches full docs.

---

## 5. Trade-Off Analysis

| Feature | ElasticSearch | Relational DB |
| :--- | :--- | :--- |
| **Search Speed** | Extremely Fast | Slow (LIKE %) |
| **Write Speed** | Slow (Analysis overhead) | Fast |
| **Consistency** | Near Real-Time (1sec refresh lag) | Strong (Immediately visible) |
| **Transactions** | No | Yes |

---

## 6. Scaling Considerations

### Sharding vs Replication
*   **Sharding**: Splits data. Increases **Write Capacity** and **Storage**.
*   **Replication**: Copies data. Increases **Read Throughput** and **Availability**.

### The "Deep Paging" Problem
*   User asks for Page 10,000.
*   ES must fetch 100,000 docs from *each shard*, sort them, and discard 99,990.
*   **Impact**: Cluster crash.
*   **Fix**: `search_after` (Cursor) instead of `from/size`.

---

## 7. Failure Scenarios & Recovery

### 1. Split Brain
*   **Scenario**: Master node disconnects. Cluster elects 2 masters.
*   **Result**: Data Corruption.
*   **Fix**: `discovery.zen.minimum_master_nodes = N/2 + 1`. (Quorum).

---

## 8. Security Considerations

### 1. No Security by Default (Historically)
*   Old ES versions had no auth.
*   **Risk**: Public IP = Data Leak.
*   **Fix**: Enable X-Pack Security (TLS + Basic Auth).

---

## 9. Performance Considerations

*   **Indexing Rate**: Heavy indexing slows down searching.
*   **Optimization**: Use `bulk` API. Disable `refresh_interval` during bulk loads.

---

## 10. Real Production Lessons

### The "Mapping Explosion"
*   **Scenario**: Logging JSON objects with dynamic keys.
*   **Event**: 1 Million unique fields created in index.
*   **Result**: Cluster State becomes too large. Nodes crash.
*   **Fix**: `index.mapping.total_fields.limit`. Strict mapping.

---

## 11. Interview Questions

### Basic
1.  What is an Inverted Index?
2.  Difference between SQL LIKE and Full Text Search.
3.  What is a Shard in Elasticsearch?
4.  What is "Near Real-Time" search?
5.  What is Tokenization?

### Intermediate
1.  How does Fuzzy Search work? (Levenshtein Distance).
2.  Explain the Scorer (TF/IDF or BM25).
3.  Why is Deep Paging slow in ES?
4.  Write vs Read tradeoffs in Search Systems.
5.  How do you handle synonyms? ("Mobile" = "Phone").

### Advanced
1.  Design a Typeahead (Autocomplete) system. (N-Grams / Edge N-Grams).
2.  Analyze the impact of updating a document in Lucene (Delete + Insert).
3.  How to optimize ES for log ingestion (Time-based indices, Rollover API)?
4.  Explain "Routing" in ES to optimize unique user queries.
5.  Compare Solr vs Elasticsearch.

### Architect-Level
1.  "We have 1PB of logs." Design the Hot-Warm-Cold architecture for ES.
2.  Critique the use of Elasticsearch as a Primary Database. (Don't do it).
3.  Design a Federated Search across SQL, NoSQL, and ES.

---

## 12. Scenario-Based System Design Problems

### 1. Design E-Commerce Search
*   **Req**: Filters (Size, Color), Text (Brand), Relevance.
*   **Implementation**: Faceted Search. `bool` query combining `must` (Text) and `filter` (Price).

### 2. Design Log Search (Splunk Clone)
*   **Req**: High Write Volume.
*   **Implementation**: Daily Indices (`logs-2023-10-01`). Index Templates. Lifecycle Policies (Delete after 30 days).

---

## 13. Summary & Architect Takeaways

1.  **SQL is not for Search**: Don't abuse `LIKE`. Use the right tool.
2.  **Denormalize Data**: ES needs flat documents. No Joins.
3.  **Hardware Matters**: ES loves RAM (Filesystem Cache) and Fast Disk (NVMe).

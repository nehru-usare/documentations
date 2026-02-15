# ELK Stack (Logging)

> **Part 5: Observability**  
> **Difficulty:** ⭐⭐⭐ (Infrastructure)  
> **Status:** Indexing...

---

## 0. Learning Objectives
*   **Beginner**: grep is not enough.
*   **Developer**: Why parsing JSON is faster than parsing Regex.
*   **Architect**: Managing an Elasticsearch Cluster without going bankrupt.

---

## 1. Context
Microservices produce massive log volumes distributed across 100s of pods. You cannot SSH to read them.
*   **ELK**: Elasticsearch, Logstash, Kibana. (Now EFK with Fluentd/Fluentbit).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. The Pipeline
1.  **Collection**: Fluentd/Filebeat runs as DaemonSet. Tails docker logs.
2.  **Processing**: Transforms logs. `INFO: User login` -> `{"level": "INFO", "msg": "User login"}`.
3.  **Storage**: Elasticsearch (Inverted Index).
4.  **UI**: Kibana. Search and Dashboard.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Structured Logging
*   **Unstructured**: `log.info("User " + user + " bought item " + item);`
    *   Hard to query: "How many items bought?" -> Requires Regex.
*   **Structured (JSON)**: `log.info("Order", "user_id", u, "item_id", i);`
    *   Output: `{"event": "Order", "user_id": 12, "item_id": 55}`.
    *   **Elasticsearch** loves JSON. Instant aggregations.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Elasticsearch Indexing
*   **Inverted Index**: Like a book index. `Token -> Document IDs`.
*   **Shards**: Indices are split into Shards. High write throughput.
*   **Replicas**: High read availability.

### 2. Hot-Warm-Cold Architecture
*   Logs are "Time Series Data". Recent logs (Hot) are queried often. Old logs (Cold) rarely.
*   **Hot Nodes**: Fast SSD. Latest 3 days.
*   **Warm Nodes**: HDD. Day 4-30.
*   **Cold Nodes**: S3/Glacier Snapshot. Day 30+.
*   *Saves massive cost*.

---

## 5. Trade-Off Analysis

| Tool | Pros | Cons |
| :--- | :--- | :--- |
| **ELK (Java)** | Powerful, Industry Standard | Heavy RAM usage (JVM) |
| **Loki (Go)** | Lightweight, Label-based | Limited full-text search |
| **Splunk** | Enterprise polished | Expensive ($$$) |

---

## 6. Scaling Considerations

### Buffer Layer (Kafka/Redis)
*   If traffic spikes, Logstash usually chokes / Elasticsearch slows down.
*   **Pattern**: App -> Fluentd -> **Kafka** -> Logstash -> ES.
*   Kafka acts as a shock absorber.

---

## 7. Failure Scenarios & Recovery

### 1. Split Brain
*   ES Master nodes lose quorum. Cluster blocks writes.
*   **Fix**: Min 3 Master-eligible nodes. Logic `N/2 + 1`.

---

## 8. Security Considerations

### 1. PII Masking
*   Logs often contain Emails/Passwords accidentally.
*   Use Logstash/Fluentd filters to **Redact** sensitive patterns (Credit Card Regex) *before* sending to storage.

---

## 9. Performance Considerations

*   **Field Explosion**:
    *   If every app sends different JSON keys (`user_id`, `userId`, `UserID`), ES creates messy mappings.
    *   **Fix**: ECS (Elastic Common Schema). Standardize naming.

---

## 10. Real Production Lessons

### "Logging is expensive"
*   Logging `DEBUG` in production can kill your performance and fill your disk.
*   **Dynamic Levels**: Configure apps to change Log Level via API (Spring Boot Actuator) without restart.

---

## 11. Interview Questions

### Basic
1.  Components of ELK.
2.  Why use Structured Logging?
3.  What is a DaemonSet?

### Intermediate
1.  Explain Hot-Warm-Cold architecture.
2.  How to handle PII in logs?
3.  Why do we need a Buffer key (Kafka)?

### Advanced
1.  Architect an ELK stack for 10TB/day ingest.
2.  Compare ELK vs Loki. (Loki indexes only metadata, ES indexes content).
3.  Debug an Elasticsearch cluster Red status (Unassigned shards).

---

## 12. Summary & Architect Takeaways

1.  **JSON or Die**: Enforce JSON logging at the library level across the org.
2.  **Cost Awareness**: Logs are the most expensive observability data. Don't log garbage.
3.  **Lifecycle**: Automate index rotation (ILM) to delete old logs.

# Time Series Databases (TSDB)

> **Part 4: Specialized DBs**  
> **Difficulty:** ⭐⭐⭐ (Domain Specific)  
> **Status:** 1000 datapoints/sec

---

## 0. Learning Objectives
*   **Beginner**: Why storing metrics in MySQL is a bad idea.
*   **Developer**: Understanding Dimensionality and Retention.
*   **Architect**: Designing an IoT ingestion pipeline.

---

## 1. Context
**Data Shape**: Timestamp + Value + Tags.
*   High Write Volume (Append Only).
*   Query by Time Range.
*   Old data becomes irrelevant (Downsampling).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. The Workload
*   **Write**: 99% Insert. 1% Update (Rare backfill). No Delete (Limit by retention).
*   **Read**: Aggregations (`AVG`, `MAX` over 1 hour).

### 2. Databases
*   **InfluxDB**: Push model. IoT standard.
*   **Prometheus**: Pull model. K8s standard.
*   **TimescaleDB**: Postgres extension. SQL interface.

---

## 3. Architecture Breakdown (Compression)

### Delta-of-Delta (Gorilla Algorithm)
*   Timestamp: 100, 101, 102, 103.
*   Store: 100, +1, +1, +1.
*   Takes 64 bits -> Compresses to 1 bit.
*   **Result**: 90% compression ratio. Crucial for cost.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Downsampling (Rollups)
*   **Raw**: Store every second for 7 days.
*   **Medium**: Store 1-minute avg for 30 days.
*   **Long**: Store 1-hour avg for 1 year.
*   **Continuous Queries**: Background jobs that aggregate Raw -> Medium.

### 2. High Cardinality Problem
*   `cpu_usage{host="server-1", app="web"}`.
*   If `host` has 1 million unique values (e.g., ephemeral containers), the index explodes.
*   TSDBs struggle with High Cardinality.

---

## 5. Trade-Off Analysis

| DB | Pros | Cons |
| :--- | :--- | :--- |
| **InfluxDB** | Specialized, Fast | Custom Query Language (Flux) |
| **TimescaleDB** | It's just Postgres (SQL) | Heavy (Row based) |
| **Prometheus** | Great for Ops | Not for business data |

---

## 6. Scaling Considerations

### Horizontal Scaling
*   Sharding by "Metric Name" or "Tag".
*   InfluxDB Clustered is paid (Enterprise). Single node is free.
*   **VictoriaMetrics** / **Thanos**: Scalable opensource alternatives.

---

## 7. Real Production Lessons

### "The IoT Flood"
*   10,000 Sensors sending data every second.
*   **Pattern**: Device -> MQTT -> Kafka -> Telegraf -> InfluxDB.
*   Kafka handles the burst/backpressure.

---

## 8. Summary & Architect Takeaways

1.  **Compression**: TSDBs are all about disk efficiency.
2.  **Retention**: Define retention policies early. Don't store raw data forever.
3.  **SQL vs NoSQL**: TimescaleDB is bridging the gap, allowing JOINs with business tables.

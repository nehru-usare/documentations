# Lambda vs Kappa Architecture

> **Part 7: High-Level Architecture**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** The Big Data Pipeline

---

## 0. Learning Objectives
*   **Beginner**: Difference between "Batch" (Nightly) and "Stream" (Real-time).
*   **Developer**: Using Kafka Streams or Spark.
*   **Architect**: Choosing between Complexity (Lambda) and Unproven purity (Kappa).

---

## 1. Problem Context
**Why does this exist?**
You need Analytics. "How many visitors today?"
*   **Batch (Hadoop)**: Accurate, comprehensive. Slow (Yesterday's data).
*   **Stream (Storm)**: Fast, real-time. Approximate, can lose data/buggy.
You want **Both**: Fast AND Accurate.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Batch Layer (Cold)
*   S3 / HDFS.
*   Process all data from beginning of time.
*   Reliable master dataset.

### 2. Speed Layer (Hot)
*   Kafka / Flink.
*   Process only recent data (last hour).
*   Compensates for the lag of the Batch Layer.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### 1. Lambda Architecture
*   **Logic**: Run computation `f()` twice.
*   Two Codebases:
    1.  **Batch View**: Hadoop MapReduce runs `f_batch()` nightly.
    2.  **Real-Time View**: Storm/Spark runs `f_speed()` continuously.
*   **Query**: Merge results. `Result = BatchView + SpeedView`.
*   *Con*: **Maintenance Nightmare**. You must write logic in Java (Hadoop) and Scala (Spark) and keep them in sync.

### 2. Kappa Architecture
*   **Logic**: Run computation `f()` once.
*   **Everything is a Stream**.
*   Batch processing is just streaming through "old" data stored in Kafka (Long retention).
*   *Pro*: One codebase.
*   *Con*: Requires very mature streaming platform.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Re-processing (Backfill)
*   **Scenario**: Bug in code. Calculation was wrong for last 30 days.
*   **Lambda**: Fix code. Re-run Batch job (Easy).
*   **Kappa**: Fix code. Spin up new Stream Consumer. Replay Kafka from 30 days ago. Swap output when caught up.

---

## 5. Trade-Off Analysis

| Feature | Lambda | Kappa |
| :--- | :--- | :--- |
| **Complexity** | High (2 Systems) | Medium (1 System) |
| **Accuracy** | High (Batch correction) | High (If log is durable) |
| **Latency** | Low | Low |
| **Cost** | High (Duplicate compute) | Medium |

---

## 6. Scaling Considerations

### The Data Lakehouse
*   Modern evolution.
*   Databricks / Snowflake.
*   Allows streaming ingestion into a transactional table format (Delta Lake / Iceberg).
*   Blurs the line between Batch and Stream.

---

## 7. Failure Scenarios & Recovery

### 1. Speed Layer Drift
*   In Lambda, if Speed layer logic diverges from Batch logic, user sees data "jump" when Batch overwrites Speed.
*   *Fix*: Shared libraries / Unified abstraction (Apache Beam).

---

## 8. Security Considerations

### 1. Immutable Logs
*   Both architectures rely on immutable raw data.
*   Ensuring ACLs on the Raw Data Lake (S3/Kafka) is critical. Use Ranger/Kerberos.

---

## 9. Performance Considerations

*   **Windowing**: Computing "Top 10 users in last 5 mins".
*   Stream processors utilize "Watermarks" to handle late-arriving data (Event time vs Processing time).

---

## 10. Real Production Lessons

### LinkedIn (Kappa Origin)
*   Jay Kreps (Creator of Kafka) invented Kappa.
*   Realized maintaining two pipelines was unsustainable.
*   Moved to "Stream Processing First".

---

## 11. Interview Questions

### Basic
1.  Difference between Batch and Stream processing.
2.  What is Lambda Architecture?
3.  What is Kappa Architecture?
4.  Why do we need a Speed Layer?
5.  What is a Data Lake?

### Intermediate
1.  Explain the downsides of Lambda Architecture.
2.  How does Kappa handle re-processing? (Replay).
3.  What is "Event Time" vs "Processing Time"?
4.  Role of Kafka in these architectures.
5.  What is Apache Beam?

### Advanced
1.  Design a real-time Ad-Fraud detection system using Kappa.
2.  Critique the storage cost of keeping infinite logs for Kappa. (Tiered Storage in Kafka).
3.  How does "Structured Streaming" in Spark unify batch and stream?
4.  Compare Flink vs Spark Streaming.
5.  Explain "Out of order" event handling.

### Architect-Level
1.  "We have a legacy Hadoop cluster. We need real-time." Architect the transition to Lambda, then Kappa.
2.  Design a Data Quality framework that validates the drift between Speed and Batch layers.
3.  Evaluate Delta Lake (Lakehouse) as a replacement for Lambda Architecture.

---

## 12. Scenario-Based System Design Problems

### 1. Design Trending Hashtags
*   **Req**: Top 10 hashtags in last 1 hour.
*   **Arch**: **Kappa**.
*   **Tool**: Flink sliding window.

### 2. Design Financial Reconciliation
*   **Req**: 100% accuracy. Daily reports.
*   **Arch**: **Batch** (or Lambda).
*   **Why**: Trust is more important than speed. Batch allows complex joins.

---

## 13. Summary & Architect Takeaways

1.  **Lambda is Legacy**: It was a bridge when Stream processing was immature.
2.  **Kappa is Future**: But requires strong engineering (Kafka mastery).
3.  **Lakehouse is Now**: Tools like Delta Lake offer the best of both worlds with SQL support.

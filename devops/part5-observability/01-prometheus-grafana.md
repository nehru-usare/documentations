# Prometheus & Grafana

> **Part 5: Observability**  
> **Difficulty:** ⭐⭐⭐ (Standard)  
> **Status:** Scraping metrics...

---

## 0. Learning Objectives
*   **Beginner**: Understanding "Time Series Data".
*   **Developer**: Exposing metrics in Java/Go (`/metrics`).
*   **Architect**: Designing a highly available monitoring stack with long-term storage (Thanos).

---

## 1. Context
**Observability**: Knowing *why* it broke, not just *that* it broke.
*   **Metrics**: "What is happening?" (CPU is 90%).
*   **Logs**: "Why?" (NullPointerException).
*   **Traces**: "Where?" (Service A called Service B).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Prometheus Architecture (Pull Model)
*   **App**: Exposes an HTTP endpoint `/metrics`.
*   **Prometheus**: Periodically (15s) scrapes that endpoint. Saves data.
*   **Grafana**: Queries Prometheus and draws pretty charts.
*   **AlertManager**: Checks rules. Sends Slack/PagerDuty notification.

### 2. Metric Types
1.  **Counter**: Only goes up (Requests, Errors). Rate().
2.  **Gauge**: Goes up and down (Memory, CPU).
3.  **Histogram**: Distribution (Latency buckets).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Exposing Metrics
*   **Spring Boot**: Add `actuator` + `micrometer-registry-prometheus`.
*   **Go**: Use `prometheus/client_golang` lib.
*   **Result**: Text format.
    ```text
    http_requests_total{method="post", handler="/api/comments"} 1024
    ```

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. PromQL (Query Language)
*   `rate(http_requests_total[5m])`: Rate of increase per second over last 5 mins.
*   `sum via (service) (...)`: Aggregation.
*   Power is in the **Labels** (Dimensions).

### 2. Service Discovery
*   How does Prometheus find 100 dynamic pods?
*   It talks to **Kubernetes API**.
*   "Give me all pods with annotation `prometheus.io/scrape: true`".

---

## 5. Trade-Off Analysis

| Feature | Pull (Prometheus) | Push (Datadog/Graphite) |
| :--- | :--- | :--- |
| **Discovery** | Centralized (Server finds Apps) | Decentralized (Apps find Server) |
| **Firewalls** | Hard (Server must reach App) | Easy (App pushes out) |
| **Scale** | Single Node bottlenecks | Distributed ingestion |

---

## 6. Scaling Considerations

### Long Term Storage (Thanos/Cortex)
*   Prometheus stores data on local disk (Retention ~15 days).
*   **Thanos**: Sidecar uploads metric blocks to Object Storage (S3).
*   Allows unlimited retention and global query view matches across clusters.

---

## 7. Failure Scenarios & Recovery

### 1. Prometheus Down
*   If scraper is down, you lose visibility (Blind flying).
*   **HA**: Run 2 Prometheus instances scraping the same targets. (Data duplication is acceptable here).

---

## 8. Security Considerations

### 1. Metric Exposure
*   `/metrics` endpoint might leak business info ("orders_processed_total").
*   Protect with Network Policy (Only Prometheus can reach it) or Basic Auth.

---

## 9. Performance Considerations

*   **Cardinality Explosion**:
    *   `http_requests_total{user_id="123"}` -> **BAD**.
    *   If you have 1 Million users, you create 1 Million time series. Prometheus RAM crashes.
    *   **Rule**: Never put unbounded values (User IDs, Emails, URLs) in Labels. Use bounded values (Status Code, Region, Service).

---

## 10. Real Production Lessons

### "Alert Fatigue"
*   If you alert on CPU > 80%, you will wake up every night.
*   **Philosophy**: Alert on **Symptoms** (Latency too high, Error rate too high), not **Causes** (CPU, RAM).
*   Let the detailed dashboard show the Cause.

---

## 11. Interview Questions

### Basic
1.  Push vs Pull monitoring.
2.  What is a Time Series DB?
3.  Difference between Counter and Gauge.

### Intermediate
1.  Write a PromQL to get 99th percentile latency. (`histogram_quantile`).
2.  What occurs if Prometheus runs out of disk space?
3.  Explain High Cardinality.

### Advanced
1.  Architect a Multi-Cluster Monitoring Solution (Federation vs Thanos).
2.  How to monitor "Batch Jobs" (PushGateway).
3.  Design an Alerting Strategy that minimizes noise.

---

## 12. Summary & Architect Takeaways

1.  **Labels are Power**: Structure your data with dimensions.
2.  **Dashboarding**: Every service needs a standard dashboard (USE Method: Utilization, Saturation, Errors).
3.  **Data Retention**: Don't use Prometheus for long-term storage. Use S3 (Thanos).

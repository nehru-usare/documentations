# 04. Observability (ELK, Prometheus, Grafana, OpenTelemetry)

> **Part 5: Infrastructure & DevOps**  
> **Difficulty:** ⭐⭐⭐⭐ (SRE)  
> **Status:** Mandatory for Microservices

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Stop using `ssh` to check logs. |
| **Developer** | Add Micrometer and OpenTelemetry to Spring Boot. |
| **Architect** | Define SLIs (Indicators) and SLOs (Objectives). |

---

## 1. Why This Topic Exists

### The Blindness Problem
In a Monolith, you `tail -f /var/log/app.log`.
In Microservices, you have 50 services * 3 replicas = 150 logs.
You cannot SSH into 150 containers.
**Observability** brings the data to you.

---

## 3. Core Concepts (The 3 Pillars)

### 1. Metrics (Is it healthy?)
*   **Data**: Time-series numbers.
*   **Examples**: CPU Usage, Memory, HTTP 500 Count, Latency p99.
*   **Tool**: **Prometheus** (Store), **Grafana** (Visualize).

### 2. Logs (What happened?)
*   **Data**: Text/JSON lines.
*   **Examples**: "NullPointerException at line 45".
*   **Tool**: **ELK Stack** (Elasticsearch, Logstash, Kibana) or **Loki**.

### 3. Traces (Where did it happen?)
*   **Data**: Spans showing a request jumping between services.
*   **Examples**: Gateway (5ms) -> Auth (20ms) -> Order (2000ms - SLOW).
*   **Tool**: **Jaeger**, **Zipkin**, **Tempo**.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Spring Boot Implementation

**1. Metrics (Micrometer)**
Default in Actuator. Exposes `/actuator/prometheus`.
Prometheus scrapes this endpoint every 15s.

**2. Tracing (OpenTelemetry/Zipkin)**
Automagically adds `traceId` and `spanId` to Logs and Http Headers.
*   *Log*: `INFO [OrderService, 1234, 5678] : Order created.`
*   *1234* = TraceID (Global).
*   *5678* = SpanID (Local).

---

## 9. Architect-Level Best Practices

1.  **Correlation ID**: Ensure `TraceID` is in the Logs. This is the **Link** between Pillars.
    *   *Flow*: Alert (High Latency) -> Dashboard (Trace) -> TraceID -> Logs (Exception).
2.  **No PII**: Don't log Passwords or Credit Cards. Use automatic masking.
3.  **Sampling**: Don't trace 100% of requests (Too expensive). Trace 1% or 5%.

---

## 14. Summary & Architect Takeaways

*   **Visibility**: If you can't see it, you can't fix it.
*   **MTTR**: Observability reduces Mean Time To Recovery.

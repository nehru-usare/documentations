# OpenTelemetry (Distributed Tracing)

> **Part 5: Observability**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** Context Propagated

---

## 0. Learning Objectives
*   **Beginner**: Knowing *where* the request failed in a microservice chain.
*   **Developer**: Adding standard instrumentation to code.
*   **Architect**: Vendor-neutral observability strategy.

---

## 1. Context
**The Problem**: Service A calls B -> C -> D. D fails. A sees "500 Internal Error". Logs show nothing in A.
*   **Tracing**: Attaches a `TraceID` to the request that follows it across network calls.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Structure
*   **Trace**: The whole journey.
*   **Span**: A single unit of work (e.g., "SQL Query", "HTTP GET /api").
*   **Context Propagation**: Passing IDs (`trace-id`, `span-id`) in HTTP Headers (`w3c-trace-context`).

### 2. OpenTelemetry (OTel)
*   A merger of OpenTracing and OpenCensus.
*   **Standard**: Unified APIs for Metrics, Logs, and Traces.
*   **Vendor Neutral**: Instrument once, send to Jaeger/Zipkin/Datadog/NewRelic.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Instrumentation
1.  **Auto-Instrumentation (Agent)**:
    *   Java/Python agent attaches to process. Intercepts HTTP/JDBC calls.
    *   Zero code changes.
2.  **Manual Instrumentation (SDK)**:
    *   `tracer.startSpan("process_order")`.
    *   Add custom attributes (`customer_tier=gold`).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Sampling
*   Tracing *every* request is too expensive (CPU/Storage).
*   **Head-Based Sampling**: Decide at start (10% of requests).
*   **Tail-Based Sampling**: Keep only "Interesting" traces (Errors, Slow requests). *Harder to implement*.

### 2. The Collector
*   **OTel Collector**: A sidecar or central deployed service.
*   Receives data from Apps (OTLP format).
*   Process/Filter/Batch.
*   Exporters -> Jaeger (Traces), Prometheus (Metrics).

---

## 5. Trade-Off Analysis

| Strategy | Pros | Cons |
| :--- | :--- | :--- |
| **Zipkin/Jaeger** | Open Source, Simple | Maintenance overhead |
| **SaaS (Datadog/Honeycomb)**| Powerful Analysis | Expensive |
| **OpenTelemetry** | **Future Proof**, portable | Implementation complexity |

---

## 6. Scaling Considerations

### Collector scaling
*   The OTel Collector is stateless. Horizontal scale.
*   Use it to scrub credentials before sending to SaaS vendors.

---

## 7. Failure Scenarios & Recovery

### 1. Broken Context
*   If Service B (Legacy) drops the headers before calling C.
*   The Trace is "Broken". You see two disconnected traces.
*   **Fix**: Update middleware libraries in B to propagate W3C headers.

---

## 8. Security Considerations

### 1. Data Leakage
*   Don't include `Authorization` headers or `Payload` in Spans.
*   OTel processors can redact attributes.

---

## 9. Performance Considerations

*   **Overhead**:
    *   Agents inject code. Can add latency.
    *   **Async Sending**: Trace data is sent via UDP/gRPC asynchronously. Should not block the main request path.

---

## 10. Real Production Lessons

### "Needle in Haystack"
*   Tracing is the *only* tool that helps debug "One specific user gets 500 error".
*   Logs are too noisy. Metrics are aggregated. Traces are specific.

---

## 11. Interview Questions

### Basic
1.  What is Distributed Tracing?
2.  What is a Trace ID?
3.  Why OTel is better than vendor agents?

### Intermediate
1.  Explain Context Propagation.
2.  Head vs Tail Sampling.
3.  Role of OTel Collector.

### Advanced
1.  Architect a Tracing solution for a mix of Monolith and Microservices.
2.  How to trace generic Async Message Queues (Kafka)? (Inject headers in message metadata).
3.  Impact of Sampling rate on error visibility.

---

## 12. Summary & Architect Takeaways

1.  **Standardize**: Use OpenTelemetry. Do not lock into vendor agents (Datadog agent).
2.  **Propagate**: Ensure every service/queue propagates headers.
3.  **Sample Smart**: 1% sampling is usually enough for performance tuning. 100% sampling on Errors.

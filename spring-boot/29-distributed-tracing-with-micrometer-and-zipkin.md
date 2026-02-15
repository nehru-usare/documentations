# Chapter 29: Distributed Tracing with Micrometer and Zipkin

## 0. Learning Objectives

- **🟢 Beginner**: Understand the concept of "Distributed Tracing" and why request IDs are needed in microservices.
- **🟡 Professional**: Master the replacement of Spring Cloud Sleuth with **Micrometer Tracing**, understand "Spans" and "Traces," and learn how to send data to **Zipkin** or **Tempo**.
- **🔴 Architect**: Deep dive into the `Observation` API internals, understand the propagation headers (B3, W3C), and design a high-performance sampling strategy to minimize tracing overhead in high-throughput environments.

---

## 1. Why This Topic Exists

### Real-World Business Problem
In a Monolith, an error is easy to trace because it happens in one process. In **Microservices**, a single user click might go through:
`Gateway -> Auth Service -> Order Service -> Inventory Service -> Payment Service`.
If the "Payment Service" fails, how do you know which "Gateway" request caused it? Tracing creates a "Single Thread of Evidence" across the entire network.

### Technical Limitations Solved
- **Visibility Gap**: Tracing allows you to see the **Latency** of every individual hop in a distributed request.
- **Correlation**: Connects logs from 10 different servers into a single coherent timeline.

---

## 2. Big Picture Architecture View

Tracing in Spring Boot 3 is powered by **Micrometer Tracing** (The successor to Spring Cloud Sleuth).

### Interaction with Other Modules
- **Web MVC / WebFlux**: Automatically intercepts incoming/outgoing requests to add tracing headers.
- **OpenTelemetry**: The industry-standard protocol that Micrometer can export to.
- **Logging**: Injects the `traceId` and `spanId` into your `MDC` logs automatically.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Trace**: The entire journey of a request from start to finish.
- **Span**: A single unit of work (e.g., one database call, one REST API call).
- **Correlation ID**: A unique ID passed in every request header.

### Simple Explanation
Think of a **Postal Package**.
- The **Tracking Number** is the **Trace ID**. No matter how many trucks (Services) the package goes through, the number stays the same.
- Every time a driver scans the package at a new warehouse, a new **Span** is created. One span for the flight, one span for the truck, one span for the final delivery.

### Minimal Working Example
Adding the tracing dependencies:
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Propagating Headers
Micrometer uses standard headers to pass IDs between services:
- **`X-B3-TraceId`**: The legacy standard (Brave/Sleuth).
- **`traceparent`**: The new **W3C Standard**.
**Tip**: If you use `RestTemplate` or `WebClient` beans managed by Spring, Micrometer will automatically inject these headers for you!

### Viewing Traces in Zipkin
Zipkin is the UI for your traces. It shows a waterfall chart:
- `OrderService: 500ms`
  - `InventoryService: 200ms`
  - `PaymentService: 250ms` (The bottleneck!)

---

## 5. Internal Mechanics (🔴 Advanced Level)

### The Observation API
Spring Boot 3 introduced the `Observation` API to unify Metrics and Tracing.
- **One Instrument**: You "Observe" a piece of code once, and Spring automatically creates both a **Metric** (for Grafana) and a **Span** (for Zipkin).
- **ObservationRegistry**: The central hub that manages collectors and reporters.

### Sampling Strategy
Sending 100% of traces to Zipkin will kill your network and fill up your storage.
- **Probability Sampling**: Only trace 10% of requests (Default is 10%).
  `management.tracing.sampling.probability=0.1`
- **Architect Note**: In production, use **Rate Limiting** sampling (e.g., max 100 traces per second) to prevent tracer-induced application crashes.

---

## 6. Under the Hood

### Brave vs. OpenTelemetry (OTEL)
Micrometer acts as a bridge. You can choose the "Engine":
- **Brave**: The tried-and-true library from OpenZipkin.
- **OTEL**: The modern, vendor-neutral standard supported by Google, AWS, and Azure. 
**Decision**: Use OTEL if you are moving to a Cloud-Native / Multi-cloud environment.

---

## 7. Real-World Use Cases

- **Latency Debugging**: Finding out why a "Search" API is taking 2 seconds (Often it's a slow DB query hidden 3 layers deep).
- **Error Analysis**: Seeing which service in a chain of 10 was the first to throw an exception.

---

## 8. Production & Performance Considerations

- **Asynchronous Work**: Tracing spans are stored in a `ThreadLocal`. If you spawn a new thread, the trace stays behind.
  - **Fix**: Use `Observation.openScope()` to manually pass the context to your background threads.
- **Disk Space**: A high-traffic system generates millions of spans. Use **Elasticsearch** or **Cassandra** as the backend for Zipkin; never use the default in-memory storage for production.

---

## 9. Architect-Level Best Practices

- **Tagging**: Add custom tags to your spans like `tenant_id` or `order_value`. This allows you to filter traces in Zipkin for specific business use cases.
- **Trace Logs link**: Always ensure your `spanId` is in your logs. Modern Log Aggregators (like Loki or Datadog) can click-through from a log line directly to the corresponding waterfall trace.
- **Standardize Formats**: Ensure all services in your mesh use the same header format (e.g., both use W3C) otherwise the trace will "Break" halfway through.

---

## 10. Common Mistakes & Anti-Patterns

- **100% Sampling**: Trying to trace every single request in a high-traffic app. This can add 5ms-10ms of latency and massive cost.
- **Manual Span creation**: Creating thousands of small spans for every `if` statement. Spans should represent significant "Operations," not every line of code.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`logging.level.io.micrometer.tracing=DEBUG`**: Shows you which headers were received and which were sent.
- **Missing Traces**: Usually means you created your own `RestTemplate` using `new RestTemplate()` instead of the **`RestTemplateBuilder`**. (Manual creation bypasses Spring's instrumentation).

---

## 12. Comparisons

### Metrics vs. Tracing 
| Feature | Metrics (Prometheus) | Tracing (Zipkin) |
| :--- | :--- | :--- |
| **Goal** | Aggregated health (AVG) | Individual request details |
| **Data Size** | Small (Numeric) | Large (JSON/Waterfall) |
| **Retention** | Long-term | Short-term (Days) |
| **Recommendation** | **Alerting & Trends** | **Debugging Latency/Errors** |

---

## 13. Interview Questions

### 🟢 Basic
1. What is a Trace ID?
2. Why is tracing important in Microservices?

### 🟡 Intermediate
1. What is the difference between a Trace and a Span?
2. How do you pass the Trace ID between two different Spring Boot services?

### 🔴 Advanced
1. What is the Spring `Observation` API and how does it relate to Micrometer Tracing?
2. Explain the difference between B3 and W3C propagation headers.

### 🔥 Tricky
1. Does distributed tracing slow down my application? (Yes, slightly. The overhead of creating spans and sending them to a reporter is non-zero. That's why we use sampling).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You have a "Gateway" that calls a "User Service" which calls a "Legacy COBOL API." The COBOL API doesn't support tracing headers. How do you maintain the trace? (The "User Service" should create a "Local Span" for the COBOL call. The waterfall will show the call was made, and the latency will be recorded, even if the COBOL system doesn't know it's part of a trace).
2. **Performance**: Your Zipkin server is overwhelmed with data. What are the three steps you take to fix this? (1. Lower the sampling probability. 2. Implement Rate-Limited sampling. 3. Use an asynchronous/buffered reporter).

---

## 15. Summary & Key Takeaways

- **Core Insight**: Tracing is the **Map** of your Microservice world.
- **Architect Mindset**: Instrument your interfaces, not your logic. Use standard headers to ensure your map works across different languages and clouds.
- **Production Reminder**: Sampling is your volume knob. Start low and only turn it up when you are debugging a specific issue.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 6: Chapter 29**

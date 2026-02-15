# Chapter 30: Monitoring, Metrics, and Grafana Integration

## 0. Learning Objectives

- **🟢 Beginner**: Understand the difference between a "Log" and a "Metric," and the basics of Prometheus scraping.
- **🟡 Professional**: Master **Micrometer** core concepts (Meters, Timers, Gauges, Counters) and learn how to expose custom business metrics using the `MeterRegistry`.
- **🔴 Architect**: Deep dive into the `Micrometer` registry internals, understand the impact of **Metric Cardinality**, and design a high-scale observability dashboard in **Grafana** using the "RED" and "USE" signals.

---

## 1. Why This Topic Exists

### Real-World Business Problem
Logs tell you *exactly* what happened to one user (The "Why"). Metrics tell you the *overall health* of your entire system (The "What"). You don't want to read a log file to find out that your CPU is at 99%. You want a **Dashboard** that alerts you before the server crashes.

### Technical Limitations Solved
- **Efficiency**: Storing 1 billion log lines is expensive. Storing a single "Number" (the CPU percentage) every 10 seconds is incredibly cheap and efficient.
- **Trend Detection**: Metrics allow you to see that your "Memory Usage" is slowly growing over 3 days, helping you find a memory leak *before* it becomes a production incident.

---

## 2. Big Picture Architecture View

Monitoring in Spring Boot is powered by **Micrometer** (The "SLF4J of Metrics").

### Interaction with Other Modules
- **Spring Boot Actuator**: Provides the `/actuator/prometheus` endpoint which Prometheus "Scrapes" (pulls data from).
- **Grafana**: The visualization engine that connects to Prometheus to show charts.
- **Core Engine**: Micrometer automatically instruments JVM, Tomcat, and HikariCP beans.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Counter**: A value that only goes UP (e.g., total requests received).
- **Gauge**: A value that goes up and down (e.g., current number of logged-in users).
- **Timer**: Measures how long an event takes (e.g., response time of an API).

### Simple Explanation
Think of a **Smart Watch**.
- **Counter**: Total steps taken today.
- **Gauge**: Your current Heart Rate.
- **Timer**: How long did your 5km run take?
The watch collects this data (Micrometer), sends it to the Cloud (Prometheus), and you view it on the phone app in a pretty chart (Grafana).

### Minimal Working Example
Integrating Prometheus support:
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
Exposing the endpoint:
```properties
management.endpoints.web.exposure.include=prometheus
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Creating Custom Metrics
Use the `MeterRegistry` bean:
```java
@Service
public class OrderService {
    private final Counter orderCounter;

    public OrderService(MeterRegistry registry) {
      this.orderCounter = registry.counter("orders.placed", "type", "retail");
    }

    public void placeOrder() {
        // Logic...
        orderCounter.increment();
    }
}
```

### High-Precision Timers
Timers collect count, sum, and max. By default, they also collect **Quantiles** (p95, p99) which are critical for understanding "Tail Latency" (the experience of your unluckiest users).

---

## 5. Internal Mechanics (🔴 Advanced Level)

### The `MeterRegistry` Internals
When you call `increment()`, Micrometer doesn't send the data anywhere.
1. **In-Memory Store**: It updates a volatile variable in RAM.
2. **Polling (Scraping)**: When Prometheus calls the `/actuator/prometheus` endpoint, Micrometer iterates over all "Meters," formats they into the Prometheus text protocol, and sends them in one go.

### Metric Cardinality (The Architect's Trap)
A metric "Tag" creates a new time series.
- **Good Tag**: `status=200` (Only 5-10 possible values).
- **Bad Tag**: `user_id=12345` (Millions of possible values).
**Result**: High cardinality will make your Prometheus server run out of memory and your Actuator endpoint take 10 seconds to respond. **Never use high-cardinality data as a tag.**

---

## 6. Under the Hood

### Percentile Histograms
Spring Boot 3 allows capturing histograms for latency:
```properties
management.metrics.distribution.percentiles-histogram.http.server.requests=true
```
This allows Grafana to calculate accurate percentiles across multiple server nodes.

---

## 7. Real-World Use Cases

- **Revenue Monitoring**: A gauge showing the total dollar value of orders processed in the last 1 minute.
- **Circuit Breaker Status**: A metric showing if your "Payment Gateway" circuit is OPEN or CLOSED.

---

## 8. Production & Performance Considerations

- **Scrape Interval**: Don't scrape every 1 second. 10s-30s is the industry standard.
- **Step Size**: For push-based registries (like CloudWatch or Datadog), ensure your "Step" matches your recording interval to avoid "Sawtooth" charts.

---

## 9. Architect-Level Best Practices

- **The RED Method**: (For APIs)
  - **R**ate: Number of requests per second.
  - **E**rrors: Number of failed requests.
  - **D**uration: Time taken per request.
- **The USE Method**: (For Infrastructure)
  - **U**tilization: Average time the resource was busy.
  - **S**aturation: Degree to which the resource has extra work it can't handle (Queue depth).
  - **E**rrors: Count of error events.

---

## 10. Common Mistakes & Anti-Patterns

- **Not Naming Metrics Correctly**: Using `my_metric` instead of `component.action.result`.
- **Calculating Averages only**: An "Average" of 500ms could mean 99 people had 10ms and 1 person had 50 seconds. **Always use Percentiles (p95/p99) to see real pain.**

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **Endpoint 404**: Check if `micrometer-registry-prometheus` is in the pom.
- **Missing Custom Metric**: Ensure you are using the same `MeterRegistry` instance and that your bean is correctly initialized.

---

## 12. Comparisons

### Counter vs. Gauge
| Feature | Counter | Gauge |
| :--- | :--- | :--- |
| **Logic** | Add-only (`n++`) | Up/Down (`n = x`) |
| **Reset** | Resets on app restart | Reflects current state |
| **Usage** | Count of events | Snapshot of status |
| **Recommendation** | **Requests, Errors** | **Thread counts, Memory** |

---

## 13. Interview Questions

### 🟢 Basic
1. What is the difference between Prometheus and Grafana?
2. What is a "Timer" used for?

### 🟡 Intermediate
1. Why is "Metric Cardinality" important?
2. What are "Tags" in Micrometer?

### 🔴 Advanced
1. Describe the difference between a Histogram and a Summary in Prometheus.
2. How do you implement a custom `MeterBinder` to instrument a legacy library?

### 🔥 Tricky
1. If you have 5 instances of your app, and you see 1,000 total requests in Grafana, does Prometheus add them up? (Yes, if your PromQL query uses `sum(rate(...))`).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You have a background job that processes Excel files. You want to see "How many rows are processed per second." Which meter type do you use? (Use a **Timer** or a **FunctionCounter**).
2. **Performance**: Your Grafana dashboard is loading very slowly. You realize you have a metric `product_view_count` with a tag `product_id`. What is the fix? (Remove the `product_id` tag as it is too high-cardinality. Use a log aggregator like ELK if you need to count specific product views).

---

## 15. Summary & Key Takeaways

- **Core Insight**: **You can't manage what you don't measure.**
- **Architect Mindset**: Dashboards are for humans; Alerts are for machines. Design your metrics to trigger automated alerts when p99 exceeds acceptable thresholds.
- **Production Reminder**: Keep your charts simple. A Grafana dashboard with 50 complex charts is impossible to read during a high-pressure production outage.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 6: Chapter 30**

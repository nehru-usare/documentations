# 🧰 Java 21 — Observability and Performance Toolkit

> “What you don’t measure, you can’t improve.” — *Peter Drucker*

Modern Java applications run in dynamic cloud environments where performance visibility is critical.  
This guide provides a **comprehensive toolkit** for observing, profiling, and optimizing **Java 21** systems — from JVM internals to distributed tracing.

---

## 🧭 Table of Contents

1. [Observability Overview](#1-observability-overview)
2. [JFR (Java Flight Recorder)](#2-jfr-java-flight-recorder)
3. [JMC (Java Mission Control)](#3-jmc-java-mission-control)
4. [Async Profiler](#4-async-profiler)
5. [Micrometer Metrics Integration](#5-micrometer-metrics-integration)
6. [Prometheus and Grafana Setup](#6-prometheus-and-grafana-setup)
7. [OpenTelemetry Tracing](#7-opentelemetry-tracing)
8. [Distributed Logging and Correlation IDs](#8-distributed-logging-and-correlation-ids)
9. [Performance Benchmarking Tools](#9-performance-benchmarking-tools)
10. [Profiling JVM in Containers](#10-profiling-jvm-in-containers)
11. [Common JVM Bottlenecks and Fixes](#11-common-jvm-bottlenecks-and-fixes)
12. [Official Resources and References](#12-official-resources-and-references)
13. [Summary](#13-summary)

---

## 1️⃣ Observability Overview

Observability in Java means **collecting and correlating three key pillars**:

| Type | Tools | Purpose |
|------|--------|----------|
| **Metrics** | Micrometer, Prometheus, Grafana | Numeric KPIs (heap, latency, throughput) |
| **Logs** | Logback, Loki, ELK | Structured event history |
| **Traces** | OpenTelemetry, Jaeger | Request flow visualization |

✅ Good observability = clear insight into system behavior without changing code.  
✅ JVM-level observability adds another dimension — **GC, JIT, memory, and threads**.

---

## 2️⃣ JFR (Java Flight Recorder)

**Java Flight Recorder (JFR)** is a low-overhead profiler built into the JDK.

### 🔹 Start JFR Recording:
```bash
java -XX:StartFlightRecording=name=app,filename=app.jfr,duration=60s,settings=profile MyApp
````

### 🔹 Continuous Recording:

```bash
java -XX:StartFlightRecording=filename=recording.jfr,maxage=1h,maxsize=250M,dumponexit=true MyApp
```

### 🔹 Analyze Results:

Open `app.jfr` in **Java Mission Control**.

### JFR Captures:

* GC events
* Memory allocations
* Thread states
* Locks and synchronization
* CPU samples
* Class loading metrics

✅ Safe for production — overhead < 1–2%.
✅ Perfect for capturing intermittent spikes or leaks.

---

## 3️⃣ JMC (Java Mission Control)

**Java Mission Control (JMC)** is the official GUI for analyzing JFR recordings.

### 🔹 Installation:

```bash
sudo apt install jmc
```

### 🔹 Key Tabs:

* **Memory** → Heap usage, GC, allocation hotspots
* **Threads** → Contention, deadlocks
* **I/O** → File and socket metrics
* **Code** → Method-level hot paths
* **Events** → GC pauses, safepoints, exceptions

> 💡 Combine JMC + JFR for continuous application performance insight.

---

## 4️⃣ Async Profiler

**Async Profiler** is a low-level native sampling profiler for JVMs.

### 🔹 Download:

[https://github.com/jvm-profiling-tools/async-profiler](https://github.com/jvm-profiling-tools/async-profiler)

### 🔹 Run Profiling:

```bash
./profiler.sh -d 30 -e cpu -f profile.html <pid>
```

### 🔹 Generate Flame Graph:

```bash
./profiler.sh -d 30 -e cpu -o flamegraph -f flame.html <pid>
```

✅ Captures CPU and allocation samples.
✅ Works even inside containers.
✅ Produces Flame Graphs for visualization.

> 🧠 Flame Graphs reveal *where* your app spends time — ideal for micro-optimizations.

---

## 5️⃣ Micrometer Metrics Integration

**Micrometer** is the de facto metrics facade for JVM and Spring Boot.

### 🔹 Maven Dependency:

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### 🔹 Basic JVM Metrics:

```java
MeterRegistry registry = new SimpleMeterRegistry();
new ClassLoaderMetrics().bindTo(registry);
new JvmMemoryMetrics().bindTo(registry);
new ProcessorMetrics().bindTo(registry);
```

### 🔹 Spring Boot Integration:

Enable `/actuator/prometheus` endpoint:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "prometheus"
```

✅ Micrometer auto-collects:

* Heap & non-heap usage
* GC time
* CPU load
* Thread states

---

## 6️⃣ Prometheus and Grafana Setup

### 🔹 Prometheus Config:

```yaml
scrape_configs:
  - job_name: 'java-service'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080']
```

### 🔹 Grafana Dashboard:

Import **JVM Micrometer Dashboard ID 4701**
[https://grafana.com/grafana/dashboards/4701](https://grafana.com/grafana/dashboards/4701)

✅ Visualize:

* GC pause times
* Heap allocation
* Active threads
* Response latency

---

## 7️⃣ OpenTelemetry Tracing

**OpenTelemetry (OTel)** provides distributed tracing and metric correlation across microservices.

### 🔹 Enable Tracing Agent:

```bash
java -javaagent:opentelemetry-javaagent.jar \
     -Dotel.service.name=order-service \
     -Dotel.exporter.otlp.endpoint=http://otel-collector:4317 \
     -jar app.jar
```

### 🔹 Spring Boot Configuration:

```yaml
management:
  tracing:
    sampling:
      probability: 1.0
```

✅ Compatible with:

* **Jaeger**
* **Grafana Tempo**
* **AWS X-Ray**
* **Elastic APM**

✅ Automatically tracks:

* HTTP requests
* Database calls
* Messaging events

> 🧠 Add traceId + spanId in logs for full request reconstruction.

---

## 8️⃣ Distributed Logging and Correlation IDs

Use structured JSON logs with **trace correlation**.

### Example Log:

```json
{
  "timestamp": "2025-10-26T10:05:00Z",
  "traceId": "abc123",
  "spanId": "def456",
  "service": "order-service",
  "level": "INFO",
  "message": "Order processed successfully"
}
```

✅ Integrate with:

* **Loki** for logs
* **Grafana** for visualization
* **OpenTelemetry** for linking traces

> 💡 Traces + Logs + Metrics = Complete observability triad.

---

## 9️⃣ Performance Benchmarking Tools

| Tool               | Description                    | Usage                      |
| ------------------ | ------------------------------ | -------------------------- |
| **JMH**            | Micro-benchmarking framework   | Fine-grained method timing |
| **wrk / Gatling**  | Load testing                   | REST API performance       |
| **JMeter**         | End-to-end performance testing | Stress/load tests          |
| **Async Profiler** | Low-level CPU/memory analysis  | Flame graphs               |
| **JFR**            | JVM event profiling            | GC, JIT, threads           |

### 🔹 Example: JMH Benchmark

```java
@Benchmark
public void testLoop() {
    for (int i = 0; i < 1000; i++);
}
```

Run with:

```bash
java -jar benchmarks.jar
```

---

## 🔟 Profiling JVM in Containers

### 🔹 Connect via PID:

```bash
docker exec -it myapp jcmd 1 VM.native_memory summary
```

### 🔹 Copy JFR File:

```bash
docker cp myapp:/app/app.jfr .
```

### 🔹 Run async-profiler inside container:

```bash
docker exec -it myapp ./profiler.sh -d 30 -e cpu -f /tmp/profile.html 1
```

✅ Works for Kubernetes pods too (`kubectl exec`).
✅ Combine with Grafana dashboards for runtime insight.

---

## 11️⃣ Common JVM Bottlenecks and Fixes

| Issue                   | Cause                      | Fix                            |
| ----------------------- | -------------------------- | ------------------------------ |
| **High GC pauses**      | Excessive allocation       | Tune G1GC, reduce object churn |
| **CPU saturation**      | Thread contention          | Use Virtual Threads            |
| **Slow startup**        | Class loading, JIT warm-up | Use GraalVM Native Image       |
| **Memory leaks**        | Unreleased collections     | Use JFR heap dump analysis     |
| **Deadlocks**           | Synchronized blocks        | Monitor with `jstack`          |
| **Blocked I/O threads** | Blocking calls             | Adopt async or virtual threads |

---

## 12️⃣ Official Resources and References

📘 **JVM Observability**

* [Java Flight Recorder Docs](https://docs.oracle.com/en/java/javase/21/jfapi/)
* [Java Mission Control](https://www.oracle.com/java/technologies/jdk-mission-control.html)
* [Async Profiler](https://github.com/jvm-profiling-tools/async-profiler)
* [JMH (Java Microbenchmark Harness)](https://openjdk.org/projects/code-tools/jmh/)
* [VisualVM](https://visualvm.github.io/)

📊 **Metrics & Tracing**

* [Micrometer Docs](https://micrometer.io/)
* [Prometheus](https://prometheus.io/)
* [Grafana Dashboards](https://grafana.com/grafana/dashboards/)
* [OpenTelemetry Java Instrumentation](https://opentelemetry.io/docs/instrumentation/java/)

💻 **Real-World Use**

* [Spring Boot Observability Docs](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
* [Red Hat OpenJDK Observability Guide](https://developers.redhat.com/)
* [GraalVM Performance Docs](https://www.graalvm.org/reference-manual/performance/)

---

## 13️⃣ Summary

You now have a **complete toolkit** for observing and tuning Java 21 systems:

✅ Collect JVM metrics with **Micrometer + Prometheus**
✅ Profile runtime behavior with **JFR + JMC + Async Profiler**
✅ Enable **OpenTelemetry tracing** for distributed requests
✅ Implement structured, correlated **JSON logging**
✅ Benchmark and validate performance before deployment

> 🧠 **Observability transforms guesswork into engineering.**
> The most reliable Java systems are the ones that can *explain themselves* in production.

> 🧭 **Next (optional advanced):**
> [24-java-performance-incident-response.md → Troubleshooting and Incident Response Playbook for JVM in Production](./24-java-performance-incident-response.md)

# 🚨 Java 21 — Performance Incident Response and Troubleshooting Playbook

> “The best engineers aren’t those who never cause incidents —  
> they’re the ones who can fix them fast.” — *Production proverb*

This playbook provides **step-by-step diagnostic workflows** and **tools** to handle **Java 21 production performance incidents**, including memory leaks, GC storms, CPU thrashing, and deadlocks.

It’s designed for **senior developers, DevOps engineers, and SREs** who operate JVM-based systems at scale.

---

## 🧭 Table of Contents

1. [Incident Response Workflow](#1-incident-response-workflow)
2. [Collecting Initial Evidence](#2-collecting-initial-evidence)
3. [High CPU Usage](#3-high-cpu-usage)
4. [Memory Leaks and OutOfMemoryError](#4-memory-leaks-and-outofmemoryerror)
5. [Garbage Collection (GC) Storms](#5-garbage-collection-gc-storms)
6. [Thread Leaks and Deadlocks](#6-thread-leaks-and-deadlocks)
7. [I/O and Network Latency Issues](#7-io-and-network-latency-issues)
8. [Slow Startup or Throughput Degradation](#8-slow-startup-or-throughput-degradation)
9. [Container-Specific Incidents (Kubernetes)](#9-container-specific-incidents-kubernetes)
10. [Log Analysis and Observability Correlation](#10-log-analysis-and-observability-correlation)
11. [Postmortem and Root Cause Analysis](#11-postmortem-and-root-cause-analysis)
12. [Official Resources and Tools](#12-official-resources-and-tools)
13. [Summary](#13-summary)

---

## 1️⃣ Incident Response Workflow

### 🚨 Step 1: Detect
- Use **Prometheus / Grafana / OpenTelemetry** alerts  
- Identify anomalies: memory, CPU, latency, error rates

### 🧩 Step 2: Capture
Collect:
- Thread dumps
- Heap dumps
- GC logs
- System metrics

### 🧠 Step 3: Analyze
- Identify patterns: leaks, deadlocks, hotspots  
- Use **JFR / JMC / async-profiler**

### 🧰 Step 4: Mitigate
- Apply temporary scaling or restarts  
- Enable rate limiting, fallbacks, or circuit breakers

### 🧾 Step 5: Document
- Root cause  
- Fix  
- Preventive measure  

---

## 2️⃣ Collecting Initial Evidence

### 🔹 List Running JVMs:
```bash
jps -l
````

### 🔹 Get System Overview:

```bash
top -H -p <pid>
```

### 🔹 Dump Threads:

```bash
jstack -l <pid> > threads.log
```

### 🔹 Dump Heap:

```bash
jmap -dump:live,format=b,file=heap.bin <pid>
```

### 🔹 GC Summary:

```bash
jstat -gcutil <pid> 1s 10
```

✅ Store all dumps with timestamps.
✅ Avoid collecting dumps too frequently — can pause app.

---

## 3️⃣ High CPU Usage

### 🔍 Symptom:

* CPU consistently > 80%
* Latency and GC time increase

### 🔧 Diagnosis Steps:

1. Identify top threads:

   ```bash
   top -H -p <pid>
   ```
2. Find thread ID in hex:

   ```bash
   printf "%x\n" <tid>
   ```
3. Match with stack trace:

   ```bash
   jstack <pid> | grep -A50 <hex_tid>
   ```

### 🔬 Common Causes:

* Infinite loops
* High GC overhead
* Tight polling (busy-waiting)
* Excessive logging or serialization

### 🩹 Mitigation:

* Throttle problematic tasks
* Use backpressure
* Enable **Virtual Threads** for blocking I/O

> 🧠 Use `async-profiler` for fine-grained CPU analysis.

---

## 4️⃣ Memory Leaks and OutOfMemoryError

### 🔍 Symptoms:

* Gradual memory growth
* Frequent GC
* `java.lang.OutOfMemoryError: Java heap space`

### 🔧 Diagnosis:

```bash
jmap -histo <pid> | head -20
```

→ Shows top object counts.

### Dump full heap:

```bash
jmap -dump:format=b,file=heap.bin <pid>
```

Analyze with:

* **Eclipse MAT**
* **VisualVM**
* **JMC (Heap Analyzer)**

### 🔬 Common Leak Sources:

* Unbounded collections (`Map`, `List`, caches)
* ThreadLocal misuse
* Static references
* Streams not closed
* Listeners not deregistered

### 🩹 Fix:

* Use **weak references** for caches
* Close streams with try-with-resources
* Monitor GC logs for retained objects

---

## 5️⃣ Garbage Collection (GC) Storms

### 🔍 Symptoms:

* GC pauses > 500ms
* CPU spikes during GC
* Throughput drops

### 🔧 Diagnosis:

Enable GC logging:

```bash
-Xlog:gc*:file=gc.log:time,level,tags
```

Analyze with:

* **GCViewer** or **GCEasy.io**

### 🔬 Common Causes:

* Excessive object allocation
* Memory leaks
* Poor heap sizing
* Misconfigured GC

### 🩹 Fix:

* Use **G1GC** or **ZGC**
* Set heap properly:

  ```bash
  -Xms512m -Xmx512m
  ```
* Reduce temporary object creation
* Use object pools only where necessary

---

## 6️⃣ Thread Leaks and Deadlocks

### 🔍 Symptoms:

* Application hangs
* Increased thread count
* `BLOCKED` threads in dump

### 🔧 Check Active Threads:

```bash
jcmd <pid> Thread.print
```

### Detect Deadlocks:

```bash
jstack <pid> | grep "Found one Java-level deadlock"
```

### 🔬 Common Causes:

* Unreleased locks
* Misuse of synchronized methods
* Executor pools without shutdown
* Blocking network calls

### 🩹 Fix:

* Use **try-lock** or **CompletableFuture**
* Virtual threads for non-blocking I/O
* Thread pooling hygiene

> 💡 Use JFR “Thread Dump” event to capture deadlocks automatically.

---

## 7️⃣ I/O and Network Latency Issues

### 🔍 Symptoms:

* Slow REST calls
* Socket timeouts
* Thread pool exhaustion

### 🔧 Diagnosis:

```bash
netstat -anp | grep <pid>
```

Check blocked I/O threads:

```bash
jstack <pid> | grep "WAITING on java.net"
```

### 🩹 Fix:

* Use async I/O (Reactor, Loom Virtual Threads)
* Configure timeouts (`connectTimeout`, `readTimeout`)
* Circuit breakers (Resilience4j)

---

## 8️⃣ Slow Startup or Throughput Degradation

### 🔍 Causes:

* JIT warm-up delay
* Class loading overhead
* Reflection-heavy frameworks

### 🔧 Fix:

* Use **AppCDS**:

  ```bash
  java -Xshare:dump
  ```
* Precompile with **GraalVM Native Image**
* Cache reflection configuration
* Reduce auto-scanning packages in Spring

> 💡 For microservices, use GraalVM or CDS for <100ms startup.

---

## 9️⃣ Container-Specific Incidents (Kubernetes)

### 🔍 Problems:

* OOMKilled pods
* CPU throttling
* Unstable autoscaling

### 🔧 Diagnosis:

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

Check resource limits:

```yaml
resources:
  limits:
    memory: "1Gi"
    cpu: "500m"
```

### 🩹 Fix:

* Use percentage heap sizing (`MaxRAMPercentage`)
* Avoid full heap GC by tuning `-XX:InitiatingHeapOccupancyPercent`
* Enable `ExitOnOutOfMemoryError`

---

## 🔟 Log Analysis and Observability Correlation

### Correlate Logs + Metrics + Traces:

Use **traceId** from OpenTelemetry to connect all signals.

Example workflow:

1. Filter logs by traceId
2. Check corresponding JFR timeline
3. Verify latency spikes in Grafana
4. Identify GC or thread contention correlation

✅ Combine structured logs, metrics, and traces for 360° diagnosis.

---

## 11️⃣ Postmortem and Root Cause Analysis

After resolution:

1. **Summarize the incident** — when, what, how, impact
2. **Analyze root cause** — not just symptom
3. **Define preventive measures**
4. **Add monitoring dashboards** for recurrence
5. **Automate alerting thresholds**

> 🧾 Every incident should make your system stronger.

---

## 12️⃣ Official Resources and Tools

📘 **Profiling & Debugging**

* [Java Flight Recorder](https://docs.oracle.com/en/java/javase/21/jfapi/)
* [Java Mission Control](https://www.oracle.com/java/technologies/jdk-mission-control.html)
* [Async Profiler](https://github.com/jvm-profiling-tools/async-profiler)
* [Eclipse MAT (Heap Analyzer)](https://www.eclipse.org/mat/)
* [VisualVM](https://visualvm.github.io/)

📊 **Cloud & Observability**

* [Micrometer Metrics](https://micrometer.io/)
* [Prometheus + Grafana](https://grafana.com/grafana/dashboards/)
* [OpenTelemetry Java](https://opentelemetry.io/docs/instrumentation/java/)
* [Resilience4j](https://resilience4j.readme.io/)

---

## 13️⃣ Summary

You’ve learned how to:

* Handle JVM performance incidents methodically
* Capture and analyze key evidence (heap, threads, GC, CPU)
* Use JFR, JMC, async-profiler, and Prometheus for diagnostics
* Resolve leaks, deadlocks, and container-specific issues
* Document root causes and automate prevention

> 🧠 **Performance incidents are opportunities to make your system smarter, not just stable.**

> 🧭 **Next (Optional Advanced):**
> [25-java-production-hardening.md → Production Hardening, Reliability, and Chaos Engineering in Java 21](./25-java-production-hardening.md)
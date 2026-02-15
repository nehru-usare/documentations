# 05. Performance Testing (Gatling, JMeter, Profiling)

> **Part 7: Performance & Scalability**  
> **Difficulty:** ⭐⭐⭐⭐ (QA / Architect)  
> **Status:** Verification

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Run a simple Load Test with JMeter. |
| **Developer** | Analyze a Heap Dump to find Memory Leaks. |
| **Architect** | Define Performance SLAs (P99 < 100ms). |

---

## 1. Why This Topic Exists

### The Production Surprise
"It worked on my laptop."
But your laptop processes 1 request.
Production processes 10,000.
**Performance Testing** simulates Production *before* you go live.

---

## 3. Core Concepts (🟢 Beginner Level)

### Types of Tests
1.  **Load Test**: Normal expected load (1000 users). Goal: Stability.
2.  **Stress Test**: Break point. (100k users). Goal: Find where it crashes.
3.  **Soak Test**: Run 80% load for 24 hours. Goal: Find Memory Leaks.

### Latency Percentiles (P99)
*   **Average**: 200ms. (Misleading).
    *   1 request = 1s.
    *   9 requests = 100ms.
    *   Avg = 190ms.
*   **P99**: 1s. (This means 1% of users are having a terrible time).
*   **Architect Rule**: Optimize for P99, not Average.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Profiling (VisualVM / IntelliJ)
When CPU is 100%, what is line of code responsible?
1.  **CPU Profiler**: Shows "Hot Methods".
2.  **Memory Profiler**: Shows "Who holds the objects".
    *   *Leak*: If `byte[]` keeps growing and GC never reclaims it.

### Gatling (Code-based Load Testing)
Scala-based DSL. Very high performance.

```scala
val scn = scenario("BasicSimulation")
  .exec(http("request_1")
  .get("/"))

setUp(
  scn.inject(atOnceUsers(1000))
).protocols(httpConf)
```

---

## 14. Summary & Architect Takeaways

*   **Don't Guess**: "I think the DB is slow". No. **Measure** it.
*   **Shift Left**: Performance test in CI, not just before release.
*   **Flame Graphs**: Learn to read them. They show exactly where the CPU burns.

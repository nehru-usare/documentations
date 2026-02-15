# 05. Debugging Common Microservices Errors (Network, Serialization)

> **Part 9: Debugging & Troubleshooting**  
> **Difficulty:** ⭐⭐⭐⭐ (Senior Dev)  
> **Status:** The Daily Grind

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Know the difference between `ConnectException` and `SocketTimeoutException`. |
| **Developer** | Fix Jackson `UnrecognizedPropertyException` without `@IgnoreUnknown`. |
| **Architect** | Debug DNS resolution issues in Kubernetes (`ndots:5`). |

---

## 1. Why This Topic Exists

### The Error Message
You see a stacktrace. Ideally, you know exactly what to do.
*   **Reality**: Most stacktraces are misleading.
*   **Goal**: Translate "Java Exception" into "Infrastructure Reality".

---

## 3. Core Concepts (🟢 Beginner Level)

### Connection Refused vs Timeout
1.  **java.net.ConnectException: Connection refused**
    *   *Meaning*: I knocked on the door, and no one answered.
    *   *Cause*: The Service is **DOWN** or the Port is wrong.
    *   *Fix*: Check if the process is running.

2.  **java.net.SocketTimeoutException: Read timed out**
    *   *Meaning*: I knocked, they opened the door, but they are taking too long to reply.
    *   *Cause*: The Service is **SLOW** (DB locked, GC pause).
    *   *Fix*: Optimize the query or increase timeout (use with caution).

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Serialization Hell (Jackson)
**Error**: `com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "middleName"`
*   *Cause*: The JSON has a field `middleName`, but your Java POJO does not.
*   *Fix 1 (Quick)*: `@JsonIgnoreProperties(ignoreUnknown = true)`. (Dangerous - you might miss data).
*   *Fix 2 (Correct)*: Add the `middleName` field to your POJO.

### DNS Issues (UnknownHostException)
**Error**: `java.net.UnknownHostException: payment-service`
*   *Cause*: Kubernetes DNS (CoreDNS) failed to resolve the name.
*   *Debug*: Run `nslookup payment-service` inside the pod.

---

## 14. Summary & Architect Takeaways

*   **Transient Failures**: Network glitches happen. Retrying (with exponential backoff) fixes 99% of `ConnectExceptions`.
*   **Fail Fast**: If DNS is down, don't wait 30 seconds. Fail immediately.

# 🚨 Performance Incident Response: Triage for Architects

> **Document Level:** Architect (15+ years experience)  
> **Focus:** GC Analysis, Heap Dumps (MAT), CPU Spikes, and Thread Starvation Triage

---

## 🏗️ Layer 1: The First 5 Minutes (The Golden Triage)

When a Java production incident hits, don't restart the app immediately—you will lose the evidence.

### 1. The Dashboard Triage
-   **High CPU + Low Throughput**: The JVM is likely stuck in **GC Death Spiral**. It's spending 90% of its time cleaning memory and 10% on business logic.
-   **High Memory + OOM**: Likely a **Memory Leak**.
-   **Low CPU + Low Throughput**: Likely **Thread Starvation** (Blocked on a DB pool or a synchronized lock).

---

## 🛠️ Layer 2: Analyzing "Stop-The-World" (STW) Pauses

### 1. The GC Log Analysis
Use tools like **gceasy.io** or local `jstat` to look for:
-   **Full GC Frequency**: If Full GCs are happening every 30 seconds, your Old Gen is full.
-   **Promotion Rate**: If many objects are moving from Young to Old Gen quickly, your Young Gen is too small.

### 2. Safeguard Logs
Check for STW pauses not related to GC. Look for "Time to Safepoint" issues where a long-running thread is preventing the JVM from pausing.

---

## 🚀 Layer 3: Heap Dump Mastery (Eclipse MAT)

When you have a heap dump (`.hprof`), follow these steps:

### 1. The Dominator Tree
In MAT, use the **Dominator Tree** to see which objects are keeping others alive. 
- **Shallow Heap**: Memory used by the object itself.
- **Retained Heap**: Memory that would be freed if this object were garbage collected.
- **Architect Rule**: 90% of memory leaks are caused by a single static `Map` or `Cache` that never expires.

### 2. Path to GC Roots
If you find a suspicious object, find its "Path to GC Roots". This tells you exactly which class and field is still holding onto it.

---

## 📜 Layer 4: CPU Spike Analysis (Top -H and JStack)

### 1. Finding the "Hot" Thread
On Linux, use `top -H -p <pid>`. This shows individual thread CPU usage.
1.  Identify the **Thread ID** (TID) consuming 100% CPU.
2.  Convert TID to **Hex** (e.g., `1234` -> `0x4d2`).
3.  Run `jstack <pid>` and search for that Hex value.
4.  **Result**: You now have the exact stack trace of the thread burning the CPU.

---

## 🏁 Layer 5: Incident Management - The "Kill" Strategy

### 1. Automatic "Capturing"
Always configure your production JVMs with these flags:
-   `-XX:+HeapDumpOnOutOfMemoryError`: Saves a heap dump automatically.
-   `-XX:HeapDumpPath=/data/dumps`: Where to save it.
-   `-XX:OnOutOfMemoryError="kill -9 %p"`: Forces a restart to restore service after the dump is saved.

---

## 🧭 Interview Prep & Architect Scenarios

### Q: How do you tell the difference between a "Memory Leak" and "Poor Sizing"?
**A**: An architect looks at the "Post-GC Heap usage". In Poor Sizing, the usage is high but stable. In a Memory Leak, the usage after a Full GC rises linearly over time.

### Q: What is the "Throttling" symptom in Kubernetes?
**A**: If you see your app taking 2 seconds for a 50ms request, but CPU usage is low, check the Kubernetes **CPU Throttling** metrics. Your JVM might be trying to multi-thread, but the CGroup is cutting off its execution time for the remainder of the period.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [23-java-observability-and-performance-toolkit.md](./23-java-observability-and-performance-toolkit.md) | Observability |
| ⏩ **Next** | [25-java-production-hardening.md](./25-java-production-hardening.md) | Hardening |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026
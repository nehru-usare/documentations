# ☁️ Cloud Native Java: Performance in the Container Age

> **Document Level:** Architect (15+ years experience)  
> **Focus:** Container Awareness, AppCDS, Project CRaC, and Memory/CPU Isolation

---

## 🏗️ Layer 1: Container Awareness (The CGroup Reality)

Earlier versions of Java (pre-10) were "Container Blind". A JVM would see 64GB host RAM even if the Docker limit was 1GB, leading to the OS killing the container (**OOMKilled**).

### 1. `UseContainerSupport`
Enabled by default in Java 10+. It allows the JVM to read **CGroup** limits and adjust the Heap size accordingly.
- **Architect Rule**: Always use **Relative Memory Limits** (`-XX:MaxRAMPercentage=75.0`) instead of absolute values (`-Xmx1G`). This allows you to change the container RAM in Kubernetes without updating your Java flags.

### 2. CPU Shares & CFS Throttling
In Kubernetes, you set CPU `requests` and `limits`. 
- **The Problem**: If you set 250m (quarter core), and your JVM sees 32 host cores, it will create 32 threads for GC and ForkJoinPool. These threads will fight for the small CPU slice, leading to massive **latency spikes** due to context switching.
- **Fix**: Use `-XX:ActiveProcessorCount=n` to manually align the JVM's thread counts with the container's CPU quota.

---

## 🛠️ Layer 2: Startup & Memory Optimization (AppCDS)

Class loading is the single biggest bottleneck for Java microservice startup.

### 1. AppCDS (Application Class Data Sharing)
- **Concept**: The JVM parses and links classes once, then saves the result into a "Share Archive" file. 
- **Runtime**: Multiple JVMs on the same host can **memory-map** that same shared archive.
- **Architect Benefit**: 30-40% faster startup and significantly lower RAM usage across multiple containers on the same physical server.

---

## 🚀 Layer 3: The Holy Grail – Project CRaC

"Checkpoint/Restore at Checkpoint" (Java 17/21).

### 1. How it works
CRaC allows you to "Snapshot" a running, warmed-up JVM state into files on disk. 
- **Restore**: You can start a new container from that snapshot in **milliseconds**. The JVM is already "Hot", with the JIT code ready to go.
- **Architect Impact**: Merges the startup benefits of GraalVM (AOT) with the peak performance of HotSpot (JIT).

---

## 📜 Layer 4: The "OOMKilled" Mystery

Why does a container crash even when the Heap is mostly empty?

### 1. RSS (Resident Set Size) vs. Heap
The container limit applies to the **RSS** of the process, which includes:
- **Heap Area** (`-Xmx`)
- **Metaspace**
- **Thread Stacks** (`Number of Threads * -Xss`)
- **Code Cache**
- **Direct Buffers** (Off-Heap)
- **Native Memory** (JNI, Profiling tools)

### 2. The Golden Ratio
For a 1GB RAM container, an architect typically sets the JVM Heap to **600-750MB**. The remaining 250-400MB is the "Safety Buffer" for the native memory regions.

---

## 🚦 Layer 5: Modern Packaging (JLink)

### 1. Custom Runtimes
Instead of shipping a 300MB JDK, use **`jlink`** to create a custom, minimal JRE that only contains the modules your app needs.
- **Outcome**: Docker images dropping from 500MB to 80MB.

---

## 🧭 Interview Prep & Architect Scenarios

### Q: How do you optimize a Java app for AWS Lambda?
**A**: 1. **Tiered Compilation**: Use `-XX:TieredStopAtLevel=1` to stop at the C1 compiler, trading peak throughput for faster startup. 2. **AppCDS**: Pre-archive the classes. 3. **GraalVM**: If the workload is small and cold-starts are the #1 pain point, move to Native Image.

### Q: Why is `-Xms` (Initial Heap) often set equal to `-Xmx` in production?
**A**: To avoid the performance cost of **Heap Expansion** during the application's ramp-up phase. For containerized environments, setting them equal ensures the JVM claims its memory upfront, making the pod's resource usage predictable for the Kubernetes scheduler.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [18-native-image-and-graalvm.md](./18-native-image-and-graalvm.md) | GraalVM |
| ⏩ **Next** | [20-java-21-best-practices.md](./20-java-21-best-practices.md) | Best Practices |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026
# Latency Numbers Every Programmer Should Know (2026)

> **Part 10: Appendix**  
> **Difficulty:** ⭐ (Reference)  
> **Status:** The Physics of Computers

---

## 1. The Numbers (Humanized)

If **1 CPU Cycle** was **1 Second**:

| Operation | Actual Time | Human Scale |
| :--- | :--- | :--- |
| **1 CPU Cycle** | 0.4 ns | **1 Second** |
| **L1 Cache Access** | 1 ns | **2.5 Seconds** (Pick up paper on desk) |
| **L2 Cache Access** | 4 ns | **10 Seconds** (Walk to bookshelf) |
| **L3 Cache Access** | 20 ns | **1 Minute** (Walk to colleague's desk) |
| **Main Memory (RAM)** | 100 ns | **4 Minutes** (Walk to water cooler) |
| **NVMe SSD I/O** | 20 µs | **14 Hours** (Drive to next city) |
| **SSD I/O** | 100 µs | **3 Days** (Drive across country) |
| **Network (Intra-DC)** | 500 µs | **2 Weeks** (Ship by boat) |
| **Disk Seek (HDD)** | 5 ms | **5 Months** (Walk around the world) |
| **Network (US to EU)** | 150 ms | **12 Years** (Voyager 1 probe) |

---

## 2. Key Takeaways

### 1. Memory is the new Disk
*   RAM access (100ns) is fast, but compared to L1 cache, it's slow.
*   **Cache Locality** matters. Linked Lists (pointers) scatter data in RAM. Arrays keep data contiguous.

### 2. Disk is Lava
*   Even NVMe (Fastest Disk) is **200x slower** than RAM.
*   Avoid disk I/O on the hot path. Use Redis / Memcached.

### 3. Network is Unknown
*   Datacenter RTT (Recieve-Transmit Time) is fast (0.5ms).
*   WAN RTT is strictly bound by **Speed of Light**.
    *   Vacuum: 300,000 km/s.
    *   Fiber: ~200,000 km/s (Refractive index).
    *   NY to London (5600km) = $5600 / 200000 \times 2 \approx 56ms$.
    *   Protocols adds overhead. Real RTT $\approx 70-80ms$.

---

## 3. Bandwidth vs Latency

*   **Latency**: Time to get the *first byte*. (Pipe length).
*   **Bandwidth**: How *many bytes* per second. (Pipe width).
*   **Analogy**: Station Wagon full of Hard Drives driving down highway.
    *   **Bandwidth**: Massive (PB/hour).
    *   **Latency**: Terrible (Hours).

---

## 4. Compression Physics

*   Compressing 1GB data takes CPU time.
*   Sending 1GB data takes Network time.
*   **Rule**: If Compression Time + Transfer Time (Small) < Transfer Time (Large), then **Compress**.
*   In modern CPUs, nearly always **Compress** (Snappy, LZ4, Zstd).

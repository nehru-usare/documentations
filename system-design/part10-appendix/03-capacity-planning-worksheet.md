# Capacity Planning Worksheet

> **Part 10: Appendix**  
> **Difficulty:** ⭐ (Tool)  
> **Status:** Use this in interviews

---

## 1. Input Variables

| Variable | Example Value | Your Value |
| :--- | :--- | :--- |
| **DAU (Daily Active Users)** | 10 Million | |
| **MAU (Monthly Active Users)** | 50 Million | |
| **Avg Requests per User/Day** | 20 | |
| **Read:Write Ratio** | 100:1 | |
| **Avg Object Size** | 10 KB (Tweet) | |
| **Media Size** | 1 MB (Image) | |
| **Retention Period** | 5 Years | |

---

## 2. Traffic Calculations

### QPS (Query Per Second)
$$ QPS = \frac{DAU \times Req/Day}{24 \times 3600} $$
*   Example: $\frac{10M \times 20}{86400} \approx 2,300$ QPS.

### Peak QPS
*   Safe multiplier: **2x to 5x**.
*   Peak QPS = $2,300 \times 3 \approx 7,000$ QPS.

---

## 3. Storage Calculations

### Daily New Data
$$ Size_{Day} = Writes_{Day} \times Size_{Object} $$
*   Writes/Day = QPS(Write) * 86400.
*   If Read:Write = 100:1, Write QPS = 23.
*   Writes/Day = 2M.
*   Size/Day = $2M \times 10KB = 20GB$.

### 5 Year Storage
$$ Size_{Total} = Size_{Day} \times 365 \times 5 $$
*   $20GB \times 1825 \approx 36TB$.

---

## 4. Bandwidth Calculations

### Ingress (Incoming)
$$ Ingress = QPS_{Write} \times Size_{Object} $$
*   $23 \times 10KB = 230 KB/s$. (Tiny).

### Egress (Outgoing)
$$ Egress = QPS_{Read} \times Size_{Object} $$
*   $2300 \times 10KB = 23 MB/s$.
*   If Media included: $2300 \times 1MB = 2.3 GB/s$. (Massive).

---

## 5. Server Estimation

### Stateless Web Servers
*   Assume 1 Tomcat Thread handles 50 QPS (blocking) or 500 QPS (reactive).
*   Required Servers = Peak QPS / Server Capacity.
*   $7000 / 500 = 14$ Servers.

### Database Servers
*   Assume standard MySQL can handle 1000 Write QPS / 5000 Read QPS.
*   Write QPS = 70 (Peak). 1 Master is enough.
*   Read QPS = 7000 (Peak). 2 Slaves needed.

### Cache Servers (Redis)
*   Rule of thumb: Cache 20% of Daily Data.
*   Daily Data = 20GB. Cache = 4GB.
*   1 Redis instance (16GB RAM) is enough.

---

## 6. Summary Table

| Metric | Value |
| :--- | :--- |
| **Total QPS** | 7,000 |
| **Total Storage** | 36 TB |
| **Bandwidth** | 2.3 GB/s |
| **Web Nodes** | 14 |
| **DB Nodes** | 1 Master, 2 Slaves |
| **Cache Nodes** | 1 |

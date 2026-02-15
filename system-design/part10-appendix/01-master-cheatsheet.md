# System Design Master Cheatsheet

> **Part 10: Appendix**  
> **Difficulty:** ⭐ (Reference)  
> **Status:** Print this out

---

## 1. Core Formulas

### Local Performance
*   **Little's Law**: $L = \lambda W$
    *   (Avg Customers in Store) = (Arrival Rate) * (Avg Time in Store).
    *   Use: Calculating Thread Pool capacity.
*   **Amdahl's Law**: $S(n) = \frac{1}{(1-P) + \frac{P}{N}}$
    *   Max speedup is limited by the serial part of the program.

### Distributed Performance
*   **Universal Scalability Law**: $C(N) = \frac{N}{1 + \alpha(N-1) + \beta N(N-1)}$
    *   Explains why adding more servers eventually *slows down* the system (Coherency penalty).

---

## 2. The Power of Two (Approx)

| Power | Exact Value | Approx | Name |
| :--- | :--- | :--- | :--- |
| $2^{10}$ | 1,024 | 1 Thousand | 1 KB |
| $2^{20}$ | 1,048,576 | 1 Million | 1 MB |
| $2^{30}$ | 1,073,741,824 | 1 Billion | 1 GB |
| $2^{40}$ | ... | 1 Trillion | 1 TB |
| $2^{50}$ | ... | 1 Quadrillion | 1 PB |

---

## 3. Availability Math

| Availability | Downtime per Year | Downtime per Day |
| :--- | :--- | :--- |
| **99% (2 nines)** | 3.65 days | 14.4 mins |
| **99.9% (3 nines)** | 8.76 hours | 1.44 mins |
| **99.99% (4 nines)** | 52.6 mins | 8.64 secs |
| **99.999% (5 nines)** | 5.26 mins | 0.86 secs |
| **99.9999% (6 nines)** | 31.5 secs | 0.08 secs |

*   **Serial Reliability**: $R_{total} = R_1 \times R_2$. (Adding components reduces reliability).
*   **Parallel Reliability**: $R_{total} = 1 - (1-R_1)(1-R_2)$. (Redundancy increases reliability).

---

## 4. Back of the Envelope Constants

*   **Char**: 2 Bytes (UTF-16 in Java) / 1 Byte (ASCII).
*   **Int**: 4 Bytes.
*   **Long/Double**: 8 Bytes.
*   **UUID**: 16 Bytes (128 bits).
*   **Reference (Pointer)**: 8 Bytes (64-bit JVM).
*   **Object Header**: 12-16 Bytes.

---

## 5. Standard Ports

| Service | Port |
| :--- | :--- |
| HTTP / HTTPS | 80 / 443 |
| SSH | 22 |
| DNS | 53 (UDP) |
| PostgreSQL | 5432 |
| MySQL | 3306 |
| Redis | 6379 |
| MongoDB | 27017 |
| Kafka | 9092 |
| Zookeeper | 2181 |
| Cassandra | 9042 |
| Elasticsearch | 9200 |

---

## 6. HTTP Status Codes

*   **200**: OK.
*   **201**: Created (POST).
*   **202**: Accepted (Async processing).
*   **301**: Moved Permanently (Cacheable).
*   **302**: Found (Temp).
*   **400**: Bad Request (Client error).
*   **401**: Unauthorized (No Token).
*   **403**: Forbidden (Bad Token / No Permission).
*   **404**: Not Found.
*   **409**: Conflict (Optimistic Lock fail).
*   **429**: Too Many Requests (Rate Limit).
*   **500**: Internal Server Error (Bug).
*   **502**: Bad Gateway (Upstream down).
*   **503**: Service Unavailable (Overloaded).
*   **504**: Gateway Timeout.

---

## 7. Magic Numbers

*   **1 Million Request/Day** $\approx$ 12 Requests/Sec.
*   **Read:Write Ratio**: Usually 100:1 (Social Media) or 1000:1.
*   **Pareto Principle**: 20% of data gets 80% of traffic. (Cache that 20%).

---

## 8. Database Choices

| Use Case | Recommended DB |
| :--- | :--- |
| **Financial / ACID** | PostgreSQL, MySQL, Oracle |
| **Unstructured / Catalog** | MongoDB, Couchbase |
| **High Write / IoT / Logs** | Cassandra, ScyllaDB |
| **Search / Full Text** | Elasticsearch, Solr |
| **Caching / PubSub / Leaderboard** | Redis, Memcached |
| **Graph / Social** | Neo4j, Neptune |
| **Time Series / Metrics** | InfluxDB, Prometheus |
| **Object Storage** | S3, MinIO |
| **Analytic / Warehouse** | Snowflake, Redshift, ClickHouse|

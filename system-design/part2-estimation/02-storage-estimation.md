# Storage Estimation

> **Part 2: Estimation**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Avoiding "Disk Full" Alerts

---

## 0. Learning Objectives
*   **Beginner**: Know that 1 Char = 1 to 4 Bytes.
*   **Developer**: Calculate DB size for 1 million table rows.
*   **Architect**: Project storage growth over 5 years combined with replication and backup multipliers.

---

## 1. Problem Context
**Why does this exist?**
"Design Instagram."
User uploads a photo.
*   How big is the HDD?
*   Can one server hold it? (No).
*   How many Hard Drives do we need to buy for next year?
Storage Estimation determines if you need a **Sharded Database** or a **Blob Store (S3)**.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Units of Measure
*   **Byte (B)**: 8 bits.
*   **Kilobyte (KB)**: 1,000 Bytes (approx).
*   **Megabyte (MB)**: 1,000 KB.
*   **Gigabyte (GB)**: 1,000 MB.
*   **Terabyte (TB)**: 1,000 GB.
*   **Petabyte (PB)**: 1,000 TB.

### 2. Data Types
*   **Integer**: 4 Bytes.
*   **Long**: 8 Bytes.
*   **Char**: 1 Byte (ASCII) - 4 Bytes (UTF-8 Emoji).
*   **Reference / Pointer**: 8 Bytes (64-bit OS).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Row Size Formula

$$ \text{Row Size} = \sum(\text{Column Sizes}) + \text{Overhead} $$

**Example: User Table**
*   `id` (Long): 8 Bytes.
*   `username` (String 20 chars): 20 Bytes.
*   `email` (String 30 chars): 30 Bytes.
*   `created_at` (Long/Timestamp): 8 Bytes.
*   **Total**: ~66 Bytes per user.

**Capacity needed for 100M users**:
$$ 100,000,000 \times 66 \text{ Bytes} \approx 6.6 \text{ GB} $$
*   *Conclusion*: Everything fits in RAM!

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. The Multipliers
Storage is never just Raw Data.
1.  **Replication Factor**: Data is stored 3 times for availability. ($3\times$).
2.  **Backups**: Daily snapshots retained for 30 days. ($30\times$).
3.  **Indexes**: B-Trees take space. Add 50% overhead. ($1.5\times$).
4.  **Compression**: Text compresses well ($0.3\times$). Images do not ($1.0\times$).

### 2. Inode Limits
*   A file system runs out of **Inodes** (File Counts) before it runs out of Disk Space (Bytes) if you store millions of tiny files.
*   *Solution*: Object Storage (S3) or Haystack (Facebook).

---

## 5. Trade-Off Analysis

| Strategy | Cost | Performance | Use Case |
| :--- | :--- | :--- | :--- |
| **SSD (Flash)** | High | Fast (IOPS) | Database (Hot Data) |
| **HDD (Magnetic)** | Low | Slow (Seek time) | Backups / Logs (Cold Data) |
| **Object Store (S3)** | Low | Medium (Network) | Images / Videos |
| **Tape** | Extremely Low | Glacier Slow | 7-year Legal Archives |

---

## 6. Scaling Considerations

### Growth Forecasting (5 Years)
*   Current: 10TB.
*   Growth Rate: 2x per year.
*   Year 1: 20TB.
*   Year 2: 40TB.
*   Year 3: 80TB.
*   Year 4: 160TB.
*   Year 5: 320TB.
*   *Architect Decision*: Do not buy 320TB today. Buy 20TB. Scale horizontally next year.

---

## 7. Failure Scenarios & Recovery

### 1. Disk Full
*   **Scenario**: DB Log partition hits 100%.
*   **Event**: Database crashes. Can't even write "Error Log".
*   **Mitigation**: Alert at 80%. Log Rotation. Separate partitions for `/var/log` and `/var/lib/mysql`.

---

## 8. Security Considerations

### 1. Encryption Overhead
*   Encrypting data at rest adds negligible size overhead (IV + Padding).
*   However, efficient **Deduplication** becomes impossible (Encrypted "A" != Encrypted "A").

---

## 9. Performance Considerations

*   **Fragmentation**: Deleting data leaves "holes".
*   **Impact**: Scanning 1GB of data might require seeking across 5GB of disk.
*   **Fix**: `OPTIMIZE TABLE` or `VACUUM`.

---

## 10. Real Production Lessons

### WhatsApp's Video Problem
*   **Scenario**: Users send 100M videos daily. Avg 5MB.
*   **Total**: 500TB / Day.
*   **Challenge**: You can't store 500TB/day forever.
*   **Solution**: Aggressive retention policy (Delete from server after delivery) + Heavy compression.

---

## 11. Interview Questions

### Basic
1.  How many MB in a TB?
2.  Estimate the size of a generic User Profile.
3.  Why do we need 3x storage for every 1x data? (Replication).
4.  What is the size of a UUID? (16 Bytes / 36 Bytes as String).
5.  What is standard video size per minute? (HD ~100MB).

### Intermediate
1.  Estimate the storage for standard "Twitter" clone for 5 years.
2.  How does indexing affect storage size?
3.  Explain RAID 1 vs RAID 5 storage usage.
4.  Why don't we store images in the SQL Database (BLOB)?
5.  What is a "Sparse File"?

### Advanced
1.  Design the storage layer for Youtube. (Tiered Storage: Hot vs Cold).
2.  Calculate the cost difference between SSD and S3 Infrequent Access.
3.  How Does Columnar Storage (Parquet) reduce size compared to Row Storage (CSV)?
4.  Explain Erasure Coding (Reed-Solomon) vs Replication.
5.  Estimate the metadata storage size for an S3 clone with 100B objects.

### Architect-Level
1.  "We have 1PB of logs." Solution? (Elasticsearch Hot/Warm/Cold architecture).
2.  Evaluate the trade-off of compression: CPU cost vs Storage Savings.
3.  Design a Deduplication engine for a Backup System (Dropbox).

---

## 12. Scenario-Based System Design Problems

### 1. Design Instagram Storage
*   **Traffic**: 100M Photos / Day.
*   **Size**: 200KB / Photo (Compressed).
*   **Daily**: 20TB.
*   **Yearly**: 7.3 PB.
*   **10 Years**: 73 PB.
*   **Architecture**: S3 Bucket (Object Store). Metadata in Sharded SQL.

### 2. Design Text Message Storage
*   **Traffic**: 1B Msg / Day.
*   **Size**: 100 Bytes / Msg.
*   **Daily**: 100GB.
*   **Yearly**: 36TB.
*   **Architecture**: Cassandra or HBase (Wide Column Store). Efficient for write-heavy text.

---

## 13. Summary & Architect Takeaways

1.  **Text is Cheap, Media is Expensive**: 1 Billion tweets < 1 Hollywood Movie.
2.  **Tiered Storage**: Move old data to cheaper disks (S3 Glacier).
3.  **Replica Overhead**: Always convert "Data Size" to "Cluster Size" ($ \times 3 $).

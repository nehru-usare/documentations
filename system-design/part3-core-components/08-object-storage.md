# Object Storage Systems

> **Part 3: Core Components**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Infinite Hard Drive

---

## 0. Learning Objectives
*   **Beginner**: Understand why you can't install an OS on S3.
*   **Developer**: Use Pre-signed URLs to upload files securely.
*   **Architect**: Design a Data Lake using Object Storage as the foundation.

---

## 1. Problem Context
**Why does this exist?**
File Systems (NFS/Ext4) have limits.
*   Limit on file count (Inodes).
*   Limit on volume size (16TB).
*   Hierarchical structure is hard to scale.
**Object Storage** (S3) is flat, scalable, and accessible via HTTP API.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Block Storage (EBS)
*   Like a physical Hard Drive.
*   Mountable by OS. Fast IO. Expensive.
*   **Use**: Databases, Boot Drives.

### 2. File Storage (EFS/NAS)
*   Network File System. Shared by multiple servers.
*   Hierarchical (Folders).
*   **Use**: Shared configs, CMS uploads.

### 3. Object Storage (S3)
*   Flat structure (Buckets). Accessed via API (`PUT`, `GET`).
*   Infinite scaling. High Latency.
*   **Use**: Images, Backups, Static Sites.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Structure
*   **Bucket**: Top-level container. Globally unique name.
*   **Key**: The filename (`/photos/2023/dog.jpg`).
*   **Value**: The data (Blob).
*   **Metadata**: Key-Value pairs attached to the object (`Content-Type: image/jpeg`).

### API access
*   `PUT /bucket/key` (Upload)
*   `GET /bucket/key` (Download)
*   `HEAD /bucket/key` (Get Metadata)

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Erasure Coding
S3 does not just copy your file 3 times (Replication).
It breaks files into "Shards" + "Parity Shards" (Reed-Solomon).
*   **Benefit**: Higher durability with less storage overhead than 3x replication.

### 2. Strong Consistency
*   S3 was eventually consistent.
*   S3 is now **Strongly Consistent**.
*   Write -> Success -> Immediate Read returns new data.

---

## 5. Trade-Off Analysis

| Feature | Object Storage | Block Storage |
| :--- | :--- | :--- |
| **Interface** | REST API (HTTP) | OS Driver (SCSI) |
| **Latency** | High (Ms to Secs) | Low (Microseconds) |
| **Throughput** | Infinite (Parallel) | Limited by Volume |
| **Cost** | Cheap | Expensive |
| **Modifiable** | No (Write Once) | Yes (Random Write) |

---

## 6. Scaling Considerations

### Prefix Rate Limits
*   S3 scales by **Prefix**.
*   Limit: ~3,500 PUTs/sec per prefix.
*   *Bad*: `bucket/2023-10-01/file1`, `bucket/2023-10-01/file2`. (Hot Prefix).
*   *Good*: `bucket/a1b2-2023/file1`. (Random Hash Prefix).

---

## 7. Failure Scenarios & Recovery

### 1. Accidental Deletion
*   **Scenario**: Dev runs script to delete `tmp` files but deletes `prod`.
*   **Mitigation**: **Versioning** (Keep history) and **MFA Delete**.

---

## 8. Security Considerations

### 1. Pre-Signed URLs
*   **Scenario**: User uploads profile picture.
*   **Bad**: Upload to Server -> Server saves to S3. (Server is bottleneck).
*   **Good**: Server generates Pre-Signed URL -> User uploads directly to S3. (Server is bypassed).

---

## 9. Performance Considerations

*   **Multipart Upload**: For files > 100MB. Upload chunks in parallel. S3 stitches them.
*   **Transfer Acceleration**: Use AWS Edge Locations to route uploads to S3 over optimized backbone.

---

## 10. Real Production Lessons

### The "Public Bucket" Scan
*   **Scenario**: Company leaves bucket "Public".
*   **Event**: Security Researcher finds it. Downloads 1TB of passports.
*   **Lesson**: Block Public Access by default at Account Level.

---

## 11. Interview Questions

### Basic
1.  Difference between Block and Object storage.
2.  What is a Bucket?
3.  How do you edit a file in S3? (You can't. Download -> Edit -> Re-upload).
4.  Can you host a website on S3? (Yes, static).
5.  What is a Pre-signed URL?

### Intermediate
1.  Explain Multipart Upload.
2.  How does S3 Versioning work?
3.  What is S3 Glacier? (Cold Storage).
4.  How to optimize listing 1 Million files? (Pagination).
5.  Is S3 a CDN? (No, but serves as Origin).

### Advanced
1.  Design a Dropbox-like system using S3 (Metadata in DB, Blob in S3).
2.  Analyze the consistency model of S3.
3.  How does Erasure Coding save space compared to Replication?
4.  Architect a solution to bypass the 5TB object size limit.
5.  Explain "S3 Select" (Querying CSV inside S3).

### Architect-Level
1.  Design a Data Lake architecture (Raw -> Clean -> Curated) using S3 lifecycle policies.
2.  Critique the performance of Hadoop (HDFS) vs S3 for Big Data analytics.
3.  Implement a Cross-Region Replication (CRR) strategy for compliance.

---

## 12. Scenario-Based System Design Problems

### 1. Design Netflix Video Storage
*   **Req**: Petabytes of video masters.
*   **Solution**: S3 Standard for current hit movies. S3 Infrequent Access for older movies. Glacier for masters.

### 2. Design Instagram Photo Store
*   **Req**: Immutable keys.
*   **Solution**: Key = `MD5(Image Content)`. Deduplication built-in. Store Metadata in Cassandra.

---

## 13. Summary & Architect Takeaways

1.  **Immutable Storage**: Object storage forces you to treat files as immutable. This is good design.
2.  **Infinite is not Free**: You pay for Storage, Requests, AND Bandwidth.
3.  **Foundation of the Cloud**: Almost every modern architecture uses S3 as the ultimate source of truth.

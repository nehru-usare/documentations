# Design a Streaming Service (YouTube/Netflix)

> **Part 9: Interview Scenarios**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** Buffering...

---

## 0. Learning Objectives
*   **Beginner**: What is a Codec.
*   **Developer**: How Adaptive Streaming works (HLS).
*   **Architect**: Designing a global Transcoding Pipeline and CDN strategy.

---

## 1. Problem Context
**The Ask**: Architect YouTube.
*   **Input**: User uploads raw video (MOV, AVI, 4K, 10GB).
*   **Output**: Playable on Phone, TV, Laptop. Smooth playback.
*   **Scale**: 500 hours uploaded per minute. Massive Read bandwidth.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Transcoding
*   Raw video is huge and unsupported by browsers.
*   **Must convert to**:
    *   **Container**: .mp4, .ts
    *   **Codec**: H.264 (Common), VP9 (Google), AV1 (New, efficient).

### 2. Adaptive Bitrate Streaming (ABS)
*   Generate multiple qualities: 360p, 720p, 1080p, 4K.
*   **Protocol (HLS/DASH)**:
    *   Break video into 10s chunks (`chunk1.ts`, `chunk2.ts`).
    *   `manifest.m3u8` list the chunks.
*   **Player Logic**: Detect bandwidth. Request appropriate chunk. If slow net, switch to 360p chunk.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Components
1.  **Upload Service**: Presigned URL to S3.
2.  **Original Storage**: S3 Bucket (Raw).
3.  **Transcoder (Workers)**: FFmpeg cluster.
4.  **Transcoded Storage**: S3 Bucket (Processed).
5.  **CDN**: Distributes chunks globally.
6.  **Metadata DB**: Title, Description, Thumbnail.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Parallel Transcoding (DAG)
*   Transcoding a 2 hour 4K movie takes forever on 1 machine.
*   **Split & Merge**:
    1.  Split raw video into 5 min segments.
    2.  Parallel Workers transcode segments to (low, med, high).
    3.  Merge segments back.
    4.  Create Manifest.
*   **Tool**: AWS MediaConvert or Custom K8s Jobs.

### 2. Security (DRM)
*   **Digital Rights Management**:
    *   Encrypt content (AES).
    *   Issue Licenses (Widevine, FairPlay).
    *   Prevents stealing Netflix content. (YouTube usually doesn't DRM unless Paid/Movies).

---

## 5. Trade-Off Analysis

| Strategy | Storage Cost | CPU Cost | User Exp |
| :--- | :--- | :--- | :--- |
| **Keep Raw** | High | Low | Failed Playback |
| **Single Format** | Medium | Medium | Buffer on slow net |
| **Adaptive (ABS)** | **High** (Many copies) | **High** | **Best** |

---

## 6. Scaling Considerations

### Latency vs Cost
*   **CDN Cache Hit Ratio** is the most important metric.
*   Serving from S3 is expensive and slow. Serving from Edge is cheap(er) and fast.
*   **Long Tail**: Popular videos in High Tier CDN. Unpopular videos in Low Tier / pull from origin.

---

## 7. Failure Scenarios & Recovery

### 1. Upload Interruption
*   User uploads 9GB/10GB and net cuts.
*   **Resumable Upload**: Offset-based upload (Tus protocol).

---

## 8. Security Considerations

### 1. Content Moderation
*   Illegal content.
*   **Pipeline step**: Machine Learning frame analysis (NSFW detection) *before* publishing.

---

## 9. Performance Considerations

*   **Pre-fetching**: Load first 10s of next video while current is playing.
*   **Protocol**: QUIC (UDP) instead of TCP to reduce buffering (YouTube uses QUIC).

---

## 10. Real Production Lessons

### Netflix "Open Connect"
*   Netflix built their own CDNs (ISP Appliances).
*   They ship hard drives (appliances) to ISPs (Comcast/Verizon).
*   Traffic is served *from inside* the ISP network. Zero backbone cost.

---

## 11. Interview Questions

### Basic
1.  What is Transcoding?
2.  Difference between Container and Codec.
3.  How does streaming work? (Chunks).
4.  Storage choice (Blob Storage).
5.  Why use CDN?

### Intermediate
1.  Explain HLS / MPEG-DASH.
2.  How to implement Resumable Uploads?
3.  Database schema for Video Metadata.
4.  Safety/Moderation pipeline.
5.  What is the manifest file?

### Advanced
1.  Design the Directed Acyclic Graph (DAG) for the processing pipeline.
2.  Optimize storage (Erasure Coding).
3.  Cost analysis: Saving bandwidth (AV1 codec) vs CPU cost to encode AV1.
4.  How to handle Livestreaming? (HLS latency issues, RTMP ingest).
5.  Critique Netflix Open Connect vs Akamai.

### Architect-Level
1.  "We have 1 Exabyte of data. Archival strategy?" (Glacier Deep Archive).
2.  Design a "Recommendation System" integration point (Async events on View).
3.  Evaluate building a P2P CDN (Torrents) for cost saving. (Legal/Quality issues).

---

## 12. Scenario-Based System Design Problems

### 1. Design Live Streaming (Twitch)
*   **Diff**: Latency is key.
*   **Arch**: RTMP Ingest -> Transcode -> Low Latency HLS (CMAF).

### 2. Design TikTok
*   **Diff**: Mobile only. Vertical. Short.
*   **Arch**: Pre-download next videos. Aggressive caching.

---

## 13. Summary & Architect Takeaways

1.  **Bandwidth is Money**: Every byte saved by better compression is millions of dollars.
2.  **CDN is King**: You are a logistics company moving bits.
3.  **Async Processing**: Nothing is real-time except the playback. Upload processing is a complex workflow.

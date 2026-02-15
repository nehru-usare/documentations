# 02. System Design Scenarios (Real-world Problems)

> **Part 10: Interview & Scenario Kit**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Architect)  
> **Status:** The Masterclass

---

## 📅 Scenario 1: Design a URL Shortener (TinyURL)

### 1. Requirements
*   **Functional**: Given a long URL, return a short URL. Redirect short URL to long URL.
*   **Non-Functional**: Highly Available (A redirect must never fail). Read-heavy (100:1 ratio).

### 2. High-Level Design
*   **Endpoint**: `POST /api/shorten { url: "..." }` -> Returns `http://tiny.url/abc1234`.
*   **Endpoint**: `GET /abc1234` -> 301 Redirect to Original.

### 3. The Hard Part (Collision)
*   **Hash**: MD5("google.com") = `a1b2c3d4...` (Too long).
*   **Base62**: Convert ID (100000) to Base62 (`abc`).
*   **Distributed ID**: Use Snowflake ID or Zookeeper to generate unique IDs.

---

## 💬 Scenario 2: Design a Chat App (WhatsApp)

### 1. Requirements
*   **Functional**: 1-on-1 Chat. Group Chat. Online Status.
*   **Non-Functional**: Low Latency (Real-time). Delivery Guarantee (Sent, Delivered, Read).

### 2. Architecture
*   **Protocol**: WebSocket (Persistent connection).
*   **Storage**: HBase / Cassandra (Write-heavy). NOT SQL.
*   **Message Queue**: Kafka (for offline users/push notifications).

### 3. The "Online Status" Problem
*   **Heartbeat**: Client sends ping every 5s. Redis stores "Last Seen".
*   If heartbeat misses for 30s -> Mark offline.

---

## 📺 Scenario 3: Design Netflix (Video Streaming)

### 1. Requirements
*   **Functional**: Upload Video. Stream Video.
*   **Non-Functional**: No buffering. High Resolution.

### 2. Architecture
*   **Upload**: User uploads `.mp4`. S3 triggers Lambda.
*   **Transcoding**: Convert `.mp4` to HLS (360p, 720p, 1080p, 4k).
*   **CDN**: Push files to Edge Servers (CloudFront/Akamai).
*   **Client**: Adaptive Bitrate Streaming (Switch to 360p if internet is slow).

---

## 14. Summary & Architect Takeaways

*   **Clarify Requirement**: Before drawing a box, ask "How many users?".
*   **Back-of-Envelope Math**: 1M users * 1KB/user = 1GB bandwidth.
*   **Trade-offs**: "I chose NoSQL because writes occur 10x more than reads".

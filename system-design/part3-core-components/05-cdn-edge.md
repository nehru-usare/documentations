# CDN & Edge Computing

> **Part 3: Core Components**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Speed of Light Hack

---

## 0. Learning Objectives
*   **Beginner**: Understand that a CDN moves files closer to the user.
*   **Developer**: Configure `Cache-Control` headers correctly.
*   **Architect**: Design Edge Logic to run code closer to the user (Serverless Edge).

---

## 1. Problem Context
**Why does this exist?**
Your server is in Virginia (USA).
Your user is in Singapore.
**Distance**: 15,000 km.
**Speed of Light**: ~300,000 km/s.
**Min Latency**: ~100ms RTT. (Optimistic). Real world ~250ms.
You cannot fix physics.
**Solution**: Put a copy of the content in a server in Singapore.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. CDN (Content Delivery Network)
*   A network of servers (PoPs - Points of Presence) distributed globally.
*   They cache **Static Content** (Images, CSS, JS, Video).

### 2. Edge Computing
*   Running **Logic** (Code) on these CDN servers, not just storing files.
*   Example: Authentication, Image Resizing, A/B Testing routing.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Push vs Pull Types

1.  **Pull CDN (Origin Fetch)**:
    *   User asks CDN for `image.jpg`.
    *   CDN doesn't have it.
    *   CDN pulls it from your Origin Server, saves it, and serves it.
    *   *Best for*: Web Assets. Low maintenance.

2.  **Push CDN**:
    *   You upload files to such CDN manually.
    *   *Best for*: Huge files (Software Installers, Game Patches).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Anycast DNS
*   User queries `cdn.example.com`.
*   DNS returns an IP Address.
*   **Magic**: This same IP address is announced by routers all over the world.
*   The internet routing protocol (BGP) routes the user to the **physically closest** datacenter.

### 2. Cache Control Headers
*   `Cache-Control: public, max-age=3600`: Cache for 1 hour.
*   `ETag`: "Fingerprint" of the file. If file hasn't changed, Server returns `304 Not Modified` (0 bytes).

---

## 5. Trade-Off Analysis

| Strategy | Latency | Cost | Freshness |
| :--- | :--- | :--- | :--- |
| **No CDN** | High | Low | Instant |
| **CDN (Static)** | Low | Medium | Lag (TTL) |
| **Edge Compute** | Lowest | High | Instant Logic |

---

## 6. Scaling Considerations

### Offloading the Origin
*   A properly configured CDN absorbs **95-99%** of traffic.
*   Your Origin Server only sees traffic when content changes or TTL expires.
*   *Scale*: CDN allows a Raspberry Pi to serve 1 Million users (serving static HTML).

---

## 7. Failure Scenarios & Recovery

### 1. Stale Content
*   **Scenario**: You deploy a Critical Bug Fix (`script.js`).
*   **Problem**: CDN has cached the buggy version for 24 hours.
*   **Fix**: **Cache Invalidation** (Purge). Or use **Versioning** (`script.v2.js`).
*   *Architect Rule*: Always use Versioning (Fingerprinting) for assets. Never rely on Purge (it's slow/unreliable).

---

## 8. Security Considerations

### 1. DDoS Sponge
*   CDNs have massive bandwidth (Tbps). They absorb Volumetric Attacks easily.

### 2. Signed URLs
*   **Scenario**: Paid Course Video.
*   **Problem**: User shares link on Reddit.
*   **Fix**: Signed URLs (AWS CloudFront Signed Cookies). Link expires in 1 hour and is tied to IP.

---

## 9. Performance Considerations

### Dynamic Content Acceleration
*   CDNs can also speed up **Dynamic API calls**.
*   How? They keep a persistent, optimized TCP connection to your Origin. They optimize the "Middle Mile".

---

## 10. Real Production Lessons

### The "Private Data" Leak
*   **Scenario**: Developer sets `Cache-Control: public` on a page that contains User Profile data (`Hello, Nehru`).
*   **Result**: CDN caches it. Next user sees `Hello, Nehru`.
*   **Fix**: `Cache-Control: private` for anything user-specific. `Vary: Cookie`.

---

## 11. Interview Questions

### Basic
1.  What is a CDN?
2.  Difference between Push and Pull CDN.
3.  What is a PoP?
4.  Why is Single Region hosting slow for global users?
5.  What is `max-age`?

### Intermediate
1.  How does Anycast DNS work?
2.  Explain Cache Invalidation vs Versioning.
3.  What is Edge Computing?
4.  Can you cache API responses in CDN? (Yes, if valid).
5.  How do Signed URLs work?

### Advanced
1.  Design a Video Streaming Architecture using CDN (HLS/DASH).
2.  Analyze the economics of Multi-CDN switching strategies.
3.  How do you debug a specific CDN node being slow? (Trace headers).
4.  Compare Edge Workers (Cloudflare) vs Lambda@Edge (AWS).
5.  Explain "Cache Sharding" in CDNs.

### Architect-Level
1.  "We need to serve personalized dynamic content in <50ms globally." Architect the solution. (Edge Rendering).
2.  Evaluate the security risks of executing Wasm at the Edge.
3.  Design a specialized CDN for live betting (Sub-second latency requirements).

---

## 12. Scenario-Based System Design Problems

### 1. Design Netflix Video Delivery
*   **Content**: Static, Large, Global.
*   **Solution**: Massive Push CDN (Open Connect). Pre-position popular movies during off-peak hours.

### 2. Design Real-Time Chat Global
*   **Content**: Dynamic, Small.
*   **Solution**: CDN for static assets. WebSocket acceleration (Global Accelerator) for chat packets.

---

## 13. Summary & Architect Takeaways

1.  **Don't reinvent the wheel**: Don't build your own global network. Rent Akamai/Cloudflare.
2.  **Versioning > Purging**: Filenames should be immutable (`app.a1b2c3.js`).
3.  **The Edge is the New Server**: Move logic (Auth, Redirection, Formatting) to the Edge to kill latency.

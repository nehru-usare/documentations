# Design a URL Shortener (TinyURL)

> **Part 9: Interview Scenarios**  
> **Difficulty:** ⭐⭐ (Beginner/Intermediate)  
> **Status:** The Warm-up Exercise

---

## 0. Learning Objectives
*   **Beginner**: Difference between HTTP 301 and 302.
*   **Developer**: Implementing Base62 encoding.
*   **Architect**: Designing a collision-free distributed ID generator for the short codes.

---

## 1. Problem Context
**The Ask**: Design TinyURL / Bit.ly.
*   **Input**: `https://www.very-long-url.com/some/path?query=123`
*   **Output**: `http://tiny.url/AbC123z`
*   **Scale**: 100M links generated/day. Read:Write = 100:1.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. HTTP Redirects
*   **301 (Moved Permanently)**: Browser caches the mapping. Next time, browser goes straight to destination. *Pros*: Fast. *Cons*: You lose analytics (Server doesn't see 2nd request).
*   **302 (Found / Temporary)**: Browser hits Shortener every time. *Pros*: Analytics. *Cons*: Server load.

### 2. The Short Code
*   Allowed characters: `[a-z]`, `[A-Z]`, `[0-9]`. Total = 26+26+10 = 62.
*   Base62 encoding.
*   Length 6: $62^6 \approx 56$ Billion combinations. (Enough).
*   Length 7: $62^7 \approx 3.5$ Trillion. (Future proof).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Components
1.  **Web Server**: Handles `create(longUrl)` and `get(shortUrl)`.
2.  **Database**: Stores `id, longUrl, shortUrl`.
3.  **KGS (Key Generation Service)**: Pre-generates unique keys to avoid runtime collisions.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Hashing vs ID Generation
*   **Approach A: Hashing (MD5)**
    *   `Hash(longUrl)`. Take first 7 chars.
    *   *Problem*: Collisions. Different URLs might hash to same prefix.
*   **Approach B: Counter + Base62**
    *   Database Auto-Increment ID: 1, 2, 3...
    *   `1000000` (Base 10) -> `4c92` (Base 62).
    *   *Problem*: Predictable (`abc`, `abd`). Competitors can scrape usage.
*   **Approach C: Distributed ID (Snowflake)**
    *   Generate random-ish 64-bit ID. Base62 encode it.
    *   *Winner*.

### 2. KGS (Offline generation)
*   Generate all 6 char keys (`aaaaaa` to `zzzzzz`). Store in a "Unused Keys" DB.
*   When user requests, Pop one key. Mark used.
*   *Fast*: No calculation at runtime.

---

## 5. Trade-Off Analysis

| Strategy | Collision Risk | Latency | Predictability |
| :--- | :--- | :--- | :--- |
| **Run-time Hash** | High | Low | Random |
| **DB Counter** | None | Medium (DB Write) | Sequential (Bad) |
| **KGS** | None | Very Low | Random |

---

## 6. Scaling Considerations

### Database
*   **Read Heavy (100:1)**.
*   **Cache**: Memcached/Redis is mandatory.
*   Store `key=shortUrl`, `value=longUrl`.
*   99% of traffic will hit Cache. DB is only for the 1% (Long Tail).
*   Eviction: LRU (Least Recently Used).

---

## 7. Failure Scenarios & Recovery

### 1. KGS Exclusion
*   Two KGS servers grab the same range of keys.
*   **Fix**: Zookeeper to assign ranges to KGS instances.

### 2. Cache Penetration
*   User requests valid-looking ID `AbC123z` that doesn't exist.
*   Hit Cache (Miss) -> Hit DB (Miss).
*   Attackers can DDOs DB using this.
*   **Fix**: **Bloom Filter**. Check if ID exists before hitting DB.

---

## 8. Security Considerations

### 1. Malicious Redirects
*   Phishing sites using TinyURL to mask URL.
*   **Fix**: Real-time scanning (Google Safe Browsing API) before redirecting.

---

## 9. Performance Considerations

*   **Geo-Distribution**:
    *   If user is in India, and Redirect Server is in US -> 200ms delay.
    *   Deploy Read replicas / Caches to Edge locations.

---

## 10. Real Production Lessons

### Instagram
*   Used PostgreSQL.
*   Needed unique IDs for photos.
*   Built a custom PL/PGSQL function to generate IDs based on Shard ID + Time + Sequence. (Similar to Snowflake).

---

## 11. Interview Questions

### Basic
1.  301 vs 302 Redirect.
2.  Why Base62 and not Base64? (+ and / cause URL issues).
3.  Calculate storage for 5 years (Variables provided).
4.  Simple DB Schema.
5.  What is a collision?

### Intermediate
1.  Design the API (`POST /api/v1/data/shorten`).
2.  How to prevent users from guessing the next URL?
3.  Cache Eviction Policy choice?
4.  How to implement Analytics? (Async event to Kafka).
5.  SQL vs NoSQL for this problem? (NoSQL Key-Value is better fit).

### Advanced
1.  Design the KGS logic to guarantee Uniqueness and concurrency.
2.  How to handle "Custom Alias" (`tiny.url/marketing-campaign`)?
3.  Explain Bloom Filter usage here.
4.  How to scale the Write component?
5.  Database cleanup (Expiry of old links).

### Architect-Level
1.  "We have 1 Trillion old links. Storage is expensive. Architect a tiering solution." (Hot in Redis, Warm in DynamoDB, Cold in S3/Glacier is overkill? Maybe just DynamoDB TTL).
2.  Design a Pastebin (Similar to URL shortener, but stores text blobls).
3.  Evaluate using a Trie for storage optimization (Not worth it, Key-Value is O(1)).

---

## 12. Scenario-Based System Design Problems

### 1. Design QR Code Generator
*   **Req**: Visual URL shortener.
*   **Add-on**: Generate PNG on the fly. Store in CDN.

### 2. Design Internal Link Sharing (Go Links) (go/eats)
*   **Req**: Employee intranet.
*   **Arch**: LDAP/SSO integration. Simple mapping.

---

## 13. Summary & Architect Takeaways

1.  **Simplicity**: It's a glorified HashMap. Don't overengineer.
2.  **Read vs Write**: Optimize for Reads (Cache, Edge).
3.  **IDs**: ID generation is the hardest part. Master Snowflake/KGS patterns.

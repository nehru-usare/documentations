# Design a Web Crawler (Google Bot)

> **Part 9: Interview Scenarios**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The Internet Cartographer

---

## 0. Learning Objectives
*   **Beginner**: How Google finds new pages.
*   **Developer**: Writing a Python script to scrape a site (BeautfulSoup).
*   **Architect**: Scaling to fetch and index 2 Billion pages per day without banning yourself.

---

## 1. Problem Context
**The Ask**: Build Google Search Indexer.
*   **Input**: Seed URLs (`cnn.com`, `wikipedia.org`).
*   **Process**: Download -> Extract Links -> Add to Queue -> Repeat.
*   **Scale**: 1 Billion Pages/month.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Crawl Strategy
*   **BFS (Breadth First Search)**: Explore neighbors first. (Go wide). *Preferred for crawlers* (Finds high-level important pages).
*   **DFS (Depth First Search)**: Drill down. (Go deep). *Bad* (Can get stuck in "Spider Traps" / infinite loops).

### 2. Politeness
*   Do not kill the target server.
*   Respect `robots.txt`.
*   Limit rates (e.g., 1 request per second per domain).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Components
1.  **URL Frontier**: The priority queue of "To Be Visited" URLs.
2.  **Fetcher (Workers)**: Downloads HTML.
3.  **DNS Resolver**: Custom optimized DNS cache.
4.  **Content Parser**: Extracts links and text.
5.  **Dedup Service**: Checks "Have we seen this page before?"
6.  **Storage**: S3 (HTML blobs) + BigTable/Cassandra (Metadata).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. The URL Frontier (Sharded Queue)
*   You cannot have one big Queue.
*   **Logic**:
    *   Front Queues (Priority): Assigns High/Low priority.
    *   Back Queues (Politeness): **One Queue per Domain**.
    *   Mapping Table: `wikipedia.org` -> Queue #55.
    *   Worker #55 only pulls from Queue #55. Handles delays/throttling for that domain locally.

### 2. Duplicate Detection
*   **URL Dedup**: Canonicalization (`google.com` == `google.com/`). Bloom Filter.
*   **Content Dedup**: 30% of web is duplicates.
    *   Use **SimHash** or **Checksum** (MD5).
    *   If `SimHash(NewPage)` is close to `SimHash(StoredPage)`, discard.

---

## 5. Trade-Off Analysis

| Feature | Centralized | Distributed |
| :--- | :--- | :--- |
| **Speed** | Slow | Fast |
| **Complexity** | Low | High (Coordination) |
| **Politeness** | Easy to enforce | Hard (Two workers might hit same domain) |
| **Verdict** | Toy | **Google Scale** |

---

## 6. Scaling Considerations

### DNS is the Bottleneck
*   DNS resolution takes 10ms - 500ms.
*   Crawlers make millions of requests.
*   **Solution**: Maintain a massive local DNS Cache (don't rely on ISP DNS). Prefetch DNS.

---

## 7. Failure Scenarios & Recovery

### 1. Spider Traps (Infinite Loops)
*   `example.com/calendar/2023/jan` -> `.../feb` -> `.../march`.
*   Infinite generated specific URLs.
*   **Fix**: Max Depth Limit. Max URL Length Limit. Cycle detection.

---

## 8. Security Considerations

### 1. Honeypots
*   Hidden links invisible to humans but visible to bots.
*   Used to ban scraper IPs.
*   **Fix**: Be polite. Identify User-Agent (`Googlebot/2.1`).

---

## 9. Performance Considerations

*   **HTML Parsing**: DOM parsing is slow.
*   **Headless Browsers**: (Puppeteer). Renders JS (Client Side Rendering). Extremely heavy on CPU.
*   *Strategy*: Only crawl Static HTML for speed. Render JS only for high-value sites.

---

## 10. Real Production Lessons

### Apache Nutch
*   Open Source Crawler (Hadoop based).
*   Uses MapReduce logic to process crawl batches.

### Google
*   Refreshes "Important" pages (CNN Home) seconds.
*   Refreshes "Unimportant" pages (Old blog post) monthly.
*   **PageRank** determines crawl frequency.

---

## 11. Interview Questions

### Basic
1.  BFS vs DFS for crawling?
2.  What is `robots.txt`?
3.  How to identify valid URL?
4.  Storage choice for HTML content? (Object Storage).
5.  What is a Seed URL?

### Intermediate
1.  Explain the URL Frontier architecture.
2.  How to deduplicate URLs?
3.  How to deduplicate Content? (SimHash).
4.  Handling dynamic content (JavaScript).
5.  DNS optimization strategies.

### Advanced
1.  Design a Distributed URL Frontier (Kafka vs Redis vs Custom).
2.  Algorithm to measure "Importance" (PageRank).
3.  How to handle "Updates"? (Re-crawling strategy).
4.  Critique storing web graph in Graph DB vs Adjacency List.
5.  Detecting and Handling "Soft 404s".

### Architect-Level
1.  "We need to archive the entire internet (Wayback Machine)." Architect the storage tier.
2.  Design a crawler that respects "Politeness" in a distributed environment without central coordination? (Consistent Hashing: Domain X always goes to Worker Y).
3.  Evaluate the legality of crawling (Copyright vs Fair Use).

---

## 12. Scenario-Based System Design Problems

### 1. Design Price Monitor (Amazon Scraper)
*   **Req**: Track price of iPhone.
*   **Diff**: Targeted crawl. CSS Selectors. Proxy rotation (to avoid bans).

### 2. Design Copyright Infringement Finder
*   **Req**: Find exact copies of my text.
*   **Tech**: SimHash / Shingling.

---

## 13. Summary & Architect Takeaways

1.  **Politeness First**: If you DDoS the web, you get banned.
2.  **Partitions**: Partition by **Host/Domain**. This solves politeness and locality.
3.  **The Web is Dirty**: Malformed HTML, traps, infinite redirects, huge files. Robustness is key.

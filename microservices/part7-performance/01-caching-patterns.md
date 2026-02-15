# 01. Caching Patterns (Cache-Aside, Write-Through, 2nd Level)

> **Part 7: Performance & Scalability**  
> **Difficulty:** ⭐⭐⭐⭐ (Architect)  
> **Status:** High Impact

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand Cache Hit vs Cache Miss. |
| **Developer** | Configure Hibernate Second Level Cache with EhCache/Redis. |
| **Architect** | Choose between Write-Through and Write-Behind strategies. |

---

## 1. Why This Topic Exists

### The Bottleneck
Database I/O is slow (Disk access, network roundtrip).
Memory is fast.
**Goal**: Reduce DB hits by 90%.

---

## 3. Core Concepts (🟢 Beginner Level)

### Eviction Policies
When cache is full, what do we delete?
1.  **LRU (Least Recently Used)**: Delete the one nobody touched for the longest time. (Standard).
2.  **TTL (Time To Live)**: Delete anything older than 10 minutes.

### The Big 4 Patterns

1.  **Cache-Aside (Lazy)**: App talks to Cache. If miss, App talks to DB.
    *   *Pros*: Resilient to cache failure.
    *   *Cons*: First request is slow (Thundering Herd risk).

2.  **Read-Through**: App talks to Cache. Cache talks to DB.
    *   *Pros*: Simple App code.
    *   *Cons*: Complex Cache setup (Cache needs DB drivers).

3.  **Write-Through**: App writes to Cache. Cache writes to DB synchronously.
    *   *Pros*: Data is always consistent.
    *   *Cons*: Slow writes (Double write penalty).

4.  **Write-Behind (Async)**: App writes to Cache. Cache queues DB write.
    *   *Pros*: Blazing fast writes.
    *   *Cons*: Data loss if Cache crashes before syncing.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Hibernate Second Level Cache
Standard JPA caching.
*   **L1 Cache**: Session scope (Transaction). On by default.
*   **L2 Cache**: SessionFactory scope (Global). Shared across threads.
    *   *Provider*: EhCache, Infinispan, Redis.
    *   *Usage*: Annotate Entity with `@Cacheable`.

---

## 9. Architect-Level Best Practices

1.  **Don't Cache Everything**: Only cache **Read-Heavy, Write-Rarely** data (e.g., Product Catalog, Country List).
2.  **Versioning**: If you change the Object structure, clear the cache. Serialization errors will kill you.
3.  **Distributed Lock**: For "Expensive Calculations", ensure only 1 server calculates it, puts in cache, and others wait.

---

## 14. Summary & Architect Takeaways

*   **Trade-off**: You trade Consistency for Availability and Partition Tolerance (CAP).
*   **Invalidation**: The hardest problem. Better to have a short TTL than a complex invalidation logic.

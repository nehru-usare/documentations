# Chapter 17: Caching Strategies (Local & Distributed)

## 0. Learning Objectives

- **🟢 Beginner**: Understand the concept of "Caching" and how to use `@Cacheable` to speed up slow operations.
- **🟡 Professional**: Master Cache Invalidation (`@CacheEvict`), `@CachePut`, and configuring multiple cache managers (Ehcache for local, Redis for distributed).
- **🔴 Architect**: Deep dive into the `CacheInterceptor` internals, understand the "Cache Stampede" problem, and design a tiered caching strategy (L1 + L2) for extreme-scale architectures.

---

## 1. Why This Topic Exists

### Real-World Business Problem
The Database is usually the slowest part of any application. If your Homepage fetches "Categories" from the DB on every single hit, and you have 10,000 users per second, your database will melt. Caching allows you to store the answer to a question so you don't have to ask it again.

### Technical Limitations Solved
- **Latency Reduction**: Fetching from memory (RAM) is 1,000x-10,000x faster than fetching from a disk-based database.
- **Cost Efficiency**: In cloud environments, caching data can significantly reduce the CPU and I/O costs of your managed database (RDS/Spanner).

---

## 2. Big Picture Architecture View

Caching in Spring is a **declarative abstraction**. The code only knows "Should I cache this?", while the configuration decides "Where does the data go?" (Redis, Caffeine, or a Map).

### Interaction with Other Modules
- **AOP**: Caching uses proxies to intercept method calls.
- **Data Layer**: Frequently used alongside Spring Data to cache repository results.
- **Security**: Be careful—never cache sensitive per-user data in a shared global cache!

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Hit**: The data was found in the cache.
- **Miss**: Data was not in the cache; we had to go to the Database.
- **TTL (Time To Live)**: How long the data remains in the cache before being deleted.

### Simple Explanation
Think of a **Student studying for a test**.
- **The Brain (Cache)**: The student remembers the answer to "What is 2+2?". (Hit).
- **The Textbook (Database)**: For a complex question, the student has to open the book and look it up. (Miss).
Once looked up, the student "Caches" the answer in their brain for the next time.

### Minimal Working Example
```java
@EnableCaching
@SpringBootApplication
public class App { }

@Service
public class ProductService {
    @Cacheable("products")
    public Product getById(Long id) {
        // Slow DB call here
        return repo.findById(id).orElseThrow();
    }
}
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### The Core Annotations
- **`@Cacheable`**: "Check the cache first. If found, return. if not, run the method and save the result."
- **`@CacheEvict`**: "Delete the entry from the cache." (Run after an Update or Delete operation).
- **`@CachePut`**: "Run the method and ALWAYS update the cache with the result."

### Choosing your Cache Provider
1. **Caffeine**: High-performance local Java cache (Best for single-node apps).
2. **Ehcache**: Full-featured local cache with disk-overflow support.
3. **Redis**: Distributed key-value store (Best for multi-node/microservices).

---

## 5. Internal Mechanics (🔴 Advanced Level)

### The `CacheInterceptor`
When a `@Cacheable` method is called:
1. The **Proxy** intercepts.
2. It uses a `KeyGenerator` (usually based on method parameters) to create a key.
3. It asks the `CacheManager` for the `Cache` object.
4. It calls `cache.get(key)`.
5. If null, it calls the `proceed()` method (your real code) and then calls `cache.put(key, result)`.

### Cache Invalidation Strategies
- **Time-Based**: Simple TTL.
- **Event-Based**: When `Order` is updated, evict the `OrderSummary` cache.
- **Write-Through**: Every write to the DB immediately updates the cache.

---

## 6. Under the Hood

### Conditional Caching
You don't have to cache everything.
```java
@Cacheable(value = "users", condition = "#id > 100", unless = "#result == null")
```
- **`condition`**: Check parameters BEFORE running.
- **`unless`**: Check the result AFTER running (e.g., "Don't cache nulls").

---

## 7. Real-World Use Cases

- **E-Commerce Product Page**: Caching product descriptions that change once a day.
- **Stock Quotes**: Caching the price for 1 second to handle a massive burst of requests during market open.

---

## 8. Production & Performance Considerations

- **Cache Stampede (Thundering Herd)**: When a popular cache key expires, 1,000 threads all see a "Miss" at once and all hit the database simultaneously.
  - **Solution**: Use **`sync = true`** in `@Cacheable`. This locks the cache entry so only ONE thread goes to the DB while others wait.
- **Serialization**: For distributed caches (Redis), your objects MUST be `Serializable`. Be wary of the overhead of converting large objects to/from JSON/Binary.

---

## 9. Architect-Level Best Practices

- **Evict on Update**: Never update the database without either evicting or updating the cache. "Stale Data" is a top source of user complaints.
- **Size Matters**: Monitor your Cache Size. A cache without a size limit will eventually cause an **OOM** (Out of Memory).
- **Custom Key Generators**: For methods with many parameters, design a custom `KeyGenerator` to avoid key collisions.

---

## 10. Common Mistakes & Anti-Patterns

- **Caching too much**: If your cache "Hit Rate" is below 10%, you are just wasting memory.
- **Infinite TTL**: Data that never expires will eventually become stale or fill up the disk.
- **Self-Invocation**: Just like `@Transactional`, `@Cacheable` won't work if you call the method from within the same class (Proxy limitation).

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **Redis Desktop Manager**: Connect to your Redis and look at the keys/values.
- **Logging**: `logging.level.org.springframework.cache=TRACE`. You will see "Cache Match" or "Cache Miss" for every call.
- **Metrics**: Watch `cache.gets` and `cache.puts` in Actuator/Prometheus.

---

## 12. Comparisons

### Local vs. Distributed Cache
| Feature | Local (Caffeine/Ehcache) | Distributed (Redis/Hazelcast) |
| :--- | :--- | :--- |
| **Speed** | Instant (Memory-to-Memory) | Fast (Network/TCP) |
| **Consistency** | Node-specific (Stale on others) | Consistent across all nodes |
| **Complexity** | Low | Higher (Requires Server/Infra) |
| **Recommendation** | Single node / Immutable data | **Microservices / High Traffic** |

---

## 13. Interview Questions

### 🟢 Basic
1. What is the difference between a cache "Hit" and a "Miss"?
2. What annotation do you use to enable caching in Spring Boot?

### 🟡 Intermediate
1. Compare `@CacheEvict` and `@CachePut`.
2. Why should you be careful with caching data that is specific to a user?

### 🔴 Advanced
1. Explain the "Thundering Herd" problem and how Spring Caching addresses it.
2. How does Spring decide which cache provider to use at runtime? (Describe the `CacheResolver` process).

### 🔥 Tricky
1. If you use `@Cacheable` on a method that returns a `Optional<User>`, what gets stored in the cache? (The `Optional` object itself. Note: Spring 4.3+ handles Optional correctly, but older versions might have issues).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You have a global application with nodes in both the US and Europe. A central Redis cache in the US causes slow response times for European users. How do you optimize this? (Use **Tiered Caching**: A small local L1 cache in Europe and the US, with the US Redis as the L2 "Source of Truth").
2. **Performance**: Your app has a memory leak. You notice that the `CaffeineCache` instance in your `UserService` is growing indefinitely. What is the fix? (Set a `maximumSize` or `expireAfterWrite` policy in the cache configuration).

---

## 15. Summary & Key Takeaways

- **Core Insight**: Caching is the **Trade-off** between Accuracy and Speed.
- **Architect Mindset**: Every cache entry is a "Debt" you must eventually repay with an Invalidation.
- **Production Reminder**: Monitor your **Hit Ratio**. A cache you aren't hitting is just an expensive memory leak.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 3: Chapter 17**

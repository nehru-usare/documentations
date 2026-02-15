# API Composition Pattern (Aggregator)

> **Part 6: Enterprise & Microservices Patterns**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Data Joiner

---

## 1. Problem Statement

### The Chatty Frontend
You have a Product Page.
1.  **Product Info**: `GET /products/1` (Product Service)
2.  **Reviews**: `GET /reviews?id=1` (Review Service)
3.  **Inventory**: `GET /inventory/1` (Inventory Service)
4.  **Recommendations**: `GET /recs?id=1` (AI Service)

**Issue**:
*   Mobile client makes 4 HTTP calls over slow 4G.
*   Latency = Sum of all latencies (Sequential).
*   Complexity in Frontend code.

### The Solution
Use an **API Composer** (or Aggregator Pattern).
Client calls `GET /product-details/1`.
Composer calls all 4 services (Internal Network is fast).
Returns 1 JSON blob.

---

## 2. In-Memory Join
This is effectively an **"Outer Join"** done in memory.

```java
@Service
public class ProductAggregator {

    public FullProductDetails getDetails(String id) {
        // 1. Parallel Execution
        CompletableFuture<Product> pFuture = CompletableFuture.supplyAsync(() -> productService.get(id));
        CompletableFuture<Reviews> rFuture = CompletableFuture.supplyAsync(() -> reviewService.get(id));
        CompletableFuture<Stock> iFuture = CompletableFuture.supplyAsync(() -> inventoryService.get(id));

        // 2. Wait for all (or Join)
        CompletableFuture.allOf(pFuture, rFuture, iFuture).join();

        // 3. Assemble
        return new FullProductDetails(
            pFuture.get(),
            rFuture.get(),
            iFuture.get()
        );
    }
}
```

---

## 3. Partial Failure Handling
What if `ReviewService` is down?
*   **Option A**: Fail the whole request (500 Error). **(Bad UX)**.
*   **Option B**: Return Partial Data. `reviews: null` or `reviews: []`. **(Resilient UX)**.

```java
CompletableFuture<Reviews> rFuture = CompletableFuture
    .supplyAsync(() -> reviewService.get(id))
    .exceptionally(ex -> new Reviews()); // Return empty on fail
```

---

## 4. Architect Takeaway
*   **Use GraphQL**: If you have many such aggregation needs, GraphQL is the modern, standardized API Composition pattern.
*   **Backend for Frontend (BFF)**: If Mobile needs minimal data but Desktop needs full data, create specific Composers for each (BFF Pattern).

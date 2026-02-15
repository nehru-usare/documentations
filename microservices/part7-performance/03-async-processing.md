# 03. Async Processing (Batching, Reactive Streams)

> **Part 7: Performance & Scalability**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Senior Developer)  
> **Status:** High Throughput

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand Blocking I/O vs Non-Blocking I/O. |
| **Developer** | Use `CompletableFuture` to run tasks in parallel. |
| **Architect** | Design a Reactive System using Spring WebFlux. |

---

## 1. Why This Topic Exists

### The Thread Limit
Tomcat has 200 threads.
If 200 requests enforce a "Sleep 1s", the 201st request waits.
**Blocking I/O kills scalability.**

### The Solution: Async
Release the thread while waiting for I/O (DB, HTTP).
When data arrives, call a callback.
*   *Result*: 1 thread can handle 10,000 requests.

---

## 3. Core Concepts (🟢 Beginner Level)

### Blocking vs Non-Blocking
*   **Blocking (Servlet)**: `User u = db.find(1);` (Thread Stuck).
*   **Non-Blocking (Reactive)**: `db.find(1).subscribe(u -> print(u));` (Thread Free).

### Batching
Network roundtrips are expensive.
*   *Bad*: Loop 100 times, insert 1 row. (100 msgs).
*   *Good*: Send 1 message with 100 rows. (1 msg).

---

## 4. Developer Deep Dive (🟡 Professional Level)

### CompletableFuture (Java 8+)
Running tasks in parallel.

```java
CompletableFuture<User> userCtx = CompletableFuture.supplyAsync(() -> api.getUser(1));
CompletableFuture<Order> orderCtx = CompletableFuture.supplyAsync(() -> db.getOrder(1));

// Wait for both
CompletableFuture.allOf(userCtx, orderCtx).join();
```

### Batching with Spring Data JPA
Enable JDBC Batching.

```yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 50
          order_inserts: true
```

*   *Effect*: `repo.saveAll(list)` sends 1 SQL statement instead of 50.

---

## 5. Internal Mechanics (🔴 Architect Level)

### Reactive Streams (Project Reactor)
The Standard: `Publisher` -> `Subscriber`.
*   **Mono**: 0 or 1 item.
*   **Flux**: 0 to N items.
*   **Backpressure**: The Subscriber tells Publisher: "I can only handle 5 items right now".

### Event Loop (Netty)
Single Threaded loop.
*   **Don't Block**: Never call `Thread.sleep` or `jdbc.query` inside the Event Loop. You will freeze the entire server.

---

## 14. Summary & Architect Takeaways

*   **Complexity**: Reactive code is hard to debug. Stacktraces are useless ("Reactor Stacktrace Hell").
*   **Use Case**: Use Async for High Concurrency (Chat, Gateway). Use Sync for standard CRUD (easier to maintain).

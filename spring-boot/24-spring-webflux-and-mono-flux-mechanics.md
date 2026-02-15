# Chapter 24: Spring WebFlux and Mono/Flux Mechanics

## 0. Learning Objectives

- **🟢 Beginner**: Understand the basic difference between MVC (Imperative) and WebFlux (Reactive).
- **🟡 Professional**: Master the use of `Mono` and `Flux`, reactive operators (`map`, `flatMap`, `zip`), and non-blocking database access with R2DBC.
- **🔴 Architect**: Deep dive into the "Netty" event loop, understand the **"Backpressure"** mechanism, and design resilient event-driven systems that can handle hundreds of thousands of concurrent connections with minimal memory.

---

## 1. Why This Topic Exists

### Real-World Business Problem
Traditional Spring MVC uses "One Thread Per Request." If you have 1,000 users waiting for a slow API call, you need 1,000 threads. This consumes significant RAM. Even worse, if those 1,000 users are consuming a "Live Feed" (infinite data), your server will quickly run out of threads and crash.

### Technical Limitations Solved
- **Non-blocking I/O**: Instead of a thread "Waiting" for data, it registers a "Callback" and goes off to do other work. When the data arrives, a thread (any thread) picks up the work.
- **Backpressure**: Prevents a fast data producer (like a high-speed DB) from overwhelming a slow consumer (like a mobile 3G connection).

---

## 2. Big Picture Architecture View

WebFlux runs on an **Event Loop** architecture (similar to Node.js), usually powered by **Project Reactor** and **Netty**.

### Interaction with Other Modules
- **WebClient**: The reactive replacement for `RestTemplate`.
- **R2DBC**: The reactive replacement for JDBC.
- **Spring Security Reactive**: Specialized security filters that handle Mono/Flux streams.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Mono**: A stream that emits **zero or one** item (e.g., getting a single User).
- **Flux**: A stream that emits **zero to N** items (e.g., a list of Employees or a live stock price feed).
- **Subscriber**: Nothing happens in WebFlux until someone "Subscribes" to the stream.

### Simple Explanation
Think of a **Subway Sandwich Shop**.
- **MVC (Imperative)**: One worker follows you from start to finish. If the bread is still in the toaster, the worker stands there doing nothing, waiting for it to pop.
- **WebFlux (Reactive)**: One worker takes your order, puts the bread in the toaster, tells a timer to "Ring me when it's done," and immediately starts helping the next customer. When the bell rings, *any* free worker takes the bread out and finishes your sandwich.

### Minimal Working Example
```java
@RestController
public class MyController {
    @GetMapping("/user/{id}")
    public Mono<User> getUser(@PathVariable String id) {
        return userService.findById(id); // Returns a "Promise" of a user
    }
}
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### The Core Operators
- **`map`**: Transform items (e.g., `Mono<User>` -> `Mono<UserDTO>`).
- **`flatMap`**: Asynchronous transformation (e.g., Use a User ID to fetch their Orders from another API).
- **`zip`**: Combine multiple independent streams into one (e.g., Fetch User and Profile simultaneously).

### Error Handling
In Reactive, you don't use `try-catch`. You use operators:
```java
return service.call()
    .onErrorResume(e -> Mono.just(new FallbackData()))
    .timeout(Duration.ofSeconds(2));
```

### Context Propagation
Since threads change constantly, `ThreadLocal` doesn't work. React provides a **`Context`** object that travels along the stream.

---

## 5. Internal Mechanics (🔴 Advanced Level)

### Backpressure (Subscription and Request)
In a reactive stream:
1. The **Subscriber** asks the **Publisher**: "Give me 10 items."
2. The Publisher sends 10 items.
3. If the Subscriber is busy, it doesn't ask for more. This prevents the Subscriber's memory from filling up.

### The Netty Event Loop
WebFlux usually uses 1 thread per CPU core. These threads are "The Event Loop." 
- **Rule**: You must NEVER call a blocking method (like a standard JDBC call or `Thread.sleep`) on an event loop thread. If you block the event loop, the entire server stops.

---

## 6. Under the Hood

### Schedulers
You can decide *where* code runs:
- **`Schedulers.parallel()`**: For CPU intensive tasks.
- **`Schedulers.boundedElastic()`**: For blocking I/O (Legacy DB calls).

---

## 7. Real-World Use Cases

- **Live Dashboards**: Streaming stock prices or server metrics using Server-Sent Events (SSE) or WebSockets.
- **Gateway/Proxy**: Aggregating data from 5 different microservices to build a single response.

---

## 8. Production & Performance Considerations

- **Memory Footprint**: WebFlux handles 10,000+ connections with only a few MB of RAM.
- **Debugging**: Standard stack traces are useless in WebFlux because the threads are always changing. 
  - **Tool**: Use `Hooks.onOperatorDebug()` or the **`reactor-debugger`** agent to get "Virtual Stack Traces."

---

## 9. Architect-Level Best Practices

- **BlockHound**: A library that you should use in tests. It will throw an error if it detects ANY blocking call on a reactive thread.
- **Immutability**: Since data is passed across threads, ensure your objects are immutable to prevent race conditions.
- **StepVerifier**: The ONLY way to test reactive code. It allows you to simulate the passage of time and verify the order of emitted items.

---

## 10. Common Mistakes & Anti-Patterns

- **Calling `.block()`**: The "Sin" of WebFlux. Calling `.block()` inside a controller turns your reactive app back into a (bad) imperative app and can cause deadlocks.
- **Ignoring the Stream**: Forgetting to return a Mono/Flux from a service method. If you don't return it, the subscription never happens, and the code never runs.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`.log()` operator**: Place this in your stream: `service.call().log().subscribe()`. It will print every signal (OnNext, OnError, OnComplete) to the console.
- **"Stuck" Requests**: Usually caused by a missing subscribe or a timeout that was never specified.

---

## 12. Comparisons

### Spring MVC vs. Spring WebFlux
| Feature | Spring MVC | Spring WebFlux |
| :--- | :--- | :--- |
| **Model** | Thread-per-request | Event Loop |
| **I/O** | Blocking | Non-blocking |
| **Latency** | Low (Simple tasks) | Slightly higher (Overhead) |
| **Throughput** | Limited by Thread Pool | High (Millions of conns) |
| **Recommendation** | **Standard Apps / CRUD** | **Streaming / Gateway / Real-time** |

---

## 13. Interview Questions

### 🟢 Basic
1. What is a `Mono`? What is a `Flux`?
2. What does "Backpressure" mean?

### 🟡 Intermediate
1. Why can't you use `ThreadLocal` in a WebFlux application?
2. What is the difference between `map` and `flatMap` in Project Reactor?

### 🔴 Advanced
1. Describe the Netty Event Loop architecture.
2. How do you handle database transactions in a reactive environment? (Describe R2DBC).

### 🔥 Tricky
1. If you have a WebFlux app and you call a standard JDBC repository, what happens to the performance? (The event loop thread will block, and the whole application's throughput will collapse).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You are building a "Chat Application" where 100,000 users are connected simultaneously via WebSockets. Do you use Spring MVC or WebFlux? (WebFlux. MVC would require 100,000 threads, which is impossible. WebFlux handles it with a few hundred threads).
2. **Performance**: Your WebFlux app is using 100% CPU but only processing 10 requests per second. What is likely happening? (Likely an "Infinite Loop" in a reactive stream or a heavy computation being done on the event loop instead of `Schedulers.parallel()`).

---

## 15. Summary & Key Takeaways

- **Core Insight**: Reactive is a **Mindset Shift**. You are no longer writing code that *does* things; you are writing a *recipe* for what should happen when data arrives.
- **Architect Mindset**: Don't use WebFlux unless you need to support massive concurrency or streaming. The "Cognitive Load" is much higher than MVC.
- **Production Reminder**: Monitor your **Event Loop latency**. If one event takes 500ms, your entire server is "Paused" for half a second.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 5: Chapter 24**

# Chapter 38: API Gateway Internals (Spring Cloud Gateway)

## 0. Learning Objectives

- **🟢 Beginner**: Understand the role of an API Gateway and how to route a single request to a backend service.
- **🟡 Professional**: Master the use of **Predicates** and **Filters**, understand how to implement "Global Headers," and learn to use **Spring Cloud Gateway's** reactive model (Netty).
- **🔴 Architect**: Deep dive into the `GatewayFilterChain` internals, understand the "Route Definition Locator" mechanism, and design a scalable Gateway architecture that handles **Rate Limiting**, **CORS**, and **Authentication** at the edge.

---

## 1. Why This Topic Exists

### Real-World Business Problem
In Microservices, you don't want your mobile app to talk directly to 20 different services. 
1. **Security**: You'd have to handle Auth on every single service.
2. **Complexity**: The mobile app would need 20 different URLs.
3. **Cross-Cutting Concerns**: You want a single place to log all traffic and block malicious users.
The **API Gateway** acts as the "Single Entry Point" for all external traffic.

### Technical Limitations Solved
- **Abstraction**: Clients only ever talk to `api.mycompany.com`. The Gateway figures out where to send the request.
- **Unified Policy**: Apply one security/rate-limiting policy to the entire platform.

---

## 2. Big Picture Architecture View

Spring Cloud Gateway is a **Non-blocking Reverse Proxy** built on Top of Spring WebFlux and Project Reactor.

### Interaction with Other Modules
- **Service Discovery (Eureka)**: Gateway uses the registry to find service IPs dynamically.
- **Spring Security**: Usually where JWT validation happens for the entire system.
- **Resilience4j**: Integrated directly to provide circuit-breaking at the edge.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Route**: The basic building block of the gateway (ID, Destination URI, Predicates, and Filters).
- **Predicate**: A "Condition." If the path is `/orders/**`, then this route matches.
- **Filter**: An "Action." Before sending the request, add a header; after getting the response, modify the body.

### Simple Explanation
Think of a **Hotel Receptionist**.
- **The Guest** (User) comes to the front desk.
- **The Predicate**: The receptionist checks the guest's intention: "Are you here for a room (Path: `/rooms`) or for the gym (Path: `/gym`)?"
- **The Filter**: Before giving the room key, the receptionist **Adds a Filter** (Checks your ID, takes a deposit).
- **The Destination**: The receptionist tells you exactly where to go.

### Minimal Working Example
`application.yml`:
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-route
          uri: lb://ORDER-SERVICE # Uses Load Balancer
          predicates:
            - Path=/orders/**
          filters:
            - AddRequestHeader=X-Gateway, SpringCloud
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Reactive Architecture (Netty)
Unlike the legacy Netflix Zuul, Spring Cloud Gateway is **Reactive**. It can handle 10,000+ concurrent requests using only a few threads because it doesn't block while waiting for the backend service.

### Powerful Predicates
You can route based on more than just the path:
- **`Header=X-Beta-User, true`**: Send beta users to a different version of the service.
- **`Method=GET`**: Only route GET requests.
- **`Before/After=2023-12-01T...`**: Route traffic only after a specific date (useful for marketing launches).

### Rate Limiting with Redis
```yaml
filters:
  - name: RequestRateLimiter
    args:
      redis-rate-limiter.replenishRate: 10
      redis-rate-limiter.burstCapacity: 20
```
This protects your backend from being overwhelmed by a "DDoS" or a runaway script.

---

## 5. Internal Mechanics (🔴 Advanced Level)

### GatewayFilterChain
When a request matches a route:
1. The `RoutePredicateHandlerMapping` identifies the route.
2. It creates a `FilteringWebHandler`.
3. A **Chain** of `GlobalFilters` and `GatewayFilters` is built.
4. Each filter executes `Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain)`.
**Architect Note**: The order matters! Security filters should always run first.

### Custom Filter Implementation
```java
@Component
public class MyCustomFilter extends AbstractGatewayFilterFactory<MyCustomFilter.Config> {
    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            // Pre-processing
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                // Post-processing
            }));
        };
    }
}
```

---

## 6. Under the Hood

### Netty Event Loop
The Gateway usually runs on **Netty**. It uses a small number of threads to listen for incoming connections. When a request arrives, it is processed asynchronously. This is why you must NEVER use blocking libraries (like standard JDBC) inside a Gateway filter.

---

## 7. Real-World Use Cases

- **Authentication Consolidation**: Validating the JWT at the Gateway and stripping the secret token before passing the request to internal (and now implicitly trusted) microservices.
- **Path Stripping**: Mapping `api.com/v1/user/1` to an internal service that only expects `/user/1`.

---

## 8. Production & Performance Considerations

- **Heap Usage**: Monitoring memory is critical. If you modify large request bodies (e.g., File Uploads) in a Gateway filter, you can quickly hit an `OutOfMemoryError`.
- **Latency**: Every filter adds a few microseconds. Keep your filter chain as short as possible.

---

## 9. Architect-Level Best Practices

- **Zero-Trust**: Do not just "Strip" the authentication. Pass a "User Context" header (like `X-User-Id`) to internal services so they know who is calling.
- **Global Logging**: Implement a `GlobalFilter` that generates a **Trace ID** and adds it to the logging MDC for every single request entering the system.
- **Isolation**: Don't put business logic in the Gateway. The Gateway should only handle **Routing**, **Security**, and **Observability**.

---

## 10. Common Mistakes & Anti-Patterns

- **Blocking Calls**: Calling a database inside a filter using standard JPA. This will block the Netty Event Loop and crash the entire Gateway.
- **Using Zuul 1.0**: It's dead. Don't use it for new projects. It is blocking and significantly slower than Spring Cloud Gateway.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`404 NOT FOUND`**: Usually means your `Path` predicate doesn't match the incoming URL. Check for trailing slashes!
- **`503 SERVICE UNAVAILABLE`**: The Gateway found the route, but the backend service is not registered in Eureka or is down.

---

## 12. Comparisons

### API Gateway vs. Load Balancer (F5/Nginx)
| Feature | API Gateway (Spring) | Load Balancer (Nginx) |
| :--- | :--- | :--- |
| **Awareness** | Layer 7 (Application) | Layer 4/7 (Network) |
| **Logic** | Java + SpEL + Filter code | C/Lua/Conf files |
| **Customization**| Infinite (Java) | Limited |
| **Recommendation** | **Complex microservice logic** | **Simple high-traffic routing**|

---

## 13. Interview Questions

### 🟢 Basic
1. What is an API Gateway?
2. Difference between a Predicate and a Filter?

### 🟡 Intermediate
1. Why is Spring Cloud Gateway built on WebFlux instead of MVC?
2. How do you implement Rate Limiting in the Gateway?

### 🔴 Advanced
1. Explain the `GatewayFilterChain` and how filters are executed.
2. How do you handle CORS (Cross-Origin Resource Sharing) at the Gateway level?

### 🔥 Tricky
1. If I have a 500MB file upload, does the Gateway load the whole file into RAM? (Only if you have a filter that reads the body. By default, it "Streams" the data, which is highly efficient).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You have an old Mobile App that expects `api.com/users` and a new Mobile App that expects `api.com/v2/users`. Both need to point to the same backend. How do you solve this? (Use **Path Rewriting**. Configure two routes in the Gateway. Route 1 matches `/users` and forwards to the service. Route 2 matches `/v2/users`, uses the `RewritePath` filter to change it back to `/users`, and forwards to the same service).
2. **Performance**: Your Gateway is running at 100% CPU. You have 50 filters. What is the first thing you investigate? (Investigate the complexity of the Predicates and Filters. Check for any "Blocking" code that was accidentally added. Use a profiler like YourKit or JProfiler to see which filter is taking the most CPU time).

---

## 15. Summary & Key Takeaways

- **Core Insight**: The Gateway is the **Shield** and **Compass** of your microservices.
- **Architect Mindset**: Keep it simple. A Gateway that is too complex becomes a "Distributed Monolith" that is hard to maintain.
- **Production Reminder**: Monitor your **Gateway Latency**. It is the baseline for all user experiences.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 8: Chapter 38**

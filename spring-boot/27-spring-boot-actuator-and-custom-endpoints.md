# Chapter 27: Spring Boot Actuator and Custom Endpoints

## 0. Learning Objectives

- **🟢 Beginner**: Understand the role of Spring Boot Actuator and how to enable basic endpoints like `/health` and `/info`.
- **🟡 Professional**: Master the configuration of endpoint exposure, understanding the "Management Port," and creating custom `@Endpoint` beans.
- **🔴 Architect**: Deep dive into the `EndpointDiscoverer` internals, understand the performance impact of frequent metrics polling, and design customized **HealthIndicators** to check complex downstream dependencies like Legacy APIs or custom Message Queues.

---

## 1. Why This Topic Exists

### Real-World Business Problem
When your app is running in production, it's a "Black Box." If it starts slowing down or behaving weirdly, you need a way to look inside without restarting the server. You need to know: Is the DB connection pool full? What is the current CPU usage? How many users are logged in? 

### Technical Limitations Solved
- **Introspection**: Actuator provides a standardized way to pull internal diagnostics from a running JVM over HTTP or JMX.
- **Production Readiness**: It bridges the gap between "Code that works on my machine" and "Software that can be managed by an SRE team."

---

## 2. Big Picture Architecture View

Actuator is a **Separate Management Layer** that sits alongside your web application.

### Interaction with Other Modules
- **Security**: Actuator endpoints MUST be secured differently than your public APIs.
- **Micrometer**: Actuator uses Micrometer to collect and format the metrics it exposes.
- **Kubernetes**: Uses `/health` for Liveness and Readiness probes.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Actuator**: A set of production-ready features for monitoring and managing your application.
- **Endpoint**: A specific diagnostic URL (e.g., `/actuator/metrics`).

### Simple Explanation
Think of a **Car's Dashboard**.
- Your **App** is the engine and wheels that get you to the destination.
- **Actuator** is the dashboard. It doesn't help you drive, but it tells you your **speed** (metrics), your **fuel level** (DB connections), and if the **engine is overheating** (heap usage). Without a dashboard, you're driving blind.

### Minimal Working Example
Adding the dependency:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
Enabling all endpoints (Dev only!):
```properties
management.endpoints.web.exposure.include=*
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Essential Endpoints
1. **`/health`**: The most important. Returns "UP" or "DOWN".
2. **`/metrics`**: Aggregated data like `http.server.requests` or `jvm.memory.used`.
3. **`/loggers`**: View and CHANGE log levels at runtime without restarting.
4. **`/env`**: See the environment properties (Passwords are masked).
5. **`/httpexchanges`**: List of the last 100 HTTP requests (Very useful for debugging flaky APIs).

### Security Configuration
Never expose `/actuator/*` to the public internet. Use a separate port:
```properties
management.server.port=9090
```
Then, use Spring Security to require individual roles for management endpoints.

---

## 5. Internal Mechanics (🔴 Advanced Level)

### Writing a Custom Endpoint
You can expose any Java bean as a management endpoint.
```java
@Component
@Endpoint(id = "inventory")
public class InventoryEndpoint {
    @ReadOperation
    public Map<String, Object> getInventory() {
        return Map.of("items", 500, "status", "STABLE");
    }
}
```
- **`@ReadOperation`**: Maps to HTTP GET.
- **`@WriteOperation`**: Maps to HTTP POST.
- **`@DeleteOperation`**: Maps to HTTP DELETE.

### Custom HealthIndicators
If you have a custom component (e.g., a file system mount), you must write a `HealthIndicator` so the server reports "DOWN" if the mount fails.
```java
@Component
public class NasHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        if (nasIsConnected()) return Health.up().build();
        return Health.down().withDetail("NAS", "Disconnected").build();
    }
}
```

---

## 6. Under the Hood

### EndpointDiscoverer
Spring Boot uses an `EndpointDiscoverer` at startup. It scans the context for `@Endpoint` or `@WebEndpoint` annotations and dynamically maps them to the `/actuator` path. This is why you don't need a Controller for actuator endpoints.

---

## 7. Real-World Use Cases

- **Zero-Downtime Patching**: Using the `/loggers` endpoint to turn on `DEBUG` logging for a specific faulty class in production for 10 minutes, then turning it back to `INFO`.
- **Dependency Analytics**: Using the `/beans` endpoint to verify if a specific bean was initialized correctly in a complex multi-profile environment.

---

## 8. Production & Performance Considerations

- **Sensitive Data Leak**: The `/env` and `/configprops` endpoints can leak API keys. Ensure `management.endpoint.env.show-values` is set to `never` or `when_authorized`.
- **Metric Cardinality**: Don't create metrics with millions of unique tags (like "user_id"). This will cause the `/metrics` endpoint to consume gigabytes of RAM.

---

## 9. Architect-Level Best Practices

- **Use the Web/JMX distinction**: Expose heavy metrics via JMX (for internal monitoring tools) and only expose light health checks via Web (HTTP).
- **Proactive Health Checks**: Don't just check if a DB is "Alive." Check if the **latency** is acceptable. A DB that takes 30 seconds to respond is functionally "Down" for your users.
- **Integration with CI/CD**: Use the `/info` endpoint to display the `Git Commit ID` and `Build Time`. This helps developers know exactly which version of the code is running in which environment.

---

## 10. Common Mistakes & Anti-Patterns

- **Exposing everything in Prod**: `exposure.include=*` in production is a critical security vulnerability. 
- **Blocking in Health Checks**: A `HealthIndicator` should be FAST. If your health check takes 5 seconds, Kubernetes will think the app is slow/dead and keep restarting it.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`404 Not Found`**: Ensure the endpoint is both **Enabled** (on by default) and **Exposed** (off by default for everything except health/info).
- **`Health is UP but DB is DOWN`**: Check if you have enabled the DB health check explicitly: `management.health.db.enabled=true`.

---

## 12. Comparisons

### Actuator vs. Custom Controllers
| Feature | Actuator Endpoints | Custom RestControllers |
| :--- | :--- | :--- |
| **Effort** | Zero (Built-in) | Manual coding |
| **Standards** | Compatible with Grafana/SRE tools | Custom format |
| **Exposure** | JMX and HTTP | HTTP only |
| **Recommendation** | **Standard diagnostics** | **Business-specific dashboards** |

---

## 13. Interview Questions

### 🟢 Basic
1. What is Spring Boot Actuator?
2. How do you view the list of all available Actuator endpoints? (/actuator).

### 🟡 Intermediate
1. Difference between enabling and exposing an endpoint?
2. How do you hide sensitive details in the `/health` endpoint?

### 🔴 Advanced
1. How do you create a custom `HealthIndicator`?
2. Describe how `EndpointDiscoverer` works internally.

### 🔥 Tricky
1. If your app is at 100% CPU, will the Actuator `/health` endpoint still respond? (Only if you have configured a dedicated "Management Thread Pool" so the main app threads don't block the diagnostics).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: Your team has 50 microservices. The DevOps team wants a single dashboard to see if any service is down. How do you implement this using Actuator? (Expose `/health` and `/info` on all services and use **Spring Boot Admin** or **Prometheus** to aggregate the status).
2. **Security**: Your security auditor says that exposing the full list of Spring Beans via `/actuator/beans` is a security risk as it reveals the internal architecture of the app. How do you restrict access to this specific endpoint to only "Senior Devs"? (Configure Spring Security to restrict the `/actuator/beans` path to `hasRole('ADMIN')` while keeping `/health` public).

---

## 15. Summary & Key Takeaways

- **Core Insight**: Observability is a **First-Class Requirement**.
- **Architect Mindset**: Instrument everything worth measuring. If it moves, check its pulse.
- **Production Reminder**: **Secure your endpoints.** An open actuator is a gift to hackers.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 6: Chapter 27**

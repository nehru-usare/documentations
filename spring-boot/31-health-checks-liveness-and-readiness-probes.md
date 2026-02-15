# Chapter 31: Health Checks, Liveness, and Readiness Probes

## 0. Learning Objectives

- **🟢 Beginner**: Understand what a "Health Check" is and why Kubernetes needs them.
- **🟡 Professional**: Master the difference between **Liveness** and **Readiness** probes, and how to configure them in `application.properties`.
- **🔴 Architect**: Deep dive into the `AvailabilityChangeEvent` internals, understand the "Graceful Shutdown" mechanics, and design a custom "Cloud-Native" startup sequence that handles slow legacy dependencies without causing deployment "Restart Loops."

---

## 1. Why This Topic Exists

### Real-World Business Problem
In a modern cloud environment (like Kubernetes), containers are fragile. A server might be "Running" but its database connection is broken. Or it might be so overwhelmed with traffic that it can't respond to new users. If the cloud platform can't see these issues, it will keep sending traffic to a "Dead" service, causing user errors.

### Technical Limitations Solved
- **Automated Self-Healing**: Probes allow the infrastructure to automatically restart a "Frozen" app or stop sending traffic to a "Busy" app.
- **Zero-Downtime Deployment**: Ensures that a new version of your app only receives traffic once it is *actually* ready to work.

---

## 2. Big Picture Architecture View

Probes are the **Communication Protocol** between your Spring Boot application and the **Orchestrator** (Kubernetes/OpenShift).

### Interaction with Other Modules
- **Spring Boot Actuator**: Provides the actual HTTP endpoints (`/health/liveness` and `/health/readiness`).
- **Core Engine**: Publishes events (`LivenessState`, `ReadinessState`) that impact the health status.
- **Graceful Shutdown**: Works in tandem with probes to ensure requests finish before the app dies.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Liveness**: "Am I alive?" If this fails, the container is killed and restarted.
- **Readiness**: "Can I handle traffic?" If this fails, the traffic is stopped but the app stays running.

### Simple Explanation
Think of a **Restaurant Cook**.
- **Liveness Check**: Are you awake and breathing? If you pass out on the floor, the manager replaces you. (Restart).
- **Readiness Check**: Is the stove hot and the ingredients ready? If you are still chopping onions, you are alive, but customers shouldn't order yet. You stay in the kitchen, but the waiter doesn't give you orders. (Stopped traffic).

### Minimal Working Example
Enabling Kubernetes-specific probes:
```properties
management.endpoint.health.probes.enabled=true
```
Endpoints created:
- `GET /actuator/health/liveness`
- `GET /actuator/health/readiness`

---

## 4. Developer Deep Dive (🟡 Professional Level)

### When to fail Liveness?
Only if the application is in an unrecoverable state (e.g., Fatal Memory Leak, Deadlock). Don't fail Liveness if the Database is down! (If you do, the app will keep restarting, which won't fix the database).

### When to fail Readiness?
- Port is open but DB is down.
- Cache is still warming up.
- Application is in "Refuse Traffic" mode during a sensitive maintenance task.

### Customizing Readiness with code
```java
@Autowired
ApplicationEventPublisher eventPublisher;

public void startMaintenance() {
    AvailabilityChangeEvent.publish(eventPublisher, this, ReadinessState.REFUSING_TRAFFIC);
}
```

---

## 5. Internal Mechanics (🔴 Advanced Level)

### AvailabilityProviders
Spring Boot 2.3+ introduced the `AvailabilityProvider` abstraction. 
1. **LivenessState**: `CORRECT` or `BROKEN`.
2. **ReadinessState**: `ACCEPTING_TRAFFIC` or `REFUSING_TRAFFIC`.
**Mechanism**: These are internal state machines. Any component in the app can publish an event that changes these states. The `/health` endpoint simply reflects the current state of these providers.

### Graceful Shutdown Flow
```properties
server.shutdown=graceful
spring.lifecycle.timeout-per-shutdown-phase=30s
```
1. Kubernetes sends a `SIGTERM` signal.
2. Spring Boot immediately flips **Readiness to REFUSING_TRAFFIC**.
3. Tomcat stops accepting NEW connections.
4. Tomcat waits for existing requests to finish (up to 30 seconds).
5. The process exits.

---

## 6. Under the Hood

### ApplicationAvailability Bean
You can inject this bean anywhere to check the current health status of the application programmatically. This is useful for background jobs that should only run if the app is "Live."

---

## 7. Real-World Use Cases

- **"Warm-up" Logic**: An app that needs to load 500MB of data into memory on start. It remains "Unready" for 60 seconds while loading, preventing users from seeing "Data not found" errors.
- **Circuit Breaker Integration**: If an external API is down, you might want to mark your app as "Unready" to signal your Load Balancer to use a different node.

---

## 8. Production & Performance Considerations

- **Probe Frequency**: In Kubernetes, checking every 1s is too much. 5s-10s is better.
- **Initial Delay**: Don't start checking liveness immediately. Give the JVM 10-20 seconds to start up the JIT compiler and initialize beans.

---

## 9. Architect-Level Best Practices

- **Separate the Probes**: Never use the simple `/health` endpoint for Kubernetes. A "Custom DB Health Check" failing should NOT cause a container restart. Always use the granular `/liveness` and `/readiness` paths.
- **Independence**: A health check should not call an external API. If the external API is slow, your health check will fail, causing a cascading failure in your cluster.
- **Logging on State Change**: Always log a high-priority message when the application changes from `ACCEPTING_TRAFFIC` to `REFUSING_TRAFFIC`. This is your most important diagnostic event.

---

## 10. Common Mistakes & Anti-Patterns

- **Database-dependent Liveness**: The classic "Spring Boot Anti-pattern." If your Liveness check queries the DB and the DB goes down, your entire cluster will start "Death Spiraling" (restarting constantly).
- **Infinite Graceful Shutdown**: Setting a timeout of 10 minutes. Kubernetes will eventually kill you anyway. Match your Spring timeout to your Kubernetes `terminationGracePeriodSeconds`.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`kubectl describe pod`**: Look at the "Events" section. It will say "Liveness probe failed: 503 Service Unavailable" or "Readiness probe failed."
- **Local Testing**: Call `GET localhost:8080/actuator/health/readiness`. If it returns 200, you're ready.

---

## 12. Comparisons

### Liveness vs. Readiness
| Feature | Liveness | Readiness |
| :--- | :--- | :--- |
| **HTTP Path** | `/health/liveness` | `/health/readiness` |
| **Infra Action** | KILL container | STOP traffic |
| **Typical Failure** | Deadlock / OOM | DB Down / Startup |
| **Frequency** | Low (every 30s) | High (every 5s) |
| **Recommendation** | **Keep it minimal** | **Check deps/state** |

---

## 13. Interview Questions

### 🟢 Basic
1. What is the difference between a Liveness and Readiness probe?
2. Which Actuator endpoint provides these probes by default?

### 🟡 Intermediate
1. Why is it dangerous to include a Database check in a Liveness probe?
2. How does "Graceful Shutdown" help in a production environment?

### 🔴 Advanced
1. Describe how `AvailabilityChangeEvent` works in Spring Boot.
2. How do you programmatically change the readiness state of an application during a background migration?

### 🔥 Tricky
1. If the liveness probe fails, does the browser see an error? (No, the infrastructure kills it before the browser gets a response. The user just sees a "Connection Reset" or "Gateway Timeout").

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You have a cluster of 50 pods. You are deploying a new version. Suddenly, 49 pods are marked "Unready" because the Liquibase migration is locking the database. What went wrong? (Individual pods shouldn't check for migration status in readiness. One pod should handle the migration, while others wait for a "DB Ready" signal without failing probes).
2. **Performance**: Your Liveness probe is causing high CPU usage. Every 1 second, it re-calculates a complex checksum. How do you fix it? (Cache the health result for 5 seconds or move the expensive check to a background thread that simply updates a "Health Flag").

---

## 15. Summary & Key Takeaways

- **Core Insight**: Probes are the **Communication Bridge**.
- **Architect Mindset**: Be conservative with Liveness; be honest with Readiness.
- **Production Reminder**: A pod that is "Unready" is still a pod you are paying for. Optimize your startup time.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 6: Chapter 31**

# 03. Centralized Configuration (Spring Cloud Config)

> **Part 2: Spring Cloud Core**  
> **Difficulty:** ⭐⭐⭐ (Developer)  
> **Status:** Standard Practice

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand the separation of Config and Code. |
| **Developer** | Set up a Config Server backed by Git and refresh beans dynamically. |
| **Architect** | Design a secure, high-availability configuration strategy. |

---

## 1. Why This Topic Exists

### The Build Cycle Problem
In a traditional app, `application.properties` is inside the JAR.
Scenario: You need to change the logging level from `INFO` to `DEBUG`.
*   **Old Way**: Edit code -> Commit -> Build JAR -> Deploy. (Take 20 mins).
*   **Desired Way**: Edit Config -> Refresh. (Takes 2 seconds).

### The Security Problem
You don't want DB passwords in your main source code repo.

---

## 2. Big Picture Architecture View

```mermaid
graph LR
    Dev[Developer] -->|Push| Git[Git Repo (Config)]
    ConfigServer[Config Server] -->|Pull| Git
    
    ServiceA -->|1. Startup| ConfigServer
    ConfigServer -->|2. Return JSON| ServiceA
    
    Actuator[Actuator /refresh] -->|3. POST| ServiceA
    ServiceA -->|4. Re-fetch| ConfigServer
```

---

## 3. Core Concepts (🟢 Beginner Level)

### Config Server
A Spring Boot application that speaks "Git" on the backend and "JSON" on the frontend.
It serves config based on: `{application}/{profile}/{label}`.

### Config Client
Microservices that fetch their configuration from the Server at startup (before the main Application Context loads).

---

## 4. Developer Deep Dive (🟡 Professional Level)

### 1. Spring Cloud Config Server
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {}
```
*`application.yml`*:
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/myorg/config-repo
```

### 2. Spring Cloud Config Client
*Dependency*: `spring-cloud-starter-config`.
*`bootstrap.yml` (Legacy)* or `application.yml`:
```yaml
spring:
  config:
    import: "optional:configserver:http://localhost:8888"
  application:
    name: order-service
```

### 3. @RefreshScope
Beans that can update their private fields at runtime.
```java
@RestController
@RefreshScope
public class MessageController {
    @Value("${welcome.message}")
    private String message; // Updates when /actuator/refresh is called
}
```

---

## 5. Internal Mechanics (🔴 Architect Level)

### The Bootstrap Context (Legacy)
Historically, Spring Cloud used a parent context (`bootstrap.yaml`) to fetch config before the main context.
New versions use `spring.config.import` in `application.yaml` to achieve the same result cleanly.

### High Availability
The Config Server is a **Single Point of Failure** (during startup).
If Config Server is down, services cannot start.
*   **Fix**: Run Config Server in a cluster (behind a Load Balancer).
*   **Failover**: Clients can retry startup connection (`spring.cloud.config.fail-fast=true` + Retry logic).

---

## 6. Production & Failure Scenarios

### Scenario: Git Repo is Down
*   **Impact**: Config Server cannot fetch latest.
*   **Behavior**: Config Server usually clones the repo to local `/tmp`. It will serve the last known local copy.

### Scenario: Dynamic Refresh Lag
*   **Event**: You change a feature flag. All 100 instances need to update.
*   **Method**: Calling `/refresh` on 100 IPs is mundane.
*   **Fix**: **Spring Cloud Bus**.
    *   Push event to RabbitMQ.
    *   All services listening on RabbitMQ receive event and trigger internal refresh.

---

## 8. Security Considerations

### Encryption at Rest ({cipher})
Don't store plain text passwords in Git.
1.  Encrypt "secret" -> `A8F...`.
2.  Commit to Git: `password: '{cipher}A8F...'`.
3.  Config Server decrypts it in memory before sending to Client.

---

## 9. Architect-Level Best Practices

1.  **Fail Fast**: If Config Server is unreachable, the Service should crash immediately. Don't start with default values (dangerous).
2.  **Versioning**: Use Git tags/Labels for production releases. Lock Prod to a specific commit hash/tag.
3.  **K8s ConfigMaps**: If on K8s, consider using **Spring Cloud Kubernetes** to read from ConfigMaps instead of a separate Config Server. It reduces infrastructure complexity.

---

## 12. Interview Questions

### Basic
1.  What does Spring Cloud Config do?
2.  What is `@RefreshScope`?

### Intermediate
1.  How do you secure passwords in the Config Repo? ({cipher}).
2.  What happens if the Config Server is down when a service starts?

### Architect-Level
1.  Compare Spring Cloud Config vs Kubernetes ConfigMaps.
    *   *Ans*: SCC offers more dynamic refresh + Git history. K8s ConfigMaps are native and uniform across polyglot apps.
2.  Identify the SPOF (Single Point of Failure) in centralized config.

---

## 14. Summary & Architect Takeaways

*   **Config is Code**: Treat it with the same discipline (Code Review, Versioning).
*   **Dynamic is Dangerous**: Changing config at runtime is powerful but risky (can crash 100 services at once). Use Feature Flags instead of raw config changes when possible.

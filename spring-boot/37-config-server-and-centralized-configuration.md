# Chapter 37: Config Server and Centralized Configuration

## 0. Learning Objectives

- **🟢 Beginner**: Understand why we shouldn't store passwords and secrets in `application.properties` inside the JAR.
- **🟡 Professional**: Master the setup of **Spring Cloud Config Server** using a Git backend, and learn how to use `@RefreshScope` to update properties without restarting.
- **🔴 Architect**: Deep dive into the `PropertySourceLocator` internals, understand the "Bootstrap Context" vs. "Main Context" lifecycle, and design a secure configuration architecture using **Vault integration** and symmetrical/asymmetrical encryption.

---

## 1. Why This Topic Exists

### Real-World Business Problem
You have 100 microservices. Each service needs the Database URL. One day, the Database IP changes. 
- **The Bad Way**: You change 100 `application.yml` files, rebuild 100 JARs, and redeploy 100 services. (Takes 2 days).
- **The Architect Way**: You change ONE line in a Git repository. Every service automatically pulls the new IP. (Takes 2 minutes).

### Technical Limitations Solved
- **Secret Management**: Keeps sensitive API keys out of your source code.
- **Environment Parity**: One JAR file can run in Dev, Test, and Prod just by pointing to different "Config Profiles."

---

## 2. Big Picture Architecture View

Centralized Configuration is the **External Source of Truth** for your application's behavior.

### Interaction with Other Modules
- **Git / SVN / Local FS**: The physical storage for properties.
- **Spring Cloud Bus**: Used to "Push" config updates to all services simultaneously.
- **HashiCorp Vault**: Used for high-security secrets (Passwords, Private Keys).

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Config Server**: A standalone service that serves `application.yml` files over HTTP.
- **Config Client**: Your microservice that calls the server at startup to fetch its settings.

### Simple Explanation
Think of a **Remote Controller for a Drone**.
- The **Drone** is your microservice. It has some basic settings, but most of its "Orders" (like Where to go, How high to fly) come from the **Remote Controller** (Config Server). If you want to change the drone's speed, you don't rebuild the drone; you just change the setting on the remote.

### Minimal Working Example
Creating a Config Server:
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
```java
@EnableConfigServer
@SpringBootApplication
public class ConfigServer { ... }
```
`application.yml` for the Server:
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/my-org/config-repo
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Property Overriding Logic
The Config Server follows a hierarchy:
1. `application.yml` (Shared by ALL apps).
2. `{app-name}.yml` (Private to that app).
3. `{app-name}-{profile}.yml` (Environment specific).
**Tip**: If a property exists in both `application.yml` and `order-service-prod.yml`, the specific one (prod) WINS.

### @RefreshScope
By default, Spring beans are created ONCE at startup. If you change a config value, the bean won't see it.
```java
@RefreshScope
@Component
public class DiscountService {
    @Value("${discount.percent}")
    private int percent;
}
```
**Trigger**: To refresh the values, call `POST /actuator/refresh`.

---

## 5. Internal Mechanics (🔴 Advanced Level)

### The Bootstrap Context (Legacy)
In older Spring Cloud versions, there was a separate `bootstrap.yml` which loaded BEFORE anything else to tell the app where the Config Server lives.
- **Modern Way**: Use `spring.config.import=optional:configserver:http://localhost:8888`. This allows Spring Boot 3 to initialize the config client during the standard `EnvironmentPostProcessor` phase.

### Encryption at Rest
Config Server supports `{cipher}` prefixed values.
- **Mechanism**: You provide a `ENCRYPT_KEY` to the Config Server. You store encrypted text in your Git repo. When a client requests the property, the Config Server **Decrypts** it on the fly and sends the plain text over a secure HTTPS connection.

---

## 6. Under the Hood

### Spring Cloud Bus
Refreshing 100 services manually via `/refresh` is a pain. 
- **Solution**: Connect all services to a Message Broker (RabbitMQ/Kafka). Call `/actuator/bus-refresh` on ONE service. It sends a message to the broker, and ALL other services refresh their config automatically.

---

## 7. Real-World Use Cases

- **Feature Toggling**: Enabling "Dark Mode" or "New Checkout Flow" across the entire company by changing one boolean in the config repo.
- **Database Rotation**: Rotating passwords every 30 days without any code changes or restarts.

---

## 8. Production & Performance Considerations

- **Server Availability**: If the Config Server is down, your microservices might fail to start. 
  - **Best Practice**: Use `spring.cloud.config.fail-fast=true` so the app crashes immediately if it can't find its config (rather than starting with bad/default values).
- **Latency**: Fetching config over HTTP at startup adds 1-2 seconds to total startup time.

---

## 9. Architect-Level Best Practices

- **Vault for Secrets**: Don't put DB passwords in Git, even if encrypted. Use **HashiCorp Vault**. Spring Cloud Config has a native "Vault" backend.
- **Versioning**: Treat your config repo like source code. Use **Pull Requests**, **Code Reviews**, and **Tags** for releases.
- **Local Overrides**: Allow developers to override centralized config with local files using `spring.config.allow-override=true` for faster debugging.

---

## 10. Common Mistakes & Anti-Patterns

- **Storing "Binary" data**: Trying to store images or certificates in Config Server. It is designed for **Text-based key-pair** data only.
- **Nested Config Servers**: Trying to make Config Server A pull from Config Server B. Keep your architecture flat and simple.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`/actuator/env`**: Check this endpoint to see where a property is coming from. It will list the "Source" (e.g., `configService:order-service-prod.yml`).
- **`404 Not Found`**: Ensure the `{app-name}` in your client matches the filename in the Git repo.

---

## 12. Comparisons

### Local Properties vs. Git Config Server
| Feature | Local application.yml | Config Server (Git) |
| :--- | :--- | :--- |
| **Change Process** | Rebuild / Redeploy | Commit / Push |
| **Visibility** | Devs see secrets | Permissions based |
| **Scaling** | Hard to sync | Automatic |
| **Recommendation** | **Small Monoliths** | **Professional Microservices** |

---

## 13. Interview Questions

### 🟢 Basic
1. Why should we use a Config Server?
2. What happens to my app if I change a value in the Git config repo? (Nothing, until you trigger a refresh).

### 🟡 Intermediate
1. What does `@RefreshScope` do?
2. How do you secure sensitive values in a Config Server?

### 🔴 Advanced
1. Explain the difference between "Config First" and "Discovery First" bootstrap modes.
2. How does Spring Cloud Bus help with configuration management?

### 🔥 Tricky
1. If I have a `@ConfigurationProperties` bean, do I still need `@RefreshScope`? (Yes, for the bean to be re-instantiated and re-bound to the new environment values).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: Your company has a rule that "No secrets can ever touch the disk." How do you configure Spring Cloud Config? (Configure the Config Server with **Vault Backend**. The server fetches secrets from Vault's memory, decrypts them, and passes them to the client over HTTPS. The Git repo only contains non-sensitive keys).
2. **Performance**: Every time your Jenkins build starts, it tries to connect to the Config Server, but the server is slow. How do you allow the build to use "Default" values if the server is down? (Set `spring.cloud.config.optional=true` or use a "Local" profile that doesn't trigger the Config Client).

---

## 15. Summary & Key Takeaways

- **Core Insight**: Externalize your **Behavior**.
- **Architect Mindset**: Configuration is code. Treat it with the same rigor (Git, PRs, Versioning).
- **Production Reminder**: **Encryption is not optional.** Never store plain text passwords in any file, anywhere.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 8: Chapter 37**

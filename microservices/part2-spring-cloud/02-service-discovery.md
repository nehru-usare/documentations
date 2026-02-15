# 02. Service Discovery (Eureka / Kubernetes)

> **Part 2: Spring Cloud Core**  
> **Difficulty:** ⭐⭐⭐ (Developer)  
> **Status:** Production-Grade

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand why we can't hardcode IP addresses. |
| **Developer** | Implement Netflix Eureka Server and Client. |
| **Architect** | Compare Client-Side Discovery (Eureka) vs Server-Side Discovery (K8s). |

---

## 1. Why This Topic Exists

### The Cloud Problem
In a physical data center, servers had static IPs.
In the Cloud, servers are ephemeral. Autoscaling groups create and destroy instances. **IP addresses change constantly.**

### The Solution: The Phonebook
We need a dynamic "Phonebook" where services register themselves ("I am User Service, my IP is 10.0.0.5").
Other services look up the Phonebook to make calls.

---

## 2. Big Picture Architecture View

```mermaid
graph TD
    ServiceA[Service A (Client)] -->|1. Register| Registry[Eureka Server]
    ServiceB[Service B (Provider)] -->|1. Register| Registry
    
    ServiceA -->|2. Get IPs of 'Service B'| Registry
    Registry -->|3. Return [10.1, 10.2]| ServiceA
    
    ServiceA -->|4. Call 10.1 (Load Balance)| ServiceB
```

---

## 3. Core Concepts (🟢 Beginner Level)

### Service Registry
The database of available service instances.

### Heartbeat
Services send a "Ping" (e.g., every 30s) to the Registry. "I am alive".
If the Registry stops receiving heartbeats (e.g., for 90s), it evicts the instance.

### Client-Side Load Balancing
The Client (Service A) gets the *list* of IPs and chooses one (Round Robin).

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Netflix Eureka
Spring Cloud's default wrapper for Netflix Eureka.

**1. Eureka Server**
```java
@SpringBootApplication
@EnableEurekaServer // The Magic Annotation
public class DiscoveryServer application {}
```
*Config (`application.yml`)*:
```yaml
eureka:
  client:
    register-with-eureka: false # I am the server
    fetch-registry: false
```

**2. Eureka Client**
```java
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {}
```
*Config*:
```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

---

## 5. Internal Mechanics (🔴 Architect Level)

### Eureka Self-Preservation Mode
Eureka is an **AP System** (Availability over Consistency).
*   *Scenario*: Network partition. Eureka Server cannot hear heartbeats from 85% of services.
*   *Logic*: "It's unlikely 85% of services died instantly. It's probably the network."
*   *Action*: **Stop Evicting**. Enter Self-Preservation.
*   *Result*: It returns IPs that might be dead. Client must handle connection errors (Retry).

### Kubernetes Service Discovery (The Modern Way)
K8s has built-in discovery logic (DNS).
*   **Service Name**: `user-service`.
*   **DNS Resolution**: `user-service.default.svc.cluster.local` resolves to the ClusterIP (Virtual IP).
*   **Load Balancing**: K8s `kube-proxy` does the load balancing (Server-Side Discovery).
*   **Spring Cloud Kubernetes**: Integrates `DiscoveryClient` with K8s API.

---

## 6. Production & Failure Scenarios

### Scenario: The Zombie Instance
*   **Event**: Service B crashes hard (kernel panic). Is cannot send a "De-register" signal.
*   **Impact**: Eureka thinks B is alive for 90 seconds (Time to live).
*   **Result**: Service A tries to call B and fails.
*   **Fix**: **Retries** on the client side (Service A).

---

## 9. Architect-Level Best Practices

1.  **Use K8s Discovery if possible**: If you are 100% on Kubernetes, Eureka is redundant complexity. Use Spring Cloud Kubernetes or just raw K8s DNS.
2.  **Eureka High Availability**: Run Eureka in a cluster (Peer Awareness). Node 1 syncs with Node 2.
3.  **Client Cache**: Eureka Clients cache the registry locally. Even if Eureka Server acts down, Services can still talk to each other.

---

## 10. Anti-Patterns & Common Mistakes

### 1. Hardcoding Load Balancer URLs
Using an AWS ELB/ALB between internal services.
*   *Cost*: Extra hop, Extra $$$.
*   *Fix*: Use Client-Side LB (Feign) to talk direct-to-pod.

### 2. Tuning Heartbeats Too Low
Setting heartbeat to 1s to detect failures faster.
*   *Risk*: DDoS your own Registry.

---

## 12. Interview Questions

### Basic
1.  What is Service Discovery?
2.  Why is Client-Side Load Balancing useful? (No bottleneck at LB).

### Intermediate
1.  How does Eureka handle network partitions? (Self-Preservation).
2.  Difference between `@EnableEurekaServer` and `@EnableDiscoveryClient`?

### Advanced
1.  Compare Eureka (Client-Side) vs Kubernetes (Server-Side) discovery.
2.  Why is Eureka considered an AP system in CAP theorem?

### Architect-Level
1.  Design a Service Discovery mechanism for a multi-region deployment. (Region-aware routing).

---

## 14. Summary & Architect Takeaways

*   **Discovery is dynamic**: Never assume an IP is permanent.
*   **Latency**: Discovery implies eventual consistency. A started service might not be visible for 30s.
*   **Resilience**: Clients must handle stale IPs (Retries).

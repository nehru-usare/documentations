# Monolith vs Microservices vs Serverless

> **Part 7: High-Level Architecture**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Great Separation

---

## 0. Learning Objectives
*   **Beginner**: Why "Microservices" is not a magic bullet.
*   **Developer**: Identify when to extract a module into a service.
*   **Architect**: Design a "Modular Monolith" that can evolve into Microservices later.

---

## 1. Problem Context
**Why does this exist?**
Startup: 3 devs. 1 Codebase. **Monolith**. Fast.
Scale: 100 devs. 1 Codebase. Merge Conflicts. Slow Builds.
**Solution**: Break it up?
*   **Microservices**: Independent deployments.
*   **Serverless**: Event-driven functions.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Monolith
*   Single Deployable Unit (`app.jar`).
*   Shared Database.
*   *Pros*: Simple Ops, Fast/Free method calls, transactions work.
*   *Cons*: Coupling, Scale Cube (can't scale *just* the payment module).

### 2. Microservices
*   Many Deployable Units.
*   Separate Databases (Database-per-service).
*   *Pros*: Independent Scaling, Tech Freedom (Java + Go).
*   *Cons*: **Distributed Systems Fallacies**. Network latency. No transactions.

### 3. Serverless (FaaS)
*   No long-running servers. Functions spin up on request.
*   *Pros*: Zero Ops, Pay-per-use.
*   *Cons*: Cold Starts, Stateless constraints.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Migration Path
1.  **Monolith**: Start here.
2.  **Modular Monolith**: Enforce boundaries in code (Java Packages/Modules). Don't let `Order` package import `User` package directly. Use Interfaces.
3.  **Microservices**: Only when Team Size > 20 or specific scaling need arises.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Database-Per-Service
*   The Golden Rule of Microservices.
*   If Service A reads Service B's DB directly, they are **Coupled**.
*   Migration becomes impossible.
*   *Correct*: Service A calls Service B's API to get data.

### 2. Cold Starts (Serverless)
*   AWS Lambda shuts down idle functions.
*   First request triggers "Init": Download Code -> Start Container -> Start Runtime (JVM).
*   *Latency*: 200ms (Node) to 3s (Java).
*   *Fix*: Provisioned Concurrency or SnapStart.

---

## 5. Trade-Off Analysis

| Feature | Monolith | Microservices | Serverless |
| :--- | :--- | :--- | :--- |
| **Development** | Fast | Slow (Contracts) | Medium |
| **Deployment** | Risky (All or nothing) | Safe (Canary) | Instant |
| **Debugging** | Easy (Local debugger) | Hard (Tracing) | Hard (Logs) |
| **Cost** | Low | High (Infra overhead) | Low (if low traffic) |

---

## 6. Scaling Considerations

### The "Death Star" Diagram
*   When you have 1000 microservices, the dependency graph looks like a Death Star.
*   **Service Mesh** (Istio/Linkerd) becomes mandatory to manage traffic, retries, and security.

---

## 7. Failure Scenarios & Recovery

### 1. Cascading Failure
*   Service A calls B, B calls C, C calls D.
*   D fails. C waits. B waits. A waits.
*   Entire system hangs.
*   *Fix*: **Circuit Breakers** and **Timeouts**.

---

## 8. Security Considerations

### 1. Expanded Attack Surface
*   Monolith: 1 Public Port (443). Internal calls are memory.
*   Microservices: 100 Internal Ports.
*   **Zero Trust**: You must encrypt (mTLS) traffic between services. Service A should not trust Service B just because it's inside the VPC.

---

## 9. Performance Considerations

*   **Latency Tax**: Every network hop adds ~2-5ms (Optimistic).
*   A request hitting 10 services adds 50ms of pure overhead.
*   *Optimization*: Async messaging (Fire and forget) to reduce user-perceived latency.

---

## 10. Real Production Lessons

### Amazon Prime Video
*   **Move**: From Serverless/Microservices -> Monolith.
*   **Why**: Monitoring stream quality involves high-frequency data transfer. Moving data between Lambdas/S3 was expensive and slow.
*   **Result**: 90% cost reduction by moving to a Monolith (on EC2/ECS).

---

## 11. Interview Questions

### Basic
1.  What is a Monolith?
2.  Why use Microservices?
3.  What is the "Database per Service" pattern?
4.  What is a Cold Start in Lambda?
5.  What is a "Distributed Monolith"? (Microservices with shared DB).

### Intermediate
1.  Explain the "Strangler Fig" pattern for migration.
2.  How do you handle Distributed Transactions? (Saga).
3.  Why is Logging hard in Microservices? (Need Request ID / Correlation ID).
4.  Serverless vs Kubernetes?
5.  When to use a Modular Monolith?

### Advanced
1.  Design a CI/CD pipeline for 500 microservices. (Independent release trains).
2.  Analyze the cost of FaaS vs Containers for high-throughput workloads. (FaaS is expensive at scale).
3.  How does Service Mesh (Sidecar) impact latency?
4.  Explain "function composition" limitations in Serverless.
5.  Critique "Nano-services" (Anti-pattern).

### Architect-Level
1.  "We have a legacy Monolith. ROI analysis implies rewrite." (Rewrite is usually a mistake. Refactor incrementally).
2.  Architect a Multi-Tenant Serverless SaaS. (Isolation vs Pool).
3.  Evaluate the operational complexity of managing 1000 RDS instances vs 1 Shared Oracle cluster.

---

## 12. Scenario-Based System Design Problems

### 1. Design Uber Migration
*   **Start**: Monolith (Python).
*   **Growth**: Dispatch, Maps, Payments teams blocked each other.
*   **End**: Microservices.
*   **Why**: Organizational scalability, not just technical.

### 2. Design Image Resizer
*   **Req**: User uploads, resize to 5 formats.
*   **Choice**: **Serverless (Lambda)**.
*   **Why**: Burst traffic. Event-driven (S3 Trigger). Stateless task.

---

## 13. Summary & Architect Takeaways

1.  **Monolith First**: Don't start with Microservices. You don't know the domain boundaries yet.
2.  **Conway's Law**: Your architecture will mirror your org chart.
3.  **Complexity Conservation**: You assume complexity. You either build it in code (Monolith) or in infrastructure (Microservices).

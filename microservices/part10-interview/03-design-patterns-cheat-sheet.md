# 03. Design Patterns Cheat Sheet

> **Part 10: Interview & Scenario Kit**  
> **Difficulty:** ⭐⭐⭐ (Architect)  
> **Status:** The Pocket Reference

---

## 🛡️ Resilience Patterns

| Pattern | Problem Solved | Key Tool | Trade-off |
|:---|:---|:---|:---|
| **Circuit Breaker** | Cascading Failures. | Resilience4j | Temporary unavailability in exchange for system stability. |
| **Retry** | Transient Network Glitches. | Resilience4j | Increases latency. Can cause DDoS if infinite. |
| **Bulkhead** | One slow component kills entire app. | Resilience4j | Resources are partitioned. Efficiency loss. |
| **Rate Limiter** | Protecting Service from overload. | Redis / Bucket4j | Rejects legitimate users during spikes. |

---

## 💾 Data & Consistency Patterns

| Pattern | Problem Solved | Key Concepts | Trade-off |
|:---|:---|:---|:---|
| **Saga** | Distributed Transactions across services. | Orchestration vs Choreography. | Eventual Consistency. Complex rollback logic. |
| **CQRS** | Read vs Write scaling. | Command Model vs Query Model. | Complexity. Sync lag between Write and Read DB. |
| **Event Sourcing** | Audit capability & Replay. | Event Log is Source of Truth. | Snapshots needed for performance. |
| **Outbox** | "Dual Write" problem (DB + Kafka). | Transactional Outbox Table. | Duplicate events possible (at-least-once). |

---

## 🧩 Architecture Patterns

| Pattern | Problem Solved | Key Concept | Trade-off |
|:---|:---|:---|:---|
| **Strangler Fig** | Migrating Legacy Monoliths. | Proxy intercepts traffic gradually. | Managing two systems during migration. |
| **BFF** | General API doesn't fit Mobile/Web. | Specific API for Specific Client. | Code duplication across BFFs. |
| **ACL** | Protecting Domain from Legacy mess. | Adapter / Facade / Translator. | Latency. Extra mapping layer. |
| **Sidecar** | Cross-cutting concerns (Logs, mTLS). | Helper container in same Pod. | Resource overhead per Pod. |

---

## 🚀 Performance Patterns

| Pattern | Problem Solved | Key Concept | Trade-off |
|:---|:---|:---|:---|
| **Cache-Aside** | DB Load reduction. | App checks Cache, then DB. | Stale data window. Thundering Herd risk. |
| **Sharding** | Database size limits. | Horizontal partitioning by Key. | No Cross-shard joins. Rebalancing is hard. |
| **Async Request** | Blocking threads kill throughput. | Reactive Streams / Message Queue. | Harder to debug (Async stacktraces). |

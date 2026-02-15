# 04. Polyglot Persistence (SQL vs NoSQL)

> **Part 4: Data & Consistency**  
> **Difficulty:** ⭐⭐⭐ (Architect)  
> **Status:** Standard Practice

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Stop trying to put everything in Oracle. |
| **Developer** | Connect Spring Boot to Mongo, Redis, and Postgres simultaneously. |
| **Architect** | Select the correct database engine for the workload. |

---

## 1. Why This Topic Exists

### The One-Size-Fits-None
*   Trying to store a Social Graph in SQL? -> Recursive Joins (Slow).
*   Trying to store Bank Transactions in Mongo? -> No ACID (Dangerous).
*   **Polyglot Persistence**: Using multiple data storage technologies, selected based on the specific needs of each service.

---

## 3. Core Concepts (The Menu)

| Type | Examples | Best For | Weakness |
|:---|:---|:---|:---|
| **RDBMS** | PostgreSQL, MySQL | Financial, Relations, ACID | Scaling Write (Vertical only) |
| **Document** | MongoDB, Couchbase | User Profiles, Catalog (JSON) | Complex Transactions |
| **Key-Value** | Redis, Memcached | Token Store, Caching, Session | Connectivity (Relations) |
| **Columnar** | Cassandra, HBase | High write volume, Analytics | Ad-hoc queries |
| **Graph** | Neo4j, Amazon Neptune | Social Networks, Fraud Detection | Simple CRUD |
| **Time-Series**| InfluxDB, Prometheus | IoT, Metrics, Logs | Updates/Deletes |

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Spring Data Support
Spring Boot makes Polyglot easy.
*   `spring-boot-starter-data-jpa` -> Entity (Table).
*   `spring-boot-starter-data-mongodb` -> Document (Collection).
*   `spring-boot-starter-data-redis` -> Hash (Key-Value).

You can use all 3 in one project (though ideally separated by Microservice).

---

## 9. Architect-Level Best Practices

1.  **Default to SQL**: If you don't know what you need, use Postgres. It supports JSON (`jsonb`) and is rock solid.
2.  **Operational Overhead**: Every new DB type = New backup strategy, new monitoring, new expertise required. Don't pick Neo4j just for fun.
3.  **Cloud Native**: Use Managed Services (AWS RDS, DynamoDB). Don't host your own Cassandra unless you are Netflix.

---

## 14. Summary & Architect Takeaways

*   **Right tool for the job**: Don't use a hammer for a screw.
*   **Consistency Boundaries**: RDBMS gives strong consistency. NOSQL usually gives eventual. Design accordingly.

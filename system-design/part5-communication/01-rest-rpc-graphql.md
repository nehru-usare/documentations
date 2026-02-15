# REST vs RPC vs GraphQL

> **Part 5: Communication**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** How Services Talk

---

## 0. Learning Objectives
*   **Beginner**: Know the difference between a GET request and a Function Call.
*   **Developer**: Implementing a gRPC Service definition (.proto).
*   **Architect**: Choosing GraphQL for Frontend flexibility vs gRPC for Backend speed.

---

## 1. Problem Context
**Why does this exist?**
Service A needs data from Service B.
*   **Style 1**: "Give me the User Resource." (REST).
*   **Style 2**: "Execute the `getUser()` function." (RPC).
*   **Style 3**: "Give me exactly these 3 fields of the User." (GraphQL).
Each style dictates the coupling, performance, and cacheability of the system.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. REST (Representational State Transfer)
*   **Focus**: Resources (`/users/1`).
*   **Verbs**: HTTP Methods (`GET`, `POST`, `PUT`, `DELETE`).
*   **Format**: Usually JSON.
*   **Pros**: Universal. Cacheable (HTTP Caching).

### 2. RPC (Remote Procedure Call)
*   **Focus**: Actions (`/deleteUser`).
*   **Verbs**: Usually valid HTTP POST (hidden).
*   **Format**: Binary (Protobuf) or JSON (JSON-RPC).
*   **Pros**: High Performance. Strong Typing.

### 3. GraphQL
*   **Focus**: Graph Traversal.
*   **Verbs**: `Query` (Read), `Mutation` (Write).
*   **Pros**: No Over-fetching. Flexible for UI.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Comparison

| Feature | REST | gRPC | GraphQL |
| :--- | :--- | :--- | :--- |
| **Protocol** | HTTP/1.1 or 2 | HTTP/2 | HTTP/1.1 or 2 |
| **Payload** | JSON (Text) | Protobuf (Binary) | JSON (Text) |
| **Browser Support** | Native | Requires gRPC-Web | Client Library needed |
| **Contract** | OpenAPI (Swagger) | .proto file | Schema (.graphql) |

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. The Over-Fetching Problem (REST)
*   **Scenario**: Mobile App needs User's *Name*.
*   **REST**: `GET /users/1` returns Name, Email, Address, History... (50KB).
*   **Waste**: 99% of bandwidth wasted using battery and data.

### 2. The N+1 Problem (GraphQL)
*   **Query**: `Get Users { bestFriend { name } }`.
*   **Execution**:
    1.  Get Users (1 Query). Returns 100 users.
    2.  For EACH user, Get Best Friend (100 Queries).
    *   **Total**: 101 Queries.
*   **Fix**: **DataLader** (Batching).

---

## 5. Trade-Off Analysis

| Strategy | Speed (Latency) | Flexibility | Coupling |
| :--- | :--- | :--- | :--- |
| **gRPC** | ⭐⭐⭐⭐⭐ | ⭐⭐ | High (Shared .proto) |
| **REST** | ⭐⭐⭐ | ⭐⭐⭐ | Medium |
| **GraphQL** | ⭐⭐ | ⭐⭐⭐⭐⭐ | Low (Client decides) |

---

## 6. Scaling Considerations

### Caching
*   **REST**: Easy. `GET /users/1` can be cached by Nginx/CDN automatically.
*   **GraphQL**: Hard. Every query is a `POST` with a unique body. CDNs can't cache it easily. (Need Persisted Queries).
*   **gRPC**: No HTTP caching.

---

## 7. Failure Scenarios & Recovery

### 1. Schema Breaking
*   **REST**: Changing JSON field name `userName` -> `name` breaks client.
*   **gRPC**: Changing Field ID (`int32 id = 1` -> `int32 id = 2`) breaks binary compatibility.
*   **GraphQL**: Deprecation workflow (`@deprecated`) allows smooth transition.

---

## 8. Security Considerations

### 1. GraphQL DoS
*   **Attack**: `query { user { friend { user { friend { ... } } } } }`. (Deeply nested cyclical query).
*   **Result**: Server crashes parsing depth 1000.
*   **Fix**: Query Depth Limiting or Complexity Scoring.

---

## 9. Performance Considerations

*   **Serialization**:
    *   JSON: CPU heavy (String parsing).
    *   Protobuf: CPU light (Bit shifting). 10x faster.
*   **Network**:
    *   gRPC uses HTTP/2 Multiplexing (One connection, many requests).

---

## 10. Real Production Lessons

### Netflix's Falcor (Pre-GraphQL)
*   **Problem**: Mobile app latency.
*   **Solution**: "One Model Everywhere".
*   **Lesson**: The Frontend always wants a Graph. If you don't give them GraphQL, they will build an ad-hoc Aggregator (BFF).

---

## 11. Interview Questions

### Basic
1.  What does REST stand for?
2.  Difference between GET and POST.
3.  What is a "Resource" in REST?
4.  Why is JSON popular?
5.  What is gRPC?

### Intermediate
1.  Explain the concept of HATEOAS. (Nobody uses it, but good to know).
2.  Why is gRPC faster than REST? (Binary + HTTP/2).
3.  What is the N+1 problem in GraphQL?
4.  Can you use gRPC in the Browser? (Not directly, need Proxy).
5.  How do you version a REST API? (URL vs Header).

### Advanced
1.  Design a GraphQL Schema for a Social Network.
2.  Analyze the overhead of HTTP/1.1 headers vs HTTP/2 header compression (HPACK).
3.  How does "Schema Stitching" or "Federation" work in GraphQL?
4.  Implement a bi-directional streaming API using gRPC.
5.  Critique the Richardson Maturity Model.

### Architect-Level
1.  "We have a Microservices Backend and a Mobile App." Architect the Communication layer. (gRPC internal, GraphQL public).
2.  Design a strategy to migrate a 100-endpoint REST API to GraphQL incrementally.
3.  Evaluate the use of JSON-API spec (standardized REST) vs GraphQL.

---

## 12. Scenario-Based System Design Problems

### 1. Design Internal Microservices
*   **Req**: Ultra-low latency. 10,000 requests/sec.
*   **Choice**: **gRPC**.
*   **Why**: Binary, Multiplexing, Strong Contracts.

### 2. Design Public API (Github)
*   **Req**: External devs need flexibility.
*   **Choice**: **GraphQL** (or REST).
*   **Why**: You don't know what data the 3rd party needs. Let them ask.

---

## 13. Summary & Architect Takeaways

1.  **Internal = gRPC**: Don't use REST for Service-to-Service if you control both sides.
2.  **External = GraphQL/REST**: Don't force external users to use binary Protobufs.
3.  **Complexity Cost**: GraphQL is complex to secure and cache. Don't use it for simple CRUD.

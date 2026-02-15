# Glossary of Terms

> **Part 10: Appendix**  
> **Difficulty:** ⭐ (Reference)  
> **Status:** Speak the Language

---

## A
*   **ACID**: Atomicity, Consistency, Isolation, Durability. Prop. of SQL DBs.
*   **Active-Active**: DR strategy where multiple DCs actively serve traffic.
*   **Anycast**: Routing logic where 1 IP maps to multiple physical locations.

## B
*   **Backpressure**: System slowing down producer when consumer is overwhelmed.
*   **Bloom Filter**: Probabilistic structure to test set membership.
*   **Blue/Green**: Deployment strategy. 2 envs. Switch Load Balancer.

## C
*   **CAP Theorem**: Consistency, Availability, Partition Tolerance. Pick 2.
*   **CDN**: Content Delivery Network. Edge caching.
*   **Consistent Hashing**: Distributed hashing scheme using a ring. Minimizes rebalancing.
*   **CQRS**: Command Query Responsibility Segregation. Split Read/Write models.
*   **CRDT**: Conflict-free Replicated Data Type. Math structure for eventual consistency.

## D
*   **Database Sharding**: Partitioning DB across multiple servers.
*   **Dead Letter Queue (DLQ)**: Queue where failed messages are parked.
*   **Distributed Tracing**: Tracking a request across microservices (Zipkin/Jaeger).

## E
*   **Eventual Consistency**: Data will become consistent *eventually* (Gossip/Async).
*   **Event Sourcing**: Storing state as a sequence of events.

## G
*   **Gossip Protocol**: Epidemic communication for cluster state (Cassandra).
*   **gRPC**: Google RPC. Uses Protobuf and HTTP/2.

## H
*   **Heartbeat**: Periodic signal to indicate liveness.
*   **Hot Key**: A specific key receiving excessive traffic (Celebrity problem).
*   **Horizontal Scaling**: Adding more machines.

## I
*   **Idempotency**: Operation that can be applied multiple times without changing result beyond first.
*   **Isolation Levels**: Read Uncommitted, Read Committed, Repeatable Read, Serializable.

## L
*   **Load Balancer**: Distributes traffic. L4 (TCP) or L7 (HTTP).
*   **Long Polling**: Emulating push by holding HTTP connection open.

## M
*   **Message Queue**: Async buffer (Kafka/RabbitMQ).
*   **Microservices**: Small, autonomous services.

## O
*   **OSI Model**: 7 layers of networking (Phy, Data, Net, Trans, Sess, Pres, App).

## P
*   **PACELC**: Extension of CAP. If Partition (P), choose A or C. Else (E), choose Latency (L) or Consistency (C).
*   **Partitioning**: See Sharding.
*   **Publisher/Subscriber**: Messaging pattern.

## Q
*   **Quorum**: Minimum votes required to commit data (W+R > N).

## R
*   **Rate Limiting**: Throttling requests to protect system.
*   **Replication**: Copying data to multiple nodes.
*   **Reverse Proxy**: Server sitting in front of web servers (Nginx).

## S
*   **Saga Pattern**: Handling distributed transactions via sequence of local transactions.
*   **Service Discovery**: Phonebook for microservices (Eureka/Consul).
*   **Single Point of Failure (SPOF)**: Component whose failure kills the system.
*   **Snowflake ID**: 64-bit distributed ID generation (Time + Machine + Seq).

## T
*   **Thundering Herd**: Many clients retrying simultaneously, causing DDoS.
*   **Throughput**: Rate of processing (Jobs/Sec).

## V
*   **Vertical Scaling**: Adding RAM/CPU to a single machine.
*   **Virtual Node**: Technique in Consistent Hashing to improve balance.

## W
*   **Write-Ahead Log (WAL)**: Append-only log used for durability in DBs.
*   **Write-Through / Write-Back**: Caching strategies.

## Z
*   **Zero Trust**: Security model ensuring no implicit trust inside network.
*   **Zookeeper**: Coordination service for distributed systems.

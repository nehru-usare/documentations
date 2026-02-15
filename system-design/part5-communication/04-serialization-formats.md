# Serialization (JSON vs Protobuf vs Avro)

> **Part 5: Communication**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Byte Whisperer

---

## 0. Learning Objectives
*   **Beginner**: Why sending text (`{"id":1}`) is wasteful.
*   **Developer**: Defining schemas with Protobuf/Avro.
*   **Architect**: Managing Schema Evolution without breaking 100 microservices.

---

## 1. Problem Context
**Why does this exist?**
Objects in Memory (Java Heap) need to be sent over Network (Wire).
You must **Serialize** (Object -> Bytes) and **Deserialize** (Bytes -> Object).
*   **JSON**: Easy 4 Humans. Slow 4 Machines. Big.
*   **Binary**: Hard 4 Humans. Fast 4 Machines. Small.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Human Readable (Text)
*   **JSON**: The King of APIs.
*   **XML**: The Grandfather. Verbose.
*   **YAML**: Config files.

### 2. Binary Optimized
*   **Protobuf (Google)**: RPC focus.
*   **Avro (Hadoop)**: Big Data focus.
*   **MessagePack**: JSON but binary.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Schema vs Schemaless

1.  **Schemaless (JSON)**:
    *   Sender sends keys: `{"name": "Bob", "age": 20}`.
    *   Receiver parses keys.
    *   **Pro**: Flexible.
    *   **Con**: "name" string is sent every time. Redundant.

2.  **Schema-Based (Protobuf)**:
    *   Both sides have `.proto` file. `1: string name`.
    *   Sender sends: `0x0A 0x03 Bob`. (Field 1, Length 3, Value Bob).
    *   **Pro**: Tiny packets.
    *   **Con**: Strict contract.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Schema Evolution
*   **Backward Compatibility**: New Code can read Old Data. (Add optional field).
*   **Forward Compatibility**: Old Code can read New Data. (Ignore unknown fields).
*   **Avro**: Stores Schema in a Registry. Powerful evolution rules.

### 2. ZigZag Encoding
*   How Protobuf makes integers small.
*   `int32` 5 usually takes 4 bytes.
*   Varint acts like 1 byte for small numbers.

---

## 5. Trade-Off Analysis

| Format | Readability | Size | Speed | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **JSON** | ⭐⭐⭐⭐⭐ | ⭐ | ⭐⭐ | Public APIs, Debugging |
| **Protobuf** | ⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Internal Microservices (gRPC) |
| **Avro** | ⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Data Lake (S3), Kafka |

---

## 6. Scaling Considerations

### Network vs CPU
*   **High Traffic**: Use Binary. Saving 50% bandwidth = 50% less cost.
*   **High CPU**: Use Binary. JSON parsing is expensive (Reflection/String creation).

---

## 7. Failure Scenarios & Recovery

### 1. The Broken Registry
*   **Scenario**: Avro relies on Schema Registry.
*   **Event**: Registry goes down.
*   **Result**: Consumers cannot interpret the binary blobs. Total outage.
*   **Fix**: Cache schemas locally on client/consumer.

---

## 8. Security Considerations

### 1. Deserialization Attacks
*   **Java Serialization**: Famous for RCE (Remote Code Execution).
*   **JSON/Proto**: Generally safe as they are data-only, not code.

---

## 9. Performance Considerations

*   **Zero-Copy**: FlatBuffers (used by Facebook) allows accessing data without unpacking the whole object.
*   **Benchmark**: Protobuf is typically 6x faster and 4x smaller than JSON.

---

## 10. Real Production Lessons

### Uber's Switch
*   **Original**: JSON over HTTP.
*   **Problem**: Massive scale. JSON parsing consumed 40% of CPU.
*   **Switch**: 100% to Protobuf (gRPC).
*   **Result**: Saved thousands of cores.

---

## 11. Interview Questions

### Basic
1.  What is Serialization?
2.  Why is JSON larger than Protobuf?
3.  Can I read a Protobuf message in Notepad?
4.  What is a Schema Registry?
5.  Why is XML frowned upon today?

### Intermediate
1.  Explain Backward vs Forward compatibility.
2.  How does MessagePack differ from JSON?
3.  What is "Varint" encoding?
4.  Why use Avro with Kafka?
5.  Cost of String parsing in JSON.

### Advanced
1.  Design a Schema Registry service.
2.  Analyze FlatBuffers vs Protobuf.
3.  How to handle "Unknown Fields" in Schema Evolution (Preserve vs Drop).
4.  Critique using Compression (Gzip) over JSON vs using Binary format. (Binary is better - Gzip burns CPU).
5.  Explain "Columnar Format" (Parquet) relation to serialization.

### Architect-Level
1.  "We have 1PB of CSV files." Architect a migration to Parquet/Avro.
2.  Standardize the serialization strategy for a Polyglot organization (Java, Go, Python).
3.  Design a system that accepts JSON from public but converts to Avro for internal pipeline.

---

## 12. Scenario-Based System Design Problems

### 1. Design Public Weather API
*   **Req**: Easy adoption.
*   **Choice**: **JSON**.
*   **Why**: DX (Developer Experience) > Performance.

### 2. Design Frequency Trading System
*   **Req**: Microsecond latency.
*   **Choice**: **SBE (Simple Binary Encoding)** or Protobuf.
*   **Why**: Every bit counts.

---

## 13. Summary & Architect Takeaways

1.  **JSON availability**: Start with JSON. Optimize to Proto only when Scale requires it.
2.  **Contracts**: Binary formats force you to define a Contract (Schema). This is good for stability.
3.  **Data Lake**: Always use Avro/Parquet for long-term storage (S3).

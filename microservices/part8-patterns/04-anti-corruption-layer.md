# 04. Anti-Corruption Layer (ACL)

> **Part 8: Real-world Architecture Patterns**  
> **Difficulty:** ⭐⭐⭐⭐ (Architect)  
> **Status:** Domain Purity

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand why you shouldn't use `LegacyUserObject` in your Service. |
| **Developer** | Write an Adapter that converts XML to a Java Record. |
| **Architect** | Protect the Integrity of the Bounded Context. |

---

## 1. Why This Topic Exists

### The Poison
You need to talk to a Mainframe.
The Mainframe returns a `CUST_REC_XML` with fields like `C_NM_1` (First Name) and `C_ST_CD` (Status).
*   **Bad**: Using `C_NM_1` everywhere in your code.
*   **Result**: Your clean code becomes polluted with Mainframe naming conventions.

### The Solution: ACL
A border control station.
1.  **Receive**: `CUST_REC_XML`.
2.  **Translate**: Map `C_NM_1` -> `firstName`.
3.  **Forward**: Your domain sees only `Customer` object.

---

## 2. Big Picture Architecture View

```mermaid
graph LR
    subgraph "Legacy System"
        Mainframe[COBOL Core]
    end
    
    subgraph "Your Microservice"
        Service[Domain Logic]
        ACL[Anti-Corruption Layer]
    end
    
    Service -->|getCustomer()| ACL
    ACL -->|SOAP Request| Mainframe
    Mainframe -->|XML Response| ACL
    ACL -->|Customer DTO| Service
```

---

## 3. Core Concepts (🟢 Beginner Level)

### Adapter Pattern vs Facade
*   **Facade**: Simplifies the interface. (Hides complexity).
*   **Adapter**: Translates the interface. (Makes square peg fit round hole).
*   **ACL**: Does BOTH.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Implementation Approaches
1.  **In-Code Class**: A simple `MianframeAdapter` class inside your service.
    *   *Pros*: Fast, easy.
    *   *Cons*: Consumes CPU of your service.
2.  **Standalone Service**: A specific Microservice just for the integration (`mainframe-connector-service`).
    *   *Pros*: Scalable, isolated crash risk.
    *   *Cons*: Network latency.

### Code Example
```java
// INTERNAL Domain Model (Clean)
public record Customer(String id, String name) {}

// INTERNAL Interface
public interface CustomerPort {
    Customer findById(String id);
}

// ADAPTER (Dirty work)
@Service
public class MainframeAdapter implements CustomerPort {
    public Customer findById(String id) {
        var xml = soapClient.call(id); // Returns weird XML
        return new Customer(xml.getC_ID(), xml.getC_NM_1());
    }
}
```

---

## 14. Summary & Architect Takeaways

*   **Isolation**: If the Mainframe changes its XML format, only the `MainframeAdapter` breaks. Your core logic remains untouched.
*   **Cost**: ACLs add code and latency. Only use them if the external model is significantly different/messy.

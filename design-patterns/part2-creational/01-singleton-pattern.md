# Singleton Pattern

> **Part 2: Creational Patterns**  
> **Difficulty:** ⭐ (Beginner)  
> **Status:** Overused & Misused

---

## 0. Learning Objectives

*   **Beginner**: Implement a basic thread-safe Singleton.
*   **Developer**: Understand why Spring Beans are Singletons by default.
*   **Architect**: Know when to use an Enum Singleton (the safest way).

---

## 1. Problem Statement

### The "One and Only" Problem
Some objects effectively "own" a resource. You can't have two of them.
*   **Database Connection Pool**: If you have 2 pools, you might exceed the database connection limit.
*   **Logger**: You want all logs to go to the same file buffer.
*   **Config Manager**: You don't want to parse `application.properties` 50 times.

### What happens if we don't use it?
1.  **Resource Exhaustion**: Opening 1000 DB connections because every request created a new `ConnectionManager`.
2.  **Inconsistency**: Config A says "Timeout=500", Config B says "Timeout=1000".

---

## 2. Real-World Analogy

**The Company CEO**
*   There is only **one** CEO.
*   If you call "GetCEO()", you get the same person every time.
*   It doesn't make sense to "instantiate" a new CEO for every meeting.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Ensure a class has only one instance and provide a global point of access to it.

### Participants
1.  **Singleton Class**: Has a `private static` instance and a `private constructor`.
2.  **Client**: Calls `Singleton.getInstance()`.

---

## 4. UML-Style Structure

*   **Class**: `Singleton`
    *   `- static instance: Singleton`
    *   `- Singleton()` (Private!)
    *   `+ static getInstance(): Singleton`

---

## 5. Java Implementation (Developer Level 🟡)

### 1. Naive Implementation (Not Thread Safe) - AVOID
```java
public class NaiveSingleton {
    private static NaiveSingleton instance;
    private NaiveSingleton() {} // Private Constructor

    public static NaiveSingleton getInstance() {
        if (instance == null) {
            instance = new NaiveSingleton();
        }
        return instance;
    }
}
```

### 2. Double-Checked Locking (Thread Safe & Efficient)
```java
public class DatabaseConnection {
    // volatile ensures changes are visible to other threads immediately
    private static volatile DatabaseConnection instance;
    
    private DatabaseConnection() {
        System.out.println("Connecting to DB...");
    }

    public static DatabaseConnection getInstance() {
        if (instance == null) { // First check (no locking)
            synchronized (DatabaseConnection.class) {
                if (instance == null) { // Second check (with locking)
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

### 3. Enum Singleton (The Best Way)
Effective Java recommends this. It handles serialization and reflection attacks automatically.
```java
public enum ConfigManager {
    INSTANCE;
    
    public String getConfig() {
        return "ShowMeTheMoney";
    }
}
// Usage: ConfigManager.INSTANCE.getConfig();
```

---

## 6. Spring Boot Implementation

Spring **does not** use the `static getInstance()` method. It uses the **Spring Container** to manage scope.

```java
@Service // This is a Singleton by default!
public class UserService {
    // Spring creates ONE instance of this class at startup.
    // It injects this SAME instance into every Controller.
}
```

*   **Diff**: In Java, the *Class* enforces Singleton. In Spring, the *Framework* enforces Singleton.
*   **Scope**: You can change it with `@Scope("prototype")` to get a new instance every time.

---

## 7. Internal Mechanics (Architect Level 🔴)

### Memory Impact
*   **Positive**: Saves RAM. 1 instance vs 10,000 instances.
*   **Negative**: The object lives for the *entire* application lifecycle. It is never Garbage Collected. (Potential Memory Leak if it holds a growing List).

### ClassLoader Issues
*   Singleton is unique *per ClassLoader*.
*   In complex App Servers (WebLogic/Tomcat), you might accidentally have two instances if different EARs load the class separately.

---

## 8. Advantages

1.  **Controlled Access**: Sole instance.
2.  **Reduced Global Namespace**: Avoids global variables (mostly).
3.  **Lazy Loading**: Object is created only when requested (saves startup time).

---

## 9. Disadvantages

1.  **Violation of SRP**: It controls its own creation *and* lifecycle.
2.  **Testing Nightmare**: Global state is the enemy of Unit Testing. You can't easy "Mock" a static `getInstance()` call.
3.  **Concurrency Bottleneck**: If the Singleton has a `synchronized` method, all threads wait for it.

---

## 10. When NOT to Use

1.  **Shared Mutable State**: If the Singleton holds a `List` that 10 threads are modifying, you will have race conditions.
2.  **DTOs/Entities**: Never make a `User` object a Singleton. `User` represents state, not service.

---

## 11. Common Mistakes

1.  **Reflection Attack**: You can use Reflection to call `private constructor`. (Enum prevents this).
2.  **Serialization**: If you deserialize a Singleton, you get a *new* instance. (You need `readResolve()` to fix this).
3.  **Overuse**: "I'll make everything a Singleton just in case."

---

## 12. Refactoring Example

### The Bad Code (Hard Dependency)
```java
class OrderService {
    void process() {
        // Hardcoded dependency. Cannot be mocked.
        Logger.getInstance().log("Processing");
    }
}
```

### Refactored (Dependency Injection)
```java
class OrderService {
    private final Logger logger;
    
    // Inject the Singleton here.
    public OrderService(Logger logger) {
        this.logger = logger;
    }
    
    void process() {
        logger.log("Processing");
    }
}
```

---

## 13. Comparison

| Pattern | Vs Singleton |
|:---|:---|
| **Static Class** | Static methods (`Math.abs`) cannot be passed as arguments or implement Interfaces. Singleton can. |
| **Factory** | Factory *creates* objects. Singleton *is* the object. Often, Factories are Singletons. |

---

## 14. Interview Questions

### Basic
1.  **Does Singleton allow parameters in the constructor?** (No, easier not to. How would `getInstance()` decide which params to use?).
2.  **Is Singleton thread-safe by default?** (No. You must synchronize it).

### Intermediate
3.  **What is the "Double-Checked Locking" problem?** (Without `volatile`, the JVM might reorder instructions, returning a partially constructed object).
4.  **How to break a Singleton?** (Reflection, Serialization, Cloning, multiple ClassLoaders).
5.  **Why is Enum the best Singleton?** (JVM guarantees 1 instance, handles serialization, prevents reflection).

### Advanced
6.  **How does Spring handle Singleton concurrency?** (It doesn't. Beans are stateless. If you add state, you must synchronize).
7.  **What is "Bill Pugh Singleton"?** (Using a static inner class holder. Lazy loaded and thread-safe without synchronization).
    ```java
    class BillPugh {
        private BillPugh() {}
        private static class Holder { static final BillPugh INSTANCE = new BillPugh(); }
        public static BillPugh getInstance() { return Holder.INSTANCE; }
    }
    ```

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: You have a `Configuration` class that reads a massive JSON file (100MB).
    *   *Design*: Singleton. Read once at startup. Cache it.

2.  **Scenario**: You need a `SequenceGenerator` for Order IDs.
    *   *Design*: Singleton. Must be synchronized to prevent duplicate IDs.

3.  **Scenario**: You are writing a testing framework.
    *   *Design*: `TestRunner` should be Singleton to coordinate results.

4.  **Scenario**: You put a cache `Map<>` inside a Singleton Service.
    *   *Design*: Ensure `ConcurrentHashMap` is used. Watch out for OOM (Out Of Memory) if the map never clears.

5.  **Scenario**: You need to mock the Database Singleton in a Unit Test.
    *   *Design*: Refactor to Dependency Injection. Pass the DB as a constructor argument.

6.  **Scenario**: You have a cluster of 5 servers. Is Singleton unique across the cluster?
    *   *Design*: **NO**. It is unique per JVM. To have a "Cluster Singleton", you need Redis or Zookeeper.

---

## 16. Summary & Architect Takeaways

*   **Use DI, not `getInstance()`**: In 2025, modern apps use Spring/Guice. You rarely write `static instance`.
*   **Stateless**: Keep Singletons stateless. If you need state, put it in a Database or Redis.
*   **Enum**: If you MUST write a manual Singleton, use `enum`.

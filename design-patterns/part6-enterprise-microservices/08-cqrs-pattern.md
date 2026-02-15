# CQRS Pattern

> **Part 6: Enterprise & Microservices Patterns**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The Scalability King

---

## 1. Core Concept
**C**ommand **Q**uery **R**esponsibility **S**egregation.
Separate your app into two Parts:
1.  **Command Side (Write)**: Handles Updates (`CreateOrder`, `CancelOrder`). Focuses on **Consistency** & **Validation**.
2.  **Query Side (Read)**: Handles Reads (`GetOrder`, `SearchOrders`). Focuses on **Performance**.

---

## 2. Problem Statement
**Traditional CRUD**:
*   You start with `User` table.
*   Requirement: "Search Users by ZipCode and LastLogin > 30 days".
*   You add Indexes. Write speed slows down.
*   Requirement: "Show User Dashboard with Order Count".
*   You do `JOIN orders`. Query speed slows down.

**The CQRS Solution**:
*   **Write**: Save `User` to normalized PostgreSQL.
*   **Read**: Sync data to ElasticSearch or a flattened Mongo collection (`UserDashboard`).
*   **Query**: `SELECT * FROM UserDashboard WHERE ...` (No Joins, Super Fast).

---

## 3. Java Implementation (Axon/Custom)

### Command (Write)
```java
@PostMapping("/users")
public void createUser(@RequestBody CreateUserCmd cmd) {
    // 1. Validate
    // 2. Save to Oracle (Relational)
    userRepository.save(new User(cmd));
    
    // 3. Publish Event
    eventBus.publish(new UserCreatedEvent(cmd));
}
```

### Event Handler (Sync)
```java
@Component
public class UserProjections {
    
    @EventListener
    public void on(UserCreatedEvent event) {
        // Save to ElasticSearch / Redis / ReadDB
        userReadRepo.save(new UserView(event));
    }
}
```

### Query (Read)
```java
@GetMapping("/users/{id}")
public UserView getUser(@PathVariable String id) {
    // Read from optimized View
    return userReadRepo.findById(id);
}
```

---

## 4. Architect Takeaway
*   **Complexity**: Increases continuously. You now handle **Eventual Consistency** (Read DB might lag behind Write DB by 100ms).
*   **Use Condition**: Only use when **Reads >>> Writes** (1000:1 ratio) or when Views are vastly different from Domain Models.

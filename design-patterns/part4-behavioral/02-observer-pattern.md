# Observer Pattern

> **Part 4: Behavioral Patterns**  
> **Difficulty:** ⭐⭐ (Intermediate)  
> **Status:** The Event Driver

---

## 0. Learning Objectives

*   **Beginner**: Implement a "Listener".
*   **Developer**: Use `Java.util.Observer` (Deprecated) vs `PropertyChangeListener` vs Custom Interface.
*   **Architect**: Design Event-Driven Architectures (EDA).

---

## 1. Problem Statement

### The Polling Problem
You have a `Store` and a `Customer`.
The Customer wants to know when the iPhone 25 is in stock.
*   **Bad Way**: Customer calls `store.hasStock()` every 5 minutes. (Polling). Wastes resources.
*   **Good Way**: Store sends an **Event** to Customer when stock arrives. (Push).

### The Solution
The `Store` (Subject) keeps a list of `Customers` (Observers). When state changes, it iterates the list and calls `notify()`.

---

## 2. Real-World Analogy

**Social Media**
*   You **Follow** a celebrity.
*   When they tweet, you get a notification.
*   They don't know *who* you are individually. They just broadcast to their "Followers List".

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

### Participants
1.  **Subject** (Publisher): Maintains list of observers. has `attach()`, `detach()`, `notify()`.
2.  **Observer** (Subscriber): Interface with `update()`.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Observer Interface
```java
public interface Observer {
    void update(String message);
}
```

### 2. The Subject (YouTube Channel)
```java
import java.util.ArrayList;
import java.util.List;

public class YouTubeChannel {
    private List<Observer> subscribers = new ArrayList<>();
    private String name;

    public YouTubeChannel(String name) { this.name = name; }

    public void subscribe(Observer o) { subscribers.add(o); }
    public void unsubscribe(Observer o) { subscribers.remove(o); }

    public void uploadVideo(String videoTitle) {
        System.out.println(name + " uploaded: " + videoTitle);
        notifySubscribers("New Video: " + videoTitle);
    }

    private void notifySubscribers(String msg) {
        for (Observer o : subscribers) {
            o.update(msg);
        }
    }
}
```

### 3. The Observers (Users)
```java
public class User implements Observer {
    private String name;

    public User(String name) { this.name = name; }

    @Override
    public void update(String message) {
        System.out.println(name + " received notification: " + message);
    }
}
```

### 4. Client
```java
YouTubeChannel channel = new YouTubeChannel("TechGuy");
User alice = new User("Alice");
User bob = new User("Bob");

channel.subscribe(alice);
channel.subscribe(bob);

channel.uploadVideo("Design Patterns in Java");
// Both Alice and Bob get printed messages.
```

---

## 6. Spring Boot Implementation

Spring uses Observer Pattern **everywhere** via the **Application Event** system.

### 1. The Event
```java
public class UserRegisteredEvent {
    private String username;
    // constructor/getters
}
```

### 2. The Publisher (Subject)
```java
@Service
public class UserService {
    @Autowired
    private ApplicationEventPublisher publisher;

    public void register(String name) {
        // ... save user ...
        publisher.publishEvent(new UserRegisteredEvent(name));
    }
}
```

### 3. The Listeners (Observers)
```java
@Component
public class EmailService {
    @EventListener
    public void sendWelcomeEmail(UserRegisteredEvent event) {
        System.out.println("Sending Email to " + event.getUsername());
    }
}

@Component
public class AuditService {
    @EventListener
    public void auditLog(UserRegisteredEvent event) {
        System.out.println("Logging Registration: " + event.getUsername());
    }
}
```
*   **Benefit**: `UserService` knows NOTHING about Email or Audit. Loose Coupling.

---

## 8. Advantages

1.  **Loose Coupling**: Subject doesn't strictly know the concrete Observer classes.
2.  **Dynamic Relationships**: Subscribers can join/leave at runtime.
3.  **Broadcasting**: Easy to send one message to many targets.

---

## 9. Disadvantages

1.  **Memory Leaks**: The "Lapsed Listener Problem". If you subscribe and forget to unsubscribe, the Subject holds a strong reference to you. You are never garbage collected.
2.  **Ordering**: Default Observers are notified in random order. Don't rely on it.

---

## 10. When NOT to Use

1.  **Complex Chains**: If Observer A notifies B, which notifies C, which notifies A... you get an infinite loop (StackOverflow).

---

## 14. Interview Questions

### Basic
1.  **What is the "Pull" vs "Push" model in Observer?**
    *   **Push**: Subject sends data in update(data).
    *   **Pull**: Subject sends "I changed", Observer calls `subject.getData()`.
2.  **Is `Observable` class in Java good?** (No, it's deprecated since Java 9. It was a Class, not Interface, limiting inheritance).

### Intermediate
3.  **How to handle async observers?** (In Spring, use `@Async` on the `@EventListener` methods).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Stock Market Ticker.
    *   *Design*: `Stock` is Subject. `Screen`, `MobileApp`, `Algorithm` are Observers.

2.  **Scenario**: Excel Sheet formulas.
    *   *Design*: Cell B1 observes Cell A1. If A1 changes, B1 recalculates.

3.  **Scenario**: Chat Room.
    *   *Design*: Room observes User (for messages). User observes Room (for broadcasting). Mediator pattern fits better here.

---

## 16. Summary & Architect Takeaways

*   **Decoupling**: This is the best way to decouple modules.
*   **Memory Management**: Always ensure you `detach()` when an object dies, or use `WeakReference`.

# Observer Pattern in Spring

> **Part 5: Patterns Inside Spring**  
> **Status:** Event Driven Architecture

---

## 1. The Event Mechanism
Spring holds a built-in Event Bus in the `ApplicationContext`.
*   **Subject**: `ApplicationContext` (which extends `ApplicationEventPublisher`).
*   **Observer**: Any Bean with `@EventListener` or implementing `ApplicationListener`.

---

## 2. Default Events
Spring publishes lifecycle events automatically:
1.  `ContextRefreshedEvent`: When the context is initialized.
2.  `ContextStartedEvent`: When `ctx.start()` is called.
3.  `ContextClosedEvent`: When the app shuts down (Good for cleanup).

```java
@Component
public class StartupListener {
    @EventListener(ContextRefreshedEvent.class)
    public void onStartup() {
        System.out.println("App is ready!");
    }
}
```

---

## 3. Custom Events (Decoupling)
Instead of calling method A -> method B -> method C, use Events.

```java
// 1. The Event
public class OrderPaidEvent { ... }

// 2. The Publisher
@Service
public class OrderService {
    @Autowired ApplicationEventPublisher publisher;
    
    public void pay() {
        // ... db logic ...
        publisher.publishEvent(new OrderPaidEvent(orderId));
    }
}

// 3. The Listeners (Multiple!)
@Component
class InventoryService {
    @EventListener
    public void onOrderPaid(OrderPaidEvent e) { ... subtract stock ... }
}

@Component
class EmailService {
    @EventListener
    public void onOrderPaid(OrderPaidEvent e) { ... send email ... }
}
```

---

## 4. Async Events
By default, Spring Events are **Synchronous**. (If email fails, transaction might rollback).
To make it **Asynchronous** (Fire and Forget), add `@Async`:

```java
@Async
@EventListener
public void sendEmail(OrderPaidEvent e) { ... }
```
*   Now the email sends in a background thread. User response is faster.

---

## 5. Architect Takeaway
*   **Use Events for Side Effects**: Core logic (DB save) should be explicit. Side effects (Email, Analytics, Audit) should be Events.

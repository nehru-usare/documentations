# Saga Pattern

> **Part 6: Enterprise & Microservices Patterns**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The Distributed Transaction

---

## 1. Problem Statement

### The "Two-Phase Commit" Problem
In a Monolith, you use `@Transactional` to save an Order and update Inventory. The Database ensures ACID.
In Microservices, **Database per Service** means you cannot use local transactions.
*   Service A (Orders) commits.
*   Service B (Inventory) fails.
*   **Result**: Data Inconsistency. You have an Order but no Stock.

### The Solution
Break the transaction into a sequence of local transactions.
If step `N` fails, execute **Compensating Transactions** to undo steps `N-1`, `N-2`, etc.

---

## 2. Core Approaches

### 1. Choreography (Events)
Each service produces an event. Other services listen and react.
*   **Order Service**: Creates Order. Publishes `OrderCreated`.
*   **Inventory Service**: Listens to `OrderCreated`. Reserves Stock. Publishes `StockReserved`.
*   **Payment Service**: Listens to `StockReserved`. Charges Card.

**Pros**: Loose Coupling.
**Cons**: Hard to track. Circular dependencies.

### 2. Orchestration (Command)
A central **Saga Orchestrator** tells participants what to do.
*   **OrderSaga**:
    1.  Call Inventory ("Reserve").
    2.  Call Payment ("Charge").
    3.  If Payment Fails -> Call Inventory ("Release").

**Pros**: Central logic. Easy to debug.
**Cons**: Tight Coupling to Orchestrator.

---

## 3. Java Implementation (Orchestration)

```java
public class CreateOrderSaga {
    
    // Steps
    public void execute(OrderDetails details) {
        try {
            inventoryService.reserve(details.getProductId());
            paymentService.charge(details.getUserId());
            shippingService.schedule(details.getAddress());
            orderService.approve(details.getOrderId());
        } catch (Exception e) {
            compensate(details);
        }
    }

    // Compensation (Undo)
    public void compensate(OrderDetails details) {
        // We don't know exactly where it failed, so we try to undo everything (idempotently)
        shippingService.cancel(details.getAddress());
        paymentService.refund(details.getUserId());
        inventoryService.release(details.getProductId());
        orderService.reject(details.getOrderId());
    }
}
```

---

## 4. Architect Takeaway
*   **Eventual Consistency**: Sagas do not guarantee immediate consistency. They guarantee **Eventual Consistency**.
*   **Frameworks**: Don't write manual Sagas. Use **Axon Framework** or **Spring State Machine**.
*   **Idempotency**: All steps and compensations MUST be idempotent. Network retries will happen.

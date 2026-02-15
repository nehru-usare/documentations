# Anemic Domain Model

> **Part 7: Anti-Patterns**  
> **Severity:** 🟡 High (Controversial)  
> **Status:** Procedural Code in Java

---

## 1. The Symptom
Your Entities look like C Structs.

```java
@Entity
public class Order {
    @Getter @Setter private BigDecimal total;
    @Getter @Setter private List<Item> items;
    @Getter @Setter private String status;
}
```

Your Services hold all the logic.
```java
@Service
public class OrderService {
    public void addItem(Order o, Item i) {
        if (o.getStatus().equals("PAID")) throw new Error();
        o.setTotal(o.getTotal().add(i.getPrice())); // Logic outside Entity
        o.getItems().add(i);
    }
}
```

---

## 2. Why is this Bad?
1.  **Not OOP**: This is Procedural Programming. You are treating objects as data buckets.
2.  **Invariance Violation**: Anyone can call `order.setTotal(-100)` because the setter is public and dumb. You cannot guarantee the Valid State of an object.
3.  **Code Scattering**: Logic for "Adding Item" might be duplicated in `OrderService`, `CartService`, and `AdminService`.

---

## 3. The Fix: Rich Domain Model
Move logic **Into the Entity**.

```java
@Entity
public class Order {
    // Private Setters!
    private BigDecimal total; 

    public void addItem(Item i) {
        if (this.status.equals("PAID")) throw new Error();
        this.total = this.total.add(i.getPrice());
        this.items.add(i);
    }
}
```

Now `OrderService` is just a coordinator:
```java
public void addItem(String id, Item i) {
    Order o = repo.findById(id);
    o.addItem(i); // Logic is inside Order
    repo.save(o);
}
```

---

## 4. Architect Takeaway
*   **Balance**: Spring naturally pushes towards Anemic Models (Stateless Services + State-only Entities).
*   **DDD**: In Domain Driven Design, Anemic Models are an Anti-Pattern. Business rules belong in the Domain Object.

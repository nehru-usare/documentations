# Mediator Pattern

> **Part 4: Behavioral Patterns**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Controller

---

## 0. Learning Objectives

*   **Beginner**: Stop objects from calling each other directly (Spaghetti coupling).
*   **Developer**: Implement a Chat Room.
*   **Architect**: Centralize complex communication logic.

---

## 1. Problem Statement

### The "Many-to-Many" Chaos
Imagine a Dialog Box.
*   `Checkbox` Checked -> Enables `TextBox`.
*   `TextBox` Filled -> Enables `SubmitButton`.
*   `ResetButton` Clicked -> Clears `TextBox`, Unchecks `Checkbox`.
*   **Issue**: If Checkbox calls TextBox directly, and Reset calls both... everything is coupled.

### The Solution
Create a `DialogMediator`. All components talk ONLY to the Mediator.
`Checkbox` says: "I was clicked."
`Mediator` decides: "Okay, I will enable TextBox."

---

## 2. Real-World Analogy

**Air Traffic Control (ATC)**
*   Pilot A doesn't talk to Pilot B.
*   Pilot A talks to ATC.
*   ATC tells Pilot B to move.
*   Safety is ensured by the Mediator (ATC).

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Define an object that encapsulates how a set of objects interact. Mediator promotes loose coupling by keeping objects from referring to each other explicitly.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Mediator Interface
```java
interface ChatMediator {
    void sendMessage(String msg, User user);
    void addUser(User user);
}
```

### 2. Concrete Mediator
```java
class ChatRoom implements ChatMediator {
    private List<User> users = new ArrayList<>();

    public void addUser(User user) { users.add(user); }

    public void sendMessage(String msg, User sender) {
        for (User u : users) {
             // Don't echo to sender
            if (u != sender) u.receive(msg);
        }
    }
}
```

### 3. Colleague (User)
```java
abstract class User {
    protected ChatMediator mediator;
    public User(ChatMediator m) { this.mediator = m; }
    
    public void send(String msg) {
        mediator.sendMessage(msg, this);
    }
    public abstract void receive(String msg);
}
```

---

## 8. Advantages

1.  **Decoupling**: Colleagues (Users) don't know about each other.
2.  **Centralized Control**: Logic is in one place, not scattered.

---

## 9. Disadvantages

1.  **God Object Risk**: The Mediator tends to become huge and complex.

---

## 14. Interview Questions

### Basic
1.  **Difference between Mediator and Facade?**
    *   **Facade**: One-way. Simple interface to a subsystem.
    *   **Mediator**: Multi-way. Cooperative interaction b/w colleagues.

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Airport Ground Control.
    *   *Design*: Mediator handles Gates, Runways, and Plane requests.

2.  **Scenario**: GUI Form Validation.
    *   *Design*: FormController (Mediator) listens to all input fields validation events.

---

## 16. Summary & Architect Takeaways

*   **Star Topology**: Moves system from Mesh (N*N connections) to Star (N connections).

# State Pattern

> **Part 4: Behavioral Patterns**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Logic Cleaner

---

## 0. Learning Objectives

*   **Beginner**: Understand State Machines (FSM).
*   **Developer**: Refactor complex `switch(state)` logic into classes.
*   **Architect**: Design robust workflows (Order Processing: New -> Paid -> Shipped).

---

## 1. Problem Statement

### The "Conditional Logic" Explosion
You have an `Order` class.
```java
void cancel() {
    if (state == NEW) { ... delete ... }
    else if (state == SHIPPED) { throw Error("Too late"); }
    else if (state == DELIVERED) { ... return process ... }
}
```
*   **Issue**: Every method (`cancel`, `pay`, `ship`) has this massive switch statement.
*   **Solution**: Make `State` an object. `Order` delegates to `state.cancel()`.

---

## 2. Real-World Analogy

**Vending Machine**
*   **State: NoCoin**: Pressing "Coke" does nothing.
*   **State: HasCoin**: Pressing "Coke" dispenses drink.
*   The button is the same. The behavior changes based on state.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

### Participants
1.  **Context**: The main object (`VendingMachine`). Has a reference to `CurrentState`.
2.  **State**: Interface (`VendingMachineState`).
3.  **ConcreteState**: `NoCoinState`, `HasCoinState`.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The State Interface
```java
interface State {
    void insertCoin();
    void pressButton();
}
```

### 2. Concrete States
```java
class NoCoinState implements State {
    public void insertCoin() {
        System.out.println("Coin accepted.");
        // Context transition logic usually happens here or in Context
    }
    public void pressButton() {
        System.out.println("Please insert coin first.");
    }
}

class HasCoinState implements State {
    public void insertCoin() {
        System.out.println("Coin already inserted.");
    }
    public void pressButton() {
        System.out.println("Dispensing Coke...");
    }
}
```

### 3. Context
```java
class VendingMachine {
    private State state;

    public void setState(State state) { this.state = state; }

    public void pressButton() {
        state.pressButton(); // Logic delegated to current state
    }
}
```

---

## 8. Advantages

1.  **Single Responsibility**: Logic for "Shipped" state is in `ShippedState.java`. Not mixed with "New" state.
2.  **Open/Closed**: Adding a new state (`ReturnedState`) is easy.

---

## 9. Disadvantages

1.  **Class Explosion**: 10 states = 10 classes.

---

## 14. Interview Questions

### Basic
1.  **Difference between State and Strategy?**
    *   **Strategy**: Client chooses the strategy once (usually).
    *   **State**: The Context *automatically* changes state during execution.
2.  **Where should transition logic live?** (Either in Context or in the ConcreteStates).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Document Workflow (Draft -> Moderation -> Published).
    *   *Design*: `DraftState.publish()` moves to Moderation. `PublishedState.publish()` throws error.

2.  **Scenario**: Uber Ride (Requesting -> Arriving -> InTrip -> Completed).

3.  **Scenario**: TCP Connection (Closed -> Listen -> SynSent -> Established).

---

## 16. Summary & Architect Takeaways

*   **Finite State Machines (FSM)**: This is the OOP implementation of FSM.
*   **Spring Statemachine**: Use this library for complex generic state machines instead of writing manual classes.

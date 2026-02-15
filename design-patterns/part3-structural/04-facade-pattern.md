# Facade Pattern

> **Part 3: Structural Patterns**  
> **Difficulty:** ⭐ (Beginner)  
> **Status:** The Cleanup Crew

---

## 0. Learning Objectives

*   **Beginner**: Simplifies "Spaghetti Code" usage.
*   **Developer**: Create a `BookingFacade` that wraps `FlightService`, `HotelService`, and `PaymentService`.
*   **Architect**: Define clear boundaries between modules.

---

## 1. Problem Statement

### The Complexity Problem
To book a trip, the client code needs to:
1.  `flightService.book(...)`
2.  `hotelService.reserve(...)`
3.  `payment.charge(...)`
4.  `email.send(...)`
*   **Result**: Client code is 50 lines long and coupled to 4 different services.

### The Solution
Create a `TravelFacade` with one method: `bookTrip()`. It handles the orchestration.

---

## 2. Real-World Analogy

**Home Theater Remote**
*   **Without Facade**: Turn on TV. Turn on DVD. Set TV Input to HDMI1. Turn on Amp. Set Amp to DVD. Lower Blinds.
*   **With Facade**: Press **"Watch Movie"**. The remote (Facade) does all 6 steps for you.

**Customer Support**
*   You call one number (Facade). The operator talks to Billing, Tech Support, and Logistics for you.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.

### Difference from Adapter
*   **Adapter**: Wraps **one** class to change its interface.
*   **Facade**: Wraps **many** classes to simplify their interface.

---

## 4. UML-Style Structure

*   `Client` depends on `Facade`.
*   `Facade` depends on `SubsystemA`, `SubsystemB`, `SubsystemC`.
*   `SubsystemA` doesn't know about `Facade`.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Subsystems (Complex)
```java
class CPU { void freeze() { ... } void jump(long pos) { ... } void execute() { ... } }
class Memory { void load(long pos, byte[] data) { ... } }
class HardDrive { byte[] read(long lba, int size) { ... } }
```

### 2. The Facade (Simple)
```java
class ComputerFacade {
    private CPU cpu;
    private Memory ram;
    private HardDrive hd;

    public ComputerFacade() {
        this.cpu = new CPU();
        this.ram = new Memory();
        this.hd = new HardDrive();
    }

    public void start() {
        // Encapsulate the complex boot sequence
        cpu.freeze();
        ram.load(0, hd.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}
```

### 3. Client
```java
ComputerFacade computer = new ComputerFacade();
computer.start(); // Easy!
```

---

## 6. Spring Boot Implementation

In Spring, **Service Layer** often acts as a Facade for the **Repository Layer**.
*   `Controller` calls `UserService` (Facade).
*   `UserService` calls `UserRepository`, `EmailService`, `AuditRepository`.

**API Gateway** (Microservices)
*   The API Gateway is a giant HTTP Facade.
*   Client calls `/buy`.
*   Gateway calls `Inventory`, `Pricing`, `Shipping` microservices.

---

## 8. Advantages

1.  **Decoupling**: Client is decoupled from the subsystem complexity.
2.  **Layering**: Helps enforce architectural layers(UI -> Facade -> Services -> DAO).
3.  **Compilation**: Changes in subsystem classes don't affect the Client (only the Facade needs recompilation).

---

## 9. Disadvantages

1.  **God Object Risk**: Facade can become a "God Class" if it does too much. Keep it thin (delegation only).
2.  **Hiding Power**: Sometimes power users *need* the low-level functions the Facade hides. (Allow access to low-level classes if necessary).

---

## 14. Interview Questions

### Basic
1.  **What is the goal of Facade?** (Simplicity).
2.  **Does Facade add new functionality?** (Usually no, it just coordinates existing functionality. But it *can* add glue logic).

### Intermediate
3.  **Can a subsystem have multiple Facades?** (Yes. You can create different Facades for different user types - e.g., `AdminFacade` vs `UserFacade`).

### Advanced
4.  **Difference between Mediator and Facade?** (Mediator centralizes communication *between* components. Facade defines a simplified interface *to* a subsystem).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Legacy System Migration.
    *   *Design*: Create a Facade over the new and old systems. The Facade decides where to route the request (Strangler Fig Pattern implementation).

2.  **Scenario**: Complex Library (e.g., FFMPEG for video).
    *   *Design*: Create `VideoConverterFacade`. Client calls `convert(mp4)`. Facade handles the 50 flags FFMPEG needs.

---

## 16. Summary & Architect Takeaways

*   **Entry Point**: Use Facade as the entry point for each layer of your application.
*   **Don't force it**: If the subsystem is already simple, you don't need a Facade.
*   **Refactoring**: It's the best way to tame a "Big Ball of Mud". Wrap the mud in a Facade.

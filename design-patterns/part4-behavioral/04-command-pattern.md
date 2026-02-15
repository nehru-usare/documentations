# Command Pattern

> **Part 4: Behavioral Patterns**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Action Object

---

## 0. Learning Objectives

*   **Beginner**: Understand how "Undo" buttons work.
*   **Developer**: Turn method calls into Objects that can be stored in a List.
*   **Architect**: Design Job Queues and Asynchronous processing systems.

---

## 1. Problem Statement

### The "Tight Coupling" Problem
You have a `Button`.
You have a `Light`.
*   Naive: `Button` calls `light.turnOn()`.
*   **Issue**: The Button is now hardcoded to the Light. What if the Button should turn on the Fan? Or Save a File?

### The Solution
Create a `Command` object (`TurnOnLightCommand`). The Button just calls `command.execute()`. It doesn't know it's a light.

---

## 2. Real-World Analogy

**Restaurant Order**
*   **Customer** (Client): "I want a Steak".
*   **Waiter** (Invoker): Writes "Steak" on a Ticket (Command Object).
*   **Kitchen** (Receiver): Reads Ticket. Cooks Steak.
*   The Waiter doesn't know how to cook. The Waiter just queues the ticket.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

### Participants
1.  **Command**: Interface with `execute()`.
2.  **ConcreteCommand**: Implements `execute()`, calls `Receiver`.
3.  **Receiver**: The actual logic class (`Light`, `Stereo`).
4.  **Invoker**: Holds the command (`RemoteControl`).

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Command Interface
```java
public interface Command {
    void execute();
    void undo(); // For Undo functionality
}
```

### 2. The Receiver (The Hardware)
```java
public class Light {
    public void on() { System.out.println("Light ON"); }
    public void off() { System.out.println("Light OFF"); }
}
```

### 3. The Concrete Command
```java
public class LightOnCommand implements Command {
    private Light light;

    public LightOnCommand(Light light) { this.light = light; }

    public void execute() { light.on(); }
    public void undo() { light.off(); }
}
```

### 4. The Invoker (Remote Control)
```java
public class RemoteControl {
    private Command slot; // Holds ANY command

    public void setCommand(Command command) { this.slot = command; }

    public void pressButton() { slot.execute(); }
    public void pressUndo() { slot.undo(); }
}
```

### 5. Client
```java
Light livingRoomLight = new Light();
LightOnCommand lightsOn = new LightOnCommand(livingRoomLight);

RemoteControl remote = new RemoteControl();
remote.setCommand(lightsOn);
remote.pressButton(); // Light ON
remote.pressUndo(); // Light OFF
```

---

## 7. Internal Mechanics (Architect Level 🔴)

### Undo/Redo Stacks
*   Maintain a `Stack<Command> history`.
*   Every `execute()` pushes to stack.
*   `Ctrl+Z` pops from stack and calls `undo()`.

### Queuing
*   ThreadPools logic works on `Runnable`. `Runnable` IS a Command interface (`run()` == `execute()`).
*   Example: `ExecutorService.submit(new SendEmailCommand())`.

---

## 8. Advantages

1.  **Decoupling**: Invoker (Remote) has no idea about Receiver (Light).
2.  **Extensibility**: You can add `FanOnCommand` without changing the Remote.
3.  **Macros**: You can create a `MacroCommand` that holds a `List<Command>` and executes them in sequence.

---

## 9. Disadvantages

1.  **Class Explosion**: Every action needs a Command class.
    *   `LightOn`, `LightOff`, `FanOn`, `FanOff`...

---

## 14. Interview Questions

### Basic
1.  **What is the main purpose of Command?** (Decouple sender from receiver).
2.  **How is `Runnable` related to Command?** (It is the Java interface for the Command pattern).

### Intermediate
3.  **How to implement Undo?** (Store state in the Command, create an `undo()` method that restores it).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Smart Home App.
    *   *Design*: Buttons on the screen are Invokers. Each button is assigned a Command (`TurnOnAC`, `LockDoor`).

2.  **Scenario**: Text Editor.
    *   *Design*: `CopyCommand`, `PasteCommand`. Allows Unlimited Undo history.

---

## 16. Summary & Architect Takeaways

*   **Transactional behavior**: You can wrap `execute()` in a transaction.
*   **Asynchronous**: Use Command to send tasks to a background thread (Job Queue pattern).

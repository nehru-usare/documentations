# Bridge Pattern

> **Part 3: Structural Patterns**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** The Hierarchy Splitter

---

## 0. Learning Objectives

*   **Beginner**: Solve the "Class Explosion" problem.
*   **Developer**: Prefer Composition over Inheritance for independent dimensions.
*   **Architect**: Design drivers (JDBC) where Abstraction (Connection common code) is bridged to Implementation (MySQL/Oracle drivers).

---

## 1. Problem Statement

### The Cartesian Product Explosion
You have a `Shape` class. Subclasses: `Circle`, `Square`.
You want to add Colors: `Red`, `Blue`.
If you use inheritance, you get:
1.  `RedCircle`
2.  `BlueCircle`
3.  `RedSquare`
4.  `BlueSquare`
*   **Total**: 4 classes.
*   **If you add Triangle**: 6 classes.
*   **If you add Green**: 9 classes.
*   **Formula**: N Shapes * M Colors.

### The Solution (Bridge)
Separate `Shape` from `Color`.
*   `Shape` classes: Circle, Square.
*   `Color` classes: Red, Blue.
*   `Shape` **has a** `Color`.
*   **Total**: 4 classes (2 + 2).
*   **Formula**: N + M.

---

## 2. Real-World Analogy

**Remote Control and TV**
*   **Abstraction**: The Remote Control (Buttons).
*   **Implementation**: The TV (Actual electronics).
*   You can have a `SonyRemote` or `SamsungRemote` work with `SonyTV` or `SamsungTV` if they share a common Bridge protocol.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Decouple an abstraction from its implementation so that the two can vary independently.

### Participants
1.  **Abstraction**: The high-level logic (`Shape`).
2.  **Implementor**: The interface for the implementation (`Color`).
3.  **RefinedAbstraction**: `Circle`.
4.  **ConcreteImplementor**: `Red`.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Implementor (The Bridge)
```java
public interface Color {
    void applyColor();
}
```

### 2. Concrete Implementors
```java
public class Red implements Color {
    public void applyColor() { System.out.println("Applying Red Color"); }
}

public class Blue implements Color {
    public void applyColor() { System.out.println("Applying Blue Color"); }
}
```

### 3. The Abstraction
```java
public abstract class Shape {
    protected Color color; // The Bridge!

    public Shape(Color color) {
        this.color = color;
    }

    abstract void draw();
}
```

### 4. Refined Abstraction
```java
public class Circle extends Shape {
    public Circle(Color color) { super(color); }

    public void draw() {
        System.out.print("Drawing Circle and ");
        color.applyColor(); // Delegate
    }
}
```

### 5. Client
```java
Shape redCircle = new Circle(new Red());
Shape blueSquare = new Square(new Blue());
```

---

## 7. Internal Mechanics (Architect Level 🔴)

### JDBC Driver
*   **Abstraction**: `java.sql.Connection`, `java.sql.Statement` (The interfaces you code against).
*   **Implementor**: `com.mysql.jdbc.ConnectionImpl`, `oracle.jdbc.OracleConnection`.
*   JDBC acts as the Bridge. You write code using the Abstraction. The Driver provides the Implementation.

---

## 8. Advantages

1.  **Decoupling**: You can change the Implementation (Platform/OS) without changing the Abstraction (UI Logic).
2.  **Scalability**: Avoids exponential class growth.

---

## 14. Interview Questions

### Basic
1.  **What problem does Bridge solve?** (Class Explosion / Cartesian Product of subclasses).
2.  **Difference between Bridge and Adapter?** (Adapter makes things work *after* they are designed. Bridge is designed *up-front* to let things vary).

### Intermediate
3.  **Is Strategy Pattern similar?** (Structurally, yes. Strategy is usually behavioral (How to do algo). Bridge is structural (How to organize hierarchy)).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Cross-Platform UI (Windows/Mac/Linux) + Rendering Engine (DirectX/OpenGL).
    *   *Design*: `Window` (Abstraction) holds a `Renderer` (Imp).
    *   `MacWindow` with `OpenGLRenderer`.
    *   `WindowsWindow` with `DirectXRenderer`.

2.  **Scenario**: Notification Senders (Email/SMS) + Message Types (Urgent/Info).
    *   *Design*: `Message` (Abstraction) holds `MessageSender` (Imp).
    *   `UrgentMessage` uses `SmsSender`. `InfoMessage` uses `EmailSender`.

---

## 16. Summary & Architect Takeaways

*   **Orthogonal Dimensions**: Whenever you see a class name like `Device_Protocol_Version`, you need a Bridge.
*   **Composition**: Once again, Composition wins over Inheritance.

# Abstract Factory Pattern

> **Part 2: Creational Patterns**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** Rare but Powerful

---

## 0. Learning Objectives

*   **Beginner**: Don't confuse it with Factory Method.
*   **Developer**: Create a "Theme" system where buttons and textboxes match.
*   **Architect**: Ensure consistency across a family of products.

---

## 1. Problem Statement

### The "Mismatched Furniture" Problem
Imagine a UI Toolkit.
You have `LightButton` and `DarkButton`.
You have `LightCheckbox` and `DarkCheckbox`.
*   **Risk**: A developer accidentally mixes `LightButton` with `DarkCheckbox`.
*   **Goal**: Ensure that if the user selects "Dark Mode", *everything* created is Dark.

---

## 2. Core Concept (Beginner Level 🟢)

### Definition
Provide an interface for creating **families** of related or dependent objects without specifying their concrete classes.

### Participants
1.  **AbstractFactory**: Declares `createButton()`, `createCheckbox()`.
2.  **ConcreteFactory1** (DarkFactory): Returns `DarkButton`, `DarkCheckbox`.
3.  **ConcreteFactory2** (LightFactory): Returns `LightButton`, `LightCheckbox`.

---

## 4. UML-Style Structure

*   `GUIFactory` (Interface)
    *   `createButton()`
    *   `createCheckbox()`
*   `DarkThemeFactory` implements `GUIFactory`
*   `LightThemeFactory` implements `GUIFactory`

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Family Interfaces
```java
interface Button { void paint(); }
interface Checkbox { void paint(); }
```

### 2. The Abstract Factory Interface
```java
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}
```

### 3. The Concrete Factories
```java
class DarkThemeFactory implements GUIFactory {
    public Button createButton() { return new DarkButton(); }
    public Checkbox createCheckbox() { return new DarkCheckbox(); }
}

class LightThemeFactory implements GUIFactory {
    public Button createButton() { return new LightButton(); }
    public Checkbox createCheckbox() { return new LightCheckbox(); }
}
```

### 4. Client (Agnostic Application)
```java
class Application {
    private Button button;
    private Checkbox checkbox;

    // Client doesn't know if it's Dark or Light.
    // It just uses the Factory provided.
    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }

    public void paint() {
        button.paint();
        checkbox.paint();
    }
}
```

---

## 8. Advantages

1.  **Consistency Guarantee**: You can't accidentally create a `LightButton` from a `DarkThemeFactory`. The compiler prevents it.
2.  **Interchangeability**: Swap the entire look-and-feel of the app by changing one line (the Factory injection).

---

## 13. Comparison

| Factory Method | Abstract Factory |
|:---|:---|
| Creates **one** product. | Creates a **family** of products. |
| Uses Inheritance (Subclasses). | Uses Composition (Factory Object). |
| Method inside a class. | A Class of its own. |

---

## 14. Interview Questions

### Basic
1.  **What is the main difference between Factory Method and Abstract Factory?** (One product vs Family of products).

### Intermediate
2.  **Does Abstract Factory use Factory Method?** (Yes, the methods inside the Abstract Factory *are* Factory Methods).

### Advanced
3.  **Implementation-wise, how do you add a new Product (e.g., `Scrollbar`)?** (Painful. You have to edit the Interface and ALL concrete factories. This is a violation of Open/Closed Principle at the Interface level).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Cross-Platform Game (DirectX vs OpenGL).
    *   *Design*: `RenderFactory`. `DirectXFactory` creates `DXTexture`, `DXMesh`. `OpenGLFactory` creates `GLTexture`, `GLMesh`.
    *   *Constraint*: You cannot render a `DXTexture` with `OpenGL`. Abstract Factory enforces this.

2.  **Scenario**: Database Drivers (Connection, Command, Result).
    *   *Design*: `sql.Connection`, `sql.Statement`. The Driver (Factory) ensures you get a MySQL Connection and a MySQL Statement.

---

## 16. Summary & Architect Takeaways

*   **Use High Caution**: This pattern is rigid. Adding a new "Product" type breaks everything. Use only when the product families are stable (e.g., UI Components, OS Drivers).
*   **Configuration**: Usually, the Concrete Factory is chosen once at startup (`main` method or Spring Config) based on env variables.

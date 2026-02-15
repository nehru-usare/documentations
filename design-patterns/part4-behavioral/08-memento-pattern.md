# Memento Pattern

> **Part 4: Behavioral Patterns**  
> **Difficulty:** ⭐⭐ (Intermediate)  
> **Status:** The Time Machine

---

## 0. Learning Objectives

*   **Beginner**: Implement "Save/Load" functionality.
*   **Developer**: Encapsulate state inside a `Memento` object so it's not exposed.
*   **Architect**: Design Undo frameworks.

---

## 1. Problem Statement

### The "Private State" Problem
You want to save the state of an Object.
*   `Object.state` is private.
*   You cannot read `private` fields from outside to save them.
*   If you make them public, you violate encapsulation.

### The Solution
The Object itself (`Originator`) creates a black-box snapshot (`Memento`). Only the Originator can peek inside the Memento. Other classes (`Caretaker`) just hold the Memento.

---

## 2. Real-World Analogy

**Video Game Save Point**
*   You save the game. The game creates a "File" (Memento).
*   You execute "Load". The game reads the file and restores health/location.
*   You don't edit the binary save file manually.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later.

### Participants
1.  **Originator**: The object to be saved. Creates Memento.
2.  **Memento**: Stores internal state. Opaque to others.
3.  **Caretaker**: Stores Memento (e.g., in a List for History) but never modifies it.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Memento
```java
// Immutable state holder
public class EditorMemento {
    private final String content;
    
    public EditorMemento(String content) { this.content = content; }
    
    public String getContent() { return content; }
}
```

### 2. The Originator (Editor)
```java
public class TextEditor {
    private String content;

    public void type(String text) { content = text; }

    public EditorMemento save() {
        return new EditorMemento(content);
    }

    public void restore(EditorMemento m) {
        content = m.getContent();
    }
}
```

### 3. The Caretaker (History)
```java
public class History {
    private Stack<EditorMemento> history = new Stack<>();

    public void push(EditorMemento m) { history.push(m); }
    public EditorMemento pop() { return history.pop(); }
}
```

---

## 6. Applications

*   **Undo/Redo**: Caretaker stores a stack of Mementos.
*   **Database Transactions**: Rollback restores previous state.

---

## 8. Advantages

1.  **Encapsulation**: State is saved without exposing private fields.
2.  **Recovery**: Easy recovery mechanism.

---

## 9. Disadvantages

1.  **Memory**: Storing 1,000 snapshots of a 10MB object = 10GB RAM.
    *   **Optimization**: Store deltas (diffs) instead of full copies.

---

## 14. Interview Questions

### Basic
1.  **Difference between Memento and Serialization?** (Serialization persists to disk. Memento is usually in-memory pattern).
2.  **Difference between Command and Memento for Undo?**
    *   **Command**: Re-executes reverse operation (`x - 5`).
    *   **Memento**: Restores state (`x = 10`).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Browser Back Button.
    *   *Design*: Memento stores URL, Scroll Position, Validated Form Data.

---

## 16. Summary & Architect Takeaways

*   **Heavyweight**: Use carefully. A full state copy is expensive.

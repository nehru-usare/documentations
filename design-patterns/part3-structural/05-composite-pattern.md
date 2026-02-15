# Composite Pattern

> **Part 3: Structural Patterns**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Recursive Tree

---

## 0. Learning Objectives

*   **Beginner**: Understand how to represent a Tree Structure (Files and Folders).
*   **Developer**: Treat "One Object" and "Many Objects" uniformly.
*   **Architect**: Design recursive APIs (Menus, Org Charts, HTML DOM).

---

## 1. Problem Statement

### The "Box within a Box" Problem
Imagine an Order System.
*   An `Order` contains `Product`s.
*   But... sometimes a `Product` is actually a `Box` containing other `Products`.
*   **Challenge**: Calculating the total price.
    *   If you treat Boxes differently from Products, you need complex `if (item.isBox)` logic nested infinitely deep.

### The Solution
Make `Box` and `Product` implement the same interface: `Item`.
*   `Product.getPrice()`: Returns its price.
*   `Box.getPrice()`: Returns sum of children.
*   The client just calls `getPrice()` and doesn't care if it's a leaf or a tree.

---

## 2. Real-World Analogy

**File System**
*   **File** (Leaf): Has size.
*   **Folder** (Composite): Has size (Sum of all files inside).
*   You can "Delete" a File. You can "Delete" a Folder. The command is the same.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.

### Participants
1.  **Component**: The common interface (`FileSystemItem`).
2.  **Leaf**: The end object (`File`). Has no children.
3.  **Composite**: The container (`Folder`). Has children. Implements operations by delegating to children.

---

## 4. UML-Style Structure

*   `Component` { `operation()` }
*   `Leaf` implements `Component` { `operation()` }
*   `Composite` implements `Component` { `children[]`, `operation()`, `add()`, `remove()` }

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Component
```java
public interface FileSystemItem {
    void print(String indent);
    int getSize();
}
```

### 2. The Leaf (File)
```java
public class File implements FileSystemItem {
    private String name;
    private int size;

    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }

    public void print(String indent) {
        System.out.println(indent + "File: " + name + " (" + size + "kb)");
    }

    public int getSize() { return size; }
}
```

### 3. The Composite (Folder)
```java
import java.util.ArrayList;
import java.util.List;

public class Folder implements FileSystemItem {
    private String name;
    private List<FileSystemItem> children = new ArrayList<>();

    public Folder(String name) { this.name = name; }

    public void add(FileSystemItem item) { children.add(item); }

    public void print(String indent) {
        System.out.println(indent + "Folder: " + name);
        for (FileSystemItem child : children) {
            child.print(indent + "  "); // Recursion!
        }
    }

    public int getSize() {
        return children.stream().mapToInt(FileSystemItem::getSize).sum();
    }
}
```

### 4. Client
```java
Folder root = new Folder("Root");
Folder video = new Folder("Video");
File movie = new File("Matrix.mkv", 1000);

video.add(movie);
root.add(video);
root.add(new File("Resume.pdf", 1));

// Client treats root exactly like a file
root.print(""); 
System.out.println("Total: " + root.getSize());
```

---

## 8. Advantages

1.  **Uniformity**: Client code is simple. It loops over a list of `Component` without knowing if they are trees or leaves.
2.  **Extensibility**: Adding new Leaf types works automatically with existing Composites.

---

## 9. Disadvantages

1.  **Value Objects Restriction**: You can't enforce type safety easily. (e.g., "A Folder can only contain Files, not other Folders"). The Composite pattern assumes anything goes.
2.  **Performance Check**: `getSize()` on the Root of a massive tree triggers a full graph traversal. Cache it if needed.

---

## 14. Interview Questions

### Basic
1.  **What data structure does Composite represent?** (A Tree).
2.  **Can a Composite contain another Composite?** (Yes, that's the whole point. Recursive structure).

### Intermediate
3.  **Does the Leaf need to implement `add/remove` methods?** (Debatable. 
    *   *Transparency*: Yes, but they throw Exceptions. 
    *   *Safety*: No, separate interfaces. GoF prefers Transparency).

### Advanced
4.  **How do you handle Parent Pointers?** (Sometimes a Leaf needs to know its Parent to delete itself. You pass `this` (the Composite) when adding a Leaf).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: HTML DOM.
    *   *Design*: `HTMLElement` (Component). `Div` (Composite). `TextNode` (Leaf). `div.render()` calls render on all children.

2.  **Scenario**: Organization Chart.
    *   *Design*: `Employee` (base). `Manager` (Composite, has list of Employees). `IndividualContributor` (Leaf).

3.  **Scenario**: GUI Toolkit.
    *   *Design*: `Panel` contains `Buttons` and other `Panels`.

---

## 16. Summary & Architect Takeaways

*   **Recursion is magical**: Writing one method `render()` that draws an entire UI tree is elegant.
*   **Don't overcomplicate**: If the hierarchy is only 1 level deep (List of Items), you don't need Composite. You just need a List.

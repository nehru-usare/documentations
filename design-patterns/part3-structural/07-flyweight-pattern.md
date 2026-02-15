# Flyweight Pattern

> **Part 3: Structural Patterns**  
> **Difficulty:** ⭐⭐⭐⭐ (Performance)  
> **Status:** Low Level Optimization

---

## 0. Learning Objectives

*   **Beginner**: Understand why `Integer a = 100; Integer b = 100; a == b` is True, but `1000 == 1000` is False.
*   **Developer**: Optimize memory usage for massive object counts (Particles, Text Characters).
*   **Architect**: Design systems that cache immutable shared data.

---

## 1. Problem Statement

### The "Million Objects" Problem
You are making a text editor.
You have a document with 1,000,000 characters.
If you do `new Character('a', font, color, size)` for every single char, you will crash RAM.
*   **Observation**: The letter 'a' in size 12 Arial is the same everywhere.
*   **Solution**: Create ONE instance of `Character('a')`. Share it 50,000 times.
*   **State Separation**:
    *   **Intrinsic**: 'a' (Shared).
    *   **Extrinsic**: X, Y position on screen (Passed as argument, not stored in object).

---

## 2. Real-World Analogy

**Public Library**
*   **Book**: "Harry Potter".
*   There are 5 physical copies.
*   But there are 1000 "Loans".
*   The Library doesn't buy a new book for every loan. It tracks `(BookID, User, Date)`. The `Book` object is the Flyweight (shared).

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Use sharing to support large numbers of fine-grained objects efficiently.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Menu (Flyweight) - Intrinsic State
```java
// Immutable Shared Object
class TreeType {
    private String name;
    private String color;
    private String texture; // Heavy data

    public TreeType(String name, String color, String texture) {
        this.name = name;
        this.color = color;
        this.texture = texture;
    }

    public void draw(int x, int y) {
        System.out.println("Drawing " + name + " at (" + x + ", " + y + ")");
    }
}
```

### 2. The Factory (Cache)
```java
import java.util.HashMap;
import java.util.Map;

class TreeFactory {
    static Map<String, TreeType> treeTypes = new HashMap<>();

    public static TreeType getTreeType(String name) {
        if (!treeTypes.containsKey(name)) {
            treeTypes.put(name, new TreeType(name, "Green", "TextureBytes"));
        }
        return treeTypes.get(name);
    }
}
```

### 3. The Context (Client) - Extrinsic State
```java
class Forest {
    public void plantTree(int x, int y, String name) {
        TreeType type = TreeFactory.getTreeType(name); // Reuses object
        type.draw(x, y); // Passes extrinsic state (x,y)
    }
}
```

---

## 6. Java/Spring Internal Examples

### 1. String Pool
Strings in Java are Flyweights.
```java
String a = "Hello";
String b = "Hello";
// a == b is true. They point to the exact same memory address.
```

### 2. Integer Cache
```java
Integer x = 127;
Integer y = 127;
// x == y is true. Java caches Integers from -128 to 127.
Integer z = 128; // New object.
```

---

## 8. Advantages

1.  **Memory RAM**: Huge savings. 1GB -> 50MB.
2.  **Performance**: Less GC pressure (fewer objects to collect).

---

## 9. Disadvantages

1.  **Complexity**: Code logic is harder (Splitting Intrinsic/Extrinsic).
2.  **CPU Overhead**: Calculating extrinsic state on the fly takes CPU cycles.

---

## 14. Interview Questions

### Basic
1.  **What is the goal of Flyweight?** (Save RAM by sharing objects).
2.  **What is Intrinsic vs Extrinsic state?** (Intrinsic = Shared/Constant. Extrinsic = Context/Changing).

### Intermediate
3.  **Why must Flyweights be immutable?** (Because they are shared. If you change the 'a' character to 'b', it changes for everyone).

---

## 16. Summary & Architect Takeaways

*   **Only for Scale**: Don't use this unless you have > 10,000 objects.
*   **Immutability is key**: You cannot share mutable objects.

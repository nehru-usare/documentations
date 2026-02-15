# Visitor Pattern

> **Part 4: Behavioral Patterns**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The Hardest Pattern

---

## 0. Learning Objectives

*   **Beginner**: Don't use this. It's too complex.
*   **Developer**: Use it for AST (Abstract Syntax Tree) traversal or File System exports.
*   **Architect**: Enable adding new operations to stable class hierarchies.

---

## 1. Problem Statement

### The "New Method" Problem
You have a stable hierarchy: `Rectangle`, `Circle`, `Triangle`.
You want to add `exportToXml()`.
*   **Simple Way**: Add `exportToXml()` method to all 3 classes.
*   **Problem**: You just modified stable code. SRP violation (Shapes shouldn't know XML).
*   **Better Way**: Create a `XmlExportVisitor`. Pass the Shape to the Visitor.

---

## 2. Real-World Analogy

**Inspector**
*   Building types: `House`, `Factory`, `Office`.
*   **Inspector 1 (Fire)**: Visits House (check alarm), Factory (check chemicals).
*   **Inspector 2 (Insurance)**: Visits House (check value), Factory (check risk).
*   The buildings don't change. The Inspectors change operations.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.

### Double Dispatch
1.  Client calls `element.accept(visitor)`.
2.  Element calls `visitor.visit(this)`.
*   Why? Because Java overrides based on *runtime type* of the receiver, but overloads based on *static type* of arguments. Double Dispatch fixes this method resolution.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. The Element Interface
```java
interface Shape {
    void accept(Visitor v);
}
```

### 2. Concrete Elements
```java
class Circle implements Shape {
    public void accept(Visitor v) { v.visit(this); } // "this" is Circle
}

class Dot implements Shape {
    public void accept(Visitor v) { v.visit(this); } // "this" is Dot
}
```

### 3. The Visitor Interface
```java
interface Visitor {
    void visit(Circle c);
    void visit(Dot d);
}
```

### 4. Concrete Visitor
```java
class XmlExportVisitor implements Visitor {
    public void visit(Circle c) { System.out.println("<circle>"); }
    public void visit(Dot d) { System.out.println("<dot>"); }
}
```

### 5. Client
```java
List<Shape> shapes = List.of(new Circle(), new Dot());
Visitor export = new XmlExportVisitor();

for (Shape s : shapes) {
    s.accept(export); // Double Dispatch magic happens here
}
```

---

## 9. Disadvantages

1.  **Complexity**: Very high.
2.  **Rigidity**: If you add a new Shape (`Triangle`), you must edit the `Visitor` interface and ALL concrete visitors. (OCP violation for Element hierarchy).

---

## 16. Summary & Architect Takeaways

*   **Use rarely**: Only for stable structures like Syntax Trees (Compilers).

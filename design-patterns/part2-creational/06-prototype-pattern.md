# Prototype Pattern

> **Part 2: Creational Patterns**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Performance Optimization

---

## 0. Learning Objectives

*   **Beginner**: Understand that "Cloning" is an alternative to "New".
*   **Developer**: Implement `clone()` correctly (Deep vs Shallow copy).
*   **Architect**: Use Prototype to optimize expensive object creation.

---

## 1. Problem Statement

### The Expensive Creation
Imagine a `DatabaseConfig` object that takes 2 seconds to initialize (Connects to DB, reads Schema, validates Metadata).
*   If you need a second `DatabaseConfig` (just to change 1 field), calling `new` takes another 2 seconds.
*   **Solution**: Copy the bits from memory (Nanoseconds). Change the 1 field.

---

## 2. Real-World Analogy

**Mitosis (Cell Division)**
*   A cell doesn't "build" a new cell layer by layer.
*   It duplicates its DNA and splits. This is cloning.

**Ctrl+C / Ctrl+V**
*   You don't re-type the whole document. You copy it and edit the specific paragraph.

---

## 3. Core Concept (Beginner Level 🟢)

### Definition
Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

### Shallow vs Deep Copy
*   **Shallow**: Copies the pointers. If Object A points to List L, Clone B points to same List L. (Dangerous).
*   **Deep**: Copies the data. Clone B gets a new List L2 with same data.

---

## 5. Java Implementation (Developer Level 🟡)

### 1. Java's `Cloneable` (The Bad Way)
Java's native cloning is notoriously broken (doesn't call constructor, throws Checked Exception).

### 2. Copy Constructor (The Good Way)
```java
public class User {
    private String name;
    private List<String> roles;

    public User(String name, List<String> roles) {
        this.name = name;
        this.roles = new ArrayList<>(roles);
    }

    // Copy Constructor
    public User(User source) {
        this.name = source.name;
        // DEEP COPY - Important!
        this.roles = new ArrayList<>(source.roles); 
    }

    public User clone() {
        return new User(this);
    }
}
```

### 3. Usage
```java
User u1 = new User("Admin", List.of("READ", "WRITE"));
User u2 = u1.clone(); // or new User(u1);
```

---

## 6. Spring Boot Implementation

Spring uses the term **Prototype Scope**, but it's *not* exactly the GoF Prototype Pattern.

*   `@Scope("prototype")`: Spring creates a **NEW** instance (calling constructor) every time. It does **NOT** clone an existing one.
*   However, you can use `BeanUtils.copyProperties(source, target)` to implement cloning easily.

---

## 7. Internal Mechanics (Architect Level 🔴)

### Serialization for Deep Copy
A common hack to get a perfect Deep Copy is to Serialize to JSON/ByteStream and Deserialize back.
```java
User copy = objectMapper.readValue(objectMapper.writeValueAsString(original), User.class);
```
*   **Performance**: Slow. Only use if the object graph is complex and manual Deep Copy is too hard.

### Security Risk
*   If you clone a Singleton, you break the Singleton pattern.
*   `Cloneable` bypasses the Constructor. Validation logic in the Constructor is **skipped**. This can lead to invalid objects.

---

## 8. Advantages

1.  **Performance**: `clone()` is usually faster than `new` for complex objects.
2.  **Decoupling**: Client code doesn't need to know the concrete class of the object it clones.

---

## 9. Disadvantages

1.  **Circular References**: Deep cloning objects with `A -> B -> A` loops leads to StackOverflow.
2.  **Constructor Bypass**: Missing initialization logic.

---

## 14. Interview Questions

### Basic
1.  **What is the difference between Shallow and Deep copy?** (Shallow copies references. Deep copies values).
2.  **Why avoid `Cloneable` interface in Java?** (Does not invoke constructor, confusing semantics, checked exception).

### Intermediate
3.  **How to perform a Deep Copy in Java?** (Copy Constructor, Serialization, or manual recursion).
4.  **Is `String` cloned in a shallow copy?** (It is copied by reference, but since String is Immutable, it acts like a Deep copy. It's safe).

### Advanced
5.  **How does Prototype pattern help with "Registry" pattern?** (You can keep a Map of default configurations [Prototypes]. When a user asks for "Config A", you clone it).

---

## 15. Scenario-Based Design Problems

1.  **Scenario**: Game Development (Spawner).
    *   *Design*: You have a definition of "Orc Warrior" (Mesh, Textures, Stats).
    *   *Action*: When spawning 100 Orcs, don't load Texture from disk 100 times. Load once. Clone 99 times.

2.  **Scenario**: Transactional Undo.
    *   *Design*: Before applying a change, Clone the State. If user clicks "Undo", restore the Clone.

---

## 16. Summary & Architect Takeaways

*   **Avoid `implements Cloneable`**. Use Copy Constructors.
*   **Immutable Objects**: The best objects. If an object is immutable, you don't even *need* to clone it usually (just share the reference).
*   **Performance**: Identify the bottleneck properly. Don't use Prototype unless object creation is PROVEN to be slow.

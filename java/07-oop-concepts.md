# 🏛️ OOP Concepts: From Inheritance to Data-Oriented Design

> **Document Level:** Architect (15+ years experience)  
> **Focus:** VTable Lookup, Composition vs. Inheritance, and Sealed Domain Modeling

---

## 🏗️ Layer 1: The Core Pillars (The Traditional View)

Every junior knows Abstraction, Encapsulation, Inheritance, and Polymorphism. An architect looks at how they are **Implemented in the JVM**.

### 1. Polymorphism & Virtual Method Lookup (VTable)
How does the JVM know which method to call when you have `Shape s = new Circle()`?
- **The Mechanism**: Each class has a **Virtual Method Table (vtable)** stored in its metadata (Metaspace). 
- **Method Invocation**: The JVM instruction `invokevirtual` looks up the method's offset in the vtable of the **actual object's class** at runtime.
- **Performance**: This lookup is extremely fast (O(1)), and the JIT often "Devirtualizes" or "Inlines" these calls if it sees only one implementation is ever used (**Monomorphic Call Site**).

---

## 🛠️ Layer 2: The Architectural Pivot - Composition over Inheritance

"Inheritance is the tightest coupling you can have."

### 1. The Fragile Base Class Problem
If you inherit from a class you don't control, and that class adds a new method with the same name as one of yours, your code might break or behave unexpectedly.
- **Architect Rule**: **Favor Composition.** Build objects by combining smaller, focused components instead of deep class hierarchies.

### 2. Interface Segregation (The 'I' in SOLID)
Don't force a class to implement methods it doesn't need. 
- **Modern Java**: Use **Functional Interfaces** and **Lambdas** to "Inject" behavior instead of inheriting it.

---

## 🚀 Layer 3: Modern OOP - Sealed Classes & Domain Modeling

Inheritance wasn't always intended for "Code Reuse"; it was intended for **Type Modeling**.

### 1. Sealed Classes (Java 17+)
Historically, if a class was `public`, anyone could extend it. `final` stopped everyone. **Sealed** classes give you a middle ground.
```java
public sealed interface PaymentMethod 
    permits CreditCard, PayPal, Crypto {}
```
- **Architect Benefit**: You define the **Closed Set** of possibilities. This allows for **Exhaustive Switch** blocks where the compiler ensures you handle every type of payment.

---

## 📜 Layer 4: Encapsulation vs. Records

Is a `record` an anti-pattern because it has public accessors?

### 1. The Data-Object vs. Service-Object Distinction
- **Service Objects**: Hide state, expose behavior (Traditional OOP/Encapsulation).
- **Data Objects (Records)**: Transparent carriers of immutable data. 
- **Architect Thinking**: Use **Records** for DTOs and API responses. Use **Classes** with private fields for your business logic and stateful services.

---

## 🧭 Interview Prep & Architect Scenarios

### Q: Why doesn't Java support Multiple Inheritance of Classes?
**A**: To avoid the **Diamond Problem** (ambiguity if two parents have same method). Java solves this by allowing multiple inheritance of **Interfaces**, where the compiler forces you to override the method if a conflict occurs in `default` methods.

### Q: How does "Abstraction" differ from "Encapsulation" in an API design?
**A**: Abstraction is about **"What it does"** (the Interface/Contract). Encapsulation is about **"How it hides"** (the access modifiers/private state). An architect focuses on Abstraction to decouple systems, and Encapsulation to maintain local invariants.

---

## 🧭 Navigation

| Direction | File | Description |
| :--- | :--- | :--- |
| ⬅️ **Back** | [06-classes-and-objects.md](./06-classes-and-objects.md) | Classes |
| ⏩ **Next** | [08-advanced-oop-topics.md](./08-advanced-oop-topics.md) | Advanced OOP |

**Author:** Nehru Usare  
**Version:** 2.0 | Expanded February 2026
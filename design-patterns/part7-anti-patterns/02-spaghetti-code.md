# Spaghetti Code

> **Part 7: Anti-Patterns**  
> **Severity:** 🔴 Critical  
> **Status:** Unreadable Flow

---

## 1. The Symptom
```java
public void process() {
    if (a) {
        if (b) {
            // ...
        } else {
            if (c) {
                 // ...
            }
        }
    } else {
        while (d) {
             if (e) break;
             // ...
        }
    }
}
```
*   **Cyclomatic Complexity**: > 10.
*   **Readability**: Zero.
*   **Debuggability**: Impossible. You can't trace the path.

---

## 2. The Cause
1.  **Rush**: "Just add this one flag to the if-statement." (Repeated 50 times).
2.  **No Design**: Writing code without planning the flow.
3.  **Global State**: Methods relying on variables changed by other methods.

---

## 3. The Refactoring Strategies

### 1. Guard Clauses (Fail Fast)
Flatten nested `if`s.
```java
// Bad
if (user != null) {
    if (user.isActive()) {
        doWork();
    }
}

// Good
if (user == null) return;
if (!user.isActive()) return;
doWork();
```

### 2. Strategy Pattern
Replace giant `switch` statements with Polymorphism.

### 3. Extract Method
Take the code inside the deepest `if` and make it a private method with a descriptive name.

---

## 4. Architect Takeaway
*   **Arrow Code**: If your code looks like an arrow `>` (deep indentation), refactor immediately.
*   **Max Indentation**: Stick to a rule of **max 3 levels** of indentation per method.

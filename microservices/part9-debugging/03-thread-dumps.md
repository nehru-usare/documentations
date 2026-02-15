# 03. Thread Dumps & Heap Dumps (VisualVM, Eclipse MAT)

> **Part 9: Debugging & Troubleshooting**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The Last Resort

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Capture a Thread Dump using `jstack`. |
| **Developer** | Identify a Deadlock in a Thread Dump. |
| **Architect** | Analyze a 10GB Heap Dump to find the leak. |

---

## 1. Why This Topic Exists

### The Frozen App
The CPU is 0%, but the app is not responding.
*   **Cause**: All threads are WAITING (blocked on DB, blocked on Lock).
*   **Fix**: You need to see *inside* the JVM. Logs won't help (the thread is stuck, so it can't log).

### The OOM (OutOfMemoryError)
`java.lang.OutOfMemoryError: Java heap space`.
*   **Cause**: You have a list that keeps growing and never shrinks.
*   **Fix**: You need to see *what objects* are filling the RAM.

---

## 3. Core Concepts (🟢 Beginner Level)

### Thread Dump (`jstack PID`)
A text file showing the stack trace of every thread.
*   **State**: `RUNNABLE` (Working), `WAITING` (Idle), `BLOCKED` (Stuck on Lock).
*   **Deadlock Detection**: Modern JVMs print "Found one Java-level deadlock:" at the bottom.

### Heap Dump (`jmap -dump:format=b,file=heap.bin PID`)
A binary file containing every object, variable, and reference.
*   **Warning**: Dumping a 10GB heap takes time and freezes the app (Stop-the-World).

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Eclipse MAT (Memory Analyzer Tool)
The standard tool for Heap Analysis.
1.  **Histogram**: "You have 10 million String objects".
2.  **Dominator Tree**: "Who is holding these Strings?" -> "Oh, it's the `static final List<User> cache` in `UserService`".
3.  **Leak Suspect Report**: "One instance of `CacheManager` occupies 90% of heap".

---

## 14. Summary & Architect Takeaways

*   **Dumping in Prod**: Use `jcmd` or `jmap`. VisualVM requires JMX (insecure).
*   **XX:+HeapDumpOnOutOfMemoryError**: Always enable this flag in Production. If it crashes, at least you have the evidence (the `.hprof` file).

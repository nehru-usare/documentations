# Threads: Lifecycle and OS Mapping

> **Part 3: Concurrency**  
> **Level:** Principal Engineer  
> **Status:** Running

---

## 0. Learning Objectives

*   **Developer**: Thread states (BLOCKED vs WAITING).
*   **Senior**: The 1:1 Mapping to Kernel Threads (Platform Threads).
*   **Architect**: Sizing thread pools based on CPU Cores vs I/O Wait.

---

## 1. Deep Technical Explanation: The Platform Thread

Until Java 21, every `java.lang.Thread` is a wrapper around an **OS Kernel Thread**.
*   **Stack Size**: 1MB (Default).
*   **Cost**: Creating a thread involves a System Call (`pthread_create`). Expensive.
*   **Context Switch**: When CPU switches threads, it must save registers and flush caches. Expensive (~1-10µs).
*   **Scalability Limit**: You cannot have 1 million threads. (1MB * 1M = 1TB RAM).

---

## 2. Thread Lifecycle (States)

1.  **NEW**: Created, not started.
2.  **RUNNABLE**: Executing (or waiting for CPU time slice).
3.  **BLOCKED**: Waiting for a Monitor Lock (`synchronized`).
4.  **WAITING**: Waiting for a signal (`Object.wait()`, `Thread.join()`).
5.  **TIMED_WAITING**: `Thread.sleep(1000)`.
6.  **TERMINATED**: Finished.

---

## 3. Production Debugging Guide

### Interrupts
*   **Mechanism**: `t.interrupt()` sets a boolean flag. It does NOT stop the thread.
*   **Cooperation**: The thread must check `Thread.interrupted()` or catch `InterruptedException`.
*   **Anti-Pattern**: Swallowing InterruptedException.
    ```java
    catch (InterruptedException e) {
        // BAD: The interrupt flag is cleared!
    }
    // GOOD:
    catch (InterruptedException e) {
        Thread.currentThread().interrupt(); // Restore flag
    }
    ```

---

## 4. Summary & Architect Takeaways

1.  **Threads are heavy**: Treat them as expensive resources (Pool them).
2.  **Responsiveness**: Long-running tasks on the UI thread (or Request thread) kill responsiveness.
3.  **Daemon Threads**: `t.setDaemon(true)` dies when the main app exits. Use for background cleaners.

---
*Next Chapter: Synchronization and Locks.*

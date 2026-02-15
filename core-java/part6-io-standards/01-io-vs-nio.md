# IO vs NIO: Streams, Channels, and Buffers

> **Part 6: Advanced I/O & Standards**  
> **Level:** Principal Engineer  
> **Status:** Buffered

---

## 0. Learning Objectives

*   **Developer**: Reading a file with `Files.readString()`.
*   **Senior**: The difference between `InputStream` and `ByteBuffer`.
*   **Architect**: Designing a high-throughput server using `Selector` (Reactor Pattern).

---

## 1. IO (java.io) - The Stream Model

### Characteristics
*   **Stream-Oriented**: Reads byte-by-byte (or char-by-char).
*   **Blocking**: `input.read()` blocks the thread until data arrives.
*   **Heap Implications**: Often creates many short-lived byte arrays.

### Use Case
*   Simple CLI tools.
*   Processing data sequentially where latency is not critical.

---

## 2. NIO (java.nio) - The Buffer Model

New I/O (Java 1.4+).

### 2.1 Buffers vs Streams
*   **Buffer**: A block of memory (Heap or Direct).
*   **Channel**: A gateway to file/socket. You read *from* Channel *into* Buffer.
*   **Flip**: You must write to buffer, flip it, then read from it. (Tricky state management).

### 2.2 Selectors (Multiplexing)
*   **Problem**: 10,000 clients -> 10,000 threads with blocking IO.
*   **Solution**: **Selector**.
    1.  One Thread monitors 10,000 Channels.
    2.  `selector.select()` blocks until *at least one* channel is ready (Read/Write).
    3.  Thread processes only active channels.
*   **Result**: Handle 10k connections with 1 Thread. (Node.js / Netty architecture).

---

## 3. Deep Concept: Zero-Copy

### FileChannel.transferTo()
*   **Standard IO**: Disk -> OS Kernel Buffer -> CPU Copy -> User Space Buffer -> CPU Copy -> Kernel Socket Buffer -> NIC. (4 Copies, 2 Context Switches).
*   **Zero-Copy**: Disk -> OS Kernel Buffer -> DMA Copy -> NIC. (0 CPU Copies).
*   **Java**: `fileChannel.transferTo(position, count, socketChannel)`.
*   **Performance**: Massive throughput gain for static file servers (Kafka uses this).

---

## 4. Summary & Architect Takeaways

1.  **Use `java.nio.file.Files`**: For all file operations. It's safer and uses NIO internally.
2.  **Netty is King**: Don't write raw NIO Selectors. It's notoriously hard to get right. Use Netty.
3.  **Direct Buffers**: Allocate outside Heap. No GC. Great for long-lived buffers.

---
*Next Chapter: The perils of Java Serialization.*

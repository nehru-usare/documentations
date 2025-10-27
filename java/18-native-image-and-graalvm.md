# 🚀 Java 21 — Native Image and GraalVM Guide

GraalVM represents the **modern evolution of the JVM** — combining polyglot execution, ahead-of-time (AOT) compilation, and advanced optimizations for cloud-native Java.

This guide teaches how to use **GraalVM** and **Native Image** to compile Java 21 applications into **standalone executables** — with **fast startup**, **low memory usage**, and **instant scalability**.

---

## 🧭 Table of Contents

1. [What Is GraalVM?](#1-what-is-graalvm)
2. [Why Native Images?](#2-why-native-images)
3. [Installing GraalVM](#3-installing-graalvm)
4. [Native Image — Overview and Benefits](#4-native-image--overview-and-benefits)
5. [Building a Native Image](#5-building-a-native-image)
6. [Native Image Options and Flags](#6-native-image-options-and-flags)
7. [Comparing JVM vs Native Execution](#7-comparing-jvm-vs-native-execution)
8. [Reflection, Proxy, and Resource Configuration](#8-reflection-proxy-and-resource-configuration)
9. [Native Image in Spring Boot 3+](#9-native-image-in-spring-boot-3)
10. [GraalVM Polyglot Capabilities](#10-graalvm-polyglot-capabilities)
11. [Performance and Observability](#11-performance-and-observability)
12. [Official Resources and References](#12-official-resources-and-references)
13. [Summary](#13-summary)

---

## 1️⃣ What Is GraalVM?

**GraalVM** is a **high-performance JDK** built by Oracle Labs, featuring:
- Advanced JIT (Just-In-Time) compiler  
- **Ahead-of-Time (AOT)** compilation (Native Image)  
- Polyglot execution (Java, JavaScript, Python, R, LLVM languages)  
- JVM interoperability and low-level optimization

> 💡 In simple terms: GraalVM = **Faster JVM + Optional Native Binary + Multi-language runtime**.

Official site: [https://www.graalvm.org/](https://www.graalvm.org/)

---

## 2️⃣ Why Native Images?

Native Image turns Java bytecode into a **fully compiled executable** — no JVM required at runtime.

| Feature | JVM | Native Image |
|----------|-----|---------------|
| Startup time | 1–2 seconds | 10–50 ms |
| Memory usage | 200–400 MB | 40–80 MB |
| Warm-up time | Needed for JIT | None |
| GC | Yes (Substrate VM GC) | Yes, minimal |
| Deployment | Requires JDK | Standalone binary |

✅ Ideal for:
- Microservices  
- Serverless functions  
- CLI tools  
- Containerized deployments  

---

## 3️⃣ Installing GraalVM

### 🔹 Step 1: Download GraalVM
From [GraalVM Downloads](https://www.graalvm.org/downloads/)

Choose:
- **Java 21 (JDK 21 based)**  
- **Community or Enterprise Edition**

### 🔹 Step 2: Set Environment Variables
```bash
export GRAALVM_HOME=/usr/lib/graalvm-jdk-21
export PATH=$GRAALVM_HOME/bin:$PATH
````

### 🔹 Step 3: Install Native Image Tool

```bash
gu install native-image
```

Check installation:

```bash
native-image --version
```

✅ Output:

```
GraalVM 21.0.0 Java 21 CE (native-image)
```

---

## 4️⃣ Native Image — Overview and Benefits

Native Image performs **Ahead-of-Time (AOT)** compilation:

* Builds a binary from `.class` and `.jar` files
* Embeds required classes, metadata, and GC
* Removes unused code (tree-shaking)

✅ Advantages:

* Instant startup
* Low memory footprint
* Faster cold-start for serverless/cloud
* No dependency on JVM at runtime

> ⚙️ The binary runs directly on the OS — no bytecode interpretation or JIT needed.

---

## 5️⃣ Building a Native Image

Example simple app:

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello from Native Java!");
    }
}
```

### Compile to class:

```bash
javac Hello.java
```

### Create Native Image:

```bash
native-image Hello
```

✅ Output:

```
Hello
```

Run:

```bash
./hello
```

Output:

```
Hello from Native Java!
```

---

## 6️⃣ Native Image Options and Flags

| Option                                     | Description                     | Example                            |
| ------------------------------------------ | ------------------------------- | ---------------------------------- |
| `--no-fallback`                            | Fails if image build incomplete | `native-image --no-fallback MyApp` |
| `--enable-https`                           | Include HTTPS libraries         | `--enable-https`                   |
| `--initialize-at-build-time`               | Pre-initialize classes          | For static config                  |
| `--report-unsupported-elements-at-runtime` | Avoid build-time errors         | Recommended                        |
| `-H:+PrintClassInitialization`             | Debug class init order          |                                    |
| `-H:ConfigurationFileDirectories=config/`  | Include reflection config       |                                    |

> 💡 Always add `--no-fallback` in CI builds to ensure full AOT success.

---

## 7️⃣ Comparing JVM vs Native Execution

| Metric          | JVM                    | Native                   |
| --------------- | ---------------------- | ------------------------ |
| Startup         | 2s                     | 0.05s                    |
| Memory          | 300MB                  | 60MB                     |
| Peak Throughput | Higher (JIT-optimized) | Slightly lower           |
| Warm-up         | Needed                 | None                     |
| Ideal Use       | Long-running servers   | Microservices, functions |

> 💡 Native images are **faster to start**, but **slightly slower in long runs** (no JIT re-optimization).

---

## 8️⃣ Reflection, Proxy, and Resource Configuration

Because AOT compilation analyzes code **statically**, you must provide configuration files for:

* Reflection
* Dynamic proxies
* JNI resources

### Example Reflection Config (reflection-config.json)

```json
[
  {
    "name": "com.example.User",
    "allDeclaredConstructors": true,
    "allDeclaredMethods": true
  }
]
```

Pass during build:

```bash
native-image -H:ReflectionConfigurationFiles=reflection-config.json MyApp
```

> ✅ Tools like `tracing-agent` can **automatically detect** reflection usage.

---

## 9️⃣ Native Image in Spring Boot 3+

Spring Boot 3.x officially supports **GraalVM Native Images**.

### Build Steps:

```bash
./mvnw -Pnative native:compile
```

Run binary:

```bash
./target/myapp
```

✅ Spring Boot 3 + GraalVM reduces cold-start time to **< 50 ms**.

> 🧩 Recommended for serverless (AWS Lambda, Azure Functions, GCP Cloud Run).

---

## 🔟 GraalVM Polyglot Capabilities

GraalVM supports multiple languages in one runtime.

### Example:

```java
import org.graalvm.polyglot.*;

public class PolyglotDemo {
    public static void main(String[] args) {
        try (Context ctx = Context.create()) {
            ctx.eval("js", "print('Hello from JavaScript!')");
        }
    }
}
```

✅ Run Python, JS, R, Ruby, or LLVM code inside Java.
✅ Share data between languages via GraalVM Context.

> 💡 GraalVM = JVM + Polyglot Runtime + Optimizing Compiler.

---

## 11️⃣ Performance and Observability

### 🔹 Measure Native Image Performance

```bash
time ./myapp
```

### 🔹 Enable Profiling

```bash
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintCompilation -XX:+PrintGC
```

For Native Images, use:

* **`--verbose`** for build-time tracing
* **`--print-analysis-call-tree`** for dependency analysis

> ✅ Combine GraalVM builds with **JFR**, **async-profiler**, or **Micrometer** for observability.

---

## 12️⃣ Official Resources and References

📘 **GraalVM & Native Image**

* [GraalVM Official Documentation](https://www.graalvm.org/)
* [Native Image User Guide](https://www.graalvm.org/latest/reference-manual/native-image/)
* [Spring Boot 3 Native Image Docs](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html)
* [GraalVM GitHub Repo](https://github.com/oracle/graal)
* [AOT Compilation Guide](https://docs.oracle.com/en/graalvm/enterprise/21/docs/reference-manual/native-image/)

📊 **Tools & Monitoring**

* [Micrometer Native Metrics](https://micrometer.io/)
* [Async Profiler](https://github.com/jvm-profiling-tools/async-profiler)
* [JITWatch (for JIT comparison)](https://github.com/AdoptOpenJDK/jitwatch)

---

## 13️⃣ Summary

You’ve learned how to:

* Build **native executables** with GraalVM
* Use **AOT compilation** to remove JVM dependency
* Optimize cloud and serverless Java deployments
* Configure reflection and dynamic features
* Explore **polyglot programming** inside the JVM

> ⚙️ **Native Image = Java that starts like Go, runs like C, and scales like the cloud.**
> Java 21 + GraalVM makes it possible to combine the maturity of the JVM with the performance of native binaries.

> 🧭 **Next (Optional Advanced):**
> [19-cloud-native-java-performance.md → Java 21 Cloud-Native Optimization (Containers, Kubernetes, and Observability)](./19-cloud-native-java-performance.md)
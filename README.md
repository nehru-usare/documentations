# 📚 Developer Documentation Repository & Architect Handbooks

Welcome to the **Developer Documentation Repository** — a comprehensive, architect-level knowledge base for building scalable, distributed systems.

This repository has evolved from basic tutorials to a full-fledged **Software Engineering Encyclopedia**, covering everything from Linux internals to advanced System Design patterns.

---

## 🏗️ The Architect Handbooks (New!)

These six handbooks are the core of this repository, designed to take you from **Junior Developer** to **Principal Architect**.

### 1. [Microservices Engineering Handbook](./microservices/)
> *Mastering distributed system Architecture with Spring Boot & Cloud.*
*   **Part 1**: Foundations (DDD, CAP).
*   **Part 2**: Spring Cloud (Gateway, Eureka).
*   **Part 3**: Resilience (Circuit Breaker).
*   **Part 4**: Data Consistency (Saga, CQRS).
*   **Part 5**: Infrastructure (Docker, K8s).
*   **Part 10**: Interview Kit.

### 2. [System Design Engineering Handbook](./system-design/)
> *Designing large-scale systems like WhatsApp, Netflix, and Uber.*
*   **Part 1**: Foundations (Scalability, Reliability).
*   **Part 3**: Core Components (LB, API Gateway).
*   **Part 6**: Distributed Patterns (Bloom Filters, Gossip).
*   **Part 9**: Scenarios (Chat, URL Shortener).
*   **Part 10**: Appendix (Cheatsheets).

### 3. [Design Patterns Engineering Handbook](./design-patterns/)
> *Advanced Java & Spring Patterns, SOLID principles, and Anti-Patterns.*
*   **Part 1**: Foundations (SOLID).
*   **Part 2-4**: GoF Patterns (Factory, Strategy, Observer).
*   **Part 5**: Spring Patterns.
*   **Part 6**: Microservices Patterns.

### 4. [DevOps Engineering Handbook](./devops/)
> *The Bridge between Code and Operations.*
*   **[Part 1: Foundations](./devops/part1-foundations/)**: Culture, CALMS, GitOps.
*   **[Part 2: CI/CD](./devops/part2-cicd/)**: Jenkins, GitHub Actions, Blue/Green.
*   **[Part 3: IaC](./devops/part3-iac/)**: Terraform, Ansible, Immutable Infra.
*   **[Part 4: Orchestration](./devops/part4-orchestration/)**: Kubernetes, Helm, Istio.
*   **[Part 5: Observability](./devops/part5-observability/)**: Prometheus, ELK, OpenTelemetry.

### 5. [Engineering Leadership Handbook](./leadership/) (New!)
> *The Process, Culture, and Soft Skills of a Principal Engineer.*
*   **[Part 1: ADRs](./leadership/01-architecture-decision-records.md)**: How to document decisions.
*   **[Part 2: RFCs](./leadership/02-rfc-design-docs.md)**: Writing technical design docs.
*   **[Part 3: Code Review](./leadership/03-code-review-guidelines.md)**: Guidelines for quality.
*   **[Part 4: Tech Debt](./leadership/04-tech-debt.md)**: Managing and paying down debt.

### 6. [Core Java Engineering Handbook](./core-java/) (New!)
> *The Principal Engineer's Guide to the JVM.*
*   **[Part 1: Foundations](./core-java/part1-foundations/)**: Internals (Memory Layout, Pass-by-Value), Generics, Design Patterns (Singleton/Builder).
*   **[Part 2: Collections](./core-java/part2-collections/)**: HashMap Internals (Treeification), ArrayList vs LinkedList (CPU Cache).
*   **[Part 3: Concurrency](./core-java/part3-concurrency/)**: Java Memory Model, Locks, Virtual Threads (Loom).
*   **[Part 4: JVM](./core-java/part4-jvm/)**: GC Algorithms (G1/ZGC), JIT (C1/C2), Memory Model.
*   **[Part 5: Modern Java](./core-java/part5-modern/)**: Streams, Records, Modules (JPMS), Java 21 features (Sequenced Collections).
*   **[Part 6: Advanced I/O](./core-java/part6-io-standards/)**: NIO (Channels/Buffers), Serialization Risks, Date/Time API.
*   **[Part 7: Evolution Guide](./core-java/java-evolution-guide.md)**: A version-by-version cheat sheet (Java 8 to 21).
*   **[📅 30-Day Study Plan](./core-java/study-plan.md)**: A daily schedule to master this handbook.

---

## 🧭 Core Technologies

Beyond the advanced handbooks, this repository contains complete guides for essential tools.

| Technology | Folder | Description |
| :--- | :--- | :--- |
| **☕ Java (Legacy)** | [`java/`](./java/) | The original tutorial-style Java content. |
| **🚀 Spring Boot** | [`spring-boot/`](./spring-boot/) | Building production-ready APIs, Security, and Testing. |
| **💾 Database** | [`database/`](./database/) | **Internals** (ACID, MVCC, B-Trees), Query Optimization, Sharding, NoSQL. |
| **🐧 Linux** | [`linux/`](./linux/) | Shell Scripting, Permissions, Networking, Process Management. |
| **🐳 Docker** | [`docker/`](./docker/) | **Internals** (Namespaces, Cgroups), Security, Multi-Stage Builds. |
| **🌿 Git** | [`git/`](./git/) | **Internals** (.git objects), Advanced Commands (Bisect/Reflog), Hooks. |

---

## 🛠️ Repository Structure

```bash
documentations/
│
├── microservices/    # The Microservices Engineering Handbook
├── system-design/    # The System Design Engineering Handbook
├── design-patterns/  # The Design Patterns Engineering Handbook
├── devops/           # The DevOps Engineering Handbook
├── leadership/       # The Engineering Leadership Handbook (New!)
├── core-java/        # The Core Java Engineering Handbook (New!)
│
├── java/             # Java Core & Advanced
├── spring-boot/      # Spring Boot Architecture
├── database/         # Database Engineering
├── linux/            # Linux System Administration
├── docker/           # Containerization
└── git/              # Version Control
```

---

## 💡 How to Use

1.  **Clone the Repo**:
    ```bash
    git clone https://github.com/nehru-usare/documentations.git
    ```
2.  **Navigate**: Go to the section you want to master.
3.  **Read**: Each folder contains numbered `.md` files in logical order.

---

## 🧾 Author & Credits

📘 **Maintained by**: Nehru Usare  
🎯 **Goal**: To be the definitive reference for Software Architects.  

> "Great developers don't just write code — they understand the system."

---
*Last Updated: 2026*
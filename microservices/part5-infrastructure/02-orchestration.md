# 02. Orchestration (Kubernetes)

> **Part 5: Infrastructure & DevOps**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (SRE)  
> **Status:** The Operating System of the Cloud

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand what K8s does (Scheduling, Healing). |
| **Developer** | Write `deployment.yaml` and `service.yaml`. |
| **Architect** | Design High Availability clusters with Affinity and Taints. |

---

## 1. Why This Topic Exists

### The Orchestration Problem
Docker runs **one** container.
What if you need:
1.  Run 5 copies (Scaling).
2.  Restart if it crashes (Healing).
3.  Deploy across 3 servers (Scheduling).
4.  Zero-Downtime update (Rolling Update).

**Kubernetes (K8s)** does this automatically.

---

## 2. Big Picture Architecture View

```mermaid
graph TD
    User --> Ingress[Ingress Controller]
    Ingress --> Svc[Service (ClusterIP)]
    
    Svc --> Pod1[Pod A]
    Svc --> Pod2[Pod B]
    Svc --> Pod3[Pod C]
    
    subgraph "Node 1"
        Pod1
    end
    
    subgraph "Node 2"
        Pod2
        Pod3
    end
```

---

## 3. Core Concepts (🟢 Beginner Level)

### Pod
The smallest unit. A wrapper around the Container.
*   Pods are ephemeral (Mortal). If they die, they are gone.

### Deployment
Manages Pods.
*   "Make sure 3 replicas of Nginx are running."
*   If a Node dies, Deployment creates new Pods on another Node.

### Service
The stable network address.
*   Since Pod IPs change, Service gives a static IP to group them.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### The YAML Manifests

**Deployment (App)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: orders
  template:
    metadata:
      labels:
        app: orders
    spec:
      containers:
      - name: app
        image: my-registry/orders:v1
        ports:
        - containerPort: 8080
```

**Service (Network)**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: orders
  ports:
  - port: 80
    targetPort: 8080
```

---

## 5. Internal Mechanics (🔴 Architect Level)

### HPA (Horizontal Pod Autoscaler)
Scales based on CPU/Memory.
*   "If CPU > 70%, add more Pods."

### Probes (Health Checks)
1.  **Liveness**: "Am I dead?" (Restart me).
2.  **Readiness**: "Am I ready to take traffic?" (Add me to Load Balancer).
3.  **Startup**: "Am I initialized?" (Don't kill me yet).

---

## 9. Architect-Level Best Practices

1.  **Requests & Limits**: ALWAYS set CPU/Memory limits. If you don't, one Java app can eat 100% RAM and crash the Node (noisy neighbor).
2.  **Namespace Isolation**: Separete `dev`, `qa`, `prod` via Namespaces.
3.  **Helm Charts**: Don't use raw YAMLs. Use Helm for templating (`values.yaml`).

---

## 12. Interview Questions

1.  Difference between Pod and Container?
2.  What happens if a Pod crashes? (Deployment restarts it).
3.  Difference between NodePort and ClusterIP?

---

## 14. Summary & Architect Takeaways

*   **Declarative**: You say "I want 3 replicas". K8s makes it happen. You don't script the "How".
*   **Abstraction**: K8s hides the underlying hardware. You deploy to the "Cluster", not "Server 1".

# Kubernetes Architecture (Control Plane)

> **Part 4: Container Orchestration**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Hard)  
> **Status:** Running

---

## 0. Learning Objectives
*   **Beginner**: Knowing that "Master Node" manages "Worker Nodes".
*   **Developer**: Why your Pod fails to schedule.
*   **Architect**: Designing High Availability (HA) clusters and etcd resilience.

---

## 1. Context
**Kubernetes (K8s)**: An open-source system for automating deployment, scaling, and management of containerized applications.
*   **Philosophy**: You tell K8s what you want (Desired State), and it continuously works to ensure the current state matches it.

---

## 2. Architecture Breakdown (The Control Plane)

The "Brain" of the cluster.

### 1. API Server (`kube-apiserver`)
*   The **Only Entry Point**. All traffic (kubectl, nodes, components) goes here.
*   Stateless, RESTful API.
*   Validates and Configures data for api objects (Pods, Services).

### 2. etcd
*   The **Database**.
*   Distributed Key-Value Store (Consistent, HA).
*   Stores the entire cluster state.
*   *Critical*: If etcd dies, the cluster is brain dead.

### 3. Scheduler (`kube-scheduler`)
*   Decides **Where** to run a Pod.
*   Filters nodes (CPU/RAM available? Taints/Tolerations?).
*   Scores nodes (Best fit?).
*   Binds Pod to Node.

### 4. Controller Manager (`kube-controller-manager`)
*   The **Reconciler**. A loop that watches state.
*   **Node Controller**: Notices node down.
*   **Job Controller**: Watches Job objects.
*   **ReplicaSet**: Ensures 3 replicas are running.

---

## 3. Worker Node Components

The "Muscle" of the cluster.

### 1. Kubelet
*   The agent running on every node.
*   Talks to API Server ("What pods should I run?").
*   Talks to Container Runtime (Docker/Containerd) -> "Start this container".
*   Reports health back.

### 2. Kube-proxy
*   Network Proxy.
*   Maintains network rules on nodes (iptables/IPVS).
*   Allows Services to balance traffic to Pods.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. The Request Flow (Starting a Pod)
1.  `kubectl apply` sends JSON to API Server.
2.  API Server validates + auth -> Writes to **etcd**.
3.  **Scheduler** (Watching API) sees new "Unbound Pod". Selects Node X. Writes binding to API.
4.  **Kubelet** on Node X (Watching API) sees Pod assigned to it.
5.  Kubelet tells Docker to start container.
6.  Kubelet updates Pod Status -> API Server -> etcd.

---

## 5. Trade-Off Analysis

| Deployment | Managed (EKS/GKE/AKS) | Self-Hosted (Kubeadm) |
| :--- | :--- | :--- |
| **Control Plane** | Managed by Cloud (99.95% SLA) | You manage etcd/backups |
| **Upgrades** | One click | Hard/Complex |
| **Cost** | ~$70/month/cluster | Free (Compute only) |
| **Verdict** | **Use Managed**. | Only for Bare Metal / Security constraints. |

---

## 6. Scaling Considerations

### etcd scaling
*   etcd uses **Raft Algorithm**. Needs Quorum (`N/2 + 1`).
*   Deploy 3 or 5 nodes. (Never even numbers).
*   Latency sensitive. Put on fast SSD.

---

## 7. Failure Scenarios & Recovery

### 1. Split Brain (Network Partition)
*   If Control Plane nodes lose touch.
*   etcd stops writing (to preserve consistency).
*   Cluster enters Read-Only mode. Existing Pods keep running. No changes allowed.

---

## 8. Security Considerations

### 1. RBAC (Role Based Access Control)
*   Determine *who* can do *what*.
*   `Role`: Permissions (GET pods).
*   `RoleBinding`: Assign Role to User "Bob".

### 2. Secrets Encryption
*   By default, Secrets in etcd are Base64 (not encrypted).
*   Enable **Encryption at Rest** in K8s config.

---

## 9. Performance Considerations

*   **List Operations**: `kubectl get pods --all-namespaces` is expensive.
*   Puts load on API Server and etcd.
*   Large clusters (5000 nodes) require tuning API Server memory and etcd IOPS.

---

## 10. Real Production Lessons

### "The CrashLoopBackOff"
*   Most common error. Container starts, crashes, K8s restarts it.
*   **Debug**: `kubectl logs` / `kubectl describe pod`.

---

## 11. Interview Questions

### Basic
1.  What is a Pod?
2.  Components of Control Plane.
3.  What is Kubelet?

### Intermediate
1.  How does `kubectl` find the API server? (Kubeconfig).
2.  Role of etcd?
3.  Explain the scheduling process.

### Advanced
1.  Architect a Multi-Region K8s Cluster (Federation).
2.  How does Kube-proxy implement Service Load Balancing? (iptables vs IPVS).
3.  Debug a scenario where API Server is unresponsive but Pods are running.

---

## 12. Summary & Architect Takeaways

1.  **Complexity**: K8s is complex because distributed systems are complex. It exposes the complexity rather than hiding it.
2.  **API First**: Everything is an API Object. This makes it automatable.
3.  **Self-Healing**: Trust the Controller loops. Don't micromanage.

# Kubernetes DaemonSet Lab

Hands-on Kubernetes DaemonSet project to understand node-level pod deployment, self-healing, node selectors, taints, tolerations, and rolling updates.

---

# Objective

Learn how DaemonSets:

* Deploy one Pod per Node
* Automatically schedule Pods on newly added Nodes
* Maintain desired state through self-healing
* Work with Node Selectors
* Work with Taints and Tolerations
* Support rolling updates
* Power real-world cluster services

---

# What is a DaemonSet?

A DaemonSet is a Kubernetes workload resource that ensures a copy of a Pod runs on every eligible node in a cluster.

As new nodes join the cluster, DaemonSet Pods are automatically created on those nodes.

If nodes are removed, the corresponding Pods are automatically removed.

---

# Why DaemonSet is Needed

Many cluster-level services must run on every node.

Examples:

* Log collection agents
* Monitoring agents
* Security agents
* Storage agents
* Networking components

Instead of manually creating a Pod on every node, Kubernetes uses DaemonSets to automate the process.

---

# Real-World Use Cases

Common DaemonSet workloads:

* Fluentd
* Fluent Bit
* Prometheus Node Exporter
* Datadog Agent
* New Relic Agent
* Falco
* Calico
* Cilium
* Storage CSI Drivers

---

# DaemonSet vs Deployment

| Deployment                             | DaemonSet                                |
| -------------------------------------- | ---------------------------------------- |
| Deploys a specified number of replicas | Deploys one Pod per eligible node        |
| Replica count controlled manually      | Pod count depends on node count          |
| Used for applications                  | Used for node-level services             |
| Examples: Nginx, Spring Boot, React    | Examples: Fluentd, Node Exporter, Calico |
| Scaling requires changing replicas     | Automatically scales with nodes          |
| Pods can run on same node              | One Pod per eligible node                |

---

# Architecture

```text
                Kubernetes Cluster
                        |
        ---------------------------------
        |               |               |
     Node-1          Node-2          Node-3
        |               |               |
   Daemon Pod      Daemon Pod      Daemon Pod
```

One Pod runs on every eligible node.

---

# Prerequisites

* Minikube
* kubectl
* Docker Desktop

Verify cluster:

```bash
kubectl get nodes
```

Example:

```text
minikube
minikube-m02
minikube-m03
```

---

# Repository Structure

```text
k8s-daemonset/
├── README.md
├── daemonset.yaml
├── daemonset-node-selector.yaml
├── daemonset-toleration.yaml
└── .gitignore
```

---

# Basic DaemonSet Deployment

Apply:

```bash
kubectl apply -f daemonset.yaml
```

Verify:

```bash
kubectl get daemonset

kubectl get pods -o wide
```

Expected:

```text
1 Pod per Node
```

Example:

```text
minikube       -> nginx-daemonset
minikube-m02   -> nginx-daemonset
minikube-m03   -> nginx-daemonset
```

---

# Verify Self-Healing

Delete a DaemonSet Pod:

```bash
kubectl delete pod <pod-name>
```

Watch:

```bash
kubectl get pods -w
```

Expected:

```text
Old Pod Deleted
        ↓
New Pod Created Automatically
```

DaemonSet continuously ensures one Pod runs on every eligible node.

---

# Node Selector Integration

## Label a Node

```bash
kubectl label node minikube-m02 logging=true
```

Verify:

```bash
kubectl get nodes --show-labels
```

---

## Deploy Node Selector DaemonSet

Apply:

```bash
kubectl apply -f daemonset-node-selector.yaml
```

YAML:

```yaml
nodeSelector:
  logging: "true"
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
logging-daemonset -> minikube-m02
```

Only nodes matching the label receive Pods.

---

# Taints and Tolerations

## Apply Taint

```bash
kubectl taint nodes minikube-m03 dedicated=logging:NoSchedule
```

Verify:

```bash
kubectl describe node minikube-m03
```

Expected:

```text
dedicated=logging:NoSchedule
```

---

## Effect of Taint

DaemonSets without tolerations cannot schedule Pods on the tainted node.

Example:

```text
minikube       ✓
minikube-m02   ✓
minikube-m03   ✗
```

---

## Deploy DaemonSet with Toleration

Apply:

```bash
kubectl apply -f daemonset-toleration.yaml
```

Toleration:

```yaml
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "logging"
  effect: "NoSchedule"
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
minikube       ✓
minikube-m02   ✓
minikube-m03   ✓
```

The Pod can now run on the tainted node.

---

# Rolling Updates

Update image:

```yaml
image: nginx:alpine
```

Apply:

```bash
kubectl apply -f daemonset.yaml
```

Monitor:

```bash
kubectl rollout status daemonset/nginx-daemonset
```

Verify:

```bash
kubectl get pods -o wide
```

---

# DaemonSet Workflow

```text
DaemonSet
      │
      ▼
Checks Cluster Nodes
      │
      ▼
Schedules One Pod Per Node
      │
      ▼
Maintains Desired State
      │
      ▼
Recreates Failed Pods
      │
      ▼
Schedules Pods on New Nodes
      │
      ▼
Ensures One Pod Per Eligible Node
```

---

# Interview Questions

### 1. What is a DaemonSet?

A DaemonSet ensures one Pod runs on every eligible node.

### 2. When should we use a DaemonSet?

When a service must run on every node.

### 3. Give examples of DaemonSet use cases.

Fluentd, Fluent Bit, Node Exporter, Calico, Datadog Agent.

### 4. Difference between Deployment and DaemonSet?

Deployment manages replicas. DaemonSet manages one Pod per node.

### 5. Does DaemonSet support rolling updates?

Yes.

### 6. Can DaemonSets use Node Selectors?

Yes.

### 7. Can DaemonSets work with Taints and Tolerations?

Yes.

### 8. What happens if a DaemonSet Pod is deleted?

Kubernetes automatically recreates it.

### 9. Does a DaemonSet run on control-plane nodes?

Only if scheduling rules allow it.

### 10. How does DaemonSet scale?

It automatically scales based on the number of eligible nodes.

---

# Key Takeaways

* DaemonSets run one Pod per eligible node.
* New nodes automatically receive Pods.
* Deleted Pods are recreated automatically.
* Node Selectors control Pod placement.
* Taints restrict scheduling.
* Tolerations allow scheduling on tainted nodes.
* DaemonSets are commonly used for monitoring, logging, networking, and security services.

---

# Cleanup

Delete DaemonSets:

```bash
kubectl delete -f daemonset.yaml

kubectl delete -f daemonset-node-selector.yaml

kubectl delete -f daemonset-toleration.yaml
```

Remove label:

```bash
kubectl label node minikube-m02 logging-
```

Remove taint:

```bash
kubectl taint nodes minikube-m03 dedicated=logging:NoSchedule-
```

Verify:

```bash
kubectl get daemonset

kubectl get pods
```
# 🛠️ Essential Kubernetes (K8s) Operational Commands

This cheat sheet contains foundational commands for managing Kubernetes clusters, investigating resources, exposing workloads, and debugging containers internally.

---

## 🏗️ 1. Cluster Diagnostics & Verification

Use these commands to verify your installation status and ensure worker nodes are actively communication with the control plane.

```bash
# Verify cluster status and list all connected physical/virtual machines
kubectl get nodes

# Show detailed node information including internal IPs, OS versions, and kernel details
kubectl get nodes -o wide
```

---

## 📦 2. Pod Management (Lifecycle & Inspections)

Commands used to create, audit, trace, and terminate discrete container workloads.

```bash
# Instantly spin up a standalone container using a specific image
kubectl run nginx-pod --image=nginx

# List all active pods running inside the current namespace
kubectl get pods

# Display detailed runtime telemetry (Node allocation, Pod IPs, Readiness status)
kubectl get pods -o wide

# Permanently delete and clean up a specific pod
kubectl delete pod nginx-pod
```

---

## 🔌 3. Service Mesh & Network Exposure

Commands to view routing tables or expose application pods to internal or external network traffic.

```bash
# List all operational routing paths and services in the current namespace
kubectl get svc

# Expose a specific pod network port onto a host node port for external access
kubectl expose pod nginx-pod --type=NodePort --port=80
```

---

## 🔍 4. In-Pod Diagnostics & Executions

Commands used to execute live troubleshooting, test localized connection loops, or inspect file systems directly inside running containers.

```bash
# Send an HTTP web request locally inside the container to verify if the server is responsive
kubectl exec -it nginx-pod -- curl localhost

# Open an interactive terminal session inside the container filesystem for manual debugging
kubectl exec -it nginx-pod -- bash
```

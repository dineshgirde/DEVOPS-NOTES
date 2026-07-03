# 🌐 Kubernetes Networking: Pods & Services Guide

This module covers the core fundamentals of Kubernetes networking, explaining how containers and pods talk to each other, alongside practical guides for both **Imperative** (command-based) and **Declarative** (manifest-based) approaches.

---

## 🧠 1. Architectural Overview: Core Networking Concepts

Kubernetes enforces a flat network model where every pod receives a unique, routable IP address. Communication is split into two distinct layers:

### A. Intra-Pod Communication (Inside the Same Pod)
*   **Definition:** Communication between different containers running inside the exact same pod.
*   **Mechanism:** All containers within a single pod share the identical **Network Namespace**. 
*   **How it Works:** They share the same IP address and can talk to each other directly over `localhost` using their respective container ports.

### B. Inter-Pod Communication (Between Different Pods)
*   **Definition:** Communication between pods running on the same node or across different worker nodes.
*   **Mechanism:** Because every pod gets a unique cluster IP, pods can communicate directly with one another without using Network Address Translation (NAT), ensuring seamless, decoupled microservices.

---

## 🛠️ 2. The Imperative Approach (Command-Based)

The imperative approach involves executing direct commands against the Kubernetes API. It is ideal for rapid prototyping, debugging, and practice, but is generally avoided in production pipelines.

### Resource Discovery & Creation
```bash
# Check the status of all available cluster worker nodes
kubectl get nodes

# Spin up a standalone pod named 'pod-1' running an Nginx container
kubectl run pod-1 --image=nginx

# List all pods running in your active namespace
kubectl get pods
```
### 📝 Step 1: Create the Manifest File (`pod.yaml`)
Save the following configuration block into a local file named `pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    app: nginx
spec:
  containers:
  - name: cont-1
    image: nginx
    ports:
    - containerPort: 80
```
---

## Exposing Applications via Services types
Pods are ephemeral (temporary). To provide a stable endpoint/IP for your pods, you create a **Service**. You can expose your running pod instantly using these configurations:

#### 1. ClusterIP (Default — Internal Cluster Traffic Only)
Restricts access exclusively to components inside the cluster (e.g., a frontend app talking to a backend database).
```bash
# Expose the pod internally on port 80
kubectl expose pod pod-1 --port=80 --target-port=80 --type=ClusterIP

# Fetch the assigned internal ClusterIP address
kubectl get svc

# Test internal connection by executing an HTTP request from another pod
kubectl exec -it <any-running-pod-name> -- curl <assigned-cluster-ip>
```
### ClusterIP service file
```
apiVersion: v1
kind: Service
metadata:
  name: my-clip-svc
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```


#### 2. NodePort (External Access via Worker Node IP)
Exposes the application externally by binding a static port RANG (typically between 30000–32767) on every worker node's physical IP address.
```bash
# Expose the pod externally across all node machines
kubectl expose pod pod-1 --port=80 --target-port=80 --type=NodePort

# Check the allocated high-range NodePort number
kubectl get svc

# Access path from your web browser:
# http://<Any-Node-IP>:<Allocated-NodePort>
```
### Nodeport service file
```
apiVersion: v1
kind: Service
metadata:
  name: my-np-svc
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 31195
```
#### 3. LoadBalancer (Cloud-Native Public Access)
Integrates natively with cloud providers (AWS, Azure, GCP) to automatically provision a public-facing infrastructure load balancer that routes external traffic directly into your cluster.
```bash
# Expose the pod to the public internet via cloud-provider balancer
kubectl expose pod pod-1 --port=80 --target-port=80 --type=LoadBalancer

# Track the status until an 'External-IP' or DNS record is assigned
kubectl get svc

# Access path from your web browser:
# http://<Assigned-LoadBalancer-Link-Or-IP>
```
### LoadBlancer service file
```
apiVersion: v1
kind: Service
metadata:
  name: my-lb-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```
--- 

### 🚀 Step 2: Apply the Resource State
Execute the tracking manifest against your cluster control plane using the following command:

```bash
# Submit the file configuration to Kubernetes
kubectl apply -f pod.yaml
```

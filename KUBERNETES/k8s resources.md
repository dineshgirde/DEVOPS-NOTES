# ☸️ Comprehensive Kubernetes (K8s) Resource, Workloads & Scaling Manual

This repository serves as a production-ready reference manual for core Kubernetes resources, workloads, networking models, deployment configurations, and resource scaling architectures.

---

## 🗺️ 1. Core Architecture & Workload Types

Kubernetes abstracts infrastructure by splitting applications into distinct workload objects based on state, scaling requirements, and node constraints.

### 📄 Pods
*   **The Baseline Unit:** The smallest and simplest deployable unit in Kubernetes. A Pod represents a single instance of a running process in your cluster.
*   **Multi-Container Support:** A Pod can hold a single container or tightly coupled sidecar containers that need to share resources.

### 🚀 Deployments (Stateless Workloads)
*   **Design Pattern:** Used for *stateless* applications (e.g., web frontends, stateless REST APIs). Pods are completely interchangeable and ephemeral.
*   **Self-Healing:** If a pod dies, a random replacement pod is spun up automatically with a brand-new random string name (e.g., `web-app-7f4b8c9d`).
*   **Lifecycle Management:** Manages underlying `ReplicaSets` and automates rolling upgrades, rollbacks, and replica counts.

### 💾 StatefulSets (Stateful Workloads)
*   **Design Pattern:** Designed specifically for applications that manage data state (e.g., Databases like MySQL, PostgreSQL, MongoDB, Kafka, or ZooKeeper).
*   **Unique Identity:** Every Pod gets a fixed, predictable, and persistent sequential name (`db-pod-0`, `db-pod-1`, `db-pod-2`).
*   **Persistent Binding:** Each Pod connects to its own dedicated storage volume. If `db-pod-1` crashes, Kubernetes recreates a new pod named exactly `db-pod-1` and binds it back to the exact same disk volume, ensuring no data loss.
*   **Ordered Lifecycle:** Scaling happens strictly sequentially ($0 \rightarrow N$) and deletions happen in exact reverse order ($N \rightarrow 0$).

### ⚙️ DaemonSets (Node-Level Agents)
*   **Design Pattern:** Ensures that a copy of a specific pod runs on every single worker node across the cluster.
*   **Automation Loop:** If you add 10 new servers/nodes to your cluster via autoscaling, the DaemonSet instantly schedules its agent pod onto them. When nodes are scaled down, the pods are gracefully collected.
*   **Primary Use Cases:** Cluster monitoring agents (Prometheus Node Exporter), log shipping utilities (Fluentd, Logstash), or core container networking daemons (Calico, Flannel).

---

## 🌐 2. Kubernetes Networking: Intra-Pod vs. Inter-Pod

Kubernetes manages network traffic cleanly by eliminating the need for dynamic port mappings or host-level port collisions.

### A. Intra-Pod Communication (Container-to-Container)
*   Containers residing within the *same* pod share an identical **Network Namespace**.
*   They share the same IP address and network stack.
*   They communicate with each other directly using `localhost` followed by their internal container ports.

### B. Inter-Pod Communication (Pod-to-Pod)
*   Every Pod receives a unique, routable IP address within the cluster network.
*   Kubernetes enforces a **Flat Network Model**: Any pod can directly communicate with any other pod in the cluster over their respective IPs without using Network Address Translation (NAT), regardless of which physical node they live on.

---

## 📈 3. Resource Scaling Architecture: Manual, Horizontal & Vertical

Scaling allows your cluster to handle sudden traffic spikes or conserve cloud costs during low-usage hours. Kubernetes supports three distinct scaling models:

### A. Manual Scaling (Imperative or Declarative)
*   **How it Works:** The administrator manually triggers a command or updates a YAML file to adjust the `replicas` count. 
*   **Speed:** Instantaneous. The underlying controller (ReplicaSet/Deployment) immediately fires up or terminates the exact delta number of pods.

### B. Horizontal Pod Autoscaler (HPA — "Scaling Out")
*   **How it Works:** Kubernetes automatically increases or decreases the **number of pods** (replicas) based on real-time resource utilization (like CPU consumption or Memory percentage thresholds).
*   **Analogy:** Adding more workers to a factory line when the workload increases.
*   **Requirement:** Requires the cluster Metrics Server to supply continuous resource usage streams.

### C. Vertical Pod Autoscaler (VPA — "Scaling Up")
*   **How it Works:** Instead of changing the number of pods, VPA automatically increases or decreases the **CPU and Memory allocations** of your *existing* pods based on historical usage.
*   **Analogy:** Upgrading a single worker to be stronger and have more stamina instead of hiring more workers.
*   **Key Behavior Note:** By default, if a container runs out of memory, VPA will gracefully restart the pod with upgraded CPU/Memory limits to match the app's requirements. 
*   **Caution:** HPA and VPA should generally not be used on the exact same resource for the same metric (like CPU), as they will fight each other for scaling control.

---

## 🔄 4. Deployment Update Strategies

When deploying container image updates (e.g., moving an application from `v1` to `v2`), Kubernetes lets you pick the right balance between resource footprint and application availability.

```text
1. Recreate       [v1 v1 v1] ──► [ All Killed ] ──► [v2 v2 v2]  (Causes Downtime)
2. RollingUpdate  [v1 v1 v1] ──► [v2 v1 v1]     ──► [v2 v2 v2]  (Zero Downtime)
3. Blue/Green     [v1 v1 v1] (Blue)  ──► Switch Traffic ──► [v2 v2 v2] (Green)
4. Canary         [v1 v1 v1] (90% Traffic)  ──► [v2] (10% Traffic to Naya Pod)
```

### 1. Recreate Strategy
*   **Mechanism:** Terminates all active `v1` pods completely before initializing any new `v2` pods.
*   **Downtime:** ❌ **Yes**. The application is unavailable during the update window.
*   **Best For:** Development environments, or apps where two versions cannot run concurrently due to database locks.

### 2. Rolling Update Strategy (Default)
*   **Mechanism:** Updates pods incrementally. It spins up a new pod, confirms it is healthy, and then terminates an old one.
*   **Downtime:** ✔️ **Zero Downtime**. Continuous user availability.
*   **Key Controls:**
    *   `maxSurge`: Configures how many extra pods can run over the desired count during an update.
    *   `maxUnavailable`: Configures how many pods can be taken offline simultaneously during a rollout.

### 3. Blue/Green Deployment
*   **Mechanism:** Spins up a completely independent, parallel environment hosting `v2` (Green) alongside live `v1` (Blue). Once Green passes checks, a Service switches traffic instantly.
*   **Downtime:** ✔️ **Zero Downtime**. Provides near-instant rollbacks if a failure occurs.
*   **Cost:** ❌ Highly resource-intensive since infrastructure footprints double during upgrades.

### 4. Canary Deployment
*   **Mechanism:** Deploys a single `v2` instance alongside your main fleet, directing a small fraction of real production traffic (e.g., 5%) to it. If logs and error metrics remain healthy, the remaining fleet is upgraded.

---

## 🔌 5. Service Types Exposed

Pods are ephemeral; they die and get replaced with new IP addresses constantly. A **Service** provides a single, permanent IP address and network endpoint that automatically routes traffic to the correct pods using label selectors.

*   **ClusterIP (Default):** Exposes the service on an **internal IP** only. The application is completely hidden from the outside world and accessible only by other workloads within the cluster.
*   **NodePort:** Exposes the service externally by opening a static, high-range port (30000–32767) on **every worker node's** physical IP address. Accessible via `http://<Node-IP>:<NodePort>`.
*   **LoadBalancer:** Natively interfaces with cloud platforms (AWS, Azure, GCP) to automatically provision a public-facing network load balancer that forwards internet traffic directly into the cluster.

---

## 🗂️ 6. Declarative Manifest Directory (YAML Specifications)

Authoring Kubernetes configurations requires clean, space-indented YAML files containing four mandatory keys: `apiVersion`, `kind`, `metadata`, and `spec`.

### 📄 Virtual Clustering: `namespace.yaml`
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test
```

### 📄 Stateless Deployment with Rolling Updates: `deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deploy
  namespace: test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-app
  template:
    metadata:
      labels:
        app: test-app
    spec:
      containers:
      - name: test-container
        image: nginx
        resources:
          # Required for Autoscalers to measure threshold math properly
          requests:
            cpu: "200m"
            memory: "100Mi"
        ports:
        - containerPort: 80
```

### 📄 Horizontal Pod Autoscaler (Horizontal Scaling): `hpa.yaml`
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: test-deploy-hpa
  namespace: test
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: test-deploy
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

### 📄 Vertical Pod Autoscaler (Vertical Scaling): `vpa.yaml`
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: test-deploy-vpa
  namespace: test
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: test-deploy
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: test-container
      minAllowed:
        cpu: "100m"
        memory: "50Mi"

# 🚀 Kubernetes Deployment Strategies & Workloads: A Practical Guide

This comprehensive guide breaks down Kubernetes deployment strategies (Recreate, Rolling Update, Blue/Green, Canary), maps out the differences between Stateless and Stateful workloads, and provides production-ready YAML manifests with operational commands.

link in-details : https://medium.com/@ezekiel.umesi/understanding-deployment-strategies-in-kubernetes-3e447d060280

---

## 🛠️ 1. Deployment Strategies Explained Simply

When you release a new version of your application (e.g., updating a container image from `v1` to `v2`), Kubernetes offers different strategies to manage how those pods are updated.
---
### 1. Recreate Strategy (Hard Reset)
*   **Concept:** Kubernetes terminates all existing `v1` pods simultaneously. Once all old pods are completely dead, it starts creating the new `v2` pods.

*   **Analogy:** Closing a retail shop for a week to renovate it, then reopening with a brand-new look.

*   **Downtime:** ❌ **Yes**. There is a distinct window where zero pods are running, making the application completely unavailable to users.

*   **Best For:** Non-critical applications, internal development/testing environments, or applications that cannot run two versions at the same time due to database schema locks.



---
### 2. Rolling Update Strategy (Gradual Phase-In — Default ✅)

*   **Concept:** Kubernetes updates pods incrementally. It spins up a few `v2` pods, waits for them to become healthy, and then terminates a few `v1` pods. This process loops until all pods are running the new version.
*   **Analogy:** Upgrading a fleet of public buses by replacing one old bus with a new hybrid bus at a time, ensuring service never stops.
*   **Downtime:** ✔️ **Zero Downtime**. Users never experience service interruptions.
*   **Key Controls (`rollingUpdate`):**
    *   `maxSurge`: Specifies how many extra pods can be created above your desired replica count during the update (e.g., `1` means temporarily running `desired + 1` pods).
    *   `maxUnavailable`: Specifies how many pods can be taken offline simultaneously during the update process.

---
### 3. Blue/Green Deployment (Instant Traffic Flip)

*   **Concept:** You maintain two identical, isolated production environments. **Blue** runs the current live version (`v1`), while **Green** hosts the new version (`v2`). Once the Green environment passes all automated tests, you update your Kubernetes Service/Router to point traffic to Green instantly.
*   **Downtime:** ✔️ **Zero Downtime**. The traffic switch happens instantly.
*   **Rollback:** Extremely easy. If an undetected bug crashes `v2`, you can route the Service traffic straight back to Blue.
*   **Trade-off:** ❌ **Resource Intensive**. It requires double the infrastructure space (CPU/RAM) since both environments run concurrently during testing.

---
### 4. Canary Deployment (Blast Radius Control)

*   **Concept:** You deploy the new version (`v2`) alongside the old version but route only a small fraction of real production traffic to it (e.g., 5% to 10%). You monitor system logs and error rates. If everything looks stable, you gradually scale up `v2` and scale down `v1`.
*   **Analogy:** Testing a new recipe on a small focus group before launching it nationwide.
*   **Pros:** ✔️ Safe production testing with real user workloads, minimizing impact if something goes wrong.
  ---

## 💾 2. Architecture Workloads: Deployment vs. StatefulSet

| Feature | Deployment (Stateless) | StatefulSet (Stateful) |
| :--- | :--- | :--- |
| **Data Nature** | Ephemeral. Data is lost when the pod dies. | Persistent. Data is safely preserved. |
| **Pod Identity** | Interchangeable. Pods have random hashes (`app-xyz12`). | Unique. Pods have fixed, sequential IDs (`pod-0`, `pod-1`). |
| **Storage Pattern** | Pods usually share a volume or use no persistent disks. | Dedicated storage per pod via `volumeClaimTemplates`. |
| **Scaling Order** | Parallel, randomized, and fast. | Strict sequential order ($0 \rightarrow N$) and reverse deletion ($N \rightarrow 0$). |
| **Best Used For** | Web frontends, stateless REST APIs, microservices. | Databases (MySQL, PostgreSQL), Kafka clusters, ZooKeeper. |

---

## ⚙️ 3. DaemonSet (The Automated Node Agent)

*   **Concept:** A DaemonSet ensures that a copy of a specific pod runs on every single node (worker machine) inside your cluster. 
*   **Automation:** When a new worker node is added to your cluster by an autoscaler, Kubernetes automatically provisions the DaemonSet pod onto it. When the node is deleted, the pod is automatically cleaned up.
*   **Best Used For:** Infra-level background tasks like log forwarding (Fluentd/Logstash), node monitoring daemons (Prometheus Node Exporter), or cluster networking plugins (Calico/Flannel).


---

## 🗂️ 4. Unified Kubernetes Manifests (YAML)

You can save these directly into individual files or split them using the standard `---` YAML separator.

### 📄 `deployment.yaml` (Stateless Rolling Update)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 6
  selector:
    matchLabels:
      app: my-app
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-pod-game
        image: mukunddeo9325/super-mario
        ports:
        - containerPort: 80
```

### 📄 `statefulset.yaml` (Stateful Database Model)
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-stateful-app
spec:
  serviceName: "my-service"
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

### 📄 `daemonset.yaml` (Node Infrastructure Daemon)
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata: 
  name: daemon
  labels:
    app: depapp
spec:
  selector: 
    matchLabels: 
      app: depapp
  template:
    metadata:
      labels:
        name: cwatch
        app: depapp
    spec:
      containers:
      - name: cwatch
        image: amazon/cloudwatch-agent:latest
        ports:
        - containerPort: 80
          protocol: TCP
```

---

## 💻 5. Crucial Kubectl Commands Checklist

Use these essential commands to manage, inspect, and troubleshoot your running workloads:

```bash
# View active pods in the current namespace
kubectl get pods

# Apply or modify resource manifests dynamically
kubectl apply -f deployment.yaml

# Monitor pod creation, deletion, and state changes live
kubectl get pods -w

# Inspect granular metadata, events, and lifecycle logs for a deployment
kubectl describe deployment my-app

# Monitor the progress of an ongoing rolling update
kubectl rollout status deployment/my-app

# Check the version history log of your deployment rollouts
kubectl rollout history deployment/my-app

# Instantly abort an update and roll back to the previous stable version
kubectl rollout undo deployment/my-app

# Update a container image instantly without editing the YAML manifest file
kubectl set image deployment/my-app my-container=nginx:latest

# Instantly scale the number of application running replicas up or down
kubectl scale deployment my-app --replicas=5
```

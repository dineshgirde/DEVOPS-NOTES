# 📈 Kubernetes HPA (Horizontal Pod Autoscaler) Lab with Minikube

## 📚 Complete Hands-On Scaling Guide (Beginner → Intermediate)
### ⚡ Automated Workload Scaling in local Kubernetes

---

## 📖 Deep Theory: Understanding Horizontal Pod Autoscaling (HPA)
In production environments, traffic is never static. Spikes occur during sales, promotions, or high-activity windows. To prevent server crashes and optimize costs, Kubernetes uses the **Horizontal Pod Autoscaler (HPA)**.

### 💡 Core Autoscaling Mechanisms:
1. **Scale Up (Horizontal):** When resource consumption (like CPU or Memory) passes a specific threshold, HPA automatically spawns new replicas (Pods) to distribute the traffic load.
2. **Scale Down:** When the traffic load decreases and resource metrics return to normal, HPA terminates the extra pods to release system resources.
3. **The Metrics Server Dependency:** HPA does not count traffic requests directly. Instead, it queries the `Metrics Server` cluster API aggregator at regular intervals (usually every 15 seconds) to check real-time CPU and Memory utilization profiles across active container workloads.

---

## 🛠️ Step 1 — Verify Minikube & Enable Metrics Server
### 💡 Theory Background
Before HPA can execute scaling decisions, it must collect resource usage telemetry data. In a default local local configuration, this telemetry data loop is disabled. We use the built-in Minikube cluster manager tool to inject the metrics aggregator framework.

#### Ensure your local minikube cluster is running securely:
```bash
minikube start
```

#### Enable the metrics collection subsystem addon:
```bash
minikube addons enable metrics-server
```

#### Verify metrics server telemetry pods are up and running:
```bash
kubectl get pods -n kube-system | grep metrics-server
```

---

## 🛠️ Step 2 — Create Deployment Manifest with Resource Limits
### 💡 Theory Background
**CRITICAL RULE:** HPA *cannot* scale a deployment unless the target containers have explicit `resources.requests.cpu` configuration blocks defined. Without these definitions, the Metrics Server cannot calculate the current usage *percentage* (Utilization %), and your HPA status will show `<unknown>` permanently.

#### Create your deployment file layout:
```bash
vim hpa-dep.yaml
```

#### Paste the deployment configuration definitions:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa-nginx
  template:
    metadata:
      labels:
        app: hpa-nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "200m"
        ports:
        - containerPort: 80
```
*(Note: `100m` means 100 milli-cores, which equals 10% of a single CPU core on the host).*

#### Deploy configuration into cluster:
```bash
kubectl apply -f hpa-dep.yaml
```

---

## 🛠️ Step 3 — Create and Expose the ClusterIP Service
### 💡 Theory Background
Pods are dynamic and temporary. To route synthetic stress traffic cleanly to our target deployment, we require a persistent internal load balancer endpoint. A **ClusterIP Service** acts as a fixed network router gateway that evenly spreads request traffic to active Nginx pods.

#### Create your network service file configuration layout:
```bash
vim service.yaml
```

#### Paste the service resource block:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hpa-service
spec:
  selector:
    app: hpa-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

#### Apply network routes:
```bash
kubectl apply -f service.yaml
```

---

## 🛠️ Step 4 — Define the Horizontal Pod Autoscaler Configuration
### 💡 Theory Background
The `autoscaling/v2` API allows us to target a specific controller architecture reference (our Nginx deployment). Here we declare standard boundaries (`minReplicas` and `maxReplicas`) and define the performance math trigger logic.

#### Create your autoscaler file workspace:
```bash
vim hpa.yaml
```

#### Paste HPA rules and metrics limits:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-nginx
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-nginx
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 20
```
*(Mathematical Interpretation: If the average CPU utilization of all running pods passes **20%** of their requested limit, HPA will calculate the scale factor and trigger server scaling).*

#### Initialize the controller engine:
```bash
kubectl apply -f hpa.yaml
```

---

## 🛠️ Step 5 — Launch a Synthetic High-Traffic Load Generator
### 💡 Theory Background
To simulate a massive real-world user spike, we spin up an isolated test pod using a small `busybox` runtime image. This pod runs an infinite shell script loop that repeatedly hits our service IP address via standard web requests (`wget`), forcing the Nginx instance to consume extra processing cycles.

#### Spin up the interactive stress execution platform:
```bash
kubectl run -i --tty load-generator --image=busybox /bin/sh
```

#### Note: Run below command directly inside the newly opened shell prompt:
```sh
while true; do wget -q -O- http://hpa-service; done
```
*(This triggers a continuous stream of requests over the cluster network).*

---

## 🛠️ Step 6 — Perform Cluster Operations & Monitor Auto-Scaling
### 💡 Theory Background
Since the load generator is occupying your current shell terminal workspace, open a separate terminal window to inspect the live scaling metrics loop.

#### Monitor the auto-scaling engine parameters in real-time:
```bash
kubectl get hpa -w
```
*(Observe how the `TARGETS` percentage jumps from `0%/20%` up past the threshold, and watch the `REPLICAS` value scale up step-by-step from 1 to 5).*

#### Review active replica container lifecycles in the cluster:
```bash
kubectl get pods
```

#### Reference Log Mapping Context:
`Screenshot 2026-04-03 at 5 45 00 PM` (Confirms full auto-scaling verification loop is successfully completed).

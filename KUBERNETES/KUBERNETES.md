# ☸️ Kubernetes Cheatsheet

## 🚀 Complete Kubernetes Guide for Beginners to Advanced Users
### 📦 Learn Kubernetes Architecture, Pods, Deployments, Networking, Storage, Security & Monitoring

---

## 📚 Table of Contents
1. Introduction to Kubernetes
2. Basic Kubernetes Operations
3. Deployments and Scaling
4. Services and Networking
5. Persistent Storage
6. ConfigMaps and Secrets
7. Ingress Controllers
8. Kubernetes Security
9. Advanced Kubernetes Features
10. Monitoring and Logging
11. High Availability & Disaster Recovery
12. References

---

## 🚀 1. Introduction to Kubernetes

### 🔹 What is Kubernetes?
Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

It helps organizations:
* Automate deployments
* Scale applications
* Manage containers
* Ensure high availability
* Handle self-healing

### 🔑 Key Concepts

| Component | Description |
| :--- | :--- |
| **Cluster** | A set of worker machines that run containerized applications |
| **Node** | A single machine inside a Kubernetes cluster |
| **Pod** | The smallest deployable unit that can contain one or more containers |
| **Service** | A stable network endpoint used to expose Pods |
| **Namespace** | A logical separation used to divide cluster resources |
| **Kubelet** | An agent running on each node that manages Pods |
| **Kubectl** | Command-line tool used to interact with Kubernetes |

### 🏗 Kubernetes Architecture

```text
                +----------------------+

                |   Control Plane      |
                |----------------------|
                | API Server           |
                | Scheduler            |
                | Controller Manager   |
                | ETCD Database        |
                +----------+-----------+
                           |
        ----------------------------------------

        |                                      |
+---------------+                   +---------------+

|   Worker Node |                   |   Worker Node |
|---------------|                   |---------------|
| Kubelet       |                   | Kubelet       |
| Kube Proxy    |                   | Kube Proxy    |
| Pods          |                   | Pods          |
+---------------+                   +---------------+
```

---

## 🛠 2. Basic Kubernetes Operations

### 🔹 Setup Kubernetes Cluster

#### Start Minikube
```bash
minikube start
```
*Minikube is used to create a single-node Kubernetes cluster for learning and testing.*

### 🔹 Working with kubectl

#### Get Cluster Information
```bash
kubectl cluster-info
```
*Displays Kubernetes control plane and cluster services information.*

#### Get All Nodes
```bash
kubectl get nodes
```
*Shows all available worker nodes in the cluster.*

### 📦 Managing Pods

#### Create Pod
```bash
kubectl run mypod --image=nginx
```
*Creates a Pod using the nginx image.*

#### List Pods
```bash
kubectl get pods
```
*Displays all running Pods.*

#### Describe Pod
```bash
kubectl describe pod mypod
```
*Shows detailed Pod information including events and status.*

#### Delete Pod
```bash
kubectl delete pod mypod
```
*Deletes the specified Pod.*

### 📂 Namespaces

#### List Namespaces
```bash
kubectl get namespaces
```
*Displays all namespaces inside the cluster.*

#### Create Namespace
```bash
kubectl create namespace mynamespace
```
*Creates a new namespace.*

#### Delete Namespace
```bash
kubectl delete namespace mynamespace
```
*Deletes the namespace and its resources.*

---

## 🚀 3. Deployments and Scaling

### 🔹 What is Deployment?
A Deployment is a Kubernetes resource used to manage Pods and ensure desired application state.

It provides:
* Scaling
* Rolling updates
* Rollbacks
* Self-healing

#### Create Deployment
```bash
kubectl create deployment myapp --image=nginx
```
*Creates a Deployment using nginx image.*

#### View Deployments
```bash
kubectl get deployments
```
*Lists all deployments.*

#### Update Deployment
```bash
kubectl set image deployment/myapp nginx=nginx:1.16
```
*Updates container image version.*

#### Rollback Deployment
```bash
kubectl rollout undo deployment/myapp
```
*Rolls back to previous version.*

### 🔹 Scaling Applications

#### Scale Deployment
```bash
kubectl scale deployment myapp --replicas=3
```
*Increases or decreases Pod replicas.*

#### Horizontal Pod Autoscaler (HPA)
```bash
kubectl autoscale deployment myapp --min=1 --max=5 --cpu-percent=80
```
*Automatically scales Pods based on CPU usage.*

---

## 🌐 4. Services and Networking

### 🔹 What is a Service?
A Service provides stable networking and communication for Pods.

Since Pods are temporary, Services ensure fixed connectivity.

#### Expose Deployment
```bash
kubectl expose deployment myapp --type=NodePort --port=80
```
*Exposes deployment externally.*

#### List Services
```bash
kubectl get services
```
*Displays all Kubernetes services.*

### 🔹 Service Types

| Type | Description |
| :--- | :--- |
| **ClusterIP** | Internal communication within cluster |
| **NodePort** | External access using node IP and port |
| **LoadBalancer** | Uses cloud load balancer |
| **ExternalName** | Maps service to DNS name |

### 🌍 Kubernetes Networking
Kubernetes networking enables communication between:
* Pod to Pod
* Pod to Service
* External traffic to cluster

Features include:
* DNS resolution
* Service discovery
* Load balancing
* Network policies

---

## 💾 5. Persistent Storage

### 🔹 What is Persistent Storage?
Persistent storage keeps application data safe even if Pods restart or fail.

#### Persistent Volume (PV)
```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: mypv

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: "/mnt/data"
```
*Persistent Volume provides actual storage resource.*

#### Persistent Volume Claim (PVC)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mypvc

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 1Gi
```
*PVC requests storage from a PV.*

---

## 🔐 6. ConfigMaps and Secrets

### 🔹 ConfigMaps
ConfigMaps store non-sensitive configuration data.

#### Create ConfigMap
```bash
kubectl create configmap myconfig --from-file=config.txt
```

### 🔒 Secrets
Secrets store sensitive data securely.

#### Create Secret
```bash
kubectl create secret generic mysecret --from-literal=username=admin
```

---

## 🌐 7. Ingress Controllers

### 🔹 What is Ingress?
Ingress manages HTTP/HTTPS traffic routing into the Kubernetes cluster.

#### Install NGINX Ingress Controller
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

---

## 🔒 8. Kubernetes Security

### 🔹 RBAC (Role-Based Access Control)
RBAC controls permissions inside Kubernetes.

#### Create Role
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role

metadata:
  namespace: mynamespace
  name: myrole
```

---

## 🔬 9. Advanced Kubernetes Features
*Note: This placeholder sections maps structural completeness from the Table of Contents for seamless configuration flow.*

---

## 📊 10. Monitoring and Logging

### 🔹 Prometheus
Prometheus collects Kubernetes metrics.

#### Install Prometheus
```bash
kubectl apply -f https://github.com/prometheus-operator/prometheus-operator/blob/main/bundle.yaml
```

### 🔹 Grafana
Grafana visualizes monitoring dashboards.

#### Install Grafana
```bash
kubectl apply -f https://raw.githubusercontent.com/grafana/grafana/main/deploy/kubernetes/grafana-deployment.yaml
```

---

## 🏥 11. High Availability & Disaster Recovery

### 🔹 High Availability (HA)
High Availability ensures Kubernetes remains operational even if nodes fail.

Features:
* Multiple master nodes
* Redundant ETCD
* Fault tolerance

#### Backup ETCD
```bash
etcdctl snapshot save /path/to/backup
```
*Creates ETCD backup snapshot.*

#### Restore ETCD
```bash
etcdctl snapshot restore /path/to/backup
```
*Restores ETCD backup.*

---

## 📚 12. References
* Kubernetes Official Documentation
* Kubernetes GitHub Repository
* Kubernetes Blog

---

## 🎯 Final Takeaway
Kubernetes simplifies container orchestration and infrastructure automation.

It helps teams:
* Deploy applications faster
* Scale automatically
* Improve reliability
* Automate infrastructure
* Ensure high availability

☸️ Kubernetes Powers Modern Cloud-Native Applications 🚀

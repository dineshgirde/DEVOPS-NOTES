# ☸️ Kubernetes RWX (ReadWriteMany) Lab using AWS EFS on EKS

## Objective
Learn:
* **What is ReadWriteMany (RWX):** Understanding shared concurrent storage access across multiple nodes.
* **How multiple pods use same storage:** Deep dive into directory sharing and real-time synchronization.
* **How to use AWS EFS with EKS:** Installing the official Container Storage Interface (CSI) plugin and mapping resources.

---

## 📖 Deep Theory: Understanding Kubernetes Storage Access Modes
Before jumping into the lab, it is crucial to understand how Kubernetes manages storage attachments. Kubernetes uses **Access Modes** to define how persistent volumes are mounted on the underlying host nodes:

1. **ReadWriteOnce (RWO):** The volume can be mounted as read-write by a **single node** only. Even if multiple pods are on that same node, pods on other nodes cannot access this storage. (Example: AWS EBS).
2. **ReadOnlyMany (ROX):** The volume can be mounted as read-only by **many nodes** simultaneously.
3. **ReadWriteMany (RWX):** The volume can be mounted as read-write by **many nodes** at the same time. This is mandatory for clustered or distributed applications where workloads scale dynamically across the cluster. (Example: AWS EFS).

---

## Architecture
```text
AWS EFS (Network File System)
   ↓ (Managed via CSI Driver)
PV (Cluster-wide Storage Resource)
   ↓ (Requested by Application Layer)
PVC (Access Mode: ReadWriteMany)
   ↓ (Shared Ingestion)
Multiple Pods (Running concurrently across different EC2 Nodes)
```

**⚠️ Important Structural Concepts:**
* **EBS ❌ does NOT support RWX:** AWS Elastic Block Store (EBS) is a block storage service. It can only attach to a single EC2 instance at a time. If you attempt to use EBS with an RWX claim, multi-node pods will fail with a `Multi-Attach error`.
* **Use Amazon Elastic File System (EFS) for RWX:** EFS is a managed network file system (NFSv4) that naturally allows thousands of concurrent connections from different servers, making it the perfect backend for **ReadWriteMany** workloads.

---

## Step 1 — Install EFS CSI Driver
### 💡 Theory Background
By default, a vanilla Kubernetes cluster does not know how to talk to AWS EFS APIs to provision network shares. The **AWS EFS CSI (Container Storage Interface) Driver** acts as a translator plugin between Kubernetes storage requests and AWS EFS service.

```bash
eksctl create addon \
  --name aws-efs-csi-driver \
  --cluster eks-cdec-prod \
  --region ap-south-1 \
  --force
```

#### Verify Installation:
The driver runs as a DaemonSet inside the `kube-system` namespace, meaning an active controller pod will deploy on every single node of your cluster. Verify that they are active:
```bash
kubectl get pods -n kube-system | grep efs
```

---

## Step 2 — Create EFS
### 💡 Theory Background
An EFS file system must exist in your AWS account before Kubernetes can map it. Because it is network-based storage, it uses **Mount Targets** inside your subnets so that EC2 instances can route local storage traffic to the EFS endpoint via IP addresses.

AWS Console Instructions:
1. Navigate to **EFS** → Click **Create File System**.
2. Select **Customize** and ensure you choose the **Same VPC as your EKS Cluster** (otherwise nodes won't find the route).
3. In the Network configuration, make sure **Mount Targets** are created across all Availability Zones (AZs) where your EKS worker nodes reside.

#### Copy your unique File System ID for later use:
* *Example:* `fs-0abc123456789`

---

## Step 3 — Create Security Group Rule
### 💡 Theory Background
EFS operates on standard Network File System (NFS) protocols over **Port 2049**. By default, AWS security groups block all inbound traffic. You must add an explicit firewall rule allowing traffic from your EKS worker nodes to the EFS filesystem endpoint.

#### Network Rule Specifications:
* **Protocol/Port:** NFS (Port 2049)
* **Source Traffic:** EKS Worker Node Security Group (This allows any instance running in your cluster to talk directly to the storage subsystem).

---

## Step 4 — Create PV and PVC
### 💡 Theory Background
* **PersistentVolume (PV):** Points directly to your infrastructure. In this case, we configure the CSI block to target your specific AWS EFS instance ID.
* **PersistentVolumeClaim (PVC):** The application’s request ticket. We declare `accessModes: [ReadWriteMany]` so that Kubernetes binds it with an RWX-compatible PV.

#### Create manifest file:
```bash
vi efs-pv-pvc.yaml
```

#### Paste the configuration definitions:
```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: efs-pv

spec:
  capacity:
    storage: 5Gi

  volumeMode: Filesystem

  accessModes:
    - ReadWriteMany

  persistentVolumeReclaimPolicy: Retain

  storageClassName: efs-sc

  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-0abc123456789

---
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: efs-pvc

spec:
  accessModes:
    - ReadWriteMany

  storageClassName: efs-sc

  resources:
    requests:
      storage: 5Gi
```
*(Make sure to replace `fs-0abc123456789` with your actual EFS Instance ID obtained from Step 2)*

#### Apply manifests to cluster:
```bash
kubectl apply -f efs-pv-pvc.yaml
```

---

## Step 5 — Verify Storage Binding
### 💡 Theory Background
Kubernetes will evaluate the StorageClass name and access capabilities. If the parameters match perfectly, the control plane will automatically bind the PVC ticket to the target infrastructure PV asset.

```bash
kubectl get pv
kubectl get pvc
```

#### Expected Output Status:
* `STATUS = Bound` (This confirms your storage claim is active and waiting for a workload deployment).

---

## Step 6 — Create Pod-1
### 💡 Theory Background
Here we mount the PVC into a standard web container directory path `/data`. Any mutations or logging written inside this directory will bypass container runtime memory layers and stream straight over the network into your remote AWS EFS share.

#### Create manifest file:
```bash
vi pod1.yaml
```

#### Paste Pod definition:
```yaml
apiVersion: v1
kind: Pod

metadata:
  name: pod-1

spec:
  containers:
  - name: app
    image: nginx

    volumeMounts:
    - mountPath: /data
      name: efs-storage

  volumes:
  - name: efs-storage
    persistentVolumeClaim:
      claimName: efs-pvc
```

#### Apply deployment:
```bash
kubectl apply -f pod1.yaml
```

---

## Step 7 — Create Pod-2 & Pod-3 (Cross-Node Scheduling)
### 💡 Theory Background
To truly test ReadWriteMany operations, we spawn duplicate pods. We use a **NodeSelector** on Pod-3 to explicitly force it onto a completely different physical infrastructure host machine. If both pods function seamlessly, our network storage layer configuration is solid.

#### Create manifest file for Pod-2:
```bash
vi pod2.yaml
```
*Paste the exact same content as `pod1.yaml` but modify the metadata identification layer:*
```yaml
metadata:
  name: pod-2
```
```bash
kubectl apply -f pod2.yaml
```

#### Create manifest file for Node-Isolated Pod-3:
```bash
vi pod3.yaml
```

#### Paste configuration definitions for Pod-3:
```yaml
apiVersion: v1
kind: Pod

metadata:
  name: pod-3

spec:
  nodeSelector:
    kubernetes.io/hostname: ip-node-ip.ap-south-1.compute.internqal

  containers:
  - name: app
    image: nginx

    volumeMounts:
    - mountPath: /data
      name: efs-storage

  volumes:
  - name: efs-storage
    persistentVolumeClaim:
      claimName: efs-pvc
```

#### Apply Pod-3 layout:
```bash
kubectl apply -f pod3.yaml
```

---

## Step 8 — Verify Pod Execution & Storage Health
Verify that all container states transition smoothly to an operational status across your infrastructure topology.

```bash
kubectl get po -o wide
```

#### Expected Operational State:
* All target pods must read `STATUS = Running`.

### Deep Dive: Troubleshooting Volumes Not Mounting
If your deployment pods hang permanently in a `ContainerCreating` or `ContainerWitherrors` state, use the following operational validation procedures:

#### a. Verify EFS Security Group Inbound Rule (Most Likely Root Cause)
* **Your EKS Node Security Group Reference:** `sg-0acbc299a7f0a6b6d`
* **Your Destination EFS Security Group Reference:** `sg-05f98b81b1308fab7`

If traffic parameters are missing, add this rule into your target EFS security network schema:
* **Type:** NFS
* **Protocol:** TCP
* **Port:** 2049
* **Source Selection:** Pass the EKS Node Security Group ID `sg-0acbc299a7f0a6b6d`

AWS Management Console Path:
* **VPC** → **Security Groups** → Select `sg-05f98b81b1308fab7` → **Inbound Rules** → **Edit Inbound Rules** → Click **Add Rule**

#### b. Verify EFS and EKS VPC Matching
Cross-verify network residency identifiers via CLI tracking:
```bash
aws eks describe-cluster \
  --name eks-cdec-b66 \
  --region ap-south-1 \
  --query "cluster.resourcesVpcConfig.vpcId" \
  --output text
```
*The resulting network string output must match your EFS system subnet target parameters precisely.*

#### c. Propagation Latency Note

## Delete nodes
```
kubectl delete -f pod1.yaml
kubectl delete -f pod2.yaml
kubectl delete -f efs-pv-pvc.yaml
```



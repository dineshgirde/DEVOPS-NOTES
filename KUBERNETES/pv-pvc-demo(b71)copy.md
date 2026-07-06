# Kubernetes RWX (ReadWriteMany) Lab using AWS EFS on EKS

## Objective

Learn:

* What is ReadWriteMany (RWX)
* How multiple pods use same storage
* How to use AWS EFS with EKS

---

# Architecture

```text id="e5w7pc"
AWS EFS
   ↓
PV
   ↓
PVC (RWX)
   ↓
Multiple Pods
```

---

# Important

EBS ❌ does NOT support RWX.

Use Amazon Elastic File System for:

```yaml id="u2x9lv"
ReadWriteMany
```

---

# Step 1 — Install EFS CSI Driver

```bash id="k8m3tw"
eksctl create addon \
  --name aws-efs-csi-driver \
  --cluster eks-cdec-prod \
  --region ap-south-1 \
  --force
```

Verify:

```bash id="r4n7qy"
kubectl get pods -n kube-system | grep efs
```

---

# Step 2 — Create EFS

AWS Console:

```text id="x6p2kr"
EFS → Create File System
```

Select:

* Same VPC as EKS
* Create mount targets in all AZs

Copy:

* File System ID

Example:

```text id="t3v8zn"
fs-0abc123456789
```

---

# Step 3 — Create Security Group Rule

Allow NFS:

```text id="j9m4xp"
Port 2049
```

Source:

* EKS worker node security group

---

# Step 4 — Create PV and PVC

Create file:

```bash id="c1q8lv"
vi efs-pv-pvc.yaml
```

Paste:

```yaml id="d7k5wn"
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

Replace:

```text id="s8n2qy"
fs-0abc123456789
```

with your EFS ID.

Apply:

```bash id="g5w1pk"
kubectl apply -f efs-pv-pvc.yaml
```

---

# Step 5 — Verify

```bash id="a3m9tc"
kubectl get pv
kubectl get pvc
```

Expected:

```text id="u6v2kr"
STATUS = Bound
```

---

# Step 6 — Create Pod-1

Create:

```bash id="n1q4xz"
vi pod1.yaml
```

Paste:

```yaml id="v7m8lp"
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

Apply:

```bash id="f4r7qw"
kubectl apply -f pod1.yaml
```

---

# Step 7 — Create Pod-2

Create:

```bash id="z2x6nc"
vi pod2.yaml
```

Paste SAME yaml but change name:

```yaml id="m9w5tr"
metadata:
  name: pod-2
```

Apply:

```bash id="h8k1pv"
kubectl apply -f pod2.yaml
```

---
## using node selector 
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

# Step 8 — Verify Pods

```bash id="q5n7ld"
kubectl get po -o wide
```

Both pods should be:

```text id="y3m8zk"
Running
```
---
## If pods are not running 
Based on all the troubleshooting, these are the **key steps** to get your pods running:

### a. Verify EFS Security Group Inbound Rule (Most Likely Fix)

Your EKS node SG:

```text
sg-0acbc299a7f0a6b6d
```

Your EFS SG:

```text
sg-05f98b81b1308fab7
```

Add this rule to the **EFS Security Group**:

```text
Type: NFS
Protocol: TCP
Port: 2049
Source: sg-0acbc299a7f0a6b6d
```

AWS Console:

```text
VPC
 → Security Groups
 → sg-05f98b81b1308fab7
 → Inbound Rules
 → Edit Inbound Rules
 → Add Rule
```

---

### b. Verify EFS and EKS are in the Same VPC

Run:

```bash
aws eks describe-cluster \
  --name eks-cdec-b66 \
  --region ap-south-1 \
  --query "cluster.resourcesVpcConfig.vpcId" \
  --output text
```

Should return:

```text
vpc-id
```

Same as EFS mount targets.

---

### c. Wait 1–2 Minutes

After adding the SG rule, give AWS networking a minute to update.

---

### d. Recreate the Pods

```bash
kubectl delete pod pod-1 pod-2
```

Even on different nodes ✅

---

# Step 9 — Write Data from Pod-1

Enter pod:

```bash id="u1v4pc"
kubectl exec -it pod-1 -- bash
```

Create file:

```bash id="k6m9qt"
echo "Hello from RWX Storage" > /data/demo.txt
```

Exit:

```bash id="n7p2xl"
exit
```

---

# Step 10 — Read Data from Pod-2

Enter:

```bash id="b4w8zr"
kubectl exec -it pod-2 -- bash
```

Read:

```bash id="s9q1tv"
cat /data/demo.txt
```

Expected:

```text id="r5x7pk"
Hello from RWX Storage
```

---

# What This Proves

```text id="v2m6qw"
Multiple pods
Multiple nodes
Same shared storage
```

This is:

```yaml id="c8k3lp"
ReadWriteMany
```

---

# Real Use Cases

RWX used in:

* Shared uploads
* WordPress
* Shared logs
* Jenkins shared workspace
* Media storage
* Kubernetes shared storage

---

# Cleanup

```bash id="x4n9wb"
kubectl delete -f pod1.yaml
kubectl delete -f pod2.yaml
kubectl delete -f efs-pv-pvc.yaml
```

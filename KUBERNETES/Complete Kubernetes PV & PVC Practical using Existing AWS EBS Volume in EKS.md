# Complete Kubernetes PV & PVC Practical using Existing AWS EBS Volume in EKS

> Static provisioning of a PersistentVolume backed by an already-existing AWS EBS volume, consumed by a Pod through a PersistentVolumeClaim, on Amazon EKS.

---

## 📌 Objective

In this practical we will learn:

- What is a **PersistentVolume (PV)**
- What is a **PersistentVolumeClaim (PVC)**
- How **EKS** uses an **AWS EBS Volume**
- How to attach an **existing EBS volume** to Kubernetes
- How a Pod stores persistent data
- How data **survives pod deletion** (persistence proof)

This document contains **ALL steps from start to end**.

---

## 🏗️ Architecture

```text
AWS EBS Volume
      ↓
PersistentVolume (PV)
      ↓
PersistentVolumeClaim (PVC)
      ↓
Pod
```

**Flow explained:** the EBS volume is the *physical* disk that already exists in your AWS account. The PV is Kubernetes' representation of that disk. The PVC is a *request/claim* made by an application for storage — it doesn't care where the storage physically lives, it just binds to a matching PV. The Pod then mounts the PVC as a volume, so the container never talks to AWS or the PV directly.

---

## ⚙️ Environment Used

| Component      | Value          |
|-----------------|----------------|
| Kubernetes      | EKS            |
| Region          | ap-south-1     |
| Cluster Name    | eks-cdec-prod  |
| Storage Type    | AWS EBS        |
| Access Mode     | ReadWriteOnce  |
| Filesystem      | ext4           |

---

## 🧠 Important Understanding Before Starting

### What is PV?

`PersistentVolume` represents the **actual storage resource** in the cluster — think of it as the Kubernetes "object form" of a real disk.

Example backing storage:
```text
AWS EBS Volume
```

### What is PVC?

A `PersistentVolumeClaim` **requests storage** from a PV.

The application (Pod) uses the **PVC**, never the EBS volume or PV directly. This keeps storage details abstracted away from the app.

### Why ReadWriteOnce (RWO)?

AWS EBS volumes can only be mounted by **one node at a time**, so they only support:

| Access Mode              | Supported |
|---------------------------|-----------|
| ReadWriteOnce (RWO)        | ✅ |
| ReadWriteMany (RWX)        | ❌ |

So for EBS, **always** use:

```yaml
accessModes:
  - ReadWriteOnce
```

> 💡 If you need `ReadWriteMany`, EBS is the wrong choice — you'd need **EFS** instead, since EFS is a network filesystem that supports multiple simultaneous mounts.

---

## 🔹 STEP 1 — Verify EKS Cluster

Check that the cluster exists and is reachable:

```bash
eksctl get cluster --region ap-south-1
```

**Expected output:**
```text
eks-cdec-prod
```

---

## 🔹 STEP 2 — Install EBS CSI Driver

EKS needs the **EBS CSI (Container Storage Interface) Driver** so Kubernetes can create, attach, and manage AWS EBS volumes on its behalf. Without this driver, Kubernetes has no way to talk to the EBS API.

### 2.1 Create IAM Service Account

This gives the CSI driver's Kubernetes pods permission (via IAM) to call AWS EBS APIs.

```bash
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster eks-cdec-prod \
  --region ap-south-1 \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --override-existing-serviceaccounts
```

### 2.2 Install EBS CSI Addon

This deploys the actual CSI driver components (controller + node plugin) into the cluster.

```bash
eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster eks-cdec-prod \
  --region ap-south-1 \
  --force
```

### 2.3 Verify CSI Driver

```bash
kubectl get pods -n kube-system | grep ebs
```

**Expected output:**
```text
ebs-csi-controller   Running
ebs-csi-node         Running
```

If it's not running, check logs:

```bash
kubectl logs -n kube-system deployment/ebs-csi-controller -c ebs-plugin
```

---

## 🔹 STEP 3 — Create EBS Volume

Go to the AWS Console:

```text
EC2 → Elastic Block Store → Volumes → Create Volume
```

| Setting | Value       |
|---------|-------------|
| Type    | gp3         |
| Size    | 20 GB       |
| AZ      | ap-south-1a |

> ⚠️ **IMPORTANT:** The EBS volume and the EKS worker node **must be in the SAME Availability Zone**. EBS volumes are zone-locked — an instance in `ap-south-1b` can never attach a volume created in `ap-south-1a`.

---

## 🔹 STEP 4 — Verify EBS Volume AZ

```bash
aws ec2 describe-volumes \
  --volume-ids vol-xxxxxxxx \
  --query "Volumes[*].AvailabilityZone"
```

**Expected output:**
```text
ap-south-1a
```

---

## 🔹 STEP 5 — Attach EBS Volume to EC2 Temporarily

**Purpose:** A brand-new/empty EBS volume has no filesystem on it. Kubernetes can't format it on the fly for you here, so we temporarily attach it to any EC2 instance just to create a filesystem manually.

```text
EC2 → Volumes → Attach Volume → (attach to a temporary EC2 instance)
```

---

## 🔹 STEP 6 — Verify Device Name

Login to the EC2 instance, then run:

```bash
lsblk
```

Example device name:
```text
nvme1n1
```

> ⚠️ The device may show up as `/dev/nvme1n1` — always confirm the exact name before formatting, since formatting the wrong disk is destructive and irreversible.

---

## 🔹 STEP 7 — Create Filesystem

🚨 **VERY IMPORTANT STEP** — get this wrong and your PV/Pod mount will fail later.

✅ Correct:

```bash
sudo mkfs.ext4 /dev/nvme1n1
```

❌ Wrong:

```bash
sudo mkfs.ext4 /dev/nvme1n1p1
```

**Why:** Kubernetes (via the CSI driver) mounts the **raw disk** directly — it does not mount a partition inside the disk. So formatting a partition (`p1`) instead of the raw device is a mismatch that will break the mount later.

If a confirmation prompt appears:

```text
Proceed anyway? (y,N)
```

Type:

```bash
y
```

---

## 🔹 STEP 8 — Detach Volume from EC2

⚠️ **IMPORTANT:** The volume must **NOT** remain attached to the EC2 instance — an EBS volume (RWO) can only be attached to one place at a time, so it must be freed before Kubernetes can claim it.

**Option A — AWS Console:**
```text
Volumes → Detach Volume
```

**Option B — AWS CLI:**

```bash
aws ec2 detach-volume --volume-id vol-xxxxxxxx
```

Verify it detached successfully:

```bash
aws ec2 describe-volumes --volume-ids vol-xxxxxxxx
```

**Expected output:**
```text
State = available
```

---

## 🔹 STEP 9 — Create PV and PVC YAML

Create the file:

```bash
vi pv-pvc.yaml
```

Paste the following:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: eks-ebs-pv

spec:
  capacity:
    storage: 10Gi

  volumeMode: Filesystem

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  storageClassName: manual

  csi:
    driver: ebs.csi.aws.com
    volumeHandle: vol-0ef5e3f9ec060dcd0
    fsType: ext4

---
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: eks-ebs-pvc

spec:
  accessModes:
    - ReadWriteOnce

  storageClassName: manual

  resources:
    requests:
      storage: 10Gi
```

**Key fields explained:**

| Field | Meaning |
|---|---|
| `volumeHandle` | The actual AWS EBS **Volume ID** — this is what links the PV to your real, pre-created disk |
| `persistentVolumeReclaimPolicy: Retain` | When the PVC/PV is deleted, the **underlying EBS volume is NOT deleted** — data stays safe in AWS |
| `storageClassName: manual` | Used to manually match a specific PVC to this specific PV (static binding, no dynamic provisioning) |

---

## 🔹 STEP 10 — Apply PV and PVC

```bash
kubectl apply -f pv-pvc.yaml
```

---

## 🔹 STEP 11 — Verify PV and PVC

```bash
kubectl get pv
kubectl get pvc
```

**Expected status:** `Bound`

```text
eks-ebs-pv    Bound
eks-ebs-pvc   Bound
```

> If status shows `Pending` instead of `Bound`, it usually means the `storageClassName`, `accessModes`, or `capacity` don't match between the PV and PVC — Kubernetes only binds them when these are compatible.

---

## 🔹 STEP 12 — Create Pod YAML

```bash
vi pod.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: ebs-test-pod

spec:
  containers:
  - name: app
    image: nginx

    volumeMounts:
    - mountPath: /data
      name: ebs-storage

  volumes:
  - name: ebs-storage
    persistentVolumeClaim:
      claimName: eks-ebs-pvc
```

**Explanation:** `mountPath: /data` means inside the container, the EBS-backed storage will appear as the `/data` directory — anything written there is actually being written to the EBS volume, not the container's ephemeral filesystem.

---

## 🔹 STEP 13 — Create Pod

```bash
kubectl apply -f pod.yaml
```

---

## 🔹 STEP 14 — Verify Pod Status

```bash
kubectl get po
```

**Expected status:** `Running`

If stuck in:

```text
ContainerCreating
```

Check why:

```bash
kubectl describe pod ebs-test-pod
```

---

## 🔹 STEP 15 — Verify Mounted Volume

Enter the pod:

```bash
kubectl exec -it ebs-test-pod -- bash
```

Check the filesystem:

```bash
df -h
```

You should see the mounted EBS storage listed there.

---

## 🔹 STEP 16 — Write Data

Create a test file:

```bash
echo "Hello from EBS Volume" > /data/test.txt
```

Verify it:

```bash
cat /data/test.txt
```

**Expected output:**
```text
Hello from EBS Volume
```

---

## 🔹 STEP 17 — Verify Persistence

Exit the pod:

```bash
exit
```

Delete the pod:

```bash
kubectl delete pod ebs-test-pod
```

Recreate it:

```bash
kubectl apply -f pod.yaml
```

Enter it again:

```bash
kubectl exec -it ebs-test-pod -- bash
```

Check the file again:

```bash
cat /data/test.txt
```

**Expected output:**
```text
Hello from EBS Volume
```

✅ This proves:

- EBS mounted successfully
- Data persistence is working
- PV and PVC are working correctly (even though the Pod was destroyed and recreated, the data on the underlying EBS volume was untouched)

---

## 🛠️ Important Troubleshooting

### Problem 1 — Pod stuck in `ContainerCreating`

```bash
kubectl describe pod ebs-test-pod
```
Look at the **Events** section at the bottom for the real error (commonly a mount timeout or AZ mismatch).

### Problem 2 — EBS CSI Controller `CrashLoopBackOff`

```bash
kubectl get pods -n kube-system | grep ebs
```

Check logs:

```bash
kubectl logs -n kube-system deployment/ebs-csi-controller -c ebs-plugin
```

### Problem 3 — `UnauthorizedOperation`

**Reason:** IAM permission missing on the CSI driver's service account.

**Fix:** Attach the policy:

```text
AmazonEBSCSIDriverPolicy
```

### Problem 4 — Wrong filesystem / mount failure

**Cause:** Filesystem was created on a partition instead of the raw disk.

❌ Wrong:
```text
/dev/nvme1n1p1
```

✅ Correct:
```text
/dev/nvme1n1
```

---

## 🧹 Cleanup

Delete the pod:

```bash
kubectl delete -f pod.yaml
```

Delete the PV and PVC:

```bash
kubectl delete -f pv-pvc.yaml
```

> The actual EBS volume will **remain in AWS** (not deleted) because of:
> ```yaml
> persistentVolumeReclaimPolicy: Retain
> ```
> This is intentional — it protects your data from accidental loss when cleaning up Kubernetes objects.

---

## ✅ Final Understanding

### Flow

```text
AWS EBS Volume
      ↓
PersistentVolume
      ↓
PersistentVolumeClaim
      ↓
Pod
```

### Real World Usage

This pattern is commonly used for stateful workloads such as:

- Jenkins
- SonarQube
- MySQL
- PostgreSQL
- Prometheus
- Other stateful applications
- File upload applications

---

## 📚 Conclusion

In this practical we learned:

- Static provisioning
- PV and PVC concepts
- AWS EBS integration with EKS
- Persistent storage
- Data persistence (survives pod deletion)
- EBS CSI Driver setup
- Troubleshooting common storage issues

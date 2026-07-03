# 📦 Kubernetes Persistent Volumes (PV) and Persistent Volume Claims (PVC) with Dynamic Volumes (EBS)

## 1. Introduction

### 🔹 Persistent Volume (PV) 
A Persistent Volume is a storage resource in a Kubernetes cluster that provides persistent storage, independent of Pod lifecycles. It is defined and managed by the cluster administrator.

### 🔹 Persistent Volume Claim (PVC) 
A Persistent Volume Claim is a request for storage by a user. Pods use PVCs to access PVs.

### 🔹 Dynamic Provisioning
Dynamic provisioning automatically creates PVs based on a PVC when a StorageClass is specified. This is particularly useful for cloud-based storage systems like AWS EBS.

---

## 🛠️ Kubernetes Persistent Volume (PV) and Persistent Volume Claim (PVC) Demo

```bash
mkdir -p /mnt/data # step 1
df -hT
```

---

## 1. Persistent Volume (PV) (step-2)

### 🔹 Example: Persistent Volume (PV)
```yaml
apiVersion: v1   # API version used for PersistentVolume resource
kind: PersistentVolume   # Declares this manifest as a PersistentVolume (PV)
metadata: 
    name: my-vol   # Name of the PV (unique identifier in the cluster)

spec: 
  capacity: 
    storage: 5Gi   # The size of the PV (5 Gigabytes)

  volumeMode: Filesystem   # Specifies that the volume will be mounted as a filesystem (not raw block)

  accessModes:
      - ReadWriteOnce   # Only one node can mount this volume for read/write at a time

  persistentVolumeReclaimPolicy: Retain   # What happens when the PVC is deleted:
                                          # Retain → Keep the data
                                          # Delete → Remove the PV and data
                                          # Recycle → Wipe and make PV available again (deprecated)

  storageClassName: admin   # StorageClass name associated with this PV (used for dynamic provisioning)

  local:
    path: /mnt/data   # Path on the node’s local filesystem where data will be stored

  nodeAffinity:   # Ensures the PV is only usable on specific nodes
    required:
      nodeSelectorTerms: 
        - matchExpressions:
            - key: kubernetes.io/hostname   # Node label key used for selection
              operator: In                  # Operator meaning "must be in this list"
              values: ["controlplane"]      # Only the node with hostname "controlplane" can use this PV
```

#### 🔹 for Nfs
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /tmp
    server: <efs_network_interface_IP>
```

#### 🔹 for EBS
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  awsElasticBlockStore:
    volumeID: "<volume id>"
    fsType: ext
```

---

## 2. Persistent Volume Claim (PVC)
```yaml
apiVersion: v1   # API version for PersistentVolumeClaim resource
kind: PersistentVolumeClaim   # Declares this manifest as a PVC
metadata: 
   name: my-pvc   # Name of the PVC (unique identifier in the namespace)

spec: 
 accessModes:
   - ReadWriteOnce   # Access mode: the PVC requires a volume that can be mounted read/write by only one node

 resources:
   requests:
     storage: 5Gi   # The PVC is requesting 5Gi of storage

 storageClassName: admin   # Must match the PV's storageClassName so that it binds correctly

 volumeName: my-vol   # Explicitly binds this PVC to the PersistentVolume named "my-vol"
```

---

## 3. Pod Using PVC
```yaml
apiVersion: v1   # API version for Pod resource
kind: Pod        # Declares this manifest as a Pod
metadata: 
  name: my-pod   # Name of the Pod

spec:
  containers:
    - name: nginx   # Container name inside the Pod
      image: nginx:latest   # Container image to run (latest Nginx)
      ports:
        - containerPort: 80   # Exposes port 80 (default HTTP port) inside the container

      volumeMounts:   # Defines where to mount volumes inside the container
        - mountPath: "/usr/share/nginx/html"   # Path inside container where volume will be mounted
          name: my-vol   # Must match the volume name defined below

  volumes:   # Volumes available to the Pod
    - name: my-vol   # Name of the volume (referenced above in volumeMounts)
      persistentVolumeClaim:
        claimName: my-pvc   # PVC name → binds this volume to the PVC "my-pvc"
```

---

## 🧪 Testing Data Persistence

```bash
kubectl exec -it my-pod -- sh
echo "Persistent Storage Test!" > /usr/share/nginx/html/index.html
exit
kubectl delete pod my-pod
kubectl apply -f pod.yaml   #change the pod name
kubectl exec -it another-pod -- cat /usr/share/nginx/html/index.html

```

<img width="2256" height="872" alt="image" src="https://github.com/user-attachments/assets/db4e16bd-9f9f-4efc-9978-d886f71a15e3" />





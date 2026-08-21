# Kubernetes PersistentVolume (PV)

A **PersistentVolume (PV)** is a piece of storage in a Kubernetes cluster that has been **provisioned by an administrator or dynamically provisioned using a StorageClass**.

PV provides persistent storage that exists independently of the lifecycle of individual Pods and containers.

---

## 1. Why Do We Need PersistentVolume?

Containers are ephemeral.

If a container is deleted:

```text
Container
   ↓
Container filesystem
   ↓
Container deleted
   ↓
Data may be lost ❌
```

For applications such as:

* Databases
* E-commerce applications
* Logging systems
* File storage
* CMS applications

we need storage that persists beyond the Pod/container lifecycle.

That's where **PersistentVolume** comes in.

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Persistent Storage
```

---

# 2. What is a PersistentVolume?

A **PersistentVolume** is a Kubernetes API resource representing storage available to the cluster.

It can be backed by different storage systems:

```text
PersistentVolume
       |
       ├── Local Storage
       ├── NFS
       ├── AWS EBS
       ├── Azure Disk
       ├── Google Persistent Disk
       └── CSI Storage
```

Example:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: my-pv

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /mnt/data
```

---

# 3. PV Architecture

The basic architecture is:

```text
                    Kubernetes Cluster
                           |
                           |
                          Pod
                           |
                           ↓
                          PVC
                           |
                           ↓
                          PV
                           |
                           ↓
                    Actual Storage
```

The important relationship is:

```text
Pod → PVC → PV → Storage
```

---

# 4. PV Components

A PersistentVolume commonly contains:

```text
PersistentVolume
│
├── Capacity
├── Access Modes
├── Reclaim Policy
├── Storage Class
├── Volume Source
└── Volume Mode
```

---

# 5. Capacity

`capacity` defines how much storage the PV provides.

Example:

```yaml
capacity:
  storage: 1Gi
```

Other examples:

```yaml
storage: 500Mi
```

```yaml
storage: 5Gi
```

```yaml
storage: 10Gi
```

Common units:

```text
Ki
Mi
Gi
Ti
```

Example:

```text
1Gi ≈ 1 gigabyte of Kubernetes storage units
```

---

# 6. Access Modes

Access modes define how a volume can be mounted.

The three traditional access modes are:

```text
ReadWriteOnce
ReadOnlyMany
ReadWriteMany
```

---

## 6.1 ReadWriteOnce — RWO

The volume can be mounted as read-write by a single node.

```yaml
accessModes:
  - ReadWriteOnce
```

Conceptually:

```text
Node 1
  |
  └── Pod
       |
       └── Read + Write
```

Commonly used for:

* Databases
* Single-instance applications
* AWS EBS-style block storage

---

## 6.2 ReadOnlyMany — ROX

The volume can be mounted as read-only by multiple nodes.

```yaml
accessModes:
  - ReadOnlyMany
```

Example:

```text
Node 1 → Read
Node 2 → Read
Node 3 → Read
```

---

## 6.3 ReadWriteMany — RWX

The volume can be mounted as read-write by multiple nodes.

```yaml
accessModes:
  - ReadWriteMany
```

Example:

```text
Node 1 → Read + Write
Node 2 → Read + Write
Node 3 → Read + Write
```

Common storage systems supporting RWX include certain NFS and distributed storage solutions.

---

# 7. Reclaim Policy

The reclaim policy defines what happens to the underlying storage after a PVC is deleted.

Common policies:

```text
Retain
Delete
Recycle
```

`Recycle` is deprecated and should not be used for new configurations.

---

## 7.1 Retain

The storage is retained after the PVC is deleted.

```yaml
persistentVolumeReclaimPolicy: Retain
```

Flow:

```text
PVC deleted
     ↓
PV remains
     ↓
Data remains
```

This is useful when you don't want important data to be automatically deleted.

Example:

```text
Database
   ↓
PVC deleted accidentally
   ↓
PV retained
   ↓
Data preserved
```

---

## 7.2 Delete

The associated storage is deleted when the PVC is deleted, depending on the storage provisioner.

```yaml
persistentVolumeReclaimPolicy: Delete
```

Flow:

```text
PVC deleted
     ↓
PV released
     ↓
Underlying storage deleted
```

This is commonly used with dynamically provisioned cloud storage.

---

# 8. StorageClass

A PV can be associated with a StorageClass.

Example:

```yaml
storageClassName: standard
```

The StorageClass determines what type of storage should be provisioned.

Conceptually:

```text
PVC
 ↓
StorageClass
 ↓
Storage Provisioner
 ↓
PV
 ↓
Storage
```

---

# 9. Volume Source

A PV needs a storage source.

For learning purposes, you may see:

```yaml
hostPath:
  path: /mnt/data
```

Example:

```yaml
spec:
  hostPath:
    path: /mnt/data
```

In production environments, storage is commonly provided through CSI drivers and cloud storage systems.

For example:

```text
AWS
 ↓
EBS CSI Driver
 ↓
PersistentVolume
```

---

# 10. Complete PV Example

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: demo-pv

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  storageClassName: manual

  hostPath:
    path: /mnt/data
```

---

# 11. Understanding the YAML

### API Version

```yaml
apiVersion: v1
```

Uses the Kubernetes core API.

### Kind

```yaml
kind: PersistentVolume
```

Specifies that the resource is a PV.

### Name

```yaml
metadata:
  name: demo-pv
```

The PV is named `demo-pv`.

### Capacity

```yaml
capacity:
  storage: 1Gi
```

The PV provides 1 GiB of storage.

### Access Mode

```yaml
accessModes:
  - ReadWriteOnce
```

The storage can be mounted read-write by one node.

### Reclaim Policy

```yaml
persistentVolumeReclaimPolicy: Retain
```

The storage is retained after the PVC is deleted.

### Storage Class

```yaml
storageClassName: manual
```

Associates the PV with the `manual` storage class.

### Storage Source

```yaml
hostPath:
  path: /mnt/data
```

Uses `/mnt/data` on the Kubernetes node as the storage source.

---

# 12. Creating a PV

Create a file:

```bash
nano pv.yaml
```

Add the PV configuration:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: demo-pv

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  storageClassName: manual

  hostPath:
    path: /mnt/data
```

Apply it:

```bash
kubectl apply -f pv.yaml
```

---

# 13. Check PV

```bash
kubectl get pv
```

Example output:

```text
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      STORAGECLASS
demo-pv   1Gi        RWO            Retain           Available   manual
```

---

# 14. PV Status

A PV can have different statuses.

```text
Available
Bound
Released
Failed
```

---

## 14.1 Available

The PV exists but is not currently bound to a PVC.

```text
PV
 ↓
Available
```

Example:

```text
demo-pv   1Gi   Available
```

---

## 14.2 Bound

The PV has been successfully associated with a PVC.

```text
PV
 ↓
PVC
 ↓
Bound
```

Example:

```text
demo-pv   1Gi   Bound
```

---

## 14.3 Released

The PVC that was using the PV has been deleted.

```text
PVC deleted
     ↓
PV
     ↓
Released
```

The PV may still contain data.

The next action depends on the reclaim policy.

---

## 14.4 Failed

The PV has encountered an error during reclamation or another operation.

```text
PV
 ↓
Failed
```

---

# 15. PV Lifecycle

The typical lifecycle is:

```text
Available
    ↓
Bound
    ↓
Released
    ↓
Reclaimed
```

More specifically:

```text
PV Created
    ↓
Available
    ↓
PVC Created
    ↓
Bound
    ↓
PVC Deleted
    ↓
Released
    ↓
Retain / Delete
```

---

# 16. PersistentVolumeClaim and PV

A PV provides storage.

A PVC requests storage.

Think of it like:

```text
PV
=
Storage available

PVC
=
Storage request
```

Example:

```text
PV
1Gi
 ↓
PVC
requests 500Mi
 ↓
Bound
```

---

# 17. PV + PVC Architecture

```text
                 Kubernetes
                     |
                     ↓
                    Pod
                     |
                     ↓
                    PVC
              "I need 500Mi"
                     |
                     ↓
                    PV
              "I provide 1Gi"
                     |
                     ↓
              Actual Storage
```

---

# 18. Example PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: demo-pvc

spec:
  accessModes:
    - ReadWriteOnce

  storageClassName: manual

  resources:
    requests:
      storage: 500Mi
```

The PVC requests:

```text
500Mi
```

from a matching PV.

---

# 19. Binding Process

Suppose we have:

```text
PV:
1Gi
RWO
StorageClass: manual
```

And:

```text
PVC:
500Mi
RWO
StorageClass: manual
```

Kubernetes can bind them:

```text
PV: 1Gi
      ↓
     MATCH
      ↓
PVC: 500Mi
      ↓
    Bound
```

Check:

```bash
kubectl get pv
kubectl get pvc
```

---

# 20. Important Matching Rules

For a PVC to bind to a PV, Kubernetes considers factors such as:

* Storage capacity
* Access modes
* StorageClass
* Volume mode
* Selector requirements, if specified

Example:

```text
PV
Capacity: 1Gi
Access: RWO
Class: manual

PVC
Request: 500Mi
Access: RWO
Class: manual

        ↓

MATCH ✅
```

---

# 21. Using PV Through a Pod

A Pod normally uses the PVC.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: pv-demo

spec:
  containers:
    - name: nginx
      image: nginx

      volumeMounts:
        - name: storage
          mountPath: /data

  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: demo-pvc
```

Notice:

```yaml
persistentVolumeClaim:
  claimName: demo-pvc
```

The Pod does **not** directly reference:

```text
demo-pv
```

Instead:

```text
Pod
 ↓
PVC
 ↓
PV
```

---

# 22. Testing Persistent Storage

Create a file inside the Pod:

```bash
kubectl exec -it pv-demo -- sh
```

Then:

```bash
echo "Hello Kubernetes" > /data/message.txt
```

Check:

```bash
cat /data/message.txt
```

Exit:

```bash
exit
```

If the container restarts, the persistent volume can continue providing the stored data.

---

# 23. PV vs Container Storage

### Container Storage

```text
Container
   ↓
Container filesystem
   ↓
Container deleted
   ↓
Data may be lost
```

### PersistentVolume

```text
Container
   ↓
PVC
   ↓
PV
   ↓
Persistent Storage
```

The storage lifecycle is independent of the container filesystem.

---

# 24. PV vs `emptyDir`

| Feature                    | `emptyDir` | PV        |
| -------------------------- | ---------- | --------- |
| Persistent storage         | ❌          | ✅         |
| Survives container restart | ✅          | ✅         |
| Survives Pod deletion      | ❌          | ✅*        |
| Intended for databases     | ❌          | ✅         |
| Can use external storage   | ❌          | ✅         |
| Can use cloud storage      | ❌          | ✅         |
| Uses PVC                   | ❌          | Usually ✅ |
| Production storage         | Limited    | ✅         |

`*` Persistence depends on the storage backend and lifecycle/reclaim configuration.

---

# 25. PV vs PVC

| PV                              | PVC                         |
| ------------------------------- | --------------------------- |
| Storage resource                | Storage request             |
| Cluster-level resource          | Namespace-level resource    |
| Created by admin or provisioner | Created by user/application |
| Provides storage                | Requests storage            |
| Can be bound to PVC             | Binds to PV                 |

Easy way to remember:

```text
PV = Provider

PVC = Claim
```

---

# 26. Static Provisioning

In static provisioning, the administrator creates the PV manually.

```text
Administrator
      ↓
Create PV
      ↓
Create PVC
      ↓
PVC binds to PV
      ↓
Pod uses PVC
```

Example:

```text
Admin
 ↓
PV
 ↓
PVC
 ↓
Pod
```

---

# 27. Dynamic Provisioning

In dynamic provisioning, Kubernetes automatically creates storage when a PVC requests it.

```text
PVC
 ↓
StorageClass
 ↓
Provisioner
 ↓
PV created automatically
 ↓
Storage
```

This is commonly used in cloud environments.

---

# 28. StorageClass and Dynamic Provisioning

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: dynamic-pvc

spec:
  storageClassName: standard

  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi
```

The user only creates the PVC.

The StorageClass/provisioner handles creating the required persistent storage.

---

# 29. Volume Modes

PV supports two volume modes:

```text
Filesystem
Block
```

---

## Filesystem

The volume is mounted as a filesystem.

```yaml
volumeMode: Filesystem
```

This is the default.

Example:

```text
/data
 ├── file1
 ├── file2
 └── file3
```

---

## Block

The volume is exposed as a raw block device.

```yaml
volumeMode: Block
```

Used for specialized workloads that need direct block-device access.

---

# 30. Useful PV Commands

### List PVs

```bash
kubectl get pv
```

### Detailed PV information

```bash
kubectl describe pv demo-pv
```

### Get PV YAML

```bash
kubectl get pv demo-pv -o yaml
```

### List PVCs

```bash
kubectl get pvc
```

### Describe PVC

```bash
kubectl describe pvc demo-pvc
```

### List StorageClasses

```bash
kubectl get storageclass
```

### Delete PV

```bash
kubectl delete pv demo-pv
```

### Delete PVC

```bash
kubectl delete pvc demo-pvc
```

---

# 31. Common Troubleshooting

## PVC stuck in Pending

Check:

```bash
kubectl get pvc
```

Then:

```bash
kubectl describe pvc <pvc-name>
```

Check:

* StorageClass
* Available PV
* Capacity
* Access modes
* Volume mode
* Provisioner

---

## PV stuck in Released

Check:

```bash
kubectl get pv
```

Then:

```bash
kubectl describe pv <pv-name>
```

Check the reclaim policy.

If:

```text
Retain
```

the PV and underlying data are intentionally retained.

---

## Pod stuck with volume mount errors

Check:

```bash
kubectl describe pod <pod-name>
```

Look at:

```text
Events:
```

Also verify:

```bash
kubectl get pv
kubectl get pvc
```

The PVC should normally be:

```text
Bound
```

---

# 32. Important Interview Questions

### Q1. What is a PersistentVolume?

A PersistentVolume is a cluster-level storage resource that provides persistent storage to Kubernetes workloads.

---

### Q2. Why do we need PV?

Because container and Pod filesystems are generally ephemeral. PV allows applications to use persistent storage.

---

### Q3. What is the difference between PV and PVC?

```text
PV → Provides storage

PVC → Requests storage
```

---

### Q4. What happens when a PVC is deleted?

It depends on the PV's reclaim policy.

```text
Retain → Storage is retained

Delete → Underlying storage may be deleted
```

---

### Q5. What are PV access modes?

```text
RWO → ReadWriteOnce

ROX → ReadOnlyMany

RWX → ReadWriteMany
```

---

### Q6. What is the default volume mode?

```text
Filesystem
```

---

### Q7. What is a reclaim policy?

It determines what happens to the PV/storage after the PVC is released.

Common policies:

```text
Retain
Delete
```

---

### Q8. What is the PV lifecycle?

```text
Available
    ↓
Bound
    ↓
Released
    ↓
Reclaimed
```

---

### Q9. Can a Pod directly use a PV?

Typically, no.

The normal pattern is:

```text
Pod → PVC → PV
```

---

### Q10. What is dynamic provisioning?

Dynamic provisioning automatically creates persistent storage when a PVC requests it, usually through a StorageClass.

---

### Q11. What is StorageClass?

A StorageClass defines a class/type of storage and enables dynamic provisioning through a storage provisioner.

---

### Q12. What is the difference between `emptyDir` and PV?

`emptyDir` is temporary storage tied to a Pod, while PV represents persistent storage that has a lifecycle independent of the Pod.

---

# 33. Real-World Example

Consider a MongoDB application running in Kubernetes.

Without persistent storage:

```text
MongoDB Pod
    ↓
MongoDB data
    ↓
Pod deleted
    ↓
Data lost ❌
```

With persistent storage:

```text
MongoDB Pod
    ↓
PVC
    ↓
PV
    ↓
Persistent Storage
```

Now:

```text
Pod deleted
    ↓
New MongoDB Pod
    ↓
Same PVC
    ↓
Same persistent storage
    ↓
Data available ✅
```

This is why persistent storage is critical for stateful applications.

---

# 34. Complete Storage Flow

Remember this architecture:

```text
                  APPLICATION
                       |
                       ↓
                      POD
                       |
                       ↓
                     PVC
              "I need 5Gi storage"
                       |
                       ↓
                STORAGE CLASS
                       |
                       ↓
                  PROVISIONER
                       |
                       ↓
                      PV
                       |
                       ↓
               ACTUAL STORAGE
                       |
          ┌────────────┼────────────┐
          ↓            ↓            ↓
        AWS EBS       NFS       Local Disk
```

---

# 35. Quick Revision

```text
PV
↓
PersistentVolume
↓
Cluster-level storage resource

PVC
↓
PersistentVolumeClaim
↓
Request for storage

StorageClass
↓
Defines storage type
↓
Enables dynamic provisioning

RWO
↓
ReadWriteOnce

ROX
↓
ReadOnlyMany

RWX
↓
ReadWriteMany

Retain
↓
Keep storage after PVC deletion

Delete
↓
Delete underlying provisioned storage

Filesystem
↓
Normal mounted filesystem

Block
↓
Raw block device
```

### One-Line Memory Trick

```text
PV = Storage Provider
PVC = Storage Request
Pod = Storage Consumer
StorageClass = Storage Automation
```

### Most Important Flow

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage
```

For DevOps interviews, make sure you can explain **PV vs PVC, access modes, reclaim policies, PV lifecycle, static vs dynamic provisioning, and what happens to data when a Pod/PVC is deleted**.

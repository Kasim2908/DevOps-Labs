# Kubernetes Volumes

Kubernetes Pods are **ephemeral** by nature. Data stored inside a container can be lost when the container restarts or is recreated.

Kubernetes Volumes provide a way to **store and share data** between containers and preserve data beyond the lifecycle of a container.

---

## 1. Why Do We Need Volumes?

By default, data written inside a container's filesystem is temporary.

For example:

```text
Pod
 └── Container
      └── /app/data
```

If the container is deleted:

```text
Container deleted
      ↓
Container filesystem deleted
      ↓
Data lost ❌
```

Volumes solve this problem by providing storage outside the container's temporary filesystem.

---

## 2. Kubernetes Volume Concept

A volume is attached to a Pod and mounted inside one or more containers.

```text
                 Pod
                  |
        ┌─────────┴─────────┐
        |                   |
   Container 1         Container 2
        |                   |
        └─────── Volume ─────┘
```

Multiple containers inside the same Pod can use the same volume.

---

# 3. Basic Volume Structure

A Kubernetes volume generally has two important parts:

### `volumes`

Defines the storage source.

### `volumeMounts`

Defines where the volume should appear inside the container.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-demo
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: storage
          mountPath: /data

  volumes:
    - name: storage
      emptyDir: {}
```

Here:

```text
volume name     → storage
mountPath       → /data
volume type     → emptyDir
```

---

# 4. `emptyDir`

`emptyDir` creates an empty directory when the Pod is assigned to a node.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: cache-volume
          mountPath: /cache

  volumes:
    - name: cache-volume
      emptyDir: {}
```

### How it works

```text
Pod created
    ↓
emptyDir created
    ↓
Container writes data
    ↓
Pod remains alive → data remains
    ↓
Pod deleted → data deleted
```

### Important

`emptyDir` survives:

* Container restart ✅

But does **not** survive:

* Pod deletion ❌
* Pod recreation ❌
* Node failure ❌

### Common use cases

* Temporary files
* Cache
* Sharing files between containers
* Scratch space

---

# 5. `hostPath`

`hostPath` mounts a directory or file from the **Kubernetes node's filesystem** into a Pod.

```text
Kubernetes Node
│
└── /data
      ↑
      │
   hostPath
      │
      ↓
Pod
└── /app/data
```

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
    - name: nginx
      image: nginx

      volumeMounts:
        - name: node-storage
          mountPath: /app/data

  volumes:
    - name: node-storage
      hostPath:
        path: /data
        type: DirectoryOrCreate
```

Here:

```text
Node path:
    /data

Container path:
    /app/data
```

Anything written to:

```text
/app/data
```

inside the container is actually stored at:

```text
/data
```

on the Kubernetes node.

---

# 6. `hostPath` Types

Kubernetes supports different `hostPath` types.

| Type                | Description                           |
| ------------------- | ------------------------------------- |
| `Directory`         | Directory must already exist          |
| `DirectoryOrCreate` | Creates directory if it doesn't exist |
| `File`              | File must already exist               |
| `FileOrCreate`      | Creates file if it doesn't exist      |
| `Socket`            | Unix socket                           |
| `CharDevice`        | Character device                      |
| `BlockDevice`       | Block device                          |

For learning purposes, `DirectoryOrCreate` is commonly used.

Example:

```yaml
hostPath:
  path: /data
  type: DirectoryOrCreate
```

---

# 7. Important `hostPath` Limitation

`hostPath` is tied to a **specific node**.

Suppose:

```text
Node 1
└── /data

Node 2
└── /data
```

A Pod running on Node 1 writes:

```text
/data/file.txt
```

If the Pod moves to Node 2, it may see a completely different `/data`.

```text
Pod
 ↓
Node 1
 ↓
/data/file.txt
```

After rescheduling:

```text
Pod
 ↓
Node 2
 ↓
/data
```

The original data may not be available.

Therefore:

> `hostPath` is generally not recommended for production application storage.

It is useful for:

* Learning
* Testing
* Node-level applications
* DaemonSets
* Accessing node files
* Local development

---

# 8. `hostPath` Example

Create:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
    - name: nginx
      image: nginx

      volumeMounts:
        - name: node-storage
          mountPath: /usr/share/nginx/html

  volumes:
    - name: node-storage
      hostPath:
        path: /data
        type: DirectoryOrCreate
```

Apply:

```bash
kubectl apply -f hostpath.yaml
```

Check:

```bash
kubectl get pods
```

Describe:

```bash
kubectl describe pod hostpath-demo
```

Check the mounted directory:

```bash
kubectl exec -it hostpath-demo -- ls /usr/share/nginx/html
```

---

# 9. `emptyDir` vs `hostPath`

| Feature                     | `emptyDir` | `hostPath`           |
| --------------------------- | ---------- | -------------------- |
| Storage location            | Node       | Node                 |
| Created automatically       | Yes        | Depends on type      |
| Survives container restart  | Yes        | Yes                  |
| Survives Pod deletion       | No         | Data remains on node |
| Node dependent              | Yes        | Yes                  |
| Production database storage | ❌          | ❌ Generally          |
| Temporary storage           | ✅          | ✅                    |
| Node-level access           | ❌          | ✅                    |

---

# 10. Persistent Storage

For production applications, Kubernetes provides persistent storage mechanisms.

The major concepts are:

```text
Storage
   ↓
PersistentVolume (PV)
   ↓
PersistentVolumeClaim (PVC)
   ↓
Pod
```

---

# 11. PersistentVolume (PV)

A **PersistentVolume** is a piece of storage available to the Kubernetes cluster.

It can be backed by:

* AWS EBS
* NFS
* Azure Disk
* Google Persistent Disk
* Local storage
* Other storage systems

Conceptually:

```text
Cluster
   |
   └── PersistentVolume
          |
          └── Storage
```

Example:

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

  hostPath:
    path: /mnt/data
```

---

# 12. PersistentVolumeClaim (PVC)

A **PersistentVolumeClaim** is a request for storage made by a user/application.

Think of it as:

```text
PV  = Available storage

PVC = Request for storage
```

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 1Gi
```

---

# 13. Pod + PVC

A Pod normally uses the PVC instead of directly referencing the PV.

```text
Pod
 |
 | uses
 ↓
PVC
 |
 | binds to
 ↓
PV
 |
 ↓
Actual Storage
```

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo
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

---

# 14. Access Modes

PVCs and PVs use access modes to define how storage can be accessed.

### ReadWriteOnce — RWO

Volume can be mounted as read-write by a single node.

```text
Node 1
 └── Pod → Read + Write
```

### ReadOnlyMany — ROX

Volume can be mounted as read-only by multiple nodes.

```text
Node 1 → Read
Node 2 → Read
Node 3 → Read
```

### ReadWriteMany — RWX

Volume can be mounted as read-write by multiple nodes.

```text
Node 1 → Read + Write
Node 2 → Read + Write
Node 3 → Read + Write
```

---

# 15. StorageClass

A `StorageClass` allows Kubernetes to dynamically provision storage.

Without dynamic provisioning:

```text
Admin
 ↓
Create PV
 ↓
Create PVC
 ↓
Pod
```

With dynamic provisioning:

```text
Pod
 ↓
PVC
 ↓
StorageClass
 ↓
Storage automatically created
```

This is especially useful in cloud environments.

---

# 16. Static vs Dynamic Provisioning

### Static Provisioning

Administrator creates storage manually.

```text
Admin
 ↓
PV
 ↓
PVC
 ↓
Pod
```

### Dynamic Provisioning

Kubernetes automatically provisions storage.

```text
PVC
 ↓
StorageClass
 ↓
Cloud Storage
 ↓
Pod
```

---

# 17. Common Kubernetes Volume Types

Some commonly encountered volume types are:

```text
emptyDir
hostPath
configMap
secret
persistentVolumeClaim
projected
NFS
CSI
```

Cloud environments commonly use CSI drivers for persistent storage.

---

# 18. Volume vs VolumeMount

This is an important interview concept.

### `volumes`

Defines **where the storage comes from**.

```yaml
volumes:
  - name: storage
    emptyDir: {}
```

### `volumeMounts`

Defines **where the storage is mounted inside the container**.

```yaml
volumeMounts:
  - name: storage
    mountPath: /data
```

Remember:

```text
volumes
   ↓
Storage definition

volumeMounts
   ↓
Container mount location
```

---

# 19. Multiple Containers Sharing a Volume

Containers inside the same Pod can share a volume.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume
spec:
  containers:

    - name: container-1
      image: busybox
      command: ["sh", "-c", "echo Hello > /data/message.txt; sleep 3600"]

      volumeMounts:
        - name: shared-storage
          mountPath: /data

    - name: container-2
      image: busybox
      command: ["sh", "-c", "cat /data/message.txt; sleep 3600"]

      volumeMounts:
        - name: shared-storage
          mountPath: /data

  volumes:
    - name: shared-storage
      emptyDir: {}
```

Both containers access:

```text
/data
```

and therefore can share files.

---

# 20. Useful Commands

### List Pods

```bash
kubectl get pods
```

### Describe Pod

```bash
kubectl describe pod <pod-name>
```

### Check Pod YAML

```bash
kubectl get pod <pod-name> -o yaml
```

### Execute into container

```bash
kubectl exec -it <pod-name> -- sh
```

### Check mounted directory

```bash
kubectl exec -it <pod-name> -- ls -la /data
```

### Check volume information

```bash
kubectl describe pod <pod-name>
```

### List PVs

```bash
kubectl get pv
```

### List PVCs

```bash
kubectl get pvc
```

### List StorageClasses

```bash
kubectl get storageclass
```

---

# 21. Important Interview Questions

### Q1. Why are Kubernetes volumes required?

Containers have ephemeral filesystems. Volumes allow applications to store data and share data between containers.

### Q2. Does `emptyDir` survive Pod deletion?

No.

`emptyDir` exists only for the lifetime of the Pod.

### Q3. What is `hostPath`?

`hostPath` mounts a file or directory from the Kubernetes node's filesystem into a Pod.

### Q4. What is the major problem with `hostPath`?

It is node-dependent. If the Pod moves to another node, the data on the original node may not be available.

### Q5. What is a PV?

A PersistentVolume is cluster storage provisioned for Kubernetes workloads.

### Q6. What is a PVC?

A PersistentVolumeClaim is a request for storage by a workload.

### Q7. What is the difference between PV and PVC?

```text
PV  → Storage resource

PVC → Request for storage
```

### Q8. What is StorageClass?

A StorageClass defines how storage should be dynamically provisioned.

### Q9. Can multiple containers share the same volume?

Yes. Containers in the same Pod can mount the same volume.

### Q10. What is the difference between `volumes` and `volumeMounts`?

```text
volumes
→ Defines storage

volumeMounts
→ Defines where storage is mounted
```

---

# 22. Kubernetes Storage Architecture

The overall architecture can be remembered as:

```text
                    Kubernetes Cluster
                           |
                    ┌──────┴──────┐
                    |             |
                   Pod           Pod
                    |
                   PVC
                    |
             StorageClass
                    |
                    ↓
                   PV
                    |
                    ↓
             Actual Storage
                    |
        ┌───────────┼───────────┐
        ↓           ↓           ↓
      AWS EBS      NFS       Local Disk
```

---

# 23. Quick Revision

```text
Volume
  ↓
Provides storage to containers

emptyDir
  ↓
Temporary Pod-level storage

hostPath
  ↓
Mounts node filesystem

PV
  ↓
Persistent storage resource

PVC
  ↓
Request for persistent storage

StorageClass
  ↓
Dynamic storage provisioning

volumeMounts
  ↓
Mount storage inside container
```

### Most Important Flow

```text
Application
     ↓
    Pod
     ↓
    PVC
     ↓
StorageClass
     ↓
     PV
     ↓
Actual Storage
```

> **Interview tip:** For DevOps interviews, understand not just the YAML syntax but **what happens to the data when a container restarts, a Pod is deleted, or a Pod moves to another node**. These scenarios are frequently more important than memorizing volume definitions.

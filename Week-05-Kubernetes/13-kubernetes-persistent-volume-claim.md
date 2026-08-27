# Kubernetes PersistentVolumeClaim (PVC) — Complete Notes

> Complete Kubernetes PVC notes from **basics to advanced level**, including concepts, YAML manifests, access modes, storage classes, dynamic provisioning, resizing, snapshots, troubleshooting, security, StatefulSets, and interview points.

---

## 1. What is a PVC?

A **PersistentVolumeClaim (PVC)** is a request for storage made by a Kubernetes workload.

A Pod should generally not need to know:

* Where the storage physically exists
* Which disk provides the storage
* How the storage is provisioned
* Which storage backend is being used

Instead:

```text
Pod
 |
 | uses
 v
PVC
 |
 | binds to
 v
PV
 |
 | backed by
 v
Storage
```

### Simple analogy

Think of:

```text
PVC = Storage Request
PV  = Actual Storage
Pod = Consumer
```

For example:

```yaml
resources:
  requests:
    storage: 10Gi
```

means:

> "I need 10 GiB of persistent storage."

---

# 2. Why Do We Need PVC?

Container filesystems are normally **ephemeral**.

If a container is deleted or recreated, data stored inside its writable container filesystem can disappear.

For example:

```text
Pod
 |
 +-- Container
      |
      +-- /app
      +-- /data
```

If `/data` is not backed by persistent storage:

```text
Pod deleted
    |
    v
Container deleted
    |
    v
Container filesystem lost
    |
    v
Data lost
```

With a PVC:

```text
Pod
 |
 v
PVC
 |
 v
PV
 |
 v
Persistent Storage
```

The Pod can be recreated while the persistent data remains.

---

# 3. PVC vs PV

| Feature                  | PVC                      | PV                    |
| ------------------------ | ------------------------ | --------------------- |
| Full name                | PersistentVolumeClaim    | PersistentVolume      |
| Created by               | Usually application/user | Admin or provisioner  |
| Purpose                  | Requests storage         | Provides storage      |
| Contains storage request | Yes                      | Yes                   |
| Contains actual backend  | No                       | Yes                   |
| Namespace                | Namespaced               | Cluster-scoped        |
| Used directly by Pod     | Yes                      | Usually indirectly    |
| Dynamic provisioning     | Can trigger it           | Created automatically |

### Remember

```text
PVC = I need storage
PV  = Here is storage
```

---

# 4. PVC Lifecycle

A typical lifecycle is:

```text
PVC Created
    |
    v
Pending
    |
    v
Bound
    |
    v
Used by Pod
    |
    v
Pod Deleted
    |
    v
PVC Deleted
    |
    v
PV Reclaimed
```

Possible PVC states:

```text
Pending
Bound
Lost
```

---

# 5. PVC YAML Structure

Basic PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: my-pvc

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi
```

Apply:

```bash
kubectl apply -f pvc.yaml
```

Check:

```bash
kubectl get pvc
```

Detailed information:

```bash
kubectl describe pvc my-pvc
```

---

# 6. Important PVC Fields

A PVC commonly contains:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-pvc

spec:
  accessModes:
    - ReadWriteOnce

  storageClassName: standard

  resources:
    requests:
      storage: 10Gi
```

Important fields:

### `metadata.name`

Name of the PVC.

```yaml
metadata:
  name: app-pvc
```

---

### `accessModes`

Defines how the volume can be mounted.

```yaml
accessModes:
  - ReadWriteOnce
```

Common modes:

```text
ReadWriteOnce
ReadOnlyMany
ReadWriteMany
ReadWriteOncePod
```

---

### `resources.requests.storage`

Requested storage size.

```yaml
resources:
  requests:
    storage: 10Gi
```

---

### `storageClassName`

Specifies which StorageClass should provision the storage.

```yaml
storageClassName: standard
```

---

### `volumeName`

Can explicitly bind the PVC to a particular PV.

```yaml
volumeName: my-pv
```

Usually this is unnecessary when using dynamic provisioning.

---

### `selector`

Can be used to select matching PVs.

```yaml
selector:
  matchLabels:
    environment: production
```

---

# 7. PVC Access Modes

Access modes determine how storage can be mounted.

## ReadWriteOnce — RWO

```text
Read + Write
```

The volume can be mounted read-write by a single node.

```yaml
accessModes:
  - ReadWriteOnce
```

Commonly used with:

* Databases
* Single-instance applications
* Block storage

Important:

> `ReadWriteOnce` means read-write from a single node, not necessarily only a single Pod.

---

# 8. ReadOnlyMany — ROX

```text
Read Only + Multiple Nodes
```

```yaml
accessModes:
  - ReadOnlyMany
```

Multiple nodes can mount the volume as read-only.

Useful for:

* Shared static content
* Reference data
* Read-only datasets

---

# 9. ReadWriteMany — RWX

```text
Read + Write + Multiple Nodes
```

```yaml
accessModes:
  - ReadWriteMany
```

Multiple nodes can mount and write to the volume.

Common backends include:

* NFS
* Azure Files
* Amazon EFS
* Other shared filesystem solutions

---

# 10. ReadWriteOncePod — RWOP

```text
Read + Write + Single Pod
```

```yaml
accessModes:
  - ReadWriteOncePod
```

This restricts read-write mounting to a **single Pod**.

This is stricter than:

```text
ReadWriteOnce
```

It is useful when the storage must be attached to exactly one Pod.

---

# 11. Important Access Mode Comparison

| Mode | Read | Write | Multiple Nodes | Typical Use       |
| ---- | ---- | ----- | -------------- | ----------------- |
| RWO  | Yes  | Yes   | No*            | Database          |
| ROX  | Yes  | No    | Yes            | Shared read-only  |
| RWX  | Yes  | Yes   | Yes            | Shared filesystem |
| RWOP | Yes  | Yes   | No             | Single Pod        |

`*` RWO is node-oriented; exact behavior depends on the storage implementation.

---

# 12. PVC and Pod Relationship

A PVC is not mounted automatically.

The Pod must reference the PVC.

PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: app-storage
          mountPath: /data

  volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: app-pvc
```

Architecture:

```text
Pod
 |
 +-- volumeMounts
 |      |
 |      v
 |    /data
 |
 +-- volumes
        |
        v
       PVC
        |
        v
       PV
        |
        v
     Storage
```

---

# 13. Complete PVC + Pod Example

### PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi
```

### Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx

spec:
  containers:
    - name: nginx
      image: nginx:latest

      volumeMounts:
        - name: nginx-storage
          mountPath: /usr/share/nginx/html

  volumes:
    - name: nginx-storage
      persistentVolumeClaim:
        claimName: nginx-pvc
```

Deploy:

```bash
kubectl apply -f pvc.yaml
kubectl apply -f pod.yaml
```

Verify:

```bash
kubectl get pvc
kubectl get pv
kubectl get pod
```

---

# 14. Static Provisioning

In static provisioning, the administrator creates the PV first.

Architecture:

```text
Admin
 |
 v
PV Created
 |
 v
PVC Created
 |
 v
PVC matches PV
 |
 v
Bound
```

Example PV:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: my-pv

spec:
  capacity:
    storage: 10Gi

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  hostPath:
    path: /mnt/data
```

PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: my-pvc

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi
```

---

# 15. Dynamic Provisioning

Dynamic provisioning automatically creates storage when a PVC requests it.

Architecture:

```text
PVC
 |
 v
StorageClass
 |
 v
CSI Provisioner
 |
 v
Cloud / Storage Backend
 |
 v
PV
 |
 v
PVC Bound
```

This is the preferred approach in many production Kubernetes environments.

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-pvc

spec:
  storageClassName: standard

  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 20Gi
```

Kubernetes can dynamically create the required PV.

---

# 16. StorageClass

A StorageClass defines how storage should be dynamically provisioned.

Example:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast-storage

provisioner: example.com/csi

parameters:
  type: fast

reclaimPolicy: Delete

volumeBindingMode: WaitForFirstConsumer
```

Important fields:

```text
provisioner
parameters
reclaimPolicy
volumeBindingMode
allowVolumeExpansion
mountOptions
```

---

# 17. Default StorageClass

A cluster may have a default StorageClass.

Check:

```bash
kubectl get storageclass
```

Example:

```text
NAME                 PROVISIONER
standard (default)   ...
```

If a PVC does not specify:

```yaml
storageClassName:
```

Kubernetes may use the default StorageClass.

---

# 18. Explicitly Disable Dynamic Provisioning

If you want a PVC without a StorageClass:

```yaml
storageClassName: ""
```

This is different from simply omitting the field.

```yaml
storageClassName: ""
```

means:

> Do not automatically use the default StorageClass.

---

# 19. PVC Binding

Kubernetes tries to find a suitable PV based on requirements such as:

```text
Storage size
Access mode
StorageClass
Selectors
Volume attributes
```

Example:

```text
PVC:
10Gi
RWO
standard

        ↓

PV:
20Gi
RWO
standard

        ↓

BOUND
```

A PV generally needs sufficient capacity and compatible attributes.

---

# 20. PVC States

## Pending

```text
PVC → Pending
```

Usually means Kubernetes has not found or provisioned suitable storage.

Check:

```bash
kubectl get pvc
kubectl describe pvc <pvc-name>
```

---

## Bound

```text
PVC → Bound
```

The PVC has successfully been associated with a PV.

Check:

```bash
kubectl get pvc
```

---

## Lost

`Lost` can indicate that the previously bound PV is no longer available.

Investigate:

```bash
kubectl describe pvc <pvc-name>
kubectl get pv
```

---

# 21. Reclaim Policies

PV reclaim policies determine what happens when the PVC is deleted.

Common policies:

```text
Retain
Delete
Recycle
```

`Recycle` is deprecated/removed from normal modern Kubernetes usage.

---

# 22. Retain Policy

```yaml
persistentVolumeReclaimPolicy: Retain
```

When PVC is deleted:

```text
PVC deleted
   |
   v
PV retained
   |
   v
Underlying data retained
```

Useful for:

* Databases
* Critical production data
* Manual recovery workflows

---

# 23. Delete Policy

```yaml
persistentVolumeReclaimPolicy: Delete
```

When the PVC is deleted, the dynamically provisioned storage resource may also be deleted according to the provisioner.

Use carefully in production.

---

# 24. PVC Storage Expansion

Many modern storage drivers support volume expansion.

StorageClass:

```yaml
allowVolumeExpansion: true
```

Initial PVC:

```yaml
resources:
  requests:
    storage: 10Gi
```

Later:

```yaml
resources:
  requests:
    storage: 20Gi
```

Apply:

```bash
kubectl apply -f pvc.yaml
```

Check:

```bash
kubectl get pvc
```

Important:

> PVC expansion depends on the StorageClass and CSI driver supporting expansion.

---

# 25. Can PVC Storage Be Reduced?

Generally:

```text
PVC expansion → Supported when configured
PVC shrinking → Not supported
```

Do not try to change:

```text
20Gi → 10Gi
```

as a normal PVC resize operation.

For shrinking, create a new smaller volume and migrate the data.

---

# 26. Volume Snapshots

Kubernetes can use the CSI snapshot framework to create volume snapshots.

Conceptually:

```text
PVC
 |
 v
VolumeSnapshot
 |
 v
Snapshot Data
```

Example:

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot

metadata:
  name: app-snapshot

spec:
  volumeSnapshotClassName: csi-snapclass

  source:
    persistentVolumeClaimName: app-pvc
```

Check:

```bash
kubectl get volumesnapshot
```

Important:

> Snapshot support depends on the CSI driver and snapshot infrastructure.

---

# 27. Restore PVC from Snapshot

A PVC can be created from a snapshot if the environment supports it.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: restored-pvc

spec:
  accessModes:
    - ReadWriteOnce

  storageClassName: standard

  resources:
    requests:
      storage: 10Gi

  dataSource:
    name: app-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
```

Architecture:

```text
Original PVC
     |
     v
VolumeSnapshot
     |
     v
Restored PVC
     |
     v
New PV
```

---

# 28. PVC Cloning

Some CSI drivers support cloning an existing PVC.

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: cloned-pvc

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 10Gi

  dataSource:
    name: source-pvc
    kind: PersistentVolumeClaim
```

This creates a new PVC from an existing PVC.

Support depends on the CSI driver.

---

# 29. PVC with Deployment

Example:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx

spec:
  replicas: 1

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

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
            claimName: app-pvc
```

---

# 30. PVC with StatefulSet

Stateful applications commonly require one storage volume per Pod.

Examples:

```text
MySQL
PostgreSQL
MongoDB
Elasticsearch
Kafka
```

A StatefulSet can use:

```yaml
volumeClaimTemplates:
```

Example:

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: database

spec:
  serviceName: database
  replicas: 3

  selector:
    matchLabels:
      app: database

  template:
    metadata:
      labels:
        app: database

    spec:
      containers:
        - name: database
          image: postgres

          volumeMounts:
            - name: database-storage
              mountPath: /var/lib/postgresql/data

  volumeClaimTemplates:
    - metadata:
        name: database-storage

      spec:
        accessModes:
          - ReadWriteOnce

        resources:
          requests:
            storage: 10Gi
```

This can result in:

```text
database-0 → database-storage-database-0
database-1 → database-storage-database-1
database-2 → database-storage-database-2
```

---

# 31. Why StatefulSet Uses PVC Templates

A Deployment typically has interchangeable Pods.

A StatefulSet provides stable identity.

For example:

```text
database-0
database-1
database-2
```

Each can have its own storage:

```text
database-0 → PVC-0
database-1 → PVC-1
database-2 → PVC-2
```

This is extremely important for stateful workloads.

---

# 32. Volume Binding Modes

StorageClass supports different binding behavior.

Common values:

```text
Immediate
WaitForFirstConsumer
```

---

## Immediate

The volume may be provisioned as soon as the PVC is created.

```text
PVC
 ↓
PV provisioned
```

---

## WaitForFirstConsumer

Provisioning waits until a Pod using the PVC is scheduled.

```text
PVC
 |
 v
Pod scheduled
 |
 v
Storage provisioned
```

This is especially useful for topology-aware storage.

For example:

```text
Availability Zone
Node
Region
```

The scheduler can make a placement decision before storage is provisioned.

---

# 33. CSI — Container Storage Interface

Modern Kubernetes storage integrations commonly use **CSI drivers**.

CSI allows storage vendors to integrate their storage systems with Kubernetes.

Architecture:

```text
Kubernetes
    |
    v
PVC
    |
    v
StorageClass
    |
    v
CSI Driver
    |
    +---- Cloud Block Storage
    |
    +---- Network File System
    |
    +---- SAN
    |
    +---- Distributed Storage
```

Examples include drivers for:

```text
AWS EBS
AWS EFS
Azure Disk
Azure Files
Google Persistent Disk
Ceph
NetApp
```

---

# 34. Important PVC/Storage Architecture

A production storage flow often looks like:

```text
Application
     |
     v
    Pod
     |
     v
    PVC
     |
     v
StorageClass
     |
     v
CSI Controller
     |
     v
Storage Backend
     |
     v
Persistent Storage
```

Node-side CSI components then handle mounting and attachment operations on appropriate nodes.

---

# 35. StorageClass Parameters

A StorageClass can contain backend-specific parameters.

Example:

```yaml
parameters:
  type: fast
```

Depending on the driver, parameters can control things such as:

```text
Disk type
Performance
Replication
Filesystem
Encryption
IOPS
Throughput
Availability zone
```

Always consult the CSI driver's documentation because parameters are driver-specific.

---

# 36. Filesystem vs Block Volume

Storage can be presented as:

```text
Filesystem
```

or:

```text
Raw Block Device
```

Filesystem example:

```yaml
volumeMounts:
  - name: storage
    mountPath: /data
```

Raw block example:

```yaml
volumeDevices:
  - name: storage
    devicePath: /dev/xvda
```

Block-mode usage is more specialized and depends on application requirements and CSI support.

---

# 37. Volume Modes

PVC supports:

```text
Filesystem
Block
```

Filesystem is the default.

Example:

```yaml
volumeMode: Filesystem
```

Block:

```yaml
volumeMode: Block
```

---

# 38. PVC Namespace

PVCs are **namespaced resources**.

Example:

```bash
kubectl get pvc -n production
```

A Pod can normally reference a PVC from its own namespace.

You cannot directly reference:

```text
namespace-a PVC
```

from a Pod in:

```text
namespace-b
```

---

# 39. PV Is Cluster Scoped

Unlike PVC, PV is not namespaced.

```bash
kubectl get pv
```

There is no:

```bash
kubectl get pv -n production
```

because PVs are cluster-scoped.

---

# 40. PVC and StorageClass Matching

When a PVC specifies:

```yaml
storageClassName: fast
```

Kubernetes looks for a compatible StorageClass.

The StorageClass determines how storage should be provisioned.

Architecture:

```text
PVC
 |
 | storageClassName: fast
 v
StorageClass "fast"
 |
 v
CSI Driver
 |
 v
Storage
```

---

# 41. PVC Selector

A PVC can select a PV using labels.

PV:

```yaml
metadata:
  name: production-pv
  labels:
    environment: production
```

PVC:

```yaml
spec:
  selector:
    matchLabels:
      environment: production
```

This is more common in static provisioning scenarios.

---

# 42. PVC Binding to Specific PV

You can specify:

```yaml
volumeName: production-pv
```

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: production-pvc

spec:
  volumeName: production-pv

  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 10Gi
```

This requests binding to a specific PV.

---

# 43. PVC Finalizers

Kubernetes resources can contain finalizers.

For PVC/PV storage operations, finalizers help prevent premature deletion while required cleanup or protection operations are still pending.

Inspect:

```bash
kubectl get pvc <name> -o yaml
```

Look for:

```yaml
metadata:
  finalizers:
```

Do not remove finalizers blindly.

---

# 44. PVC Protection

Kubernetes has protection mechanisms for storage resources.

For example, a PVC that is actively being used by a Pod may be protected from immediate deletion.

This helps prevent accidental data loss.

Typical finalizers include:

```text
kubernetes.io/pvc-protection
```

---

# 45. Mount Options

Some StorageClasses support mount options.

Example:

```yaml
mountOptions:
  - noatime
```

These are backend/driver/filesystem dependent.

Do not blindly copy mount options between storage drivers.

---

# 46. Storage Capacity

Kubernetes uses storage quantities such as:

```text
Mi
Gi
Ti
```

Examples:

```yaml
storage: 512Mi
storage: 10Gi
storage: 1Ti
```

Be careful with:

```text
Gi
GB
```

They are not the same unit convention.

---

# 47. Checking PVC Information

Basic:

```bash
kubectl get pvc
```

Detailed:

```bash
kubectl describe pvc app-pvc
```

YAML:

```bash
kubectl get pvc app-pvc -o yaml
```

Specific namespace:

```bash
kubectl get pvc -n production
```

All namespaces:

```bash
kubectl get pvc -A
```

---

# 48. Find the PV Used by PVC

```bash
kubectl get pvc app-pvc
```

Example:

```text
NAME      STATUS   VOLUME      CAPACITY
app-pvc   Bound    pvc-abc123  10Gi
```

The:

```text
VOLUME
```

column shows the bound PV.

Then:

```bash
kubectl get pv pvc-abc123
```

---

# 49. Troubleshooting PVC Pending

If:

```text
STATUS = Pending
```

run:

```bash
kubectl describe pvc <pvc-name>
```

Check:

```text
Events
StorageClass
Access modes
Requested size
Provisioner
CSI driver
Topology
PV availability
```

Also:

```bash
kubectl get storageclass
kubectl get pv
```

---

# 50. Common Reason: No StorageClass

PVC:

```yaml
storageClassName: fast
```

But:

```text
fast
```

does not exist.

Check:

```bash
kubectl get storageclass
```

Fix the PVC or create/configure the correct StorageClass.

---

# 51. Common Reason: No Matching PV

For static provisioning:

```text
PVC requests:
20Gi

Available PV:
10Gi
```

The PVC cannot bind to that PV.

Create a compatible PV or change the request appropriately.

---

# 52. Common Reason: Wrong Access Mode

PVC:

```yaml
accessModes:
  - ReadWriteMany
```

But the storage backend only supports:

```text
ReadWriteOnce
```

The claim may remain pending or provisioning may fail.

Always verify storage driver capabilities.

---

# 53. Common Reason: Topology Constraints

A Pod may require:

```text
Node A
```

while the storage is available only in:

```text
Zone B
```

This can cause scheduling/provisioning problems.

`WaitForFirstConsumer` can help topology-aware provisioning when supported.

---

# 54. PVC Events

One of the most useful troubleshooting commands:

```bash
kubectl describe pvc <pvc-name>
```

Look at:

```text
Events:
```

For example:

```text
ProvisioningFailed
FailedBinding
ExternalProvisioning
```

The event message often gives the actual reason.

---

# 55. Pod Cannot Mount PVC

If PVC is already:

```text
Bound
```

but the Pod fails, check:

```bash
kubectl describe pod <pod-name>
```

Look for events such as:

```text
FailedMount
FailedAttachVolume
FailedScheduling
MountVolume.SetUp failed
```

Then inspect:

```bash
kubectl get pvc
kubectl get pv
kubectl describe pv <pv-name>
```

---

# 56. Verify PVC Is Mounted

Inside a running Pod:

```bash
kubectl exec -it <pod-name> -- df -h
```

Check:

```bash
kubectl exec -it <pod-name> -- mount
```

Check files:

```bash
kubectl exec -it <pod-name> -- ls -lah /data
```

Create test data:

```bash
kubectl exec -it <pod-name> -- sh -c 'echo hello > /data/test.txt'
```

Then:

```bash
kubectl exec -it <pod-name> -- cat /data/test.txt
```

---

# 57. Testing Persistence

Create data:

```bash
kubectl exec -it app -- sh -c 'echo persistent-data > /data/test.txt'
```

Delete the Pod:

```bash
kubectl delete pod app
```

Create/recreate the Pod.

Then:

```bash
kubectl exec -it <new-pod> -- cat /data/test.txt
```

If the PVC remains bound to the same persistent storage, the data should remain.

---

# 58. PVC Does Not Mean Backup

This is extremely important.

A PVC provides **persistent storage**, but it is not automatically a backup.

```text
Persistence ≠ Backup
```

For production data, consider:

```text
Snapshots
Backups
Replication
Disaster Recovery
Application-level backups
Off-site copies
```

---

# 59. PVC Security Considerations

Important areas:

```text
RBAC
Namespace isolation
Storage encryption
Network security
Filesystem permissions
Pod security
CSI driver permissions
Cloud IAM
```

Do not assume that a PVC automatically encrypts your data.

Encryption depends on the storage backend and configuration.

---

# 60. File Permissions

A container may receive a mounted volume but still fail to write to it.

Example:

```text
Permission denied
```

Investigate:

```bash
kubectl exec -it <pod> -- id
kubectl exec -it <pod> -- ls -ld /data
```

Depending on the workload, you may need:

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
```

Use the permissions appropriate for your application and storage driver.

---

# 61. PVC with MySQL Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mysql-pvc

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 20Gi
```

Pod:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: mysql

spec:
  containers:
    - name: mysql
      image: mysql:8

      env:
        - name: MYSQL_ROOT_PASSWORD
          value: root

      volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql

  volumes:
    - name: mysql-data
      persistentVolumeClaim:
        claimName: mysql-pvc
```

The important part is:

```text
MySQL
 |
 v
/var/lib/mysql
 |
 v
PVC
 |
 v
PV
 |
 v
Persistent Storage
```

---

# 62. PVC with WordPress

WordPress commonly needs persistent storage for:

```text
wp-content
uploads
plugins
themes
```

Example:

```yaml
volumeMounts:
  - name: wordpress-storage
    mountPath: /var/www/html/wp-content

volumes:
  - name: wordpress-storage
    persistentVolumeClaim:
      claimName: wordpress-pvc
```

---

# 63. PVC with Stateful Applications

PVCs are especially important for:

```text
Databases
Message queues
Search engines
Object stores
Distributed databases
```

However, simply adding a PVC does not automatically make a distributed application highly available.

You still need:

```text
Replication
Failover
Backups
Recovery strategy
Application-level clustering
```

---

# 64. Deployment vs StatefulSet Storage

### Deployment

```text
Deployment
 |
 +-- Pod
      |
      +-- Shared PVC
```

Suitable when application design allows shared storage or a single replica.

### StatefulSet

```text
StatefulSet
 |
 +-- Pod-0 → PVC-0
 |
 +-- Pod-1 → PVC-1
 |
 +-- Pod-2 → PVC-2
```

Better for workloads requiring stable identities and dedicated storage.

---

# 65. Important Production Design Rule

Do not blindly use:

```text
ReadWriteOnce
```

for an application with multiple replicas.

For example:

```text
Deployment replicas: 3
PVC: RWO
```

may create scheduling/storage constraints depending on the storage backend.

If multiple Pods across multiple nodes need concurrent writes, you may need:

```text
RWX
```

or an application-specific storage architecture.

---

# 66. Storage Performance

PVC capacity is not the only consideration.

Production storage may need:

```text
IOPS
Throughput
Latency
Availability
Replication
Durability
Encryption
Snapshots
Backup
Topology
```

For databases, storage latency can have a major effect on performance.

---

# 67. StorageClass for Fast Storage

Conceptually:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast

provisioner: example.com/csi

parameters:
  type: high-performance

allowVolumeExpansion: true

reclaimPolicy: Retain

volumeBindingMode: WaitForFirstConsumer
```

PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: database-pvc

spec:
  storageClassName: fast

  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 100Gi
```

---

# 68. PVC Monitoring

Monitor:

```text
Capacity
Usage
IOPS
Latency
Throughput
Errors
Mount failures
Provisioning failures
```

Kubernetes itself does not magically provide complete storage monitoring.

Production environments commonly combine Kubernetes metrics with cloud/storage monitoring.

---

# 69. Useful Commands

### List PVCs

```bash
kubectl get pvc
```

### All namespaces

```bash
kubectl get pvc -A
```

### Watch PVC

```bash
kubectl get pvc -w
```

### Describe PVC

```bash
kubectl describe pvc <name>
```

### Get YAML

```bash
kubectl get pvc <name> -o yaml
```

### List PVs

```bash
kubectl get pv
```

### Describe PV

```bash
kubectl describe pv <name>
```

### List StorageClasses

```bash
kubectl get storageclass
```

### Describe StorageClass

```bash
kubectl describe storageclass <name>
```

---

# 70. Useful Debugging Flow

When PVC is not working:

```text
1. kubectl get pvc
        |
        v
2. Is PVC Pending?
        |
       Yes
        |
        v
3. kubectl describe pvc
        |
        v
4. Check Events
        |
        v
5. Check StorageClass
        |
        v
6. Check CSI Driver
        |
        v
7. Check PV
        |
        v
8. Check topology
        |
        v
9. Check Pod events
```

---

# 71. PVC Troubleshooting Cheat Sheet

| Problem                 | Check                                     |
| ----------------------- | ----------------------------------------- |
| PVC Pending             | `kubectl describe pvc`                    |
| No PV                   | `kubectl get pv`                          |
| Wrong StorageClass      | `kubectl get sc`                          |
| Provisioning failure    | PVC events                                |
| Mount failure           | Pod events                                |
| Permission denied       | `securityContext`, filesystem permissions |
| Pod cannot schedule     | Node/storage topology                     |
| Volume expansion failed | StorageClass + CSI driver                 |
| Data disappeared        | Reclaim policy + backend                  |
| RWX unavailable         | CSI/storage backend capability            |

---

# 72. Common Mistakes

## Mistake 1 — Confusing PV and PVC

Incorrect:

```text
PVC = actual disk
```

Correct:

```text
PVC = request
PV = storage resource
```

---

## Mistake 2 — Assuming PVC Means Backup

Incorrect:

```text
PVC protects against all data loss
```

Correct:

```text
PVC provides persistence.
Backup requires a separate strategy.
```

---

## Mistake 3 — Assuming RWO Means One Pod

RWO is generally about mounting read-write from one node.

For strict single-Pod access, use:

```text
ReadWriteOncePod
```

when supported.

---

## Mistake 4 — Ignoring StorageClass

A dynamically provisioned PVC depends on the StorageClass and provisioner.

---

## Mistake 5 — Ignoring Topology

Cloud storage can be tied to:

```text
Availability Zone
Region
Node
```

This matters when scheduling Pods.

---

## Mistake 6 — Manually deleting PVs

Be extremely careful:

```bash
kubectl delete pv <pv>
```

Depending on the backend and reclaim policy, this can have serious consequences.

---

# 73. PVC Best Practices

### 1. Use Dynamic Provisioning

Prefer StorageClass + CSI drivers where appropriate.

### 2. Use Meaningful Names

```text
mysql-data
postgres-data
app-uploads
```

### 3. Choose Access Mode Carefully

Use:

```text
RWO
RWX
RWOP
```

based on actual application requirements.

### 4. Enable Expansion When Needed

Use:

```yaml
allowVolumeExpansion: true
```

where supported.

### 5. Protect Critical Data

Use:

```text
Retain
Backups
Snapshots
Replication
```

as appropriate.

### 6. Monitor Storage

Monitor capacity and performance.

### 7. Test Recovery

Do not assume your backup/snapshot process works.

Actually perform restoration tests.

---

# 74. Advanced PVC Architecture

A production architecture may look like:

```text
                  Kubernetes Cluster
                         |
                         v
                  ┌──────────────┐
                  │     Pod      │
                  └──────┬───────┘
                         |
                         v
                  ┌──────────────┐
                  │     PVC      │
                  └──────┬───────┘
                         |
                         v
                  ┌──────────────┐
                  │ StorageClass │
                  └──────┬───────┘
                         |
                         v
                  ┌──────────────┐
                  │ CSI Driver   │
                  └──────┬───────┘
                         |
             ┌───────────┴───────────┐
             |                       |
             v                       v
       Block Storage           File Storage
             |                       |
             v                       v
         Persistent              Shared
          Volume               Filesystem
```

---

# 75. PVC vs `emptyDir`

| Feature                       | PVC         | emptyDir        |
| ----------------------------- | ----------- | --------------- |
| Persistent after Pod deletion | Yes         | No              |
| Storage backend               | External/PV | Node filesystem |
| Suitable for database         | Yes         | No              |
| Survives container restart    | Yes         | Yes             |
| Survives Pod deletion         | Yes         | No              |
| Dynamic provisioning          | Yes         | No              |

---

# 76. PVC vs `hostPath`

| Feature                        | PVC               | hostPath            |
| ------------------------------ | ----------------- | ------------------- |
| Kubernetes storage abstraction | Yes               | No                  |
| Portable                       | More              | Less                |
| Production cloud use           | Common            | Usually discouraged |
| Node-specific                  | Backend dependent | Yes                 |
| Dynamic provisioning           | Yes               | No                  |

`hostPath` is often useful for local development or special node-level workloads, but it should not be treated as a general replacement for production persistent storage.

---

# 77. Important Interview Question

### What happens when a Pod using a PVC is deleted?

Usually:

```text
Pod deleted
    |
    v
PVC remains
    |
    v
PV remains
    |
    v
Data remains
```

The exact behavior depends on the storage lifecycle and whether the PVC itself is deleted.

---

# 78. What Happens When PVC Is Deleted?

It depends on the PV's reclaim policy and provisioning mechanism.

```text
PVC deleted
     |
     v
PV reclaim policy
     |
     +---- Retain → storage/PV retained
     |
     +---- Delete → provisioned storage may be deleted
```

---

# 79. Can Multiple Pods Use One PVC?

Yes, **if the storage backend and access mode support it**.

For example:

```text
Pod-1
  |
  +----+
       |
      PVC → RWX storage
       |
  +----+
  |
Pod-2
```

But:

```text
RWO
```

does not generally allow simultaneous read-write mounting from multiple nodes.

---

# 80. Can Multiple PVCs Use One PV?

Normally a PV is bound to a single PVC at a time.

Conceptually:

```text
PVC
 |
 v
PV
```

not:

```text
PVC-1 ─┐
       ├── PV
PVC-2 ─┘
```

---

# 81. Can a PVC Be Used Across Namespaces?

A PVC is namespace-scoped.

A Pod normally uses a PVC from the same namespace.

If two namespaces need the same underlying data, use an appropriate shared storage architecture rather than trying to reference one PVC cross-namespace.

---

# 82. PVC and Disaster Recovery

For production workloads, think beyond:

```text
PVC
```

A stronger architecture is:

```text
Application
    |
    v
PVC
    |
    +---- Snapshot
    |
    +---- Backup
    |
    +---- Replication
    |
    +---- Off-site copy
```

Your disaster recovery plan should answer:

```text
How much data can we lose?
How quickly must service recover?
Where are backups stored?
Can backups be restored?
What happens if the cluster is lost?
```

---

# 83. PVC Interview Questions

## Beginner

### 1. What is PVC?

A PersistentVolumeClaim is a Kubernetes request for persistent storage.

### 2. What is the difference between PV and PVC?

```text
PV = storage resource
PVC = request for storage
```

### 3. Is PVC namespaced?

Yes.

### 4. Is PV namespaced?

No.

### 5. Why use PVC?

To provide persistent storage to Pods.

---

# 84. Intermediate Interview Questions

### 6. What is StorageClass?

A StorageClass defines how storage is dynamically provisioned.

### 7. What is dynamic provisioning?

Automatic creation of storage/PVs in response to PVC requests.

### 8. What is static provisioning?

An administrator creates PVs before PVCs request them.

### 9. What is RWO?

Read-write access from a single node.

### 10. What is RWX?

Read-write access from multiple nodes, if supported by the backend.

### 11. What is RWOP?

Read-write access from a single Pod.

### 12. What happens when a Pod is deleted?

The PVC normally remains, so persistent data can remain available to a replacement Pod.

---

# 85. Advanced Interview Questions

### 13. What is CSI?

Container Storage Interface, a standard mechanism for integrating storage systems with Kubernetes.

### 14. What is `WaitForFirstConsumer`?

A volume binding mode that delays provisioning/binding until a Pod using the PVC is scheduled, helping topology-aware placement.

### 15. Can PVC storage be expanded?

Yes, if the StorageClass and CSI driver support volume expansion.

### 16. Can PVC storage be reduced?

Normal PVC resizing supports expansion, not shrinking.

### 17. Does deleting a PVC delete data?

It depends on the PV reclaim policy and storage provisioner.

### 18. What causes PVC Pending?

Possible causes include:

```text
No matching PV
Wrong StorageClass
Provisioner failure
CSI problems
Unsupported access mode
Topology constraints
Insufficient capacity
```

### 19. How do you troubleshoot PVC Pending?

```bash
kubectl describe pvc <pvc>
kubectl get pv
kubectl get storageclass
kubectl describe storageclass <sc>
```

Then inspect relevant Pod and CSI/controller events.

---

# 86. Scenario-Based Interview Question

### Scenario

You created:

```yaml
resources:
  requests:
    storage: 100Gi
```

But PVC remains:

```text
Pending
```

### Troubleshooting

First:

```bash
kubectl get pvc
```

Then:

```bash
kubectl describe pvc <name>
```

Check events.

Then:

```bash
kubectl get storageclass
```

Then:

```bash
kubectl get pv
```

If dynamically provisioned, verify:

```text
StorageClass
CSI provisioner
CSI controller
backend capacity
topology
```

---

# 87. Scenario: Pod Cannot Mount PVC

Check:

```bash
kubectl describe pod <pod>
```

Look for:

```text
FailedAttachVolume
FailedMount
MountVolume.SetUp failed
```

Then:

```bash
kubectl describe pvc <pvc>
kubectl describe pv <pv>
```

Check:

```text
Node
CSI driver
Access mode
Filesystem
Permissions
Storage backend
```

---

# 88. Scenario: Application Lost Data

Do not immediately assume Kubernetes lost the data.

Check:

```text
Was the PVC deleted?
Was the PV deleted?
What was reclaimPolicy?
Was the backend storage deleted?
Was the application writing to the mounted path?
Was the volume actually mounted?
Was the data stored somewhere else?
Were backups available?
```

Commands:

```bash
kubectl get pvc
kubectl get pv
kubectl describe pv <pv>
kubectl describe pvc <pvc>
```

---

# 89. Important Mental Model

Always remember:

```text
                  REQUEST
                    |
                    v
                  PVC
                    |
                    v
              StorageClass
                    |
                    v
               CSI Driver
                    |
                    v
                   PV
                    |
                    v
             Actual Storage
                    |
                    v
                   DATA
```

And:

```text
Pod
 |
 | volume reference
 v
PVC
 |
 v
Persistent Storage
```

---

# 90. One-Minute PVC Revision

```text
PVC = PersistentVolumeClaim

PVC requests storage.

PV provides storage.

StorageClass defines how storage is provisioned.

Dynamic provisioning automatically creates storage.

Static provisioning uses pre-created PVs.

PVC is namespaced.

PV is cluster-scoped.

Common access modes:
RWO
ROX
RWX
RWOP

RWO = ReadWriteOnce
ROX = ReadOnlyMany
RWX = ReadWriteMany
RWOP = ReadWriteOncePod

PVC states:
Pending
Bound
Lost

PV reclaim policies:
Retain
Delete

Use StatefulSet + volumeClaimTemplates
for stateful applications requiring per-Pod storage.

Use CSI drivers for modern storage integration.

PVC persistence is NOT the same as backup.

For troubleshooting:
kubectl get pvc
kubectl describe pvc
kubectl get pv
kubectl get storageclass
kubectl describe pod
```

---

# 91. Final PVC Architecture

```text
                         Kubernetes
                             |
                             v
                      ┌─────────────┐
                      │     Pod     │
                      └──────┬──────┘
                             |
                      volumeMount
                             |
                             v
                      ┌─────────────┐
                      │     PVC     │
                      └──────┬──────┘
                             |
                    Dynamic Provisioning
                             |
                             v
                    ┌─────────────────┐
                    │  StorageClass   │
                    └────────┬────────┘
                             |
                             v
                    ┌─────────────────┐
                    │   CSI Driver    │
                    └────────┬────────┘
                             |
                 ┌───────────┴───────────┐
                 |                       |
                 v                       v
          Block Storage             File Storage
                 |                       |
                 v                       v
              PV / Volume           PV / Volume
                 |                       |
                 └───────────┬───────────┘
                             |
                             v
                       Persistent Data
```

---

# 92. Final Checklist

* [ ] Understand PV vs PVC
* [ ] Understand StorageClass
* [ ] Understand static provisioning
* [ ] Understand dynamic provisioning
* [ ] Understand RWO
* [ ] Understand ROX
* [ ] Understand RWX
* [ ] Understand RWOP
* [ ] Understand PVC lifecycle
* [ ] Understand reclaim policies
* [ ] Understand volume expansion
* [ ] Understand volume snapshots
* [ ] Understand PVC cloning
* [ ] Understand CSI
* [ ] Understand StatefulSet storage
* [ ] Understand `volumeClaimTemplates`
* [ ] Understand topology-aware provisioning
* [ ] Understand PVC troubleshooting
* [ ] Understand storage security
* [ ] Understand backup vs persistence
* [ ] Practice creating PVCs
* [ ] Practice mounting PVCs into Pods
* [ ] Practice PVC troubleshooting
* [ ] Practice StatefulSet + PVC
* [ ] Practice volume expansion
* [ ] Practice snapshot/restore where supported

---

## Quick Command Reference

```bash
# PVC
kubectl get pvc
kubectl get pvc -A
kubectl describe pvc <pvc>
kubectl get pvc <pvc> -o yaml
kubectl get pvc -w

# PV
kubectl get pv
kubectl describe pv <pv>
kubectl get pv -o yaml

# StorageClass
kubectl get storageclass
kubectl get sc
kubectl describe sc <storage-class>

# Pods
kubectl get pods
kubectl describe pod <pod>
kubectl exec -it <pod> -- df -h
kubectl exec -it <pod> -- mount

# Storage debugging
kubectl get events --sort-by=.lastTimestamp
kubectl get pvc,pv,storageclass
```

---

## Key Takeaway

> **A PVC is Kubernetes' way of asking for persistent storage. The PVC connects the application to persistent storage, while the StorageClass and CSI driver determine how that storage is provisioned and managed.**

```text
Pod
 ↓
PVC
 ↓
StorageClass
 ↓
CSI Driver
 ↓
PV / Volume
 ↓
Persistent Storage
 ↓
Persistent Data
```

Mastering **PVC + PV + StorageClass + CSI + StatefulSet** gives you the foundation required to work with persistent storage in real-world Kubernetes environments.

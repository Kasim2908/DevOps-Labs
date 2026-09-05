# Kubernetes StatefulSet — Complete Notes

## 📌 Overview

A **StatefulSet** is a Kubernetes workload controller designed for **stateful applications** that require:

* Stable Pod identity
* Stable network identity
* Persistent storage
* Ordered deployment
* Ordered scaling
* Ordered termination

Common examples:

* MySQL
* PostgreSQL
* MongoDB
* Cassandra
* Kafka
* ZooKeeper
* Elasticsearch

---

# 1. StatefulSet vs Deployment

| Feature                    | Deployment       | StatefulSet   |
| -------------------------- | ---------------- | ------------- |
| Pod identity               | Temporary        | Stable        |
| Pod names                  | Random/generated | Predictable   |
| Example                    | `web-7d9f8`      | `mysql-0`     |
| Persistent identity        | ❌                | ✅             |
| Stable DNS                 | ❌                | ✅             |
| Persistent storage per Pod | Not automatic    | ✅             |
| Ordered creation           | ❌                | ✅             |
| Ordered deletion           | ❌                | ✅             |
| Best for                   | Stateless apps   | Stateful apps |

### Deployment

```text
Deployment
    |
    ├── Pod-A
    ├── Pod-B
    └── Pod-C
```

Pods are generally interchangeable.

### StatefulSet

```text
StatefulSet
    |
    ├── mysql-0
    ├── mysql-1
    └── mysql-2
```

Each Pod has a stable identity.

---

# 2. Basic StatefulSet Architecture

```text
                 StatefulSet
                      |
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       mysql-0     mysql-1     mysql-2
          |           |           |
        PVC-0       PVC-1       PVC-2
          |           |           |
         PV-0        PV-1        PV-2
```

A StatefulSet manages the Pods while each Pod can have its own persistent storage.

---

# 3. Stable Pod Identity

If a StatefulSet has:

```yaml
replicas: 3
```

Kubernetes creates:

```text
mysql-0
mysql-1
mysql-2
```

The ordinal number is important:

```text
mysql-0 → ordinal 0
mysql-1 → ordinal 1
mysql-2 → ordinal 2
```

If `mysql-1` crashes:

```text
mysql-1 ❌
```

Kubernetes recreates:

```text
mysql-1 ✅
```

The identity remains stable.

---

# 4. StatefulSet YAML

Basic example:

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: mysql

spec:
  serviceName: mysql
  replicas: 3

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:
      - name: mysql
        image: mysql:8.0

        ports:
        - containerPort: 3306

        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql

  volumeClaimTemplates:
  - metadata:
      name: mysql-data

    spec:
      accessModes:
      - ReadWriteOnce

      resources:
        requests:
          storage: 10Gi
```

---

# 5. Important StatefulSet Fields

## `serviceName`

```yaml
serviceName: mysql
```

Associates the StatefulSet with a Service, usually a Headless Service.

---

## `replicas`

```yaml
replicas: 3
```

Creates:

```text
mysql-0
mysql-1
mysql-2
```

---

## `selector`

```yaml
selector:
  matchLabels:
    app: mysql
```

Identifies the Pods managed by the StatefulSet.

The selector must match the labels in the Pod template.

---

## `template`

Defines the Pod specification.

```yaml
template:
  metadata:
    labels:
      app: mysql
```

---

## `volumeClaimTemplates`

Creates a separate PVC for each StatefulSet Pod.

```yaml
volumeClaimTemplates:
- metadata:
    name: mysql-data
```

For three replicas:

```text
mysql-data-mysql-0
mysql-data-mysql-1
mysql-data-mysql-2
```

---

# 6. StatefulSet and Persistent Storage

A StatefulSet is commonly used with:

```text
PVC → PV → Storage
```

Architecture:

```text
mysql-0
   |
   ↓
PVC mysql-data-mysql-0
   |
   ↓
PV
   |
   ↓
Persistent Storage
```

Similarly:

```text
mysql-1 → PVC → PV
mysql-2 → PVC → PV
```

Each Pod gets its own storage.

---

# 7. PVC vs PV

## PersistentVolume — PV

A PV represents storage available to the Kubernetes cluster.

Examples of storage backends:

* AWS EBS
* Azure Disk
* Google Persistent Disk
* NFS
* Local storage

---

## PersistentVolumeClaim — PVC

A PVC requests storage.

Example:

```yaml
resources:
  requests:
    storage: 10Gi
```

Simple mental model:

```text
PVC = "I need 10 GB storage."

PV = "Here is 10 GB storage."
```

---

# 8. VolumeClaimTemplates

Example:

```yaml
volumeClaimTemplates:
- metadata:
    name: mysql-data

  spec:
    accessModes:
    - ReadWriteOnce

    resources:
      requests:
        storage: 10Gi
```

If:

```yaml
replicas: 3
```

Kubernetes creates:

```text
mysql-0
   |
   └── mysql-data-mysql-0

mysql-1
   |
   └── mysql-data-mysql-1

mysql-2
   |
   └── mysql-data-mysql-2
```

This is one of the most important StatefulSet features.

---

# 9. Why Separate PVCs?

Instead of:

```text
mysql-0 ─┐
mysql-1 ─┼──→ One Storage
mysql-2 ─┘
```

StatefulSet normally provides:

```text
mysql-0 → PVC-0 → PV-0

mysql-1 → PVC-1 → PV-1

mysql-2 → PVC-2 → PV-2
```

This gives every instance its own persistent data.

---

# 10. Headless Service

StatefulSets commonly use a **Headless Service**.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql

spec:
  clusterIP: None

  selector:
    app: mysql

  ports:
  - port: 3306
    targetPort: 3306
```

The important configuration is:

```yaml
clusterIP: None
```

---

# 11. Why Headless Service?

A normal Service provides a virtual IP:

```text
Client
   |
   ↓
Service IP
   |
   ├── Pod-1
   ├── Pod-2
   └── Pod-3
```

This is useful for stateless applications.

Stateful applications may need to communicate with a specific instance.

For example:

```text
mysql-0
mysql-1
mysql-2
```

A Headless Service enables DNS-based discovery of individual Pods.

---

# 12. Stable DNS

StatefulSet + Headless Service provides predictable DNS.

General pattern:

```text
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

Example:

```text
mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
mysql-2.mysql.default.svc.cluster.local
```

Short names can also be used within the same namespace:

```text
mysql-0.mysql
mysql-1.mysql
mysql-2.mysql
```

---

# 13. Complete Network Architecture

```text
                  Headless Service
                         |
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       mysql-0        mysql-1        mysql-2
          |              |              |
        PVC-0          PVC-1          PVC-2
          |              |              |
        PV-0           PV-1           PV-2
```

The Headless Service handles discovery.

PVC/PV handles persistence.

StatefulSet handles Pod identity and lifecycle.

---

# 14. StatefulSet Scaling

Suppose:

```yaml
replicas: 3
```

Pods:

```text
mysql-0
mysql-1
mysql-2
```

Scale to five:

```bash
kubectl scale statefulset mysql --replicas=5
```

Kubernetes creates:

```text
mysql-3
mysql-4
```

Final state:

```text
mysql-0
mysql-1
mysql-2
mysql-3
mysql-4
```

---

# 15. Scaling Down

Suppose:

```text
mysql-0
mysql-1
mysql-2
mysql-3
mysql-4
```

Scale down:

```bash
kubectl scale sts mysql --replicas=3
```

Pods are removed from the highest ordinal:

```text
mysql-4
mysql-3
```

Remaining:

```text
mysql-0
mysql-1
mysql-2
```

---

# 16. PVC Behavior During Scaling

Scaling down removes the corresponding Pods, but their PVCs are generally retained.

Example:

```text
mysql-2
   |
PVC mysql-data-mysql-2
```

After the Pod is removed:

```text
mysql-data-mysql-2
```

can remain.

Check:

```bash
kubectl get pvc
```

This helps protect persistent data.

---

# 17. Pod Management Policy

StatefulSet supports:

```yaml
podManagementPolicy: OrderedReady
```

This is the default.

Pods are created in order:

```text
mysql-0
   ↓
mysql-1
   ↓
mysql-2
```

Kubernetes waits for the previous Pod to become ready before proceeding.

---

# 18. Parallel Pod Management

Another option:

```yaml
podManagementPolicy: Parallel
```

Pods can be created or terminated without waiting for ordinal ordering.

Conceptually:

```text
mysql-0 ─┐
mysql-1 ─┼──→ Parallel
mysql-2 ─┘
```

The Pods still retain their stable identities.

---

# 19. Ordered Termination

With the default ordered behavior, StatefulSet termination proceeds from the highest ordinal.

Example:

```text
mysql-2
   ↓
mysql-1
   ↓
mysql-0
```

This can be useful for clustered applications that need controlled shutdown.

---

# 20. StatefulSet Update Strategy

StatefulSet supports different update strategies.

Common strategy:

```yaml
updateStrategy:
  type: RollingUpdate
```

If the image changes:

```yaml
image: mysql:8.0
```

to:

```yaml
image: mysql:8.4
```

Kubernetes can progressively update the Pods.

Typically the highest ordinal is updated first:

```text
mysql-2
   ↓
mysql-1
   ↓
mysql-0
```

---

# 21. OnDelete Strategy

Another strategy:

```yaml
updateStrategy:
  type: OnDelete
```

With `OnDelete`, changing the StatefulSet template does not automatically replace existing Pods.

You manually delete a Pod:

```bash
kubectl delete pod mysql-2
```

Kubernetes then recreates it using the updated configuration.

---

# 22. Partitioned Rolling Update

StatefulSet supports partitions.

Example:

```yaml
updateStrategy:
  type: RollingUpdate
  rollingUpdate:
    partition: 2
```

For:

```text
mysql-0
mysql-1
mysql-2
mysql-3
mysql-4
```

Pods below the partition:

```text
mysql-0
mysql-1
```

remain on the old version.

Pods at or above the partition:

```text
mysql-2
mysql-3
mysql-4
```

can be updated.

This can be useful for controlled or canary-style updates.

---

# 23. StatefulSet + MySQL

A common learning architecture is:

```text
                    MySQL Cluster
                         |
              ┌──────────┴──────────┐
              ↓                     ↓
           mysql-0                mysql-1
           PRIMARY                REPLICA
              |                     |
             PVC                   PVC
              |                     |
             PV                    PV
              |
        Binary Logging
              |
              └──────────────→ Replication
```

However:

> A StatefulSet by itself does NOT create MySQL replication.

It only provides the Kubernetes infrastructure.

---

# 24. Kubernetes vs Database Responsibilities

```text
                  Kubernetes
                      |
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
    Pod Identity   Storage       Networking
        |             |             |
   StatefulSet      PVC/PV    Headless Service


                    MySQL
                      |
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
   Replication       GTID       Failover
        |             |             |
    Data Sync      Tracking    Primary Election
```

Kubernetes manages:

* Pods
* Storage attachments
* Networking
* Lifecycle
* Scheduling

MySQL manages:

* Data
* Replication
* Consistency
* Primary/replica roles
* Database-level failover

---

# 25. MySQL Primary–Replica Architecture

A simplified replication architecture:

```text
                    Application
                         |
                         ↓
                     MySQL
                         |
                       mysql-0
                       PRIMARY
                          |
                    Binary Log
                          |
                          ↓
                       mysql-1
                       REPLICA
```

Writes go to the primary.

The primary records changes in the binary log.

The replica consumes those changes and applies them.

---

# 26. MySQL Server IDs

Each replication member needs a unique server ID.

Primary:

```ini
[mysqld]
server-id=1
log-bin=mysql-bin
binlog_format=ROW
```

Replica:

```ini
[mysqld]
server-id=2
relay-log=mysql-relay-bin
```

The IDs must be unique.

---

# 27. Replication User

A dedicated replication account should be used rather than the root account.

Conceptually:

```sql
CREATE USER 'repl'@'%' IDENTIFIED BY 'replpass';

GRANT REPLICATION REPLICA
ON *.*
TO 'repl'@'%';
```

Exact replication syntax can vary with the MySQL version.

---

# 28. Replica Connection

The replica needs information about the primary:

```text
Primary hostname
Primary port
Replication username
Replication password
Replication coordinates or GTID
```

The stable hostname can be:

```text
mysql-0.mysql
```

instead of a Pod IP.

This is an important reason StatefulSet is useful for distributed systems.

---

# 29. GTID

GTID means:

> Global Transaction Identifier

It gives transactions unique identifiers.

Conceptually:

```text
Transaction
     ↓
   GTID
     ↓
Binary Log
     ↓
 Replica
```

GTID-based replication can simplify replication management compared with manually tracking binary-log file positions.

---

# 30. Initialization Logic

A stateful database may need different configuration depending on its ordinal.

Conceptually:

```bash
if hostname == mysql-0
then
    start as PRIMARY
else
    start as REPLICA
fi
```

For example:

```text
mysql-0
   ↓
PRIMARY configuration

mysql-1
   ↓
REPLICA configuration
```

This initialization logic normally requires scripts, ConfigMaps, images, or an operator.

---

# 31. ConfigMap

A ConfigMap can store non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: mysql-config

data:
  my.cnf: |
    [mysqld]
    binlog_format=ROW
```

Use ConfigMaps for configuration that is not secret.

Do not store passwords in ConfigMaps.

---

# 32. Secret

Sensitive information should be stored using Kubernetes Secrets.

Example:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: mysql-secret

type: Opaque

stringData:
  root-password: rootpass
  replication-password: replpass
```

Use:

```yaml
env:
- name: MYSQL_ROOT_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-secret
      key: root-password
```

---

# 33. Readiness Probe

A readiness probe determines whether a Pod should receive traffic.

Example:

```yaml
readinessProbe:
  exec:
    command:
    - mysqladmin
    - ping

  initialDelaySeconds: 10
  periodSeconds: 10
```

Meaning:

```text
Is MySQL ready to accept requests?
```

---

# 34. Liveness Probe

A liveness probe determines whether the container should be restarted.

Example:

```yaml
livenessProbe:
  exec:
    command:
    - mysqladmin
    - ping

  initialDelaySeconds: 30
  periodSeconds: 10
```

Mental model:

```text
Liveness
   ↓
Should Kubernetes restart me?

Readiness
   ↓
Should Kubernetes send traffic to me?
```

---

# 35. Complete StatefulSet + Service Example

## Headless Service

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql

spec:
  clusterIP: None

  selector:
    app: mysql

  ports:
  - port: 3306
    targetPort: 3306
```

## StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: mysql

spec:
  serviceName: mysql
  replicas: 3

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:
      - name: mysql
        image: mysql:8.0

        ports:
        - containerPort: 3306

        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password

        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql

  volumeClaimTemplates:
  - metadata:
      name: mysql-data

    spec:
      accessModes:
      - ReadWriteOnce

      resources:
        requests:
          storage: 10Gi
```

---

# 36. Deploy the StatefulSet

Apply the Secret:

```bash
kubectl apply -f mysql-secret.yaml
```

Apply the Service:

```bash
kubectl apply -f mysql-service.yaml
```

Apply the StatefulSet:

```bash
kubectl apply -f mysql-statefulset.yaml
```

---

# 37. Verify StatefulSet

```bash
kubectl get statefulsets
```

or:

```bash
kubectl get sts
```

Example:

```text
NAME    READY
mysql   3/3
```

---

# 38. Verify Pods

```bash
kubectl get pods
```

Expected:

```text
NAME      READY   STATUS
mysql-0   1/1     Running
mysql-1   1/1     Running
mysql-2   1/1     Running
```

---

# 39. Verify PVCs

```bash
kubectl get pvc
```

Example:

```text
NAME                   STATUS   CAPACITY
mysql-data-mysql-0     Bound    10Gi
mysql-data-mysql-1     Bound    10Gi
mysql-data-mysql-2     Bound    10Gi
```

---

# 40. Verify PVs

```bash
kubectl get pv
```

Check that the PVCs are correctly bound to PersistentVolumes.

---

# 41. Verify Service

```bash
kubectl get svc
```

For the Headless Service:

```text
CLUSTER-IP
None
```

---

# 42. Describe StatefulSet

```bash
kubectl describe statefulset mysql
```

This helps troubleshoot:

* Scheduling problems
* Container errors
* PVC issues
* Events
* Configuration problems

---

# 43. Test Pod Identity

```bash
kubectl exec -it mysql-0 -- hostname
```

Expected:

```text
mysql-0
```

Test:

```bash
kubectl exec -it mysql-1 -- hostname
```

Expected:

```text
mysql-1
```

---

# 44. Test DNS

From another Pod:

```bash
nslookup mysql-0.mysql
```

Also:

```bash
nslookup mysql-1.mysql
```

and:

```bash
nslookup mysql-2.mysql
```

This demonstrates stable network discovery.

---

# 45. Test Persistent Storage

Create some data inside the database.

Then delete the Pod:

```bash
kubectl delete pod mysql-0
```

Check:

```bash
kubectl get pods
```

Kubernetes recreates:

```text
mysql-0
```

The corresponding PVC remains:

```text
mysql-data-mysql-0
```

The persistent data can be mounted again.

---

# 46. Failure Scenario

Suppose:

```text
mysql-1
   ↓
Crash
```

StatefulSet detects:

```text
Desired replicas = 3
Current replicas = 2
```

It recreates:

```text
mysql-1
```

The Pod gets:

```text
Same ordinal identity
+
Same PVC
+
Persistent storage
```

---

# 47. Important: StatefulSet Does Not Automatically Provide Database Failover

This is a common misconception.

Having:

```text
mysql-0
mysql-1
mysql-2
```

does not automatically mean:

```text
mysql-0 = Primary
mysql-1 = Replica
mysql-2 = Replica
```

Nor does Kubernetes automatically perform database-level primary election.

You need:

* Database replication
* Appropriate configuration
* Health checks
* Failover mechanism
* Potentially a database operator

---

# 48. Production Architecture

A production database architecture can look like:

```text
                       Application
                            |
                       DB Endpoint
                            |
                 ┌──────────┴──────────┐
                 ↓                     ↓
              PRIMARY                REPLICA
                 |                     |
                PVC                   PVC
                 |                     |
                PV                    PV
                 |
          Database Replication
                 |
          ┌──────┴──────┐
          ↓             ↓
       Backups       Monitoring
```

Production environments may additionally use:

* Database Operators
* Automated backups
* Monitoring
* Alerting
* Disaster recovery
* Encryption
* Network policies
* Resource limits
* Pod disruption budgets
* Anti-affinity
* Storage classes
* Multi-zone deployment

---

# 49. Important Commands Cheat Sheet

### Create

```bash
kubectl apply -f statefulset.yaml
```

### List StatefulSets

```bash
kubectl get sts
```

### List Pods

```bash
kubectl get pods
```

### List PVCs

```bash
kubectl get pvc
```

### List PVs

```bash
kubectl get pv
```

### List Services

```bash
kubectl get svc
```

### Describe StatefulSet

```bash
kubectl describe sts mysql
```

### Scale

```bash
kubectl scale sts mysql --replicas=5
```

### Delete Pod

```bash
kubectl delete pod mysql-1
```

### Check Logs

```bash
kubectl logs mysql-0
```

### Enter Pod

```bash
kubectl exec -it mysql-0 -- bash
```

### Check StatefulSet YAML

```bash
kubectl get sts mysql -o yaml
```

---

# 50. StatefulSet Troubleshooting

## Pods Not Starting

Check:

```bash
kubectl get pods
kubectl describe pod mysql-0
```

Look for:

* Image errors
* Resource problems
* Scheduling problems
* Configuration errors

---

## PVC Pending

Check:

```bash
kubectl get pvc
kubectl describe pvc mysql-data-mysql-0
```

Possible causes:

* No matching PV
* StorageClass problem
* CSI driver problem
* Insufficient storage

---

## DNS Not Working

Check:

```bash
kubectl get svc
```

Confirm:

```text
clusterIP: None
```

Then test:

```bash
nslookup mysql-0.mysql
```

---

## Pod Stuck During Creation

Check:

```bash
kubectl describe sts mysql
kubectl describe pod mysql-0
```

Because `OrderedReady` is the default, a problem with an earlier Pod can prevent later Pods from being created.

---

# 51. Important Interview Questions

## Q1. What is StatefulSet?

**Answer:**

> StatefulSet is a Kubernetes workload controller used for stateful applications that require stable Pod identities, persistent storage, stable network identities, and ordered deployment or scaling behavior.

---

## Q2. StatefulSet vs Deployment?

**Answer:**

> Deployment is primarily designed for stateless workloads where Pods are interchangeable. StatefulSet is designed for workloads where each Pod requires a stable identity and potentially persistent storage.

---

## Q3. Why does StatefulSet use predictable Pod names?

Because stateful applications may need to identify individual instances.

Example:

```text
mysql-0
mysql-1
mysql-2
```

---

## Q4. What is a Headless Service?

A Service with:

```yaml
clusterIP: None
```

It does not provide a normal virtual ClusterIP and enables DNS-based discovery of individual Pods.

---

## Q5. Why use Headless Service with StatefulSet?

> StatefulSet provides stable Pod identity, while the Headless Service provides stable DNS-based network discovery for those Pods.

---

## Q6. What is `volumeClaimTemplates`?

It is a StatefulSet feature that automatically creates a separate PVC for each Pod.

---

## Q7. What happens when a StatefulSet Pod is deleted?

Kubernetes recreates the Pod using the same ordinal identity.

Example:

```text
mysql-1
   ↓
deleted
   ↓
recreated as mysql-1
```

---

## Q8. Does StatefulSet automatically replicate database data?

**No.**

StatefulSet provides infrastructure features.

Database replication must be configured separately.

---

## Q9. Does StatefulSet automatically provide database failover?

**No.**

Database-level failover requires additional database configuration or an appropriate operator/system.

---

## Q10. Why shouldn't we use Pod IPs for database communication?

Pod IPs can change when Pods are recreated.

Stable DNS names provided by StatefulSet + Headless Service are more appropriate.

Example:

```text
mysql-0.mysql
```

---

# 52. Key Differences to Remember

```text
Deployment
    ↓
Stateless
    ↓
Interchangeable Pods
```

```text
StatefulSet
    ↓
Stateful
    ↓
Stable Identity
    ↓
Stable DNS
    ↓
Persistent Storage
```

---

# 53. Final Mental Model

Remember these three components:

```text
             Stateful Application
                      |
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
  StatefulSet   Headless Service   PVC/PV
        |             |             |
        ↓             ↓             ↓
  Pod Identity     DNS/Network     Storage
```

### Simple Rule

```text
StatefulSet
    =
Stable Identity

Headless Service
    =
Stable Network Identity

PVC/PV
    =
Persistent Storage
```

Together:

```text
StatefulSet
     +
Headless Service
     +
PVC/PV
     ↓
Reliable foundation for Stateful Applications
```

---

# 54. One-Line Interview Summary

> **StatefulSet manages stateful workloads by providing stable Pod identities and ordered lifecycle behavior, while Headless Services provide stable DNS discovery and PVC/PV provide persistent storage for each Pod.**

---

# 55. Recommended Learning Progression

After mastering these notes, practice in this order:

```text
1. Create a basic StatefulSet
        ↓
2. Create a Headless Service
        ↓
3. Add volumeClaimTemplates
        ↓
4. Verify PVC/PV creation
        ↓
5. Test stable Pod names
        ↓
6. Test stable DNS
        ↓
7. Delete and recreate Pods
        ↓
8. Scale StatefulSet
        ↓
9. Test RollingUpdate
        ↓
10. Test OnDelete
        ↓
11. Configure MySQL
        ↓
12. Configure MySQL replication
        ↓
13. Add readiness/liveness probes
        ↓
14. Add monitoring and backups
        ↓
15. Deploy on AWS Kubernetes/EKS
```

---

## 🎯 Key Takeaways

* **StatefulSet** is designed for stateful workloads.
* StatefulSet Pods have **stable names and ordinals**.
* StatefulSet commonly works with a **Headless Service**.
* Headless Services provide **stable DNS discovery**.
* `volumeClaimTemplates` creates **individual PVCs** for StatefulSet Pods.
* PVCs request storage from PVs.
* StatefulSet supports **ordered creation, scaling, and termination**.
* `RollingUpdate` supports controlled StatefulSet updates.
* `OnDelete` gives manual control over Pod replacement.
* StatefulSet does **not automatically implement database replication**.
* StatefulSet does **not automatically provide database failover**.
* Database systems are responsible for their own **replication and consistency mechanisms**.
* For production databases, consider **operators, backups, monitoring, security, and disaster recovery**.

**Core formula:**

```text
StatefulSet
     +
Headless Service
     +
PVC/PV
     +
Database Replication
     ↓
Production-oriented Stateful Application
```

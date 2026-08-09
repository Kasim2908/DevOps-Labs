# ReplicaSets – Cheat Sheet

A quick-reference guide for **Kubernetes ReplicaSets**, including concepts, YAML, commands, troubleshooting, and interview essentials.

---

# 1. What is a ReplicaSet?

A **ReplicaSet** ensures that a specified number of identical Pods are running at any given time.

```text
ReplicaSet
    |
    ├── Pod
    ├── Pod
    └── Pod
```

If a managed Pod is deleted or disappears, the ReplicaSet creates a replacement.

---

# 2. Core Responsibility

```text
Desired State
      ↓
ReplicaSet Controller
      ↓
Compare Current State
      ↓
Create/Delete Pods
      ↓
Desired State Achieved
```

Example:

```text
Desired replicas = 3
Current replicas = 2
        ↓
Create 1 Pod
        ↓
Current replicas = 3
```

---

# 3. ReplicaSet YAML

Basic example:

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx-rs

spec:
  replicas: 3

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
          image: nginx:1.25
          ports:
            - containerPort: 80
```

---

# 4. Important YAML Fields

| Field                      | Purpose                       |
| -------------------------- | ----------------------------- |
| `apiVersion`               | Kubernetes API version        |
| `kind`                     | Resource type                 |
| `metadata.name`            | ReplicaSet name               |
| `spec.replicas`            | Desired number of Pods        |
| `spec.selector`            | Identifies managed Pods       |
| `spec.template`            | Defines the Pod specification |
| `template.metadata.labels` | Labels assigned to Pods       |
| `containers`               | Container configuration       |

---

# 5. Labels and Selectors

ReplicaSets use selectors to identify Pods.

```yaml
selector:
  matchLabels:
    app: nginx
```

Pod template:

```yaml
template:
  metadata:
    labels:
      app: nginx
```

Relationship:

```text
ReplicaSet Selector
       |
       | app=nginx
       ↓
Pod Label
       |
       ↓
Managed Pod
```

### Important

The ReplicaSet selector and Pod template labels must be compatible.

---

# 6. Create a ReplicaSet

Using YAML:

```bash
kubectl apply -f replicaset.yaml
```

---

# 7. List ReplicaSets

```bash
kubectl get rs
```

Short form:

```bash
kubectl get replicasets
```

---

# 8. Get Detailed Information

```bash
kubectl describe rs nginx-rs
```

---

# 9. Get ReplicaSet YAML

```bash
kubectl get rs nginx-rs -o yaml
```

---

# 10. Get ReplicaSet as JSON

```bash
kubectl get rs nginx-rs -o json
```

---

# 11. Check ReplicaSet Status

```bash
kubectl get rs
```

Example:

```text
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   3         3         3       5m
```

### Meaning

```text
DESIRED
→ Number of Pods Kubernetes should maintain

CURRENT
→ Number of Pods currently associated with the ReplicaSet

READY
→ Number of Pods currently Ready
```

---

# 12. List Pods Managed by a ReplicaSet

If the ReplicaSet selector is:

```yaml
matchLabels:
  app: nginx
```

Run:

```bash
kubectl get pods -l app=nginx
```

---

# 13. Show Pod Labels

```bash
kubectl get pods --show-labels
```

---

# 14. Get Pod Ownership

```bash
kubectl get pod <pod-name> -o yaml
```

Look for:

```yaml
ownerReferences:
```

Example:

```yaml
ownerReferences:
  - apiVersion: apps/v1
    kind: ReplicaSet
    name: nginx-rs
```

Ownership:

```text
ReplicaSet
    ↓
Pod
```

---

# 15. Scale a ReplicaSet

Scale up:

```bash
kubectl scale rs nginx-rs --replicas=5
```

Scale down:

```bash
kubectl scale rs nginx-rs --replicas=2
```

Check:

```bash
kubectl get rs
kubectl get pods
```

---

# 16. Scale Using YAML

Change:

```yaml
spec:
  replicas: 3
```

to:

```yaml
spec:
  replicas: 5
```

Apply:

```bash
kubectl apply -f replicaset.yaml
```

---

# 17. Test Self-Healing

List Pods:

```bash
kubectl get pods
```

Delete one:

```bash
kubectl delete pod <pod-name>
```

Watch:

```bash
kubectl get pods -w
```

The ReplicaSet creates a replacement Pod.

```text
3 Pods
  ↓
Delete 1
  ↓
2 Pods
  ↓
ReplicaSet detects difference
  ↓
Creates replacement
  ↓
3 Pods
```

---

# 18. Delete a ReplicaSet

```bash
kubectl delete rs nginx-rs
```

By default, the ReplicaSet's dependent Pods are also deleted.

---

# 19. Delete ReplicaSet but Keep Pods

Use orphan deletion:

```bash
kubectl delete rs nginx-rs --cascade=orphan
```

The ReplicaSet is deleted while its dependent Pods can remain orphaned.

---

# 20. ReplicaSet Events

Check events associated with a ReplicaSet:

```bash
kubectl describe rs nginx-rs
```

Check cluster events:

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

# 21. ReplicaSet Troubleshooting

## Step 1: Check ReplicaSet

```bash
kubectl get rs
```

## Step 2: Describe ReplicaSet

```bash
kubectl describe rs <name>
```

## Step 3: Check Pods

```bash
kubectl get pods
```

## Step 4: Describe Pod

```bash
kubectl describe pod <pod-name>
```

## Step 5: Check logs

```bash
kubectl logs <pod-name>
```

## Step 6: Check events

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

# 22. Common Pod Problems

| Status              | Possible Cause                    |
| ------------------- | --------------------------------- |
| `Pending`           | Scheduling/resource problem       |
| `ContainerCreating` | Container setup still in progress |
| `Running`           | Pod is running                    |
| `CrashLoopBackOff`  | Container repeatedly crashes      |
| `ImagePullBackOff`  | Image cannot be pulled            |
| `ErrImagePull`      | Image pull failed                 |
| `Completed`         | Container finished successfully   |
| `Failed`            | Container/Pod failed              |

---

# 23. ReplicaSet vs Pod

| ReplicaSet              | Pod                        |
| ----------------------- | -------------------------- |
| Controller              | Workload unit              |
| Manages Pods            | Runs containers            |
| Maintains replica count | Does not maintain replicas |
| Provides self-healing   | Can be recreated           |
| Uses selectors          | Has labels                 |

---

# 24. ReplicaSet vs Deployment

| ReplicaSet                      | Deployment                    |
| ------------------------------- | ----------------------------- |
| Manages Pods                    | Manages ReplicaSets           |
| Maintains replica count         | Manages application releases  |
| No rolling update functionality | Supports rolling updates      |
| No rollback mechanism           | Supports rollback             |
| Basic controller                | Higher-level controller       |
| Usually managed by Deployment   | Common production abstraction |

Relationship:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

---

# 25. ReplicaSet vs ReplicationController

| ReplicaSet                   | ReplicationController    |
| ---------------------------- | ------------------------ |
| Modern controller            | Legacy controller        |
| `apps/v1`                    | Older API                |
| Supports set-based selectors | Equality-based selectors |
| Used by Deployments          | Rarely used today        |

---

# 26. Does a ReplicaSet Perform Rolling Updates?

**No.**

A ReplicaSet maintains the desired number of Pods.

Rolling updates are handled by **Deployments**.

```text
Deployment
    ↓
Rolling Update
    ↓
New ReplicaSet
    ↓
New Pods
```

---

# 27. Does a ReplicaSet Support Rollback?

**No.**

Rollback is a Deployment feature.

```bash
kubectl rollout undo deployment/<name>
```

---

# 28. Does Changing Replica Count Create a New ReplicaSet?

**No.**

Changing:

```yaml
spec:
  replicas: 3
```

to:

```yaml
spec:
  replicas: 5
```

only changes the desired Pod count.

It does not change the Pod template.

---

# 29. What Changes Create a New ReplicaSet Under a Deployment?

Changes to the Deployment's **Pod template** create a new ReplicaSet.

Examples:

```yaml
image: nginx:1.25
```

changed to:

```yaml
image: nginx:1.26
```

Other examples:

* Environment variables
* Container command
* Container arguments
* Resource configuration
* Volume configuration
* Pod labels
* Pod annotations

Concept:

```text
Pod Template Changes
       ↓
New Deployment Revision
       ↓
New ReplicaSet
```

---

# 30. ReplicaSet Reconciliation

The controller continuously compares:

```text
Desired State
      vs
Current State
```

Example:

```text
Desired = 5
Current = 3
```

Action:

```text
Create 2 Pods
```

Another example:

```text
Desired = 2
Current = 5
```

Action:

```text
Remove 3 Pods
```

---

# 31. High Availability

Multiple replicas can improve availability:

```text
ReplicaSet
    |
    ├── Pod → Node 1
    ├── Pod → Node 2
    └── Pod → Node 3
```

However, simply setting replicas does not guarantee that Pods are distributed across different nodes.

For stronger workload distribution, consider:

* Pod anti-affinity
* Topology spread constraints
* Multiple worker nodes

---

# 32. ReplicaSet Across Nodes

A ReplicaSet does not directly decide which Node runs a Pod.

The Kubernetes scheduler is responsible for scheduling Pods onto suitable Nodes.

```text
ReplicaSet
    ↓
Creates Pod
    ↓
Scheduler
    ↓
Suitable Node
```

---

# 33. Important Commands

```bash
# Create/update
kubectl apply -f replicaset.yaml

# List
kubectl get rs

# Detailed information
kubectl describe rs <name>

# YAML
kubectl get rs <name> -o yaml

# JSON
kubectl get rs <name> -o json

# Scale
kubectl scale rs <name> --replicas=5

# Pods
kubectl get pods

# Pods by label
kubectl get pods -l app=nginx

# Show labels
kubectl get pods --show-labels

# Watch Pods
kubectl get pods -w

# Delete Pod
kubectl delete pod <pod-name>

# Events
kubectl get events --sort-by=.lastTimestamp

# Delete ReplicaSet
kubectl delete rs <name>

# Delete RS and orphan Pods
kubectl delete rs <name> --cascade=orphan
```

---

# 34. Quick Troubleshooting Flow

```text
ReplicaSet Problem
       ↓
kubectl get rs
       ↓
kubectl describe rs <name>
       ↓
kubectl get pods
       ↓
kubectl describe pod <name>
       ↓
kubectl logs <name>
       ↓
kubectl get events
```

Check:

* Replica count
* Labels
* Selectors
* Pod status
* Container image
* Resources
* Scheduling
* Node health
* Events
* Application logs

---

# 35. ReplicaSet Interview Quick Answers

### What is a ReplicaSet?

> A ReplicaSet ensures that a specified number of matching Pods are running.

### What happens when a Pod is deleted?

> The ReplicaSet detects that the actual number of Pods is below the desired count and creates a replacement.

### What does a ReplicaSet use to identify Pods?

> Label selectors.

### Does ReplicaSet perform rolling updates?

> No. Deployments provide rolling updates.

### Does ReplicaSet support rollback?

> No. Deployments provide rollback functionality.

### Can a ReplicaSet scale Pods?

> Yes, by changing the desired replica count.

### Can a ReplicaSet manage existing Pods?

> It can adopt eligible existing Pods that match its selector and ownership rules.

### What is the modern replacement for ReplicationController?

> ReplicaSet.

### What manages ReplicaSets in a typical application?

> Deployment.

### What manages Pods?

> ReplicaSet.

---

# 36. Most Important Relationship

Memorize:

```text
Deployment
     |
     | manages
     ↓
ReplicaSet
     |
     | manages
     ↓
Pod
     |
     | runs
     ↓
Container
```

---

# 37. One-Minute Revision

```text
ReplicaSet
──────────

Purpose:
Maintain desired number of Pods.

Key field:
spec.replicas

Pod identification:
selector + labels

Self-healing:
Yes

Scaling:
Yes

Rolling update:
No

Rollback:
No

Usually managed by:
Deployment

Modern API:
apps/v1
```

---

# 38. Final Cheat Sheet

```text
┌─────────────────────────────────────┐
│          ReplicaSet                 │
├─────────────────────────────────────┤
│ Maintains Pod replicas              │
│ Uses label selectors                │
│ Provides self-healing               │
│ Supports scaling                    │
│ Does not perform rolling updates    │
│ Does not provide rollback           │
│ Usually managed by Deployment       │
└─────────────────────────────────────┘
```

### Core Commands

```bash
kubectl get rs
kubectl describe rs <name>
kubectl get rs <name> -o yaml
kubectl scale rs <name> --replicas=5
kubectl get pods
kubectl get pods -l <label>
kubectl delete pod <pod>
kubectl get events --sort-by=.lastTimestamp
kubectl delete rs <name>
```

### Core Concept

> **ReplicaSet = Maintain the desired number of matching Pods.**


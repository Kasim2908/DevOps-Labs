# Deployments – Cheat Sheet

A quick-reference guide for **Kubernetes Deployments** covering core concepts, YAML, commands, scaling, rolling updates, rollbacks, strategies, troubleshooting, and interview essentials.

---

# 1. What is a Deployment?

A **Deployment** provides declarative management of application Pods and ReplicaSets.

It is commonly used to:

* Create application Pods
* Maintain desired replicas
* Perform rolling updates
* Roll back to previous versions
* Scale applications
* Manage application revisions

Relationship:

```text
Deployment
    |
    └── ReplicaSet
          |
          ├── Pod
          ├── Pod
          └── Pod
```

---

# 2. Deployment Responsibilities

```text
Deployment
    |
    ├── Desired replicas
    ├── Pod template
    ├── Rolling updates
    ├── Rollbacks
    ├── Revision history
    └── ReplicaSet management
```

---

# 3. Basic Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

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
          image: nginx:1.27
          ports:
            - containerPort: 80
```

---

# 4. Important Deployment YAML Fields

| Field                      | Purpose                                                        |
| -------------------------- | -------------------------------------------------------------- |
| `apiVersion`               | Kubernetes API version                                         |
| `kind`                     | Resource type                                                  |
| `metadata.name`            | Deployment name                                                |
| `spec.replicas`            | Desired number of Pods                                         |
| `spec.selector`            | Identifies Pods managed by the Deployment                      |
| `spec.template`            | Pod template                                                   |
| `template.metadata.labels` | Pod labels                                                     |
| `containers`               | Container configuration                                        |
| `strategy`                 | Deployment update strategy                                     |
| `minReadySeconds`          | Time a Pod must remain Ready before being considered available |
| `progressDeadlineSeconds`  | Maximum time Kubernetes waits for Deployment progress          |
| `revisionHistoryLimit`     | Number of old ReplicaSets retained                             |

---

# 5. Create a Deployment

Using YAML:

```bash
kubectl apply -f deployment.yaml
```

Imperatively:

```bash
kubectl create deployment nginx \
  --image=nginx:1.27 \
  --replicas=3
```

---

# 6. List Deployments

```bash
kubectl get deployments
```

Short form:

```bash
kubectl get deploy
```

---

# 7. Get Detailed Deployment Information

```bash
kubectl describe deployment nginx-deployment
```

---

# 8. Get Deployment YAML

```bash
kubectl get deployment nginx-deployment -o yaml
```

---

# 9. Get Deployment as JSON

```bash
kubectl get deployment nginx-deployment -o json
```

---

# 10. Check Deployment Status

```bash
kubectl get deployment
```

Example:

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           5m
```

### Meaning

```text
READY
→ Ready Pods / Desired Pods

UP-TO-DATE
→ Pods using the current Pod template

AVAILABLE
→ Pods available to serve according to Deployment availability rules
```

---

# 11. List ReplicaSets

```bash
kubectl get rs
```

A Deployment normally creates and manages ReplicaSets.

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

---

# 12. List Deployment Pods

```bash
kubectl get pods
```

Or by label:

```bash
kubectl get pods -l app=nginx
```

---

# 13. Scale a Deployment

Scale up:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Scale down:

```bash
kubectl scale deployment nginx-deployment --replicas=2
```

Verify:

```bash
kubectl get deployment
kubectl get pods
```

---

# 14. Scale Using YAML

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
kubectl apply -f deployment.yaml
```

---

# 15. Update Container Image

Use:

```bash
kubectl set image deployment/nginx-deployment \
  nginx=nginx:1.28
```

Check:

```bash
kubectl get deployment
```

---

# 16. Rolling Update

A Deployment uses `RollingUpdate` by default.

```yaml
strategy:
  type: RollingUpdate
```

Concept:

```text
Old Pods
   ↓
Create New Pods
   ↓
New Pods become Ready
   ↓
Remove Old Pods
   ↓
New Version Running
```

---

# 17. Monitor a Rolling Update

```bash
kubectl rollout status deployment/nginx-deployment
```

Watch Pods:

```bash
kubectl get pods -w
```

---

# 18. Check Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

Example:

```text
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

---

# 19. Inspect a Specific Revision

```bash
kubectl rollout history deployment/nginx-deployment \
  --revision=2
```

---

# 20. Rollback a Deployment

Rollback to the previous revision:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Check:

```bash
kubectl rollout status deployment/nginx-deployment
```

---

# 21. Rollback to a Specific Revision

```bash
kubectl rollout undo deployment/nginx-deployment \
  --to-revision=2
```

Verify:

```bash
kubectl rollout status deployment/nginx-deployment
```

---

# 22. Restart a Deployment

Restart all Pods managed by the Deployment:

```bash
kubectl rollout restart deployment/nginx-deployment
```

Monitor:

```bash
kubectl rollout status deployment/nginx-deployment
```

---

# 23. Pause a Deployment

```bash
kubectl rollout pause deployment/nginx-deployment
```

A paused Deployment allows you to make multiple changes before resuming the rollout.

---

# 24. Resume a Deployment

```bash
kubectl rollout resume deployment/nginx-deployment
```

---

# 25. RollingUpdate Strategy

Example:

```yaml
strategy:
  type: RollingUpdate

  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

---

# 26. maxSurge

`maxSurge` controls how many Pods can temporarily exist above the desired replica count during a rollout.

Example:

```yaml
replicas: 5

strategy:
  rollingUpdate:
    maxSurge: 1
```

Conceptually:

```text
Desired = 5

Maximum temporary Pods ≈ 6
```

`maxSurge` can be expressed as:

* Number
* Percentage

Example:

```yaml
maxSurge: 2
```

or:

```yaml
maxSurge: 25%
```

---

# 27. maxUnavailable

`maxUnavailable` controls how many Pods can be unavailable during a rolling update.

Example:

```yaml
replicas: 5

strategy:
  rollingUpdate:
    maxUnavailable: 1
```

Conceptually:

```text
Desired = 5
Potential minimum available ≈ 4
```

It can be specified as:

```yaml
maxUnavailable: 1
```

or:

```yaml
maxUnavailable: 25%
```

---

# 28. Zero-Downtime Configuration

A common configuration is:

```yaml
strategy:
  type: RollingUpdate

  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Combined with a properly configured readiness probe, this can help maintain application availability during updates.

---

# 29. Recreate Strategy

Instead of a rolling update:

```yaml
strategy:
  type: Recreate
```

Behavior:

```text
Old Pods
   ↓
Terminate Old Pods
   ↓
Create New Pods
```

There can be downtime during the transition.

---

# 30. RollingUpdate vs Recreate

| RollingUpdate                        | Recreate                                    |
| ------------------------------------ | ------------------------------------------- |
| Gradually replaces Pods              | Deletes old Pods first                      |
| Usually preferred for stateless apps | Useful when old/new versions cannot coexist |
| Can support high availability        | Can cause downtime                          |
| Default strategy                     | Must be explicitly configured               |

---

# 31. Readiness Probe

Example:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

A readiness probe tells Kubernetes whether the application is ready to receive traffic.

During a rolling update, readiness is important because the Deployment should not treat an unready new Pod as available.

---

# 32. Liveness Probe

Example:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 10
```

A liveness probe helps Kubernetes determine whether a container needs to be restarted.

### Remember

```text
Readiness
→ Can this Pod receive traffic?

Liveness
→ Is this container still healthy?
```

---

# 33. Startup Probe

Example:

```yaml
startupProbe:
  httpGet:
    path: /health
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```

A startup probe is useful for applications that take a long time to start.

Concept:

```text
Startup Probe
      ↓
Application starts
      ↓
Liveness/Readiness checks become effective
```

---

# 34. minReadySeconds

Example:

```yaml
spec:
  minReadySeconds: 30
```

A newly created Pod must remain Ready for the configured number of seconds before it is considered available.

Useful for avoiding declaring a Pod available immediately after it becomes Ready.

---

# 35. progressDeadlineSeconds

Example:

```yaml
spec:
  progressDeadlineSeconds: 600
```

It defines how long Kubernetes waits for Deployment progress before reporting that the Deployment has failed to make progress.

Check:

```bash
kubectl describe deployment <name>
```

Look at the Deployment conditions.

---

# 36. revisionHistoryLimit

Example:

```yaml
spec:
  revisionHistoryLimit: 5
```

Controls how many old ReplicaSets are retained for Deployment revision history.

This is useful for:

* Rollbacks
* Limiting old ReplicaSets
* Reducing unnecessary stored revision history

---

# 37. Change Cause

You can record a human-readable change cause.

Example:

```bash
kubectl annotate deployment nginx-deployment \
  kubernetes.io/change-cause="Updated nginx to 1.28"
```

Then:

```bash
kubectl rollout history deployment/nginx-deployment
```

---

# 38. Deployment Revision Concept

Suppose:

```text
Revision 1 → nginx:1.25
Revision 2 → nginx:1.26
Revision 3 → nginx:1.27
```

Each Pod-template change can create a new Deployment revision and corresponding ReplicaSet.

Concept:

```text
Deployment
   |
   ├── Revision 1 → ReplicaSet
   ├── Revision 2 → ReplicaSet
   └── Revision 3 → ReplicaSet
```

---

# 39. Does Scaling Create a New Revision?

Normally, changing only:

```yaml
spec:
  replicas: 3
```

to:

```yaml
spec:
  replicas: 5
```

does **not** create a new Deployment revision because the Pod template has not changed.

---

# 40. What Creates a New Revision?

Changes to the Pod template can create a new revision.

Examples:

```text
Container image
Environment variables
Container command
Container arguments
Resources
Volumes
Pod labels
Pod annotations
```

Concept:

```text
Pod Template Change
       ↓
New ReplicaSet
       ↓
New Deployment Revision
```

---

# 41. Deployment Self-Healing

Delete a managed Pod:

```bash
kubectl delete pod <pod-name>
```

Watch:

```bash
kubectl get pods -w
```

The ReplicaSet managed by the Deployment creates a replacement.

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod deleted
    ↓
ReplicaSet detects difference
    ↓
Replacement Pod
```

---

# 42. Deployment and ReplicaSet Relationship

A Deployment does not directly manage individual Pods in the same way a ReplicaSet does.

Typical hierarchy:

```text
Deployment
     |
     ↓
ReplicaSet
     |
     ↓
Pods
```

The Deployment manages ReplicaSets.

The ReplicaSet maintains the desired number of Pods.

---

# 43. Deployment vs ReplicaSet

| Deployment                      | ReplicaSet                       |
| ------------------------------- | -------------------------------- |
| Higher-level controller         | Lower-level controller           |
| Manages ReplicaSets             | Manages Pods                     |
| Supports rolling updates        | Does not perform rolling updates |
| Supports rollback               | No native rollout rollback       |
| Maintains application revisions | Maintains replica count          |
| Common production abstraction   | Usually managed by Deployment    |

---

# 44. Deployment vs StatefulSet

| Deployment                                      | StatefulSet                                |
| ----------------------------------------------- | ------------------------------------------ |
| Usually for stateless applications              | For stateful applications                  |
| Pods are interchangeable                        | Pods have stable identities                |
| Pod names are not stable                        | Stable Pod names                           |
| Volumes are not inherently tied to Pod identity | Supports stable storage identity           |
| Common for web applications                     | Common for databases and clustered systems |

---

# 45. Deployment vs DaemonSet

| Deployment                     | DaemonSet                           |
| ------------------------------ | ----------------------------------- |
| Controls desired replica count | Runs Pods on eligible Nodes         |
| Used for applications          | Used for node-level workloads       |
| Can have arbitrary replicas    | Typically one Pod per eligible Node |
| Supports rolling updates       | Also supports update strategies     |

Common DaemonSet use cases:

* Log collectors
* Monitoring agents
* Node-level networking components

---

# 46. Deployment Troubleshooting Flow

When a Deployment is not working:

```text
kubectl get deployment
        ↓
kubectl describe deployment
        ↓
kubectl rollout status
        ↓
kubectl get rs
        ↓
kubectl get pods
        ↓
kubectl describe pod
        ↓
kubectl logs
        ↓
kubectl get events
```

---

# 47. Check Deployment Conditions

```bash
kubectl describe deployment <name>
```

Look for:

```text
Conditions:
  Available
  Progressing
  ReplicaFailure
```

These conditions provide useful information about Deployment state.

---

# 48. Common Deployment Problems

| Problem                    | Possible Cause                                   |
| -------------------------- | ------------------------------------------------ |
| `Pending`                  | Insufficient resources or scheduling constraints |
| `ImagePullBackOff`         | Image cannot be pulled                           |
| `ErrImagePull`             | Invalid image or registry issue                  |
| `CrashLoopBackOff`         | Application repeatedly crashes                   |
| Readiness failure          | Application is not ready                         |
| Rollout stuck              | New Pods not becoming Ready                      |
| `ProgressDeadlineExceeded` | Deployment failed to make progress               |
| `0/3 Available`            | Pods not becoming available                      |

---

# 49. Troubleshoot Invalid Image

Update to an invalid image:

```bash
kubectl set image deployment/nginx-deployment \
  nginx=nginx:invalid-version
```

Check:

```bash
kubectl get pods
```

Then:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
ImagePullBackOff
```

Rollback:

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

# 50. Troubleshoot Failed Readiness Probe

Check Pods:

```bash
kubectl get pods
```

Describe:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
Readiness probe failed
```

Check application logs:

```bash
kubectl logs <pod-name>
```

---

# 51. Check Deployment Events

```bash
kubectl describe deployment <name>
```

Cluster-wide:

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

# 52. Check Current Image

```bash
kubectl get deployment nginx-deployment \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

---

# 53. Check Number of Replicas

```bash
kubectl get deployment nginx-deployment \
  -o jsonpath='{.spec.replicas}'
```

---

# 54. Check Available Replicas

```bash
kubectl get deployment nginx-deployment \
  -o jsonpath='{.status.availableReplicas}'
```

---

# 55. Check Pod Template

```bash
kubectl get deployment nginx-deployment \
  -o jsonpath='{.spec.template}'
```

---

# 56. Watch Deployment Changes

```bash
kubectl get deployment nginx-deployment -w
```

Watch Pods:

```bash
kubectl get pods -w
```

Watch ReplicaSets:

```bash
kubectl get rs -w
```

---

# 57. Force a New Rollout

One common method:

```bash
kubectl rollout restart deployment/nginx-deployment
```

This recreates the Pods through a new rollout.

---

# 58. Deployment with Resources

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"

  limits:
    cpu: "500m"
    memory: "256Mi"
```

Complete container example:

```yaml
containers:
  - name: nginx
    image: nginx:1.27

    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"

      limits:
        cpu: "500m"
        memory: "256Mi"
```

---

# 59. Deployment with Environment Variables

```yaml
env:
  - name: APP_ENV
    value: "production"

  - name: LOG_LEVEL
    value: "info"
```

Changing the Pod template can trigger a new Deployment revision.

---

# 60. Deployment with ConfigMap

Example:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

Or:

```yaml
env:
  - name: APP_MODE
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: mode
```

---

# 61. Deployment with Secret

Example:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

Avoid placing sensitive values directly in Deployment YAML.

---

# 62. Deployment with Namespace

Create in a specific namespace:

```bash
kubectl apply -f deployment.yaml -n production
```

Check:

```bash
kubectl get deployment -n production
```

---

# 63. Delete a Deployment

```bash
kubectl delete deployment nginx-deployment
```

The Deployment's managed resources are normally deleted according to Kubernetes ownership/deletion behavior.

---

# 64. Important Commands Cheat Sheet

```bash
# Create/update
kubectl apply -f deployment.yaml

# Create imperatively
kubectl create deployment <name> --image=<image>

# List Deployments
kubectl get deployment

# Describe
kubectl describe deployment <name>

# Get YAML
kubectl get deployment <name> -o yaml

# Scale
kubectl scale deployment <name> --replicas=5

# Update image
kubectl set image deployment/<name> <container>=<image>:<tag>

# Rollout status
kubectl rollout status deployment/<name>

# Rollout history
kubectl rollout history deployment/<name>

# Inspect revision
kubectl rollout history deployment/<name> --revision=2

# Rollback
kubectl rollout undo deployment/<name>

# Rollback to revision
kubectl rollout undo deployment/<name> --to-revision=2

# Restart
kubectl rollout restart deployment/<name>

# Pause
kubectl rollout pause deployment/<name>

# Resume
kubectl rollout resume deployment/<name>

# List ReplicaSets
kubectl get rs

# List Pods
kubectl get pods

# Watch Pods
kubectl get pods -w

# Check events
kubectl get events --sort-by=.lastTimestamp

# Delete
kubectl delete deployment <name>
```

---

# 65. Deployment Rollout Workflow

```text
1. Create Deployment
        ↓
2. Deployment creates ReplicaSet
        ↓
3. ReplicaSet creates Pods
        ↓
4. Application becomes Ready
        ↓
5. Update Pod template
        ↓
6. New ReplicaSet created
        ↓
7. Rolling update starts
        ↓
8. New Pods become Ready
        ↓
9. Old Pods are scaled down
        ↓
10. New version becomes active
```

---

# 66. Rollback Workflow

```text
Current Version
      ↓
Problem Detected
      ↓
kubectl rollout undo
      ↓
Previous ReplicaSet
      ↓
Previous Pod Template
      ↓
Old Version Restored
```

---

# 67. One-Minute Revision

```text
Deployment
──────────

Purpose:
Manage application releases and Pods declaratively.

Manages:
ReplicaSets

ReplicaSet manages:
Pods

Default strategy:
RollingUpdate

Update:
kubectl set image

Check rollout:
kubectl rollout status

History:
kubectl rollout history

Rollback:
kubectl rollout undo

Restart:
kubectl rollout restart

Scale:
kubectl scale deployment

Strategies:
RollingUpdate
Recreate

Important fields:
replicas
selector
template
strategy
minReadySeconds
progressDeadlineSeconds
revisionHistoryLimit
```

---

# 68. Interview Quick Answers

### What is a Deployment?

> A Deployment is a Kubernetes controller that declaratively manages application Pods and ReplicaSets and provides controlled application updates and rollbacks.

### What does a Deployment manage?

> Deployments manage ReplicaSets, which in turn manage Pods.

### What is the default Deployment strategy?

> `RollingUpdate`.

### What is a rolling update?

> A rolling update gradually replaces old Pods with new Pods while maintaining the configured availability constraints.

### What is `maxSurge`?

> It controls how many additional Pods can be created above the desired replica count during a rolling update.

### What is `maxUnavailable`?

> It controls how many Pods can be unavailable during a rolling update.

### What is `Recreate`?

> It terminates existing Pods before creating new Pods.

### How do you check rollout status?

```bash
kubectl rollout status deployment/<name>
```

### How do you rollback?

```bash
kubectl rollout undo deployment/<name>
```

### How do you rollback to a specific revision?

```bash
kubectl rollout undo deployment/<name> --to-revision=<number>
```

### Does changing replicas create a new revision?

> No. Changing only the replica count does not change the Pod template.

### Does changing the image create a new revision?

> Yes. Changing the Pod template causes a new Deployment revision and ReplicaSet.

### How do you restart a Deployment?

```bash
kubectl rollout restart deployment/<name>
```

### What is `minReadySeconds`?

> The minimum amount of time a newly Ready Pod must remain Ready before it is considered available.

### What is `progressDeadlineSeconds`?

> The amount of time Kubernetes waits for Deployment progress before reporting a stalled rollout.

---

# 69. Final Mental Model

Memorize this:

```text
                 Deployment
                     |
                     | manages
                     ↓
                ReplicaSet
                     |
                     | maintains
                     ↓
                   Pods
                     |
                     | run
                     ↓
                Containers
```

For application updates:

```text
Change Pod Template
        ↓
New ReplicaSet
        ↓
Rolling Update
        ↓
New Pods
        ↓
Readiness Checks
        ↓
Old ReplicaSet Scaled Down
```

For failures:

```text
Deployment Problem
        ↓
get deployment
        ↓
describe deployment
        ↓
rollout status
        ↓
get rs
        ↓
get pods
        ↓
describe pod
        ↓
logs
        ↓
events
        ↓
rollback if required
```

> **Key takeaway:** A Kubernetes Deployment manages application releases through ReplicaSets, providing declarative updates, scaling, rolling deployments, revision history, and rollback capabilities.


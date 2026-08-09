# Deployments – Interview Questions

This document contains commonly asked **Kubernetes Deployment interview questions**, ranging from basic concepts to advanced and scenario-based questions.

---

# Basic Deployment Interview Questions

## 1. What is a Deployment in Kubernetes?

A **Deployment** is a Kubernetes controller that manages the lifecycle of application Pods.

It provides declarative updates for Pods and ReplicaSets and supports features such as:

* Scaling
* Rolling updates
* Rollbacks
* Revision history
* Self-healing through ReplicaSets

Typical hierarchy:

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

## 2. Why do we use Deployments in Kubernetes?

Deployments make it easier to manage applications that need:

* Multiple Pod replicas
* Rolling updates
* Rollbacks
* Scaling
* Version management
* High availability

Instead of manually creating and updating Pods, you define the desired state in a Deployment.

---

## 3. What is the difference between a Deployment and a Pod?

| Pod                                 | Deployment                                     |
| ----------------------------------- | ---------------------------------------------- |
| Runs containers                     | Manages application Pods                       |
| Does not provide rollout management | Supports rolling updates                       |
| Can be recreated when deleted       | Maintains desired replicas through ReplicaSets |
| Represents a running workload unit  | Provides higher-level workload management      |

Example:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Containers
```

---

## 4. What is the difference between Deployment and ReplicaSet?

A **Deployment manages ReplicaSets**, while a **ReplicaSet manages Pods**.

```text
Deployment
    |
    └── ReplicaSet
          |
          ├── Pod
          ├── Pod
          └── Pod
```

### Deployment provides:

* Rolling updates
* Rollbacks
* Revision history
* Deployment strategies
* ReplicaSet management

### ReplicaSet provides:

* Desired Pod count
* Pod replacement
* Basic scaling

---

## 5. Can a Deployment exist without a ReplicaSet?

A Deployment normally creates and manages ReplicaSets.

When you create a Deployment, Kubernetes creates a ReplicaSet that creates and maintains the Pods.

```bash
kubectl get deployment
kubectl get rs
kubectl get pods
```

---

# Deployment YAML Questions

## 6. What does a basic Deployment YAML look like?

Example:

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
          image: nginx:1.25
          ports:
            - containerPort: 80
```

---

## 7. What are the important fields in a Deployment YAML?

Common fields include:

```yaml
apiVersion:
kind:
metadata:
spec:
  replicas:
  selector:
  strategy:
  template:
```

The `template` defines the Pod specification.

The `selector` determines which Pods belong to the Deployment.

---

## 8. What is `replicas` in a Deployment?

`replicas` defines the desired number of Pods.

Example:

```yaml
spec:
  replicas: 5
```

This tells Kubernetes to maintain five Pod replicas.

---

## 9. What is the purpose of `selector` in a Deployment?

The selector identifies the Pods managed by the Deployment.

Example:

```yaml
selector:
  matchLabels:
    app: nginx
```

The Pod template must have matching labels:

```yaml
template:
  metadata:
    labels:
      app: nginx
```

---

## 10. What is the purpose of the Pod template in a Deployment?

The Pod template defines how Pods should be created.

It includes:

* Container image
* Container name
* Ports
* Environment variables
* Resource requests/limits
* Volumes
* Probes
* Security settings

Example:

```yaml
template:
  spec:
    containers:
      - name: nginx
        image: nginx:1.25
```

---

# Deployment Commands

## 11. How do you create a Deployment?

Using YAML:

```bash
kubectl apply -f deployment.yaml
```

Or using the imperative command:

```bash
kubectl create deployment nginx --image=nginx
```

---

## 12. How do you list Deployments?

```bash
kubectl get deployments
```

Short form:

```bash
kubectl get deploy
```

---

## 13. How do you get detailed information about a Deployment?

```bash
kubectl describe deployment nginx-deployment
```

---

## 14. How do you get the Deployment YAML?

```bash
kubectl get deployment nginx-deployment -o yaml
```

---

## 15. How do you check the Pods created by a Deployment?

```bash
kubectl get pods
```

You can also use labels:

```bash
kubectl get pods -l app=nginx
```

---

## 16. How do you scale a Deployment?

Using `kubectl scale`:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Or modify the manifest:

```yaml
spec:
  replicas: 5
```

and apply it:

```bash
kubectl apply -f deployment.yaml
```

---

# Rolling Updates

## 17. What is a rolling update?

A rolling update gradually replaces old Pods with new Pods instead of stopping all old Pods at once.

Example:

```text
Old version:
Pod v1
Pod v1
Pod v1

        ↓ Rolling Update

Pod v1
Pod v1
Pod v2

        ↓

Pod v1
Pod v2
Pod v2

        ↓

Pod v2
Pod v2
Pod v2
```

This allows an application to remain available during the update.

---

## 18. How do you update a Deployment image?

Using:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.26
```

Then check the rollout:

```bash
kubectl rollout status deployment/nginx-deployment
```

---

## 19. What happens when you update the image of a Deployment?

The Deployment detects a change in the Pod template.

It creates a new ReplicaSet.

The new ReplicaSet is gradually scaled up while the old ReplicaSet is scaled down according to the Deployment's update strategy.

```text
Deployment
   |
   ├── Old ReplicaSet
   |      ├── Old Pod
   |      └── Old Pod
   |
   └── New ReplicaSet
          ├── New Pod
          └── New Pod
```

---

## 20. How do you check the status of a rollout?

```bash
kubectl rollout status deployment/nginx-deployment
```

Example:

```text
Waiting for deployment "nginx-deployment" rollout to finish...
deployment "nginx-deployment" successfully rolled out
```

---

## 21. How do you view Deployment rollout history?

```bash
kubectl rollout history deployment/nginx-deployment
```

To inspect a specific revision:

```bash
kubectl rollout history deployment/nginx-deployment --revision=2
```

---

# Rollbacks

## 22. How do you rollback a Deployment?

Use:

```bash
kubectl rollout undo deployment/nginx-deployment
```

This rolls the Deployment back to the previous revision.

---

## 23. How do you rollback to a specific revision?

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

---

## 24. What happens during a rollback?

Kubernetes changes the Deployment back to the desired Pod template from the selected revision.

A new rollout occurs using the previous configuration.

Conceptually:

```text
Current:
v3

Rollback:
v3 → v2
```

The Deployment manages the transition through ReplicaSets.

---

## 25. How do you pause a Deployment rollout?

```bash
kubectl rollout pause deployment/nginx-deployment
```

You can resume it with:

```bash
kubectl rollout resume deployment/nginx-deployment
```

---

# Deployment Strategies

## 26. What Deployment strategies are available?

The two main Deployment strategies are:

1. `RollingUpdate`
2. `Recreate`

---

## 27. What is the RollingUpdate strategy?

`RollingUpdate` gradually replaces old Pods with new Pods.

Example:

```yaml
strategy:
  type: RollingUpdate
```

This is the default Deployment strategy.

---

## 28. What is the Recreate strategy?

With `Recreate`, Kubernetes terminates the existing Pods before creating the new Pods.

Example:

```yaml
strategy:
  type: Recreate
```

Conceptually:

```text
Old Pods
   ↓
Delete old Pods
   ↓
No application Pods
   ↓
Create new Pods
```

This can cause downtime, so it is generally used when old and new application versions cannot run simultaneously.

---

## 29. What is the difference between RollingUpdate and Recreate?

| RollingUpdate                                 | Recreate                           |
| --------------------------------------------- | ---------------------------------- |
| Gradually replaces Pods                       | Deletes old Pods first             |
| Usually minimizes downtime                    | Can cause downtime                 |
| Default strategy                              | Must be explicitly configured      |
| Supports old/new versions running temporarily | Old and new versions don't overlap |

---

# maxSurge and maxUnavailable

## 30. What is `maxSurge`?

`maxSurge` defines the maximum number of Pods that can exist above the desired replica count during a rolling update.

Example:

```yaml
strategy:
  rollingUpdate:
    maxSurge: 1
```

If:

```text
Desired replicas = 3
maxSurge = 1
```

Kubernetes can temporarily have up to:

```text
4 Pods
```

during the update.

---

## 31. What is `maxUnavailable`?

`maxUnavailable` defines how many Pods can be unavailable during a rolling update.

Example:

```yaml
strategy:
  rollingUpdate:
    maxUnavailable: 1
```

If:

```text
Desired replicas = 3
maxUnavailable = 1
```

Kubernetes can temporarily have one unavailable Pod during the rollout.

---

## 32. What is the difference between maxSurge and maxUnavailable?

| maxSurge                            | maxUnavailable                       |
| ----------------------------------- | ------------------------------------ |
| Controls extra Pods                 | Controls unavailable Pods            |
| Allows capacity above desired count | Allows capacity below desired count  |
| Helps speed up replacement          | Controls availability during rollout |

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

---

# Deployment Status

## 33. What does `kubectl get deployment` show?

Example:

```bash
kubectl get deployment
```

Output may look like:

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           10m
```

### Meaning

* `READY` → Ready Pods / desired Pods
* `UP-TO-DATE` → Pods using the latest Pod template
* `AVAILABLE` → Pods available to serve traffic
* `AGE` → Deployment age

---

## 34. What does `3/3` under READY mean?

It means:

```text
3 Ready Pods
out of
3 desired Pods
```

So the Deployment currently has the desired number of Ready Pods.

---

## 35. What is `Available` in a Deployment?

`Available` represents the number of Pods considered available according to the Deployment's availability rules.

A Pod generally needs to be Ready for the required period before it contributes to availability.

---

# Deployment and ReplicaSet Relationship

## 36. How many ReplicaSets can a Deployment have?

A Deployment can have multiple ReplicaSets over its lifetime.

For example:

```text
Deployment
   |
   ├── ReplicaSet v1
   ├── ReplicaSet v2
   └── ReplicaSet v3
```

Usually, the current ReplicaSet is active while older ReplicaSets are retained according to the Deployment's revision history configuration.

---

## 37. Why does a Deployment create a new ReplicaSet when the image changes?

A change to the Pod template creates a new Deployment revision.

The Deployment creates a new ReplicaSet to represent that revision.

For example:

```text
nginx:1.25
     ↓
ReplicaSet v1

nginx:1.26
     ↓
ReplicaSet v2
```

---

## 38. Does changing the number of replicas create a new ReplicaSet?

No.

Changing only:

```yaml
spec:
  replicas: 5
```

does not change the Pod template.

Therefore, it does not normally create a new Deployment revision or ReplicaSet.

Changing the Pod template, such as the image, environment variables, or container configuration, creates a new revision.

---

# Deployment Revision Questions

## 39. What is a Deployment revision?

A revision represents a version of the Deployment's Pod template.

For example:

```text
Revision 1 → nginx:1.24
Revision 2 → nginx:1.25
Revision 3 → nginx:1.26
```

Revision history allows Kubernetes to support rollback.

---

## 40. What is `revisionHistoryLimit`?

`revisionHistoryLimit` determines how many old ReplicaSets are retained by a Deployment.

Example:

```yaml
spec:
  revisionHistoryLimit: 5
```

This allows a limited number of previous revisions to be retained.

---

# Probes and Deployments

## 41. How does a readiness probe affect a Deployment rollout?

A readiness probe determines whether a Pod is ready to receive traffic.

During a rolling update, Kubernetes uses Pod readiness when determining whether new Pods are ready before continuing the rollout.

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

---

## 42. What happens if the new Pods never become Ready?

The Deployment rollout may remain incomplete.

Check:

```bash
kubectl rollout status deployment/nginx-deployment
```

Then investigate:

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

Common causes include:

* Application startup failure
* Incorrect readiness probe
* Incorrect port
* Configuration errors
* Missing dependencies
* Insufficient resources

---

# Deployment Troubleshooting

## 43. A Deployment shows `0/3` Ready. How do you troubleshoot?

Start with:

```bash
kubectl get deployment
kubectl get pods
```

Then:

```bash
kubectl describe deployment <deployment-name>
```

Inspect individual Pods:

```bash
kubectl describe pod <pod-name>
```

Check logs:

```bash
kubectl logs <pod-name>
```

Check events:

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

## 44. A Deployment rollout is stuck. What would you check?

Check:

```bash
kubectl rollout status deployment/<deployment-name>
```

Then:

```bash
kubectl get rs
kubectl get pods
kubectl describe deployment <deployment-name>
```

Look for:

* Pods stuck in Pending
* ImagePullBackOff
* CrashLoopBackOff
* Readiness probe failures
* Insufficient resources
* Scheduling constraints
* Failed admission policies
* Configuration or Secret problems

---

## 45. What does `ImagePullBackOff` mean during a Deployment?

It usually means Kubernetes cannot successfully pull the container image and is backing off before retrying.

Possible causes:

* Incorrect image name
* Incorrect image tag
* Private registry authentication failure
* Registry unavailable
* Network problems
* Image does not exist

Check:

```bash
kubectl describe pod <pod-name>
```

---

## 46. What does `CrashLoopBackOff` mean?

It means a container repeatedly starts and crashes, and Kubernetes is progressively delaying restart attempts.

Check:

```bash
kubectl logs <pod-name>
```

For the previous container instance:

```bash
kubectl logs <pod-name> --previous
```

Also inspect:

```bash
kubectl describe pod <pod-name>
```

---

# Scenario-Based Interview Questions

## 47. You have 5 replicas and update the image. How does Kubernetes perform the update?

Assuming the default `RollingUpdate` strategy, Kubernetes creates a new ReplicaSet and gradually replaces the old Pods.

```text
Before:

RS-v1
 ├── Pod
 ├── Pod
 ├── Pod
 ├── Pod
 └── Pod

After update:

RS-v1
 ├── Pod
 └── Pod

RS-v2
 ├── Pod
 ├── Pod
 └── Pod
```

Eventually:

```text
RS-v1 → 0 Pods
RS-v2 → 5 Pods
```

---

## 48. What happens if the new application version is broken?

The rollout can become stuck if new Pods fail to become Ready.

You should investigate the new Pods and, if necessary, roll back:

```bash
kubectl rollout undo deployment/<deployment-name>
```

---

## 49. A Deployment has 10 replicas. You update the image and suddenly traffic drops. What could be wrong?

Possible causes include:

* New Pods failing readiness checks
* Incorrect application configuration
* Incorrect environment variables
* New application version errors
* Incorrect Service selectors
* Network or dependency problems
* Insufficient resources
* Bad readiness/liveness probes

Useful commands:

```bash
kubectl get pods
kubectl get deployment
kubectl describe deployment <deployment-name>
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

---

## 50. You updated a Deployment but no new ReplicaSet was created. Why?

If the change does not modify the Pod template, a new ReplicaSet is not required.

For example, changing:

```yaml
spec:
  replicas: 3
```

to:

```yaml
spec:
  replicas: 5
```

changes the desired number of Pods but does not create a new revision.

---

## 51. A Deployment has 3 replicas, but one Pod is Pending. What would you check?

Check:

```bash
kubectl describe pod <pending-pod>
```

Common causes:

* Insufficient CPU or memory
* Node selector mismatch
* Node affinity
* Taints and tolerations
* Resource quotas
* PersistentVolume issues
* Scheduling constraints

Also check:

```bash
kubectl get nodes
kubectl get events --sort-by=.lastTimestamp
```

---

## 52. How would you achieve zero-downtime deployments?

Common considerations include:

1. Use the `RollingUpdate` strategy.
2. Configure appropriate `maxUnavailable` and `maxSurge`.
3. Configure a correct readiness probe.
4. Run multiple replicas.
5. Ensure adequate cluster capacity.
6. Use proper application shutdown handling.
7. Test backward/forward compatibility between application versions.

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

---

## 53. What happens if `maxUnavailable` is set to 0?

It means Kubernetes should not intentionally reduce the number of available Pods below the desired number during the rolling update.

This can improve availability but may require additional capacity, especially when combined with `maxSurge`.

---

## 54. What happens if `maxSurge` is set to 0?

No additional Pods can be created above the desired replica count during the rollout.

The Deployment must make progress by removing/replacing existing Pods according to the allowed availability constraints.

---

# Advanced Questions

## 55. Can a Deployment manage multiple ReplicaSets?

Yes.

A Deployment can retain multiple ReplicaSets representing different revisions.

```text
Deployment
   |
   ├── RS revision 1
   ├── RS revision 2
   └── RS revision 3
```

Only the appropriate ReplicaSet or ReplicaSets are scaled according to the current rollout state.

---

## 56. What is the purpose of `minReadySeconds`?

`minReadySeconds` specifies how long a newly created Pod must remain Ready before the Deployment considers it available.

Example:

```yaml
spec:
  minReadySeconds: 30
```

This can help prevent a rollout from progressing immediately when a Pod becomes Ready only briefly.

---

## 57. What is `progressDeadlineSeconds`?

`progressDeadlineSeconds` specifies how long Kubernetes should wait for Deployment progress before reporting that the Deployment has failed to make progress.

Example:

```yaml
spec:
  progressDeadlineSeconds: 600
```

This helps identify stalled rollouts.

---

## 58. What does `kubectl rollout status` do?

It watches the progress of a Deployment rollout.

Example:

```bash
kubectl rollout status deployment/nginx
```

It is useful in CI/CD pipelines because the command can wait for the rollout to complete.

---

## 59. How can you restart a Deployment without changing the image?

You can use:

```bash
kubectl rollout restart deployment/nginx
```

This triggers a new rollout by updating the Pod template metadata.

---

## 60. What is the difference between `kubectl apply` and `kubectl create` for Deployments?

### `kubectl create`

Creates a resource.

```bash
kubectl create deployment nginx --image=nginx
```

### `kubectl apply`

Creates or updates a resource from a declarative configuration.

```bash
kubectl apply -f deployment.yaml
```

For infrastructure managed through Git or CI/CD, declarative `kubectl apply` workflows are commonly used.

---

# Important Kubernetes Deployment Commands

## Create

```bash
kubectl apply -f deployment.yaml
```

## List

```bash
kubectl get deployments
```

## Detailed information

```bash
kubectl describe deployment <name>
```

## Scale

```bash
kubectl scale deployment <name> --replicas=5
```

## Update image

```bash
kubectl set image deployment/<name> <container>=<image>:<tag>
```

## Check rollout

```bash
kubectl rollout status deployment/<name>
```

## View rollout history

```bash
kubectl rollout history deployment/<name>
```

## Rollback

```bash
kubectl rollout undo deployment/<name>
```

## Rollback to a specific revision

```bash
kubectl rollout undo deployment/<name> --to-revision=2
```

## Pause rollout

```bash
kubectl rollout pause deployment/<name>
```

## Resume rollout

```bash
kubectl rollout resume deployment/<name>
```

## Restart Deployment

```bash
kubectl rollout restart deployment/<name>
```

---

# Quick Revision Table

| Topic                          | Key Point                                   |
| ------------------------------ | ------------------------------------------- |
| Deployment                     | Manages application rollout and ReplicaSets |
| ReplicaSet                     | Maintains desired Pod count                 |
| Pod                            | Runs application containers                 |
| Default strategy               | `RollingUpdate`                             |
| Other strategy                 | `Recreate`                                  |
| Rolling update                 | Gradually replaces old Pods                 |
| Rollback                       | `kubectl rollout undo`                      |
| Scale                          | `kubectl scale deployment`                  |
| Image update                   | `kubectl set image`                         |
| Rollout status                 | `kubectl rollout status`                    |
| Rollout history                | `kubectl rollout history`                   |
| Pause rollout                  | `kubectl rollout pause`                     |
| Resume rollout                 | `kubectl rollout resume`                    |
| Restart                        | `kubectl rollout restart`                   |
| Extra Pods during update       | `maxSurge`                                  |
| Unavailable Pods during update | `maxUnavailable`                            |
| Minimum Ready duration         | `minReadySeconds`                           |
| Stalled rollout detection      | `progressDeadlineSeconds`                   |
| Revision retention             | `revisionHistoryLimit`                      |

---

# Deployment Interview Cheat Sheet

Remember this flow:

```text
Deployment
    |
    | manages
    ↓
ReplicaSet
    |
    | manages
    ↓
Pods
    |
    | runs
    ↓
Containers
```

### Most important commands

```bash
kubectl get deploy
kubectl describe deploy <name>
kubectl get rs
kubectl get pods
kubectl set image deployment/<name> <container>=<image>:<tag>
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout restart deployment/<name>
kubectl scale deployment/<name> --replicas=5
```

### Most important concepts

```text
Deployment
├── Scaling
├── Rolling Updates
├── Rollbacks
├── Revision History
└── ReplicaSet Management
```

---

# Interview Tip

A strong short answer for **"What is a Kubernetes Deployment?"** is:

> **A Deployment is a Kubernetes workload controller that declaratively manages application Pods through ReplicaSets and provides scaling, rolling updates, rollbacks, and revision management.**

For a **"How does a Deployment update an application?"** question, remember:

```text
Update Pod Template
       ↓
New ReplicaSet
       ↓
Scale New ReplicaSet Up
       ↓
Scale Old ReplicaSet Down
       ↓
New Version Running
```

This is the core concept behind Kubernetes Deployment rolling updates.


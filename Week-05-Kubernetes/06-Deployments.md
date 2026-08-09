# 06 - Deployments

A **Deployment** is one of the most commonly used Kubernetes workload resources. It provides declarative management of application Pods and ReplicaSets and makes it easier to perform application updates, scaling, rollbacks, and controlled releases.

---

# 1. What is a Deployment?

A Kubernetes Deployment manages a set of Pods and provides controlled updates to those Pods.

A Deployment typically manages:

```text
Deployment
    |
    ↓
ReplicaSet
    |
    ↓
Pods
    |
    ↓
Containers
```

For example:

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
```

This Deployment tells Kubernetes:

> Keep three Pods running with the specified Pod template.

---

# 2. Why Do We Need Deployments?

A ReplicaSet can maintain a desired number of Pods, but it does not provide a complete application release-management mechanism.

A Deployment adds capabilities such as:

* Declarative application management
* Rolling updates
* Rollbacks
* Revision history
* Scaling
* Controlled application releases
* Deployment status tracking
* ReplicaSet management

Without a Deployment:

```text
ReplicaSet
    ↓
Pods
```

With a Deployment:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

The Deployment becomes the higher-level controller responsible for application releases.

---

# 3. Deployment Architecture

The relationship is:

```text
                Deployment
                    |
                    |
          manages ReplicaSets
                    |
          +---------+---------+
          |                   |
     Old ReplicaSet      New ReplicaSet
          |                   |
       Old Pods            New Pods
```

During an update, Kubernetes may temporarily have multiple ReplicaSets.

For example:

```text
Deployment
    |
    +---- ReplicaSet v1
    |          |
    |       0 Pods
    |
    +---- ReplicaSet v2
               |
             3 Pods
```

The Deployment keeps revision information so that previous versions can be restored.

---

# 4. Deployment API Version

The standard API version is:

```yaml
apiVersion: apps/v1
```

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
```

Deployments belong to the Kubernetes `apps` API group.

---

# 5. Basic Deployment YAML

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

Apply it:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get deployments
```

---

# 6. Understanding the Deployment YAML

Let's break the YAML down.

## apiVersion

```yaml
apiVersion: apps/v1
```

Specifies the Kubernetes API group and version.

---

## kind

```yaml
kind: Deployment
```

Specifies that the resource is a Deployment.

---

## metadata

```yaml
metadata:
  name: nginx-deployment
```

Defines the name of the Deployment.

---

## replicas

```yaml
spec:
  replicas: 3
```

Defines the desired number of Pods.

```text
replicas: 3
        ↓
Kubernetes tries to maintain
        ↓
3 Pods
```

---

## selector

```yaml
selector:
  matchLabels:
    app: nginx
```

The selector identifies the Pods managed by the Deployment's ReplicaSet.

---

## template

```yaml
template:
  metadata:
    labels:
      app: nginx
```

Defines the Pod template.

The labels must match the Deployment selector.

---

# 7. Deployment and Pod Template

The Pod template is extremely important.

Example:

```yaml
template:
  metadata:
    labels:
      app: nginx

  spec:
    containers:
      - name: nginx
        image: nginx:1.27
```

The Deployment uses this template when creating Pods.

Conceptually:

```text
Deployment
    |
    ↓
Pod Template
    |
    ↓
ReplicaSet
    |
    ↓
Pods
```

---

# 8. Deployment Selector

Example:

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

These values must be compatible.

```text
Deployment Selector
        |
        | app=nginx
        ↓
Pod Labels
        |
        | app=nginx
        ↓
Matching Pods
```

A common mistake is creating a selector that does not match the Pod template labels.

---

# 9. Create a Deployment

Using YAML:

```bash
kubectl apply -f deployment.yaml
```

Verify:

```bash
kubectl get deployment
```

Example:

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           30s
```

---

# 10. Create a Deployment Imperatively

You can also create a Deployment using:

```bash
kubectl create deployment nginx \
  --image=nginx:1.27
```

Specify replicas:

```bash
kubectl create deployment nginx \
  --image=nginx:1.27 \
  --replicas=3
```

Verify:

```bash
kubectl get deployment
```

---

# 11. List Deployments

```bash
kubectl get deployments
```

Short form:

```bash
kubectl get deploy
```

For all namespaces:

```bash
kubectl get deployments -A
```

---

# 12. Describe a Deployment

```bash
kubectl describe deployment nginx-deployment
```

This provides information about:

* ReplicaSets
* Desired replicas
* Available replicas
* Deployment strategy
* Conditions
* Events
* Pod template
* Container image

---

# 13. Get Deployment YAML

```bash
kubectl get deployment nginx-deployment -o yaml
```

This is useful for understanding the actual resource stored in Kubernetes.

---

# 14. Get Deployment as JSON

```bash
kubectl get deployment nginx-deployment -o json
```

Useful when working with:

* Scripts
* JSONPath
* Automation
* Kubernetes APIs

---

# 15. Understand Deployment Status

Run:

```bash
kubectl get deployment
```

Example:

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           5m
```

## READY

Example:

```text
3/3
```

Means three Pods are Ready out of three desired replicas.

## UP-TO-DATE

Indicates how many Pods are using the current Pod template.

## AVAILABLE

Indicates how many Pods are considered available according to Deployment availability rules.

---

# 16. Deployment Creates a ReplicaSet

When you create:

```yaml
kind: Deployment
```

Kubernetes creates a ReplicaSet.

Check:

```bash
kubectl get rs
```

Example:

```text
NAME                          DESIRED   CURRENT   READY
nginx-deployment-6f8b9d7c9f   3         3         3
```

Then:

```bash
kubectl get pods
```

Example:

```text
nginx-deployment-6f8b9d7c9f-abcde
nginx-deployment-6f8b9d7c9f-fghij
nginx-deployment-6f8b9d7c9f-klmno
```

---

# 17. Complete Ownership Hierarchy

The complete relationship is:

```text
Deployment
     |
     ↓
ReplicaSet
     |
     ↓
Pod
     |
     ↓
Container
```

Each layer has a different responsibility.

| Resource   | Responsibility                 |
| ---------- | ------------------------------ |
| Deployment | Application release management |
| ReplicaSet | Maintain desired Pod count     |
| Pod        | Run one or more containers     |
| Container  | Run application process        |

---

# 18. Scaling Deployments

A Deployment can be scaled easily.

Scale to five replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Check:

```bash
kubectl get deployment
```

Then:

```bash
kubectl get pods
```

---

# 19. Scaling Down

Scale to two:

```bash
kubectl scale deployment nginx-deployment --replicas=2
```

Verify:

```bash
kubectl get pods
```

Kubernetes removes unnecessary Pods until the desired state is reached.

---

# 20. Declarative Scaling

Instead of using:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

you can update:

```yaml
spec:
  replicas: 5
```

Then:

```bash
kubectl apply -f deployment.yaml
```

This is the declarative Kubernetes approach.

---

# 21. Deployment Reconciliation

Kubernetes continuously tries to make the actual state match the desired state.

Example:

```text
Desired:
5 Pods

Current:
3 Pods
```

Deployment/ReplicaSet controllers work toward:

```text
Current:
5 Pods
```

Another example:

```text
Desired:
2 Pods

Current:
5 Pods
```

The controllers reduce the number of Pods.

---

# 22. Self-Healing

Suppose a Deployment manages three Pods:

```text
Pod A
Pod B
Pod C
```

Delete Pod B:

```bash
kubectl delete pod <pod-name>
```

The ReplicaSet notices:

```text
Desired = 3
Current = 2
```

It creates a replacement.

```text
Pod A
Pod B → deleted
Pod C

        ↓

Pod A
Pod C
Pod D → replacement
```

The Deployment itself is not directly creating the replacement Pod; its managed ReplicaSet maintains the desired count.

---

# 23. Updating a Deployment

Suppose the Deployment uses:

```yaml
image: nginx:1.27
```

Change it to:

```yaml
image: nginx:1.28
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Or use:

```bash
kubectl set image deployment/nginx-deployment \
  nginx=nginx:1.28
```

Kubernetes detects that the Pod template has changed.

---

# 24. What Happens During an Update?

The sequence is approximately:

```text
Deployment
     |
     ↓
Pod Template Changes
     |
     ↓
New ReplicaSet Created
     |
     ↓
New Pods Created
     |
     ↓
New Pods Become Ready
     |
     ↓
Old Pods Scaled Down
```

This is called a **rolling update**.

---

# 25. Rolling Updates

The default Deployment strategy is:

```yaml
strategy:
  type: RollingUpdate
```

The purpose is to replace old Pods gradually rather than terminating everything at once.

Example:

```text
Old Version
3 Pods

    ↓

Old: 2
New: 1

    ↓

Old: 1
New: 2

    ↓

Old: 0
New: 3
```

---

# 26. Monitor a Rollout

Use:

```bash
kubectl rollout status deployment/nginx-deployment
```

Successful rollout:

```text
deployment "nginx-deployment" successfully rolled out
```

You can also watch Pods:

```bash
kubectl get pods -w
```

---

# 27. Deployment Revision

When the Pod template changes, the Deployment can create a new revision.

Example:

```text
Revision 1
nginx:1.25

        ↓

Revision 2
nginx:1.26

        ↓

Revision 3
nginx:1.27
```

Each revision represents a different Deployment state.

---

# 28. View Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

Example:

```text
deployment.apps/nginx-deployment

REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

---

# 29. Inspect a Revision

```bash
kubectl rollout history deployment/nginx-deployment \
  --revision=2
```

This can help identify what changed in a previous revision.

---

# 30. Rollback

If a new release causes problems, rollback to the previous revision:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Check:

```bash
kubectl rollout status deployment/nginx-deployment
```

---

# 31. Rollback to a Specific Revision

List revisions:

```bash
kubectl rollout history deployment/nginx-deployment
```

Rollback:

```bash
kubectl rollout undo deployment/nginx-deployment \
  --to-revision=2
```

Verify:

```bash
kubectl rollout status deployment/nginx-deployment
```

---

# 32. Deployment Rollback Example

Suppose:

```text
Revision 1 → nginx:1.25
Revision 2 → nginx:1.26
Revision 3 → nginx:1.27
```

Revision 3 is broken.

Run:

```bash
kubectl rollout undo deployment/nginx-deployment \
  --to-revision=2
```

Result:

```text
Revision 2
nginx:1.26
```

The Deployment returns to the previous Pod template.

---

# 33. RollingUpdate Strategy Configuration

Example:

```yaml
strategy:
  type: RollingUpdate

  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

These two fields control the behavior of the rollout.

---

# 34. maxSurge

`maxSurge` controls how many additional Pods can be created above the desired number of replicas during a rolling update.

Example:

```yaml
replicas: 4

strategy:
  rollingUpdate:
    maxSurge: 1
```

The Deployment can temporarily have:

```text
4 desired Pods
+
1 extra Pod

= 5 Pods
```

`maxSurge` can be a number or percentage.

Example:

```yaml
maxSurge: 2
```

or:

```yaml
maxSurge: 25%
```

---

# 35. maxUnavailable

`maxUnavailable` controls how many Pods can be unavailable during a rolling update.

Example:

```yaml
replicas: 4

strategy:
  rollingUpdate:
    maxUnavailable: 1
```

The rollout can temporarily tolerate one unavailable Pod.

It can also be expressed as a percentage:

```yaml
maxUnavailable: 25%
```

---

# 36. maxSurge and maxUnavailable Together

Consider:

```yaml
replicas: 5

strategy:
  type: RollingUpdate

  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

The goal is:

```text
Desired = 5

Maximum temporary Pods = 6

Intentionally unavailable = 0
```

This configuration can help maintain availability during an update, assuming Pods become Ready correctly and sufficient cluster capacity exists.

---

# 37. Percentage Values

Both can use percentages.

Example:

```yaml
rollingUpdate:
  maxSurge: 25%
  maxUnavailable: 25%
```

Kubernetes calculates the corresponding Pod counts based on the desired replica count.

---

# 38. Recreate Strategy

A Deployment can use:

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

Unlike a rolling update, the old and new versions do not normally run simultaneously during the transition.

---

# 39. RollingUpdate vs Recreate

## RollingUpdate

```text
Old + New
   ↓
Gradual replacement
   ↓
New only
```

Advantages:

* Reduced downtime
* Gradual rollout
* Better availability
* Can be monitored during rollout

## Recreate

```text
Old
 ↓
Nothing
 ↓
New
```

Advantages:

* Ensures old and new versions do not overlap
* Useful when versions cannot safely coexist

Disadvantage:

* Can cause downtime

---

# 40. Readiness Probes and Deployments

A readiness probe determines whether a Pod is ready to receive traffic.

Example:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80

  initialDelaySeconds: 5
  periodSeconds: 5
```

During a rolling update, readiness is especially important.

Concept:

```text
New Pod Created
      ↓
Readiness Probe
      ↓
Not Ready
      ↓
Deployment waits
      ↓
Ready
      ↓
Continue rollout
```

---

# 41. Liveness Probe

A liveness probe determines whether a container is still functioning correctly.

Example:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 80

  initialDelaySeconds: 10
  periodSeconds: 10
```

Remember:

```text
Readiness
→ Should this Pod receive traffic?

Liveness
→ Should this container be restarted?
```

---

# 42. Startup Probe

For slow-starting applications:

```yaml
startupProbe:
  httpGet:
    path: /health
    port: 80

  failureThreshold: 30
  periodSeconds: 10
```

Startup probes are useful when an application needs significant time to initialize.

---

# 43. minReadySeconds

Example:

```yaml
spec:
  minReadySeconds: 30
```

This means a newly Ready Pod must remain Ready for at least 30 seconds before it is considered available by the Deployment.

This can help avoid considering a Pod available immediately after it first becomes Ready.

---

# 44. progressDeadlineSeconds

Example:

```yaml
spec:
  progressDeadlineSeconds: 600
```

This specifies how long Kubernetes waits for Deployment progress before reporting that the Deployment has failed to make progress.

Check Deployment conditions:

```bash
kubectl describe deployment <name>
```

---

# 45. Deployment Conditions

Run:

```bash
kubectl describe deployment nginx-deployment
```

Important conditions include:

```text
Available
Progressing
ReplicaFailure
```

These help determine the state of the Deployment.

---

# 46. Deployment Status

Useful status information includes:

```yaml
status:
  replicas:
  updatedReplicas:
  readyReplicas:
  availableReplicas:
  unavailableReplicas:
```

For example:

```text
replicas: 5
updatedReplicas: 5
readyReplicas: 5
availableReplicas: 5
```

This generally indicates a healthy Deployment.

---

# 47. Pause a Deployment

You can pause a rollout:

```bash
kubectl rollout pause deployment/nginx-deployment
```

This is useful when making multiple changes that you want to apply as part of a controlled rollout.

---

# 48. Resume a Deployment

Resume:

```bash
kubectl rollout resume deployment/nginx-deployment
```

Check:

```bash
kubectl rollout status deployment/nginx-deployment
```

---

# 49. Restart a Deployment

To restart the Pods:

```bash
kubectl rollout restart deployment/nginx-deployment
```

Monitor:

```bash
kubectl rollout status deployment/nginx-deployment
```

This is useful when you want to recreate Pods without changing the application image.

---

# 50. Deployment Revision History

Deployment revision history is maintained through ReplicaSets.

Example:

```text
Deployment
    |
    +── ReplicaSet v1
    |
    +── ReplicaSet v2
    |
    +── ReplicaSet v3
```

Older ReplicaSets can be retained so that Kubernetes can perform rollbacks.

---

# 51. revisionHistoryLimit

Example:

```yaml
spec:
  revisionHistoryLimit: 5
```

This controls how many old ReplicaSets are retained.

A smaller value reduces retained history.

A larger value provides more previous revisions for rollback.

---

# 52. Changing Replicas vs Changing Pod Template

This is an important interview concept.

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

changes only the desired Pod count.

It does **not** change the Pod template.

Therefore, it does not normally create a new Deployment revision.

---

# 53. Changing the Image

Changing:

```yaml
image: nginx:1.27
```

to:

```yaml
image: nginx:1.28
```

changes the Pod template.

Therefore:

```text
Pod Template Change
       ↓
New ReplicaSet
       ↓
New Deployment Revision
       ↓
Rolling Update
```

---

# 54. Other Pod Template Changes

Changes to the following can result in a new Deployment revision:

* Container image
* Environment variables
* Container command
* Container arguments
* Resource requests
* Resource limits
* Volumes
* Volume mounts
* Pod labels
* Pod annotations
* Probes

The general rule is:

> Changes to the Deployment's Pod template can create a new revision.

---

# 55. Deployment Self-Healing

Suppose:

```text
replicas: 3
```

Current Pods:

```text
Pod A
Pod B
Pod C
```

Delete Pod B:

```bash
kubectl delete pod <pod-name>
```

The ReplicaSet notices:

```text
Desired = 3
Current = 2
```

It creates a replacement.

This demonstrates reconciliation and self-healing.

---

# 56. Deployment Does Not Directly Schedule Pods

The Deployment manages the application state through ReplicaSets.

The Kubernetes scheduler decides where Pods should run.

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Scheduler
    ↓
Node
```

The scheduler considers factors such as:

* Available resources
* Node selectors
* Affinity
* Taints and tolerations
* Topology constraints

---

# 57. Deployment and Services

A Deployment creates and manages Pods, but it does not provide stable networking by itself.

For stable application access, a Service is commonly used.

```text
             Service
                |
                ↓
          Deployment Pods
          ├── Pod
          ├── Pod
          └── Pod
```

Example Service selector:

```yaml
selector:
  app: nginx
```

Deployment Pod labels:

```yaml
labels:
  app: nginx
```

The Service selects the Pods using labels.

---

# 58. Deployment with Service

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web

spec:
  replicas: 3

  selector:
    matchLabels:
      app: web

  template:
    metadata:
      labels:
        app: web

    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```

Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service

spec:
  selector:
    app: web

  ports:
    - port: 80
      targetPort: 80
```

Architecture:

```text
Client
  |
  ↓
Service
  |
  +---- Pod
  +---- Pod
  +---- Pod
```

---

# 59. Deployment with Environment Variables

Example:

```yaml
containers:
  - name: app
    image: nginx:1.27

    env:
      - name: APP_ENV
        value: production

      - name: LOG_LEVEL
        value: info
```

Changing these values changes the Pod template and can trigger a new Deployment revision.

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

Sensitive information should not be hard-coded directly into Deployment manifests.

---

# 62. Deployment with Resource Requests and Limits

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

Complete example:

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

Resource requests are important for scheduling.

---

# 63. Deployment Troubleshooting

When a Deployment is not working, follow a systematic process.

## Step 1: Check Deployment

```bash
kubectl get deployment
```

## Step 2: Describe Deployment

```bash
kubectl describe deployment <name>
```

## Step 3: Check rollout

```bash
kubectl rollout status deployment/<name>
```

## Step 4: Check ReplicaSets

```bash
kubectl get rs
```

## Step 5: Check Pods

```bash
kubectl get pods
```

## Step 6: Describe failing Pod

```bash
kubectl describe pod <pod-name>
```

## Step 7: Check logs

```bash
kubectl logs <pod-name>
```

## Step 8: Check events

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

# 64. Common Deployment Problems

## ImagePullBackOff

Possible causes:

* Incorrect image name
* Incorrect image tag
* Private registry authentication problem
* Registry unavailable

Check:

```bash
kubectl describe pod <pod-name>
```

---

## CrashLoopBackOff

Possible causes:

* Application crashes
* Incorrect command
* Missing configuration
* Missing environment variables
* Dependency failure

Check:

```bash
kubectl logs <pod-name>
```

---

## Pending

Possible causes:

* Insufficient CPU
* Insufficient memory
* Node selector mismatch
* Taints/tolerations
* Affinity constraints
* PVC issues

Check:

```bash
kubectl describe pod <pod-name>
```

---

## Readiness Probe Failure

Check:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
Readiness probe failed
```

Then inspect:

```bash
kubectl logs <pod-name>
```

---

# 65. Stuck Rollout

A Deployment may become stuck when new Pods cannot become available.

Check:

```bash
kubectl rollout status deployment/<name>
```

Then:

```bash
kubectl get pods
```

And:

```bash
kubectl describe pod <pod-name>
```

Possible causes include:

* Invalid image
* Application crash
* Readiness failure
* Insufficient resources
* Scheduling problems
* Configuration errors

---

# 66. ProgressDeadlineExceeded

A Deployment may report:

```text
ProgressDeadlineExceeded
```

This indicates that the Deployment did not make sufficient progress within `progressDeadlineSeconds`.

Investigate:

```bash
kubectl describe deployment <name>
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.lastTimestamp
```

---

# 67. Invalid Image Example

Suppose the Deployment currently uses:

```yaml
image: nginx:1.27
```

Change it to:

```yaml
image: nginx:this-image-does-not-exist
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get pods
```

You may see:

```text
ImagePullBackOff
```

Inspect:

```bash
kubectl describe pod <pod-name>
```

Rollback:

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

# 68. Deployment Rollout Failure Recovery

A typical recovery process:

```text
New Version Deployed
        ↓
Rollout Fails
        ↓
Check rollout status
        ↓
Inspect Pods
        ↓
Inspect logs/events
        ↓
Identify problem
        ↓
Rollback
        ↓
Verify healthy version
```

Commands:

```bash
kubectl rollout status deployment/<name>

kubectl get pods

kubectl describe pod <pod-name>

kubectl logs <pod-name>

kubectl rollout undo deployment/<name>

kubectl rollout status deployment/<name>
```

---

# 69. Deployment vs ReplicaSet

| Feature                 | Deployment          | ReplicaSet                              |
| ----------------------- | ------------------- | --------------------------------------- |
| Maintains Pods          | Through ReplicaSets | Yes                                     |
| Scaling                 | Yes                 | Yes                                     |
| Rolling updates         | Yes                 | No                                      |
| Rollback                | Yes                 | No                                      |
| Revision history        | Yes                 | Not as an application release mechanism |
| Manages ReplicaSets     | Yes                 | No                                      |
| Common production usage | Yes                 | Usually indirectly                      |

Remember:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

---

# 70. Deployment vs StatefulSet

| Deployment                        | StatefulSet                            |
| --------------------------------- | -------------------------------------- |
| Generally for stateless workloads | Generally for stateful workloads       |
| Pods are interchangeable          | Pods have stable identities            |
| No stable Pod identity            | Stable Pod identity                    |
| Common for web applications       | Common for databases/clustered systems |
| Updates are deployment-oriented   | Updates preserve StatefulSet identity  |

---

# 71. Deployment vs DaemonSet

| Deployment                    | DaemonSet                 |
| ----------------------------- | ------------------------- |
| Desired replica count         | Pod on each eligible Node |
| Application workloads         | Node-level workloads      |
| Multiple interchangeable Pods | Usually one Pod per Node  |
| Example: web server           | Example: log collector    |

---

# 72. Deployment Best Practices

## Use Deployments for Stateless Applications

Examples:

* Web servers
* REST APIs
* Frontend applications
* Microservices
* Workers

---

## Use Specific Image Tags

Prefer:

```yaml
image: nginx:1.27
```

over:

```yaml
image: nginx:latest
```

Specific versions make deployments more predictable.

---

## Configure Readiness Probes

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
```

This helps Kubernetes determine when a new Pod is ready.

---

## Configure Resource Requests

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
```

This helps the scheduler make appropriate placement decisions.

---

## Use Rolling Updates Carefully

For highly available applications:

```yaml
rollingUpdate:
  maxUnavailable: 0
  maxSurge: 1
```

Choose values based on application requirements and cluster capacity.

---

# 73. Deployment with Labels

A good labeling strategy makes resources easier to manage.

Example:

```yaml
metadata:
  labels:
    app: payment
    component: backend
    environment: production
```

Pod template:

```yaml
template:
  metadata:
    labels:
      app: payment
      component: backend
      environment: production
```

You can then query:

```bash
kubectl get pods -l app=payment
```

---

# 74. Deployment in a Namespace

Create:

```bash
kubectl apply -f deployment.yaml -n production
```

Check:

```bash
kubectl get deployment -n production
```

Describe:

```bash
kubectl describe deployment <name> -n production
```

---

# 75. JSONPath with Deployments

Get the current image:

```bash
kubectl get deployment nginx-deployment \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

Get desired replicas:

```bash
kubectl get deployment nginx-deployment \
  -o jsonpath='{.spec.replicas}'
```

Get available replicas:

```bash
kubectl get deployment nginx-deployment \
  -o jsonpath='{.status.availableReplicas}'
```

---

# 76. Useful Deployment Commands

## Create

```bash
kubectl apply -f deployment.yaml
```

## List

```bash
kubectl get deployments
```

## Describe

```bash
kubectl describe deployment <name>
```

## Scale

```bash
kubectl scale deployment <name> --replicas=5
```

## Update image

```bash
kubectl set image deployment/<name> \
  <container>=<image>:<tag>
```

## Rollout status

```bash
kubectl rollout status deployment/<name>
```

## Rollout history

```bash
kubectl rollout history deployment/<name>
```

## Rollback

```bash
kubectl rollout undo deployment/<name>
```

## Specific rollback

```bash
kubectl rollout undo deployment/<name> \
  --to-revision=2
```

## Restart

```bash
kubectl rollout restart deployment/<name>
```

## Pause

```bash
kubectl rollout pause deployment/<name>
```

## Resume

```bash
kubectl rollout resume deployment/<name>
```

## Delete

```bash
kubectl delete deployment <name>
```

---

# 77. Deployment Workflow

The typical production workflow looks like:

```text
Developer builds application
          ↓
Container image created
          ↓
Image pushed to registry
          ↓
Deployment manifest updated
          ↓
kubectl apply
          ↓
New ReplicaSet created
          ↓
Rolling update
          ↓
New Pods created
          ↓
Readiness checks
          ↓
New Pods become available
          ↓
Old Pods removed
          ↓
Deployment completed
```

---

# 78. Complete Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: web-app

  labels:
    app: web-app

spec:
  replicas: 3

  revisionHistoryLimit: 5

  minReadySeconds: 10

  progressDeadlineSeconds: 600

  strategy:
    type: RollingUpdate

    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0

  selector:
    matchLabels:
      app: web-app

  template:
    metadata:
      labels:
        app: web-app

    spec:
      containers:
        - name: web
          image: nginx:1.27

          ports:
            - containerPort: 80

          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"

            limits:
              cpu: "500m"
              memory: "256Mi"

          readinessProbe:
            httpGet:
              path: /
              port: 80

            initialDelaySeconds: 5
            periodSeconds: 5

          livenessProbe:
            httpGet:
              path: /
              port: 80

            initialDelaySeconds: 10
            periodSeconds: 10
```

---

# 79. Deployment Lifecycle

A Deployment lifecycle can be understood as:

```text
Create
  ↓
Scale
  ↓
Update
  ↓
Rolling Update
  ↓
Monitor
  ↓
Successful
```

If the update fails:

```text
Update
  ↓
Failure
  ↓
Troubleshoot
  ↓
Rollback
  ↓
Previous Version
```

---

# 80. Important Interview Questions

## Beginner

### 1. What is a Kubernetes Deployment?

A Deployment is a Kubernetes controller used to declaratively manage application Pods and ReplicaSets.

### 2. What does a Deployment manage?

A Deployment manages ReplicaSets.

### 3. What does a ReplicaSet manage?

A ReplicaSet manages Pods.

### 4. What is the default Deployment strategy?

`RollingUpdate`.

### 5. How do you create a Deployment?

```bash
kubectl apply -f deployment.yaml
```

### 6. How do you scale a Deployment?

```bash
kubectl scale deployment <name> --replicas=5
```

---

# 81. Intermediate Interview Questions

### 7. What happens when you change the image in a Deployment?

The Pod template changes, causing a new Deployment revision and ReplicaSet. Kubernetes then performs a rollout according to the Deployment strategy.

### 8. What is a rolling update?

A rolling update gradually replaces old Pods with new Pods.

### 9. What is `maxSurge`?

It controls the number of additional Pods that can temporarily be created above the desired replica count.

### 10. What is `maxUnavailable`?

It controls how many Pods can be unavailable during a rolling update.

### 11. What is the difference between RollingUpdate and Recreate?

RollingUpdate gradually replaces Pods, while Recreate terminates old Pods before creating new ones.

### 12. How do you check rollout status?

```bash
kubectl rollout status deployment/<name>
```

### 13. How do you rollback?

```bash
kubectl rollout undo deployment/<name>
```

---

# 82. Advanced Interview Questions

### 14. Why does a Deployment create a new ReplicaSet during an update?

Because the Pod template has changed. The Deployment needs a separate ReplicaSet to manage Pods using the new Pod template while retaining the old ReplicaSet for rollout and rollback purposes.

### 15. Does changing only `replicas` create a new ReplicaSet?

Normally, no. Changing replicas does not modify the Pod template.

### 16. What happens if a new Pod fails its readiness probe?

The Pod is not considered Ready/available, and the Deployment may wait before progressing further with the rollout.

### 17. What happens if a new image cannot be pulled?

The new Pods may enter `ErrImagePull` or `ImagePullBackOff`, causing the rollout to become unhealthy or stuck.

### 18. How would you troubleshoot a stuck Deployment?

Use:

```bash
kubectl get deployment
kubectl describe deployment <name>
kubectl rollout status deployment/<name>
kubectl get rs
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.lastTimestamp
```

### 19. What is `progressDeadlineSeconds`?

It defines how long Kubernetes waits for Deployment progress before reporting that the Deployment has failed to make progress.

### 20. What is `revisionHistoryLimit`?

It controls how many old ReplicaSets are retained for Deployment revision history.

---

# 83. Scenario-Based Interview Questions

## Scenario 1: New Pods Are Not Starting

You deploy a new image and see:

```text
ImagePullBackOff
```

What do you do?

```text
kubectl describe pod <pod>
```

Check:

* Image name
* Image tag
* Registry access
* ImagePullSecrets

If necessary:

```bash
kubectl rollout undo deployment/<name>
```

---

## Scenario 2: Deployment Is Stuck

You run:

```bash
kubectl rollout status deployment/app
```

and the rollout never completes.

Investigate:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl describe deployment app
kubectl get events --sort-by=.lastTimestamp
```

Common causes:

* Readiness failure
* Application crash
* Invalid image
* Insufficient resources
* Scheduling problems

---

## Scenario 3: Application Version Is Broken

Current versions:

```text
v1 → working
v2 → working
v3 → broken
```

Rollback:

```bash
kubectl rollout undo deployment/app
```

Or:

```bash
kubectl rollout undo deployment/app --to-revision=2
```

---

# 84. Key Concepts to Memorize

```text
Deployment
    ↓
Manages ReplicaSets
    ↓
ReplicaSet
    ↓
Maintains Pods
    ↓
Pods
    ↓
Run Containers
```

Deployment provides:

```text
✓ Declarative management
✓ Scaling
✓ Rolling updates
✓ Rollbacks
✓ Revision history
✓ Rollout monitoring
```

---

# 85. Deployment Mental Model

Think of a Deployment as an **application release manager**.

The Deployment answers:

> What version of my application should be running, how many replicas should exist, and how should Kubernetes move from the old version to the new version?

The ReplicaSet answers:

> How many Pods matching this particular Pod template should be running?

The Pod answers:

> Run these containers.

Therefore:

```text
Deployment
  = Release Management

ReplicaSet
  = Replica Management

Pod
  = Application Execution
```

---

# 86. Final Summary

A Kubernetes Deployment is a higher-level workload controller used to manage application releases.

The key hierarchy is:

```text
Deployment
      ↓
ReplicaSet
      ↓
Pods
      ↓
Containers
```

The most important Deployment features are:

```text
Rolling Updates
       +
Scaling
       +
Revision History
       +
Rollback
       +
Self-Healing
```

The most important commands are:

```bash
kubectl get deployment
kubectl describe deployment <name>

kubectl scale deployment <name> --replicas=5

kubectl set image deployment/<name> \
  <container>=<image>:<tag>

kubectl rollout status deployment/<name>

kubectl rollout history deployment/<name>

kubectl rollout undo deployment/<name>

kubectl rollout undo deployment/<name> \
  --to-revision=2

kubectl rollout restart deployment/<name>

kubectl rollout pause deployment/<name>

kubectl rollout resume deployment/<name>
```

The most important update concept is:

```text
Pod Template Changes
        ↓
New ReplicaSet
        ↓
New Deployment Revision
        ↓
Rolling Update
        ↓
New Pods Become Ready
        ↓
Old Pods Scaled Down
```

> **Key takeaway:** A Deployment is the standard Kubernetes abstraction for managing stateless application releases. It uses ReplicaSets to maintain Pods and provides rolling updates, scaling, revision history, and rollback capabilities.


# Deployments – Labs

This lab covers practical exercises for understanding and working with **Kubernetes Deployments**.

The labs progress from basic Deployment creation to scaling, rolling updates, rollbacks, rollout strategies, troubleshooting, and advanced scenarios.

---

# Prerequisites

Make sure you have:

* A running Kubernetes cluster
* `kubectl` installed and configured
* Basic knowledge of Pods
* Basic knowledge of ReplicaSets
* Familiarity with YAML

Verify the cluster:

```bash
kubectl cluster-info
```

Check the nodes:

```bash
kubectl get nodes
```

---

# Lab 1: Create a Basic Deployment

## Objective

Create a Deployment with three nginx Pods.

## Step 1: Create the YAML file

Create:

```text
deployment.yaml
```

Add:

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

## Step 2: Apply the Deployment

```bash
kubectl apply -f deployment.yaml
```

Expected:

```text
deployment.apps/nginx-deployment created
```

## Step 3: Check the Deployment

```bash
kubectl get deployments
```

Expected:

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           ...
```

## Step 4: Check the ReplicaSet

```bash
kubectl get rs
```

You should see a ReplicaSet created by the Deployment.

## Step 5: Check the Pods

```bash
kubectl get pods
```

Expected:

```text
nginx-deployment-xxxxx
nginx-deployment-yyyyy
nginx-deployment-zzzzz
```

---

# Lab 2: Understand the Deployment Hierarchy

## Objective

Understand the relationship between Deployment, ReplicaSet, and Pods.

Run:

```bash
kubectl get deployment
kubectl get rs
kubectl get pods
```

The relationship should look like:

```text
Deployment
    |
    └── ReplicaSet
          |
          ├── Pod
          ├── Pod
          └── Pod
```

Get the Deployment details:

```bash
kubectl describe deployment nginx-deployment
```

Look for the ReplicaSet information.

---

# Lab 3: Inspect Deployment YAML

## Objective

Understand the Deployment's desired state.

Run:

```bash
kubectl get deployment nginx-deployment -o yaml
```

Important sections:

```yaml
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
```

The important concept is:

```text
Deployment
    ↓
Pod Template
    ↓
ReplicaSet
    ↓
Pods
```

---

# Lab 4: Scale a Deployment

## Objective

Scale the Deployment from three replicas to five.

Run:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Check:

```bash
kubectl get deployment
```

Expected:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   5/5     5            5
```

Check Pods:

```bash
kubectl get pods
```

You should now have five Pods.

---

# Lab 5: Scale Down a Deployment

## Objective

Scale the Deployment from five replicas to two.

Run:

```bash
kubectl scale deployment nginx-deployment --replicas=2
```

Verify:

```bash
kubectl get deployment
kubectl get pods
```

Expected:

```text
READY
2/2
```

---

# Lab 6: Scale Using YAML

## Objective

Change the desired replica count declaratively.

Edit:

```text
deployment.yaml
```

Change:

```yaml
replicas: 3
```

to:

```yaml
replicas: 4
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Verify:

```bash
kubectl get deployment
kubectl get pods
```

The Deployment should maintain four replicas.

---

# Lab 7: Perform a Rolling Update

## Objective

Update the nginx image from version `1.25` to `1.26`.

First check the current image:

```bash
kubectl get deployment nginx-deployment -o wide
```

Update the image:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.26
```

Check the rollout:

```bash
kubectl rollout status deployment/nginx-deployment
```

Expected:

```text
deployment "nginx-deployment" successfully rolled out
```

---

# Lab 8: Watch a Rolling Update

## Objective

Observe how Kubernetes replaces old Pods with new Pods.

Run:

```bash
kubectl get pods -w
```

In another terminal, update the image:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.27
```

Observe the Pods.

You may see:

```text
Old Pods
   ↓
New Pods created
   ↓
New Pods become Ready
   ↓
Old Pods terminated
```

The Deployment creates a new ReplicaSet for the new Pod template.

---

# Lab 9: Observe ReplicaSets During a Rolling Update

Before updating:

```bash
kubectl get rs
```

Update the image:

```bash
kubectl set image deployment/nginx-deployment nginx=1.27
```

Then:

```bash
kubectl get rs
```

You should now see multiple ReplicaSets.

Example:

```text
NAME                         DESIRED   CURRENT   READY
nginx-deployment-old         0         0         0
nginx-deployment-new         3         3         3
```

The Deployment retains the old ReplicaSet for revision history.

---

# Lab 10: Check Rollout History

## Objective

View Deployment revisions.

Run:

```bash
kubectl rollout history deployment/nginx-deployment
```

You may see:

```text
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

To inspect a specific revision:

```bash
kubectl rollout history deployment/nginx-deployment --revision=2
```

---

# Lab 11: Add a Change Cause

## Objective

Make rollout history easier to understand.

You can annotate a Deployment before or after a change:

```bash
kubectl annotate deployment nginx-deployment \
  kubernetes.io/change-cause="Updated nginx to 1.27"
```

Then check:

```bash
kubectl rollout history deployment/nginx-deployment
```

A meaningful change cause makes deployment history easier to understand.

---

# Lab 12: Rollback a Deployment

## Objective

Rollback the Deployment to the previous revision.

First check history:

```bash
kubectl rollout history deployment/nginx-deployment
```

Rollback:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Check the rollout:

```bash
kubectl rollout status deployment/nginx-deployment
```

Verify the image:

```bash
kubectl get deployment nginx-deployment \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

---

# Lab 13: Rollback to a Specific Revision

## Objective

Rollback to a specific Deployment revision.

First:

```bash
kubectl rollout history deployment/nginx-deployment
```

Suppose you want revision 2:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

Check:

```bash
kubectl rollout status deployment/nginx-deployment
```

Then:

```bash
kubectl get pods
```

---

# Lab 14: Pause a Deployment

## Objective

Understand how to pause a Deployment rollout.

Pause:

```bash
kubectl rollout pause deployment/nginx-deployment
```

Check:

```bash
kubectl rollout status deployment/nginx-deployment
```

While paused, you can make additional Pod-template changes without immediately progressing the rollout.

---

# Lab 15: Resume a Deployment

Resume the paused Deployment:

```bash
kubectl rollout resume deployment/nginx-deployment
```

Check:

```bash
kubectl rollout status deployment/nginx-deployment
```

The Deployment should continue progressing.

---

# Lab 16: Restart a Deployment

## Objective

Restart all Pods managed by a Deployment without changing the application image.

Run:

```bash
kubectl rollout restart deployment/nginx-deployment
```

Check:

```bash
kubectl rollout status deployment/nginx-deployment
```

Watch the Pods:

```bash
kubectl get pods -w
```

This is commonly useful when you need to recreate Pods after configuration or external changes.

---

# Lab 17: Configure RollingUpdate

## Objective

Explicitly configure the rolling-update strategy.

Update `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 4

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0

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

Apply:

```bash
kubectl apply -f deployment.yaml
```

Verify:

```bash
kubectl get deployment nginx-deployment -o yaml
```

---

# Lab 18: Understand maxSurge

## Objective

Understand how `maxSurge` affects a rolling update.

Given:

```yaml
replicas: 4

strategy:
  rollingUpdate:
    maxSurge: 1
```

The Deployment can temporarily create one additional Pod during the rollout.

Conceptually:

```text
Desired replicas = 4

Maximum temporary Pods:
4 + 1 = 5
```

Observe the rollout:

```bash
kubectl get pods -w
```

Then trigger an update:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.28
```

---

# Lab 19: Understand maxUnavailable

## Objective

Understand how `maxUnavailable` controls availability during a rollout.

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
```

With four replicas:

```text
Desired replicas = 4
maxUnavailable = 1
```

Kubernetes can temporarily have one fewer available Pod during the rollout.

Conceptually:

```text
4 desired Pods
      ↓
1 may be unavailable
      ↓
At least approximately 3 available
```

The exact rollout behavior also depends on readiness and the other Deployment settings.

---

# Lab 20: RollingUpdate vs Recreate

## Objective

Understand the difference between the two Deployment strategies.

## RollingUpdate

```yaml
strategy:
  type: RollingUpdate
```

Pods are replaced gradually.

```text
Old Pods
   ↓
New Pods
   ↓
Old Pods removed
```

## Recreate

Change the Deployment to:

```yaml
strategy:
  type: Recreate
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Update the image:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.28
```

Watch:

```bash
kubectl get pods -w
```

With `Recreate`, existing Pods are terminated before new Pods are created.

---

# Lab 21: Configure a Readiness Probe

## Objective

Understand how readiness affects a Deployment rollout.

Create or update the Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-probe-deployment
spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx-probe

  template:
    metadata:
      labels:
        app: nginx-probe

    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80

          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
```

Apply:

```bash
kubectl apply -f deployment-probe.yaml
```

Check:

```bash
kubectl get pods
```

The Deployment waits for new Pods to become Ready before considering them available.

---

# Lab 22: Create a Deployment with a Broken Readiness Probe

## Objective

Understand what happens when new Pods never become Ready.

Create:

```text
broken-probe.yaml
```

Use:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-probe
spec:
  replicas: 3

  selector:
    matchLabels:
      app: broken-probe

  template:
    metadata:
      labels:
        app: broken-probe

    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80

          readinessProbe:
            httpGet:
              path: /does-not-exist
              port: 80
            initialDelaySeconds: 2
            periodSeconds: 5
```

Apply:

```bash
kubectl apply -f broken-probe.yaml
```

Check:

```bash
kubectl get pods
```

Then:

```bash
kubectl describe pod <pod-name>
```

Look for readiness probe failures.

Check:

```bash
kubectl rollout status deployment/broken-probe
```

The rollout may not complete successfully because the new Pods are not becoming Ready.

---

# Lab 23: Troubleshoot a Failed Deployment

## Scenario

A Deployment is stuck during rollout.

Run:

```bash
kubectl get deployment
```

Then:

```bash
kubectl rollout status deployment/<name>
```

Check ReplicaSets:

```bash
kubectl get rs
```

Check Pods:

```bash
kubectl get pods
```

Inspect the Deployment:

```bash
kubectl describe deployment <name>
```

Inspect the affected Pod:

```bash
kubectl describe pod <pod-name>
```

Check logs:

```bash
kubectl logs <pod-name>
```

Check previous container logs if necessary:

```bash
kubectl logs <pod-name> --previous
```

Check events:

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

# Lab 24: Test an Invalid Image Deployment

## Objective

Understand what happens when a Deployment references an invalid image.

Update the image:

```bash
kubectl set image deployment/nginx-deployment \
  nginx=nginx:this-image-does-not-exist
```

Check:

```bash
kubectl get pods
```

You may see:

```text
ImagePullBackOff
```

or:

```text
ErrImagePull
```

Inspect:

```bash
kubectl describe pod <pod-name>
```

Check rollout:

```bash
kubectl rollout status deployment/nginx-deployment
```

The rollout may become stuck because the new Pods cannot start successfully.

---

# Lab 25: Recover from a Failed Deployment

## Objective

Rollback after deploying a broken image.

First check:

```bash
kubectl rollout history deployment/nginx-deployment
```

Rollback:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Check:

```bash
kubectl rollout status deployment/nginx-deployment
```

Verify:

```bash
kubectl get pods
```

The Deployment should return to the previous working revision.

---

# Lab 26: Understand minReadySeconds

## Objective

Understand how `minReadySeconds` affects Deployment availability.

Add:

```yaml
spec:
  minReadySeconds: 30
```

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-minready
spec:
  replicas: 3

  minReadySeconds: 30

  selector:
    matchLabels:
      app: nginx-minready

  template:
    metadata:
      labels:
        app: nginx-minready

    spec:
      containers:
        - name: nginx
          image: nginx:1.27
```

Apply:

```bash
kubectl apply -f minready.yaml
```

The Pod must remain Ready for the configured period before being considered available.

---

# Lab 27: Understand progressDeadlineSeconds

## Objective

Understand how Kubernetes detects a stalled Deployment.

Example:

```yaml
spec:
  progressDeadlineSeconds: 60
```

This means the Deployment reports a stalled rollout if sufficient progress is not observed within the configured deadline.

Check:

```bash
kubectl describe deployment <deployment-name>
```

Look at the Conditions section.

---

# Lab 28: Compare Deployment and ReplicaSet

## Objective

Observe what additional functionality a Deployment provides.

Create a ReplicaSet:

```bash
kubectl get rs
```

Create a Deployment:

```bash
kubectl create deployment web --image=nginx:1.27 --replicas=3
```

Check:

```bash
kubectl get deployment
kubectl get rs
kubectl get pods
```

Now update the Deployment:

```bash
kubectl set image deployment/web nginx=nginx:1.28
```

Check ReplicaSets:

```bash
kubectl get rs
```

You should see a new ReplicaSet.

This demonstrates:

```text
Deployment
    ↓
Manages ReplicaSets
    ↓
ReplicaSets manage Pods
```

---

# Lab 29: Observe Deployment Revision History

## Objective

Create multiple revisions and inspect them.

Start with:

```bash
kubectl create deployment revision-demo --image=nginx:1.25 --replicas=3
```

Update:

```bash
kubectl set image deployment/revision-demo nginx=nginx:1.26
```

Update again:

```bash
kubectl set image deployment/revision-demo nginx=nginx:1.27
```

Check:

```bash
kubectl rollout history deployment/revision-demo
```

Check ReplicaSets:

```bash
kubectl get rs
```

You should see multiple ReplicaSets corresponding to Deployment revisions.

---

# Lab 30: Deployment with Environment Variables

## Objective

Understand how changes to the Pod template trigger a new Deployment revision.

Create:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: env-demo
spec:
  replicas: 2

  selector:
    matchLabels:
      app: env-demo

  template:
    metadata:
      labels:
        app: env-demo

    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          env:
            - name: APP_ENV
              value: "production"
```

Apply:

```bash
kubectl apply -f env-demo.yaml
```

Now change:

```yaml
value: "staging"
```

Apply again:

```bash
kubectl apply -f env-demo.yaml
```

Check:

```bash
kubectl get rs
```

A new ReplicaSet should be created because the Pod template changed.

---

# Lab 31: Changing Replicas vs Changing the Pod Template

## Objective

Understand when a new ReplicaSet is created.

### Change only replicas

```yaml
spec:
  replicas: 5
```

This changes the desired number of Pods.

It does not change the Pod template.

Therefore, it does not normally create a new ReplicaSet.

### Change the image

```yaml
containers:
  - name: nginx
    image: nginx:1.28
```

This changes the Pod template.

A new ReplicaSet is created.

### Key Concept

```text
Change replicas
     ↓
Scale existing ReplicaSet

Change Pod template
     ↓
New Deployment revision
     ↓
New ReplicaSet
```

---

# Lab 32: Deployment Self-Healing

## Objective

Observe how a Deployment recovers from Pod deletion.

Check Pods:

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

A replacement Pod should be created.

The actual ownership chain is:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
```

The ReplicaSet is directly responsible for maintaining the Pod count.

---

# Lab 33: Deployment Rollout Interview Challenge

## Scenario

You have:

```text
replicas = 5
maxSurge = 1
maxUnavailable = 0
```

You deploy a new application version.

### Questions

1. How many Pods can temporarily exist?
2. Can an existing available Pod be intentionally taken below the desired availability?
3. Why might additional cluster capacity be required?

### Expected reasoning

With five desired replicas and `maxSurge: 1`, the Deployment can temporarily create up to one additional Pod during the rollout.

With `maxUnavailable: 0`, the rollout is configured to avoid intentionally reducing the number of available Pods below the desired count.

Therefore, the cluster may need capacity for an additional Pod.

---

# Lab 34: Deployment Rollback Interview Challenge

## Scenario

Your application has the following revisions:

```text
Revision 1 → nginx:1.25
Revision 2 → nginx:1.26
Revision 3 → nginx:1.27
```

Revision 3 is broken.

## Task

Rollback to revision 2.

```bash
kubectl rollout undo deployment/<deployment-name> --to-revision=2
```

Then verify:

```bash
kubectl rollout status deployment/<deployment-name>
```

Check the image:

```bash
kubectl get deployment <deployment-name> \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

---

# Lab 35: Zero-Downtime Deployment Exercise

## Objective

Configure a Deployment for high availability during rolling updates.

Use:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zero-downtime-demo
spec:
  replicas: 4

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

  selector:
    matchLabels:
      app: zero-downtime

  template:
    metadata:
      labels:
        app: zero-downtime

    spec:
      containers:
        - name: nginx
          image: nginx:1.27

          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
```

Apply:

```bash
kubectl apply -f zero-downtime.yaml
```

Update the image:

```bash
kubectl set image deployment/zero-downtime-demo \
  nginx=nginx:1.28
```

Watch:

```bash
kubectl get pods -w
```

Check rollout:

```bash
kubectl rollout status deployment/zero-downtime-demo
```

---

# Lab 36: Deployment Cleanup

Delete the Deployments created during the labs:

```bash
kubectl delete deployment nginx-deployment
kubectl delete deployment nginx-probe-deployment
kubectl delete deployment broken-probe
kubectl delete deployment nginx-minready
kubectl delete deployment revision-demo
kubectl delete deployment env-demo
kubectl delete deployment web
kubectl delete deployment zero-downtime-demo
```

If some Deployments do not exist, Kubernetes may report:

```text
Error from server (NotFound)
```

This is safe to ignore.

Verify:

```bash
kubectl get deployments
kubectl get rs
kubectl get pods
```

---

# Lab 37: Complete Deployment Workflow

This lab combines the major concepts.

## Step 1: Create Deployment

```bash
kubectl create deployment workflow-demo \
  --image=nginx:1.25 \
  --replicas=3
```

## Step 2: Verify

```bash
kubectl get deployment
kubectl get rs
kubectl get pods
```

## Step 3: Update the image

```bash
kubectl set image deployment/workflow-demo \
  nginx=nginx:1.26
```

## Step 4: Monitor rollout

```bash
kubectl rollout status deployment/workflow-demo
```

## Step 5: Check history

```bash
kubectl rollout history deployment/workflow-demo
```

## Step 6: Update again

```bash
kubectl set image deployment/workflow-demo \
  nginx=nginx:1.27
```

## Step 7: Check ReplicaSets

```bash
kubectl get rs
```

## Step 8: Rollback

```bash
kubectl rollout undo deployment/workflow-demo
```

## Step 9: Verify

```bash
kubectl rollout status deployment/workflow-demo
kubectl get pods
```

This workflow demonstrates:

```text
Create
  ↓
Scale
  ↓
Update
  ↓
Rolling Update
  ↓
New ReplicaSet
  ↓
Revision History
  ↓
Rollback
```

---

# Deployment Troubleshooting Checklist

When a Deployment is not working, follow this sequence:

## 1. Check Deployment

```bash
kubectl get deployment
```

## 2. Describe Deployment

```bash
kubectl describe deployment <name>
```

## 3. Check ReplicaSets

```bash
kubectl get rs
```

## 4. Check Pods

```bash
kubectl get pods
```

## 5. Describe the failing Pod

```bash
kubectl describe pod <pod-name>
```

## 6. Check logs

```bash
kubectl logs <pod-name>
```

## 7. Check previous logs

```bash
kubectl logs <pod-name> --previous
```

## 8. Check events

```bash
kubectl get events --sort-by=.lastTimestamp
```

## 9. Check rollout

```bash
kubectl rollout status deployment/<name>
```

## 10. Check rollout history

```bash
kubectl rollout history deployment/<name>
```

## 11. Rollback if necessary

```bash
kubectl rollout undo deployment/<name>
```

---

# Deployment Labs Summary

After completing these labs, you should be able to:

* Create Deployments
* Understand Deployment YAML
* Scale Deployments
* Understand Deployment and ReplicaSet relationships
* Perform rolling updates
* Monitor rollouts
* View revision history
* Roll back deployments
* Roll back to specific revisions
* Pause and resume rollouts
* Restart Deployments
* Configure `RollingUpdate`
* Understand `maxSurge`
* Understand `maxUnavailable`
* Configure `Recreate`
* Configure readiness probes
* Understand `minReadySeconds`
* Understand `progressDeadlineSeconds`
* Troubleshoot failed rollouts
* Handle invalid container images
* Understand Deployment revisions
* Configure zero-downtime rolling updates

---

# Important Commands Cheat Sheet

```bash
# Create/update Deployment
kubectl apply -f deployment.yaml

# Create Deployment imperatively
kubectl create deployment <name> --image=<image>

# List Deployments
kubectl get deployments

# Describe Deployment
kubectl describe deployment <name>

# Get Deployment YAML
kubectl get deployment <name> -o yaml

# Scale Deployment
kubectl scale deployment <name> --replicas=5

# Update image
kubectl set image deployment/<name> <container>=<image>:<tag>

# Check rollout
kubectl rollout status deployment/<name>

# View rollout history
kubectl rollout history deployment/<name>

# Inspect a revision
kubectl rollout history deployment/<name> --revision=2

# Rollback
kubectl rollout undo deployment/<name>

# Rollback to specific revision
kubectl rollout undo deployment/<name> --to-revision=2

# Pause rollout
kubectl rollout pause deployment/<name>

# Resume rollout
kubectl rollout resume deployment/<name>

# Restart Deployment
kubectl rollout restart deployment/<name>

# List ReplicaSets
kubectl get rs

# List Pods
kubectl get pods

# Watch Pods
kubectl get pods -w

# Check events
kubectl get events --sort-by=.lastTimestamp

# Delete Deployment
kubectl delete deployment <name>
```

---

# Key Concepts to Remember

## Deployment Hierarchy

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
    | run
    ↓
Containers
```

## Rolling Update

```text
Old ReplicaSet
      |
      | scale down
      ↓
Old Pods

New ReplicaSet
      |
      | scale up
      ↓
New Pods
```

## Rollback

```text
Current Revision
       ↓
Deployment rollback
       ↓
Previous Revision
       ↓
Previous ReplicaSet
       ↓
Previous Pod Template
```

## Reconciliation

```text
Desired State
      ↓
Deployment Controller
      ↓
ReplicaSet
      ↓
Pods
      ↓
Actual State
```

---

# Interview Practice Questions

After completing the labs, you should be able to answer these without referring to documentation:

1. What is a Deployment?
2. How is a Deployment different from a ReplicaSet?
3. What happens when you update a Deployment's image?
4. Why does a Deployment create a new ReplicaSet?
5. What is a rolling update?
6. What is the default Deployment strategy?
7. What is the difference between `RollingUpdate` and `Recreate`?
8. What is `maxSurge`?
9. What is `maxUnavailable`?
10. How do you check rollout status?
11. How do you rollback a Deployment?
12. How do you rollback to a specific revision?
13. What is Deployment revision history?
14. What is `revisionHistoryLimit`?
15. What is `minReadySeconds`?
16. What is `progressDeadlineSeconds`?
17. What happens if a new Pod never becomes Ready?
18. How would you troubleshoot a stuck rollout?
19. Why might a Deployment show `ImagePullBackOff`?
20. What is the relationship between Deployment, ReplicaSet, and Pod?
21. Does changing only `replicas` create a new ReplicaSet?
22. Does changing the container image create a new ReplicaSet?
23. How do readiness probes affect rolling updates?
24. How do you restart a Deployment?
25. How would you configure a Deployment for zero-downtime updates?

---

# Final Interview Cheat Sheet

Remember these four concepts:

```text
1. Deployment
   → Manages application releases

2. ReplicaSet
   → Maintains Pod replicas

3. RollingUpdate
   → Gradually replaces old Pods

4. Rollback
   → Returns to a previous revision
```

The most important workflow to remember:

```text
Deployment
    ↓
Pod Template Changes
    ↓
New ReplicaSet
    ↓
Rolling Update
    ↓
New Pods Become Ready
    ↓
Old ReplicaSet Scaled Down
    ↓
New Version Running
```

And when something goes wrong:

```text
kubectl get deployment
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
        ↓
kubectl rollout undo
```

> **Key takeaway:** A Kubernetes Deployment provides declarative application management and uses ReplicaSets to maintain Pods. Its most important production features are rolling updates, scaling, revision history, and rollback.


# ReplicaSets – Labs

This lab covers practical exercises for understanding and working with **Kubernetes ReplicaSets**.

## Prerequisites

Make sure you have:

* A running Kubernetes cluster
* `kubectl` installed and configured
* Basic knowledge of Pods and YAML

Verify your cluster:

```bash
kubectl cluster-info
```

Check available nodes:

```bash
kubectl get nodes
```

---

# Lab 1: Create a ReplicaSet

## Objective

Create a ReplicaSet with three nginx Pods.

## Step 1: Create the YAML file

Create:

```text
replicaset.yaml
```

Add:

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

## Step 2: Create the ReplicaSet

```bash
kubectl apply -f replicaset.yaml
```

Expected output:

```text
replicaset.apps/nginx-rs created
```

## Step 3: Verify the ReplicaSet

```bash
kubectl get rs
```

Expected:

```text
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   3         3         3       ...
```

## Step 4: Check the Pods

```bash
kubectl get pods
```

You should see three Pods:

```text
nginx-rs-xxxxx
nginx-rs-yyyyy
nginx-rs-zzzzz
```

## Step 5: Check ReplicaSet details

```bash
kubectl describe rs nginx-rs
```

### Verify

You should see:

* Desired replicas: 3
* Current replicas: 3
* Ready replicas: 3
* Selector: `app=nginx`

---

# Lab 2: Understand ReplicaSet and Pod Ownership

## Objective

Identify which ReplicaSet owns a Pod.

List Pods:

```bash
kubectl get pods
```

Choose one Pod:

```bash
kubectl get pod <pod-name> -o yaml
```

Look for:

```yaml
ownerReferences:
```

You should find a reference similar to:

```yaml
ownerReferences:
  - apiVersion: apps/v1
    kind: ReplicaSet
    name: nginx-rs
```

This demonstrates:

```text
ReplicaSet
    |
    └── Pod
```

---

# Lab 3: Delete a Pod and Observe Self-Healing

## Objective

Understand how a ReplicaSet maintains the desired number of Pods.

First check the Pods:

```bash
kubectl get pods
```

You should have three Pods.

Delete one:

```bash
kubectl delete pod <pod-name>
```

Immediately check:

```bash
kubectl get pods
```

You should eventually see a new Pod.

Example:

```text
Before:

Pod A
Pod B
Pod C

Delete Pod B

After:

Pod A
Pod C
Pod D
```

The ReplicaSet automatically creates a replacement Pod.

## Verify ReplicaSet status

```bash
kubectl get rs nginx-rs
```

The desired replica count should remain:

```text
3
```

### Key Learning

A ReplicaSet continuously reconciles:

```text
Desired State = 3 Pods
        ↓
Current State = 2 Pods
        ↓
ReplicaSet creates 1 Pod
        ↓
Current State = 3 Pods
```

---

# Lab 4: Scale a ReplicaSet

## Objective

Scale the ReplicaSet from three Pods to five Pods.

Current state:

```bash
kubectl get rs
```

Scale to five:

```bash
kubectl scale rs nginx-rs --replicas=5
```

Verify:

```bash
kubectl get rs
```

Expected:

```text
NAME       DESIRED   CURRENT   READY
nginx-rs   5         5         5
```

Check Pods:

```bash
kubectl get pods
```

You should now have five nginx Pods.

---

# Lab 5: Scale Down a ReplicaSet

## Objective

Reduce the ReplicaSet from five Pods to two.

Run:

```bash
kubectl scale rs nginx-rs --replicas=2
```

Check:

```bash
kubectl get rs
```

Expected:

```text
NAME       DESIRED   CURRENT   READY
nginx-rs   2         2         2
```

Check Pods:

```bash
kubectl get pods
```

Only two Pods should remain.

---

# Lab 6: Scale Using YAML

## Objective

Change the desired replica count declaratively.

Edit:

```text
replicaset.yaml
```

Change:

```yaml
replicas: 3
```

to:

```yaml
replicas: 4
```

Apply the configuration:

```bash
kubectl apply -f replicaset.yaml
```

Verify:

```bash
kubectl get rs
kubectl get pods
```

The ReplicaSet should now maintain four Pods.

---

# Lab 7: Inspect ReplicaSet Labels and Selectors

## Objective

Understand how ReplicaSets identify Pods.

Check the ReplicaSet:

```bash
kubectl get rs nginx-rs -o yaml
```

Find:

```yaml
selector:
  matchLabels:
    app: nginx
```

Now check a Pod:

```bash
kubectl get pod <pod-name> --show-labels
```

Expected:

```text
app=nginx
```

The relationship is:

```text
ReplicaSet Selector
        |
        | app=nginx
        ↓
Pod Label
        |
        | app=nginx
        ↓
Pod belongs to ReplicaSet
```

---

# Lab 8: Find Pods Managed by a ReplicaSet

Use the ReplicaSet selector:

```bash
kubectl get pods -l app=nginx
```

You can also inspect the ReplicaSet:

```bash
kubectl describe rs nginx-rs
```

Look for:

```text
Selector: app=nginx
```

---

# Lab 9: Change Pod Labels and Observe the Effect

## Objective

Understand why labels and selectors are important.

First check the Pods:

```bash
kubectl get pods --show-labels
```

Choose a Pod and change its label:

```bash
kubectl label pod <pod-name> app=test --overwrite
```

Check the Pods:

```bash
kubectl get pods --show-labels
```

Then check the ReplicaSet:

```bash
kubectl get rs
```

### What to observe

Changing a Pod's labels can affect whether it matches the ReplicaSet selector.

This demonstrates why selectors should be designed carefully.

> Avoid modifying labels on production-managed Pods unless you understand the ownership implications.

---

# Lab 10: Inspect ReplicaSet Events

## Objective

Learn how to troubleshoot ReplicaSet behavior.

Run:

```bash
kubectl describe rs nginx-rs
```

At the bottom, inspect:

```text
Events:
```

You may see events related to:

* Successful Pod creation
* Pod deletion
* Scaling
* Failed Pod creation

You can also check cluster events:

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

# Lab 11: Test ReplicaSet with an Invalid Image

## Objective

Understand how ReplicaSets behave when Pods cannot start correctly.

Create a file:

```text
bad-replicaset.yaml
```

Use:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-bad-rs
spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx-bad

  template:
    metadata:
      labels:
        app: nginx-bad

    spec:
      containers:
        - name: nginx
          image: nginx:does-not-exist
```

Apply:

```bash
kubectl apply -f bad-replicaset.yaml
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

Inspect the Pod:

```bash
kubectl describe pod <pod-name>
```

Look at the Events section.

### Key Learning

The ReplicaSet can successfully create the desired Pods, but it does not guarantee that the containers inside those Pods will successfully start.

---

# Lab 12: Fix the Invalid Image

Change:

```yaml
image: nginx:does-not-exist
```

to:

```yaml
image: nginx:1.25
```

Apply:

```bash
kubectl apply -f bad-replicaset.yaml
```

Check:

```bash
kubectl get pods
```

The Pods should eventually become:

```text
Running
```

---

# Lab 13: Create a ReplicaSet with Multiple Replicas

## Objective

Create a ReplicaSet running five Apache Pods.

Create:

```text
apache-rs.yaml
```

Use:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: apache-rs
spec:
  replicas: 5

  selector:
    matchLabels:
      app: apache

  template:
    metadata:
      labels:
        app: apache

    spec:
      containers:
        - name: apache
          image: httpd:2.4
          ports:
            - containerPort: 80
```

Apply:

```bash
kubectl apply -f apache-rs.yaml
```

Verify:

```bash
kubectl get rs
kubectl get pods -l app=apache
```

---

# Lab 14: Delete the ReplicaSet

## Objective

Understand what happens when a ReplicaSet is deleted.

Delete:

```bash
kubectl delete rs nginx-rs
```

Check:

```bash
kubectl get rs
```

Then:

```bash
kubectl get pods
```

Normally, deleting the ReplicaSet also deletes its dependent Pods.

---

# Lab 15: Delete ReplicaSet and Keep Pods

## Objective

Understand orphaned Pods.

First create a ReplicaSet with three Pods.

Then delete it using orphan deletion:

```bash
kubectl delete rs nginx-rs --cascade=orphan
```

Check:

```bash
kubectl get rs
```

The ReplicaSet should be gone.

Now check:

```bash
kubectl get pods
```

The Pods can remain because they have been orphaned from the ReplicaSet.

### Important

Orphaning resources should be done deliberately. It is generally not the normal way to manage production workloads.

---

# Lab 16: Inspect the ReplicaSet YAML

Run:

```bash
kubectl get rs nginx-rs -o yaml
```

Identify these sections:

```yaml
metadata:
spec:
  replicas:
  selector:
  template:
status:
```

Pay particular attention to:

```yaml
spec:
  replicas: 3
```

and:

```yaml
status:
  replicas: 3
  readyReplicas: 3
```

The `spec` represents the desired state.

The `status` represents the observed state.

---

# Lab 17: Desired State vs Current State

## Objective

Understand Kubernetes reconciliation.

Start with:

```text
Desired replicas = 3
Current replicas = 3
```

Delete one Pod:

```bash
kubectl delete pod <pod-name>
```

For a short period:

```text
Desired replicas = 3
Current replicas = 2
```

The ReplicaSet controller detects the difference and creates a replacement.

Eventually:

```text
Desired replicas = 3
Current replicas = 3
```

This is the basic Kubernetes reconciliation model.

---

# Lab 18: ReplicaSet Troubleshooting

## Scenario

A ReplicaSet expects three Pods, but the Pods are not becoming Ready.

Run:

```bash
kubectl get rs
```

Then:

```bash
kubectl get pods
```

Check the ReplicaSet:

```bash
kubectl describe rs <replicaset-name>
```

Check the Pod:

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

### Troubleshooting checklist

Check:

* Pod status
* Container status
* Image name
* Image tag
* Image pull errors
* Resource requests
* Node availability
* Scheduling events
* Labels
* Selectors
* Container logs

---

# Lab 19: ReplicaSet Selector Troubleshooting

## Scenario

A ReplicaSet unexpectedly does not manage the Pods you expected.

Inspect the selector:

```bash
kubectl get rs <replicaset-name> -o yaml
```

Check:

```yaml
selector:
  matchLabels:
    app: nginx
```

Then check Pod labels:

```bash
kubectl get pods --show-labels
```

Confirm that the Pods have:

```text
app=nginx
```

If the labels do not match the selector, the ReplicaSet will not select those Pods.

---

# Lab 20: Compare ReplicaSet and Deployment

Create a ReplicaSet:

```bash
kubectl get rs
```

Create a Deployment:

```bash
kubectl create deployment nginx-deployment --image=nginx:1.25 --replicas=3
```

Check:

```bash
kubectl get deployment
kubectl get rs
kubectl get pods
```

You should observe:

```text
Deployment
    |
    └── ReplicaSet
          |
          ├── Pod
          ├── Pod
          └── Pod
```

This is one of the most important relationships to understand before learning Deployment rollouts.

---

# Lab 21: Observe Deployment Creating a ReplicaSet

Create a Deployment:

```bash
kubectl create deployment nginx-deployment --image=nginx:1.25 --replicas=3
```

Check:

```bash
kubectl get deployment
```

Then:

```bash
kubectl get rs
```

Then:

```bash
kubectl get pods
```

Observe:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

Now update the image:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.26
```

Check ReplicaSets:

```bash
kubectl get rs
```

You should see a new ReplicaSet.

This demonstrates that **Deployments use ReplicaSets to implement application revisions and rolling updates**.

---

# Lab 22: Interview Challenge – Self-Healing

## Task

Create a ReplicaSet with three Pods.

Then:

1. Delete one Pod.
2. Watch the Pods.
3. Identify the replacement Pod.
4. Explain why the new Pod was created.

### Commands

```bash
kubectl get pods -w
```

In another terminal:

```bash
kubectl delete pod <pod-name>
```

Observe the replacement Pod.

### Expected Concept

```text
ReplicaSet
    ↓
Desired = 3
    ↓
One Pod deleted
    ↓
Current = 2
    ↓
Controller reconciles
    ↓
New Pod created
    ↓
Current = 3
```

---

# Lab 23: Interview Challenge – Scaling

## Task

Start with three replicas and scale to six.

```bash
kubectl scale rs nginx-rs --replicas=6
```

Verify:

```bash
kubectl get rs
kubectl get pods
```

Then scale down to two:

```bash
kubectl scale rs nginx-rs --replicas=2
```

Verify again:

```bash
kubectl get rs
kubectl get pods
```

### Interview Question

**What is responsible for ensuring the desired number of Pods exists?**

Answer:

> The ReplicaSet controller continuously reconciles the desired replica count with the actual number of matching Pods.

---

# Lab 24: Cleanup

Delete the ReplicaSets created during the labs:

```bash
kubectl delete rs nginx-rs
kubectl delete rs nginx-bad-rs
kubectl delete rs apache-rs
```

If you created a Deployment:

```bash
kubectl delete deployment nginx-deployment
```

Verify:

```bash
kubectl get rs
kubectl get deployments
kubectl get pods
```

---

# Lab Summary

By completing these labs, you should be able to:

* Create a ReplicaSet
* Understand ReplicaSet YAML
* Understand selectors and labels
* Scale ReplicaSets
* Delete and recreate Pods
* Understand self-healing
* Inspect ReplicaSet status
* Inspect ReplicaSet events
* Troubleshoot failed Pods
* Understand ReplicaSet ownership
* Understand orphaned Pods
* Understand ReplicaSet and Deployment relationships
* Understand Kubernetes reconciliation

---

# Important Commands Cheat Sheet

```bash
# Create ReplicaSet
kubectl apply -f replicaset.yaml

# List ReplicaSets
kubectl get rs

# Detailed ReplicaSet information
kubectl describe rs <name>

# Get ReplicaSet YAML
kubectl get rs <name> -o yaml

# List Pods
kubectl get pods

# List Pods using a label
kubectl get pods -l app=nginx

# Show Pod labels
kubectl get pods --show-labels

# Scale ReplicaSet
kubectl scale rs <name> --replicas=5

# Delete a Pod
kubectl delete pod <pod-name>

# Watch Pods
kubectl get pods -w

# Check events
kubectl get events --sort-by=.lastTimestamp

# Delete ReplicaSet
kubectl delete rs <name>

# Delete ReplicaSet while orphaning Pods
kubectl delete rs <name> --cascade=orphan
```

---

# Key Concepts to Remember

```text
ReplicaSet
    |
    | maintains
    ↓
Desired number of Pods
    |
    | uses
    ↓
Label Selector
    |
    | identifies
    ↓
Pods
```

### Core Formula

```text
Desired State
      ↓
ReplicaSet Controller
      ↓
Compare with Current State
      ↓
Create/Delete Pods
      ↓
Desired State Achieved
```

### Production Relationship

```text
Deployment
     ↓
ReplicaSet
     ↓
Pods
     ↓
Containers
```

> **Remember:** A ReplicaSet's primary responsibility is to ensure that the desired number of matching Pods are running. In production, ReplicaSets are usually managed indirectly through Deployments.


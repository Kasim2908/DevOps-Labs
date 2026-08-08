# ☸️ Kubernetes Pods

## Overview

A Pod is the smallest deployable unit in Kubernetes.

A Pod represents one or more containers that are deployed and managed together.

Most Pods contain a single application container, but Kubernetes also supports multi-container Pods.

---

# Pod Architecture

```text
                    Pod
                     │
          ┌──────────┴──────────┐
          │                     │
     Container A           Container B
          │                     │
          └──────────┬──────────┘
                     │
              Shared Network
                     │
                  Pod IP
```

Containers in the same Pod can share:

- Network namespace
- Pod IP address
- localhost communication
- Volumes when configured

---

# Pod vs Container

## Container

A container is an isolated process that runs an application.

Examples:

- nginx
- Node.js
- Python
- Redis

## Pod

A Pod is the Kubernetes abstraction that hosts one or more containers.

```text
Kubernetes
    │
   Pod
    │
 ┌──┴──┐
 │     │
 C1    C2
```

Kubernetes schedules Pods rather than individual containers.

---

# Why Does Kubernetes Use Pods?

A Pod allows tightly coupled containers to share:

- Network
- Storage
- Lifecycle
- Execution environment

Example:

```text
Pod
│
├── Application
│
└── Logging Sidecar
```

The application can write logs to a shared volume while the sidecar processes those logs.

---

# Pod Networking

Containers in the same Pod normally share the Pod's network namespace.

Therefore:

```text
Pod IP = 10.x.x.x

Container A → Same Pod IP
Container B → Same Pod IP
```

They can communicate through:

```text
localhost
```

Example:

```text
Container A
     │
     │ localhost:8080
     ▼
Container B
```

Containers in the same Pod must coordinate their port usage because they share the same network namespace.

---

# Pod YAML

Basic Pod definition:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod

spec:
  containers:
    - name: nginx
      image: nginx:latest
```

---

# Kubernetes YAML Structure

## apiVersion

Defines the API version.

```yaml
apiVersion: v1
```

## kind

Defines the Kubernetes resource type.

```yaml
kind: Pod
```

## metadata

Contains identifying information.

```yaml
metadata:
  name: nginx-pod
```

## spec

Defines the desired configuration.

```yaml
spec:
  containers:
```

---

# Imperative vs Declarative

## Imperative

```bash
kubectl run nginx --image=nginx
```

Directly tells Kubernetes what action to perform.

## Declarative

```bash
kubectl apply -f nginx-pod.yaml
```

Defines the desired state in YAML.

Production environments generally prefer declarative configuration because it can be stored in Git and reviewed.

---

# Pod Lifecycle

The main Pod phases are:

```text
Pending
Running
Succeeded
Failed
Unknown
```

## Pending

The Pod has been accepted but cannot yet run successfully.

Possible causes:

- Scheduling problems
- Insufficient resources
- Image pulling
- Taints
- Affinity rules

---

## Running

The Pod has been bound to a node and at least one container has started.

---

## Succeeded

All containers terminated successfully.

Common with Jobs.

---

## Failed

One or more containers terminated unsuccessfully.

---

## Unknown

The control plane cannot determine the Pod state, often because of communication problems with the node.

---

# Container States

Container states are different from Pod phases.

A container can be:

```text
Waiting
Running
Terminated
```

This distinction is important during troubleshooting.

---

# Restart Policies

Pods support:

```yaml
restartPolicy: Always
```

```yaml
restartPolicy: OnFailure
```

```yaml
restartPolicy: Never
```

## Always

Containers are restarted when they terminate.

Default for Pods.

## OnFailure

Containers are restarted when they terminate unsuccessfully.

## Never

Containers are not restarted after termination.

---

# Multi-Container Pods

A Pod can contain multiple containers.

```text
                 Pod
                  │
        ┌─────────┴─────────┐
        │                   │
     NGINX               Sidecar
        │                   │
        └─────────┬─────────┘
                  │
           Shared Network
```

Multiple containers should generally be used when they are tightly coupled and need to share the same lifecycle/network context.

---

# Sidecar Pattern

A sidecar runs alongside the main application.

Example:

```text
Pod
│
├── Application Container
│
└── Logging Sidecar
```

Possible uses:

- Log processing
- Proxies
- Monitoring agents
- Configuration synchronization

---

# Init Containers

Init containers run before the normal application containers.

```text
Init Container
      │
      ▼
Completes Successfully
      │
      ▼
Application Container
```

Example:

```yaml
initContainers:
  - name: init-message
    image: busybox:1.36
    command:
      - sh
      - -c
      - echo "Initializing application..."
```

Init containers are useful for:

- Initialization
- Preparing files
- Preparing configuration
- Waiting for prerequisites
- Downloading required data

---

# Init Container Execution

Multiple init containers run sequentially.

```text
init-1
  ↓
init-2
  ↓
application
```

The next init container does not start until the previous one succeeds.

---

# Sidecar vs Init Container

| Init Container | Sidecar |
|---|---|
| Runs before application | Runs alongside application |
| Usually exits | Usually keeps running |
| Initialization tasks | Supporting tasks |
| Sequential | Concurrent |
| Must complete successfully | Usually long-running |

---

# Static Pods

Static Pods are managed directly by the kubelet rather than by a Deployment or another controller.

A common location on kubeadm control-plane nodes is:

```text
/etc/kubernetes/manifests/
```

Conceptually:

```text
Manifest
   ↓
kubelet
   ↓
Static Pod
```

They are commonly used for Kubernetes control-plane components in kubeadm-based clusters.

---

# Pod Troubleshooting

Pod troubleshooting is an important DevOps skill.

Basic workflow:

```text
kubectl get pod
       ↓
kubectl describe pod
       ↓
Check Events
       ↓
kubectl logs
       ↓
Find root cause
```

---

# Common Pod Problems

## 1. Pending

Example:

```text
STATUS: Pending
```

Possible causes:

- No suitable node
- Insufficient CPU/memory
- Node selector mismatch
- Node affinity mismatch
- Taints and tolerations
- Storage/scheduling constraints

Check:

```bash
kubectl describe pod <pod>
```

Pay attention to:

```text
Events:
```

---

# 2. ErrImagePull

Kubernetes cannot pull the specified container image.

Possible causes:

- Wrong image name
- Wrong image tag
- Private registry
- Authentication failure
- Network problems

Check:

```bash
kubectl describe pod <pod>
```

Look at Events.

---

# 3. ImagePullBackOff

Kubernetes repeatedly fails to pull the image and starts backing off between attempts.

Important:

`ImagePullBackOff` is usually a symptom, not the root cause.

Investigate:

```bash
kubectl describe pod <pod>
```

---

# 4. CrashLoopBackOff

The container starts and repeatedly crashes.

Example:

```text
Application starts
       ↓
Application crashes
       ↓
Container restarts
       ↓
Application crashes
       ↓
Backoff
```

Check:

```bash
kubectl logs <pod>
```

Also:

```bash
kubectl describe pod <pod>
```

If the container has restarted:

```bash
kubectl logs <pod> --previous
```

---

# 5. OOMKilled

OOM means:

```text
Out Of Memory
```

A container can be killed when it exceeds its memory limit.

Example:

```yaml
resources:
  limits:
    memory: "32Mi"
```

If the process exceeds the limit, it may be terminated with:

```text
Reason: OOMKilled
```

Check:

```bash
kubectl describe pod <pod>
```

---

# Important Troubleshooting Commands

## List Pods

```bash
kubectl get pods
```

## Detailed Pod Information

```bash
kubectl describe pod <pod>
```

## Logs

```bash
kubectl logs <pod>
```

## Logs From Specific Container

```bash
kubectl logs <pod> -c <container>
```

## Previous Container Logs

```bash
kubectl logs <pod> --previous
```

## Execute Command

```bash
kubectl exec -it <pod> -- /bin/bash
```

## Execute in Specific Container

```bash
kubectl exec -it <pod> -c <container> -- /bin/bash
```

## Get Pod YAML

```bash
kubectl get pod <pod> -o yaml
```

## Get Pod IP and Node

```bash
kubectl get pod <pod> -o wide
```

## Get Pod Container Name

```bash
kubectl get pod <pod> \
  -o jsonpath='{.spec.containers[*].name}'
```

## Get Pod IP

```bash
kubectl get pod <pod> \
  -o jsonpath='{.status.podIP}'
```

## Get Events

```bash
kubectl get events --sort-by=.lastTimestamp
```

## Port Forward

```bash
kubectl port-forward pod/<pod> 8080:80
```

---

# Production Insight

Standalone Pods are generally not used for long-running production applications.

Instead:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Containers
```

The Deployment/ReplicaSet controller can maintain the desired number of Pods.

If a Pod disappears:

```text
Desired = 3
Actual = 2
      ↓
Controller
      ↓
Create replacement Pod
      ↓
Actual = 3
```

---

# Key Takeaways

- A Pod is the smallest deployable Kubernetes unit.
- A Pod can contain one or multiple containers.
- Containers in the same Pod share the network namespace.
- Containers can communicate through localhost.
- Volumes can be shared between containers.
- Init containers run before application containers.
- Sidecars run alongside application containers.
- Static Pods are managed directly by kubelet.
- Pod phases and container states are different concepts.
- `kubectl describe` and Events are critical for troubleshooting.
- `CrashLoopBackOff` is a symptom, not necessarily the root cause.
- `OOMKilled` indicates an out-of-memory termination.
- Production applications are generally managed by higher-level controllers.

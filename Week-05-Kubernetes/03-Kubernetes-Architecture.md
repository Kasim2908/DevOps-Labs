# 🏛 Kubernetes Architecture

## What is a Kubernetes Cluster?

A Kubernetes Cluster is a collection of machines that work together to run containerized applications.

A cluster consists of:

- Control Plane
- Worker Nodes

---

# Cluster Architecture

```
                 Kubernetes Cluster

         +---------------------------+
         |                           |
         |      Control Plane        |
         |                           |
         +---------------------------+
                   |
   -----------------------------------------
   |                                       |
+-----------+                      +-----------+
| Worker 1  |                      | Worker 2  |
+-----------+                      +-----------+
```

---

# Control Plane

The Control Plane is the brain of Kubernetes.

Responsibilities:

- Makes decisions
- Maintains cluster state
- Schedules Pods
- Manages cluster

Components:

- API Server
- etcd
- Scheduler
- Controller Manager

---

## API Server

The API Server is the entry point of Kubernetes.

Responsibilities:

- Authentication
- Authorization
- Validation
- Accepts kubectl requests
- Communicates with all cluster components

---

## etcd

etcd is a distributed key-value database.

Stores:

- Pods
- Nodes
- Deployments
- Secrets
- ConfigMaps
- Services
- Cluster State

Think of etcd as Kubernetes' memory.

---

## Scheduler

The Scheduler decides where Pods should run.

It considers:

- CPU
- Memory
- Node Labels
- Affinity
- Taints
- Resource Requests

---

## Controller Manager

Maintains the Desired State.

Example:

Desired:
3 Pods

Actual:
2 Pods

Controller Manager automatically creates the missing Pod.

---

# Worker Node

Applications run on Worker Nodes.

Each Worker Node contains:

- kubelet
- kube-proxy
- Container Runtime

---

## kubelet

The node agent.

Responsibilities:

- Talks to API Server
- Starts Pods
- Monitors Pods
- Reports status

---

## kube-proxy

Responsible for networking.

Handles:

- Service networking
- Load balancing
- Pod communication

---

## Container Runtime

Responsible for running containers.

Examples:

- containerd
- CRI-O

---

# Request Flow

kubectl apply

↓

API Server

↓

etcd

↓

Controller Manager

↓

Scheduler

↓

kubelet

↓

Container Runtime

↓

Pod Running

---

# Desired State

Kubernetes follows the Desired State Model.

You define:

"I want 3 replicas."

Kubernetes continuously ensures that 3 replicas remain running.

---

# Production Insight

In production, engineers define the desired state using YAML files.

Kubernetes continuously works to match the actual state with the desired state.

This reconciliation process is one of Kubernetes' core principles.

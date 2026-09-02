☸️ Kubernetes Quality of Service (QoS)

Overview

Kubernetes Quality of Service (QoS) determines how Kubernetes classifies Pods based on the CPU and memory requests and limits configured for their containers.

QoS classification is especially important when a node is under resource pressure.

Kubernetes assigns every Pod one of three QoS classes:

Guaranteed
Burstable
BestEffort

These classes influence how Pods are treated during resource pressure, especially during out-of-memory (OOM) situations.

Why Kubernetes QoS Exists

A Kubernetes node has finite resources.

For example:

Node
│
├── CPU
│
└── Memory

Multiple Pods compete for those resources:

Node
│
├── Pod A
├── Pod B
├── Pod C
└── Pod D

If workloads consume more resources than the node can safely provide, Kubernetes needs mechanisms to determine:

Which workloads have guaranteed resource allocations

Which workloads can use resources above their requests

Which workloads have no resource requests or limits

Which Pods are more vulnerable during resource pressure

QoS classification provides one part of this mechanism.

CPU and Memory Resources

QoS classification is based primarily on:

resources:
  requests:
    cpu: ...
    memory: ...
  limits:
    cpu: ...
    memory: ...

Example:

resources:
  requests:
    cpu: "500m"
    memory: "256Mi"

  limits:
    cpu: "1"
    memory: "512Mi"

There are two important concepts.

Resource Requests

A request represents the amount of a resource that Kubernetes uses for scheduling decisions.

Example:

resources:
  requests:
    cpu: "500m"
    memory: "256Mi"

This tells the scheduler that the container requires:

CPU    = 500m
Memory = 256Mi

The scheduler uses requests when deciding whether a Pod can fit on a node.

Conceptually:

Pod Request
     │
     ▼
Scheduler
     │
     ▼
Suitable Node

Requests do not necessarily mean the container will continuously consume exactly that amount.

Resource Limits

A limit defines an upper boundary for a container's resource consumption.

Example:

resources:
  limits:
    cpu: "1"
    memory: "512Mi"

Conceptually:

Request
   │
   ▼
Normal expected allocation

Limit
   │
   ▼
Maximum allowed boundary

CPU and memory behave differently when limits are reached.

CPU Limit Behavior

CPU is a compressible resource.

If a container attempts to use more CPU than its configured limit, CPU usage can be throttled.

Example:

resources:
  limits:
    cpu: "500m"

The container may be restricted from continuously consuming more than its CPU limit.

Conceptually:

Application wants:
1000m CPU

CPU limit:
500m CPU

Result:
CPU is throttled

CPU overuse normally does not result in the same immediate kill behavior as memory overuse.

Memory Limit Behavior

Memory is different from CPU.

Memory is not normally throttled in the same way.

If a container exceeds its memory limit, it can be terminated by the kernel's OOM mechanism.

Example:

resources:
  limits:
    memory: "256Mi"

If the container consumes more memory than allowed:

Memory usage
     │
     ▼
Exceeds limit
     │
     ▼
OOM condition
     │
     ▼
Container may be killed

You may observe:

Reason: OOMKilled

Check it with:

kubectl describe pod <pod>

CPU and Memory Units

CPU Units

Kubernetes CPU can be expressed in cores or millicores.

Examples:

cpu: "1"

means:

1 CPU core

Example:

cpu: "500m"

means:

500 millicores
= 0.5 CPU

Common examples:

100m = 0.1 CPU
250m = 0.25 CPU
500m = 0.5 CPU
1000m = 1 CPU
2000m = 2 CPU

Memory Units

Memory can be specified using units such as:

Mi
Gi
M
G

Examples:

memory: "128Mi"
memory: "256Mi"
memory: "512Mi"
memory: "1Gi"

For Kubernetes resource specifications, binary units such as Mi and Gi are commonly used.

Kubernetes QoS Classes

Every Pod is assigned exactly one QoS class:

                 Pod
                  │
          ┌───────┼────────┐
          │       │        │
          ▼       ▼        ▼
      Guaranteed Burstable BestEffort

The three classes are:

Guaranteed

Burstable

BestEffort

1. Guaranteed QoS

A Pod receives the Guaranteed QoS class when its containers meet the conditions required for the Guaranteed class.

For the normal Pod case, every container must have:

CPU request = CPU limit
Memory request = Memory limit

Example:

apiVersion: v1
kind: Pod

metadata:
  name: guaranteed-pod

spec:
  containers:
    - name: app
      image: nginx:latest

      resources:
        requests:
          cpu: "500m"
          memory: "256Mi"

        limits:
          cpu: "500m"
          memory: "256Mi"

Here:

CPU request  = 500m
CPU limit    = 500m

Memory request = 256Mi
Memory limit   = 256Mi

Therefore:

QoS = Guaranteed

Guaranteed QoS Rule

For a typical single-container Pod:

CPU request == CPU limit
AND
Memory request == Memory limit

Example:

resources:
  requests:
    cpu: "1"
    memory: "1Gi"

  limits:
    cpu: "1"
    memory: "1Gi"

Result:

Guaranteed

Guaranteed QoS with Multiple Containers

QoS classification applies to the entire Pod.

For example:

spec:
  containers:

    - name: app
      image: nginx
      resources:
        requests:
          cpu: "500m"
          memory: "256Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"

    - name: sidecar
      image: busybox
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "100m"
          memory: "128Mi"

Both containers satisfy:

request == limit

Therefore the Pod can be:

Guaranteed

2. Burstable QoS

A Pod receives the Burstable QoS class when it does not qualify as Guaranteed but at least one container has a CPU or memory request or limit.

Example:

apiVersion: v1
kind: Pod

metadata:
  name: burstable-pod

spec:
  containers:
    - name: app
      image: nginx

      resources:
        requests:
          cpu: "250m"
          memory: "128Mi"

        limits:
          cpu: "500m"
          memory: "256Mi"

Here:

CPU:
request = 250m
limit   = 500m

Memory:
request = 128Mi
limit   = 256Mi

Request and limit are different.

Therefore:

QoS = Burstable

Burstable QoS Concept

Burstable workloads have a baseline request and can potentially consume more resources up to their configured limits.

Conceptually:

Resource Usage
      │
      │             Limit
      │               ▲
      │               │
      │        ┌──────┘
      │        │
      │   ┌────┘
      │   │
      │   │ Request
      │   ▲
      └──────────────────────► Time

For example:

Request = 250m CPU
Limit   = 500m CPU

The workload has:

Baseline = 250m
Maximum  = 500m

Burstable Example: Request Only

A Pod can also be Burstable when a container has a request but no limit.

Example:

resources:
  requests:
    cpu: "250m"
    memory: "128Mi"

Because resource configuration exists but the Pod does not satisfy Guaranteed requirements:

QoS = Burstable

Burstable Example: Limit Only

A container with a resource limit but no explicit request can also result in a non-BestEffort QoS class.

For example:

resources:
  limits:
    memory: "256Mi"

Kubernetes applies resource semantics to scheduling and QoS classification, so do not assume that "request missing" automatically means BestEffort.

The exact effective request behavior should be understood together with Kubernetes' handling of limits and defaults.

3. BestEffort QoS

A Pod receives the BestEffort QoS class when none of its containers has CPU or memory requests or limits.

Example:

apiVersion: v1
kind: Pod

metadata:
  name: besteffort-pod

spec:
  containers:
    - name: app
      image: nginx

No resources are configured:

resources:

Therefore:

QoS = BestEffort

BestEffort Concept

BestEffort Pods do not have explicit CPU or memory resource requests or limits.

Conceptually:

Pod
│
├── CPU request: Not specified
├── CPU limit:   Not specified
├── Memory request: Not specified
└── Memory limit:   Not specified

Result:

BestEffort

QoS Classification Decision Tree

A simplified way to remember QoS classification:

                    Pod
                     │
                     ▼
        Are resource requests/limits
             configured appropriately?
                     │
          ┌──────────┴──────────┐
          │                     │
         Yes                    No
          │                     │
          ▼                     ▼
   Does every container       Does any container
   have request == limit      have CPU/memory
   for required resources?   request or limit?
          │                     │
      ┌───┴───┐             ┌───┴───┐
      │       │             │       │
     Yes      No           Yes      No
      │       │             │       │
      ▼       ▼             ▼       ▼
Guaranteed  Burstable    Burstable BestEffort

QoS Comparison

QoS Class

Resource Configuration

General Priority During Node Pressure

Guaranteed

Requests and limits are equal for required resources

Highest

Burstable

Some resource configuration exists but not Guaranteed

Middle

BestEffort

No CPU or memory requests/limits

Lowest

The QoS class is only one factor in Kubernetes eviction behavior.

How to Check a Pod's QoS Class

Use:

kubectl get pod <pod-name> -o jsonpath='{.status.qosClass}'

Example:

kubectl get pod nginx -o jsonpath='{.status.qosClass}'

Possible output:

Guaranteed

or:

Burstable

or:

BestEffort

Check QoS Using describe

Run:

kubectl describe pod <pod-name>

Look for:

QoS Class:

Example:

QoS Class: Guaranteed

Check QoS for All Pods

You can inspect all Pods with:

kubectl get pods -o custom-columns=NAME:.metadata.name,QOS:.status.qosClass

Example:

NAME              QOS
nginx             Guaranteed
backend           Burstable
debug             BestEffort

Practical Lab: Guaranteed Pod

Create:

apiVersion: v1
kind: Pod

metadata:
  name: qos-guaranteed

spec:
  containers:
    - name: app
      image: nginx:latest

      resources:
        requests:
          cpu: "500m"
          memory: "256Mi"

        limits:
          cpu: "500m"
          memory: "256Mi"

Apply:

kubectl apply -f guaranteed.yaml

Check:

kubectl get pod qos-guaranteed

Then:

kubectl get pod qos-guaranteed \
  -o jsonpath='{.status.qosClass}'

Expected:

Guaranteed

Practical Lab: Burstable Pod

Create:

apiVersion: v1
kind: Pod

metadata:
  name: qos-burstable

spec:
  containers:
    - name: app
      image: nginx:latest

      resources:
        requests:
          cpu: "250m"
          memory: "128Mi"

        limits:
          cpu: "500m"
          memory: "256Mi"

Apply:

kubectl apply -f burstable.yaml

Check:

kubectl get pod qos-burstable \
  -o jsonpath='{.status.qosClass}'

Expected:

Burstable

Practical Lab: BestEffort Pod

Create:

apiVersion: v1
kind: Pod

metadata:
  name: qos-besteffort

spec:
  containers:
    - name: app
      image: nginx:latest

Apply:

kubectl apply -f besteffort.yaml

Check:

kubectl get pod qos-besteffort \
  -o jsonpath='{.status.qosClass}'

Expected:

BestEffort

Compare All Three

Run:

kubectl get pods \
  -o custom-columns=NAME:.metadata.name,QOS:.status.qosClass

Expected conceptually:

NAME               QOS
qos-guaranteed     Guaranteed
qos-burstable      Burstable
qos-besteffort     BestEffort

QoS and Scheduling

QoS classification and scheduling are related to resources but are not the same thing.

The scheduler primarily uses:

Resource Requests

to determine whether a Pod fits on a node.

Example:

Node Capacity
CPU = 4
Memory = 8Gi

Pod:

CPU Request = 1
Memory Request = 2Gi

The scheduler considers the request:

1 CPU
2Gi memory

when making placement decisions.

Requests vs Limits vs QoS

These concepts must not be confused.

Requests
   │
   ├── Used heavily by scheduler
   │
   └── Represents requested baseline resources

Limits
   │
   ├── Defines container resource ceilings
   │
   └── Influences runtime enforcement

QoS
   │
   ├── Classifies the Pod
   │
   └── Matters during resource pressure

QoS and Node Resource Pressure

Suppose a node has:

Memory Capacity = 8Gi

Several Pods consume memory:

Pod A → 3Gi
Pod B → 3Gi
Pod C → 2Gi
Pod D → 2Gi

Total:

10Gi

The node cannot safely satisfy all workloads.

The node may enter:

Memory Pressure

Kubernetes has eviction mechanisms to protect node stability.

Memory Pressure

Check node conditions:

kubectl describe node <node-name>

Look for:

Conditions:

You may see:

MemoryPressure

You can also use:

kubectl get nodes

and:

kubectl describe node <node-name>

Eviction

When a node experiences resource pressure, Kubernetes may evict Pods.

Conceptually:

Node Resource Pressure
        │
        ▼
Kubelet detects pressure
        │
        ▼
Eviction decisions
        │
        ▼
Pods terminated
        │
        ▼
Node resources recovered

QoS class contributes to how Pods are prioritized for eviction.

QoS and Eviction Priority

A simplified mental model is:

BestEffort
    ↓
Burstable
    ↓
Guaranteed

In general, BestEffort Pods are more vulnerable to eviction than Burstable Pods, while Guaranteed Pods receive the strongest protection.

However, QoS class alone does not determine the complete eviction order.

Other factors, especially:

resource usage relative to requests

Pod priority

node pressure

eviction thresholds

can affect eviction decisions.

Important: QoS Is Not Pod Priority

Do not confuse:

QoS Class

with:

Priority

QoS:

Guaranteed
Burstable
BestEffort

Pod priority is configured using Kubernetes PriorityClasses.

For example:

priorityClassName: high-priority

They solve different problems.

QoS vs Priority

QoS

Priority

Based on resource configuration

Based on PriorityClass

Classifies Pods

Assigns scheduling/eviction priority

Guaranteed/Burstable/BestEffort

User-defined priority values

Related to resource management

Related to Pod priority

A high-priority Pod is not automatically Guaranteed.

QoS and OOMKilled

OOMKilled means a container was terminated because of an out-of-memory condition.

Example:

Container
   │
   ▼
Memory consumption increases
   │
   ▼
Memory pressure / limit exceeded
   │
   ▼
OOM handling
   │
   ▼
Container killed

Check:

kubectl describe pod <pod>

and:

kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState.terminated.reason}'

Possible output:

OOMKilled

QoS Does Not Mean "Guaranteed to Never Die"

A common misconception is:

Guaranteed = Cannot be killed

This is incorrect.

Guaranteed QoS does not mean:

The Pod can never be terminated.

A Guaranteed Pod can still be affected by:

Node failure

Application failure

Explicit deletion

Deployment changes

Cluster maintenance

Hardware failure

Severe node-level conditions

Other Kubernetes lifecycle events

Guaranteed mainly describes the Pod's resource configuration and its relative treatment under resource pressure.

QoS Does Not Guarantee Performance

Another misconception:

Guaranteed QoS = Guaranteed application performance

Not necessarily.

QoS classification does not guarantee:

zero latency

zero CPU throttling

unlimited resources

node availability

network performance

application correctness

It is a resource-management classification.

LimitRange and QoS

A Kubernetes namespace can have a LimitRange.

A LimitRange can provide:

default CPU requests

default CPU limits

default memory requests

default memory limits

minimum resource constraints

maximum resource constraints

Example concept:

Namespace
    │
    ▼
LimitRange
    │
    ▼
Default Resources
    │
    ▼
Pod

This is important because resource defaults can affect the final resource configuration of containers and therefore their QoS classification.

Example LimitRange

Example:

apiVersion: v1
kind: LimitRange

metadata:
  name: resource-limits

spec:
  limits:
    - type: Container

      default:
        cpu: "500m"
        memory: "256Mi"

      defaultRequest:
        cpu: "250m"
        memory: "128Mi"

Create:

kubectl apply -f limitrange.yaml

Inspect:

kubectl describe limitrange resource-limits

ResourceQuota and QoS

ResourceQuota limits aggregate resource consumption within a namespace.

Example:

apiVersion: v1
kind: ResourceQuota

metadata:
  name: compute-quota

spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"

ResourceQuota and QoS are different concepts.

ResourceQuota
    │
    └── Namespace-level aggregate resource limits

QoS
    │
    └── Pod-level resource classification

QoS and Namespaces

QoS is assigned to individual Pods.

For example:

Namespace: production

Pod A → Guaranteed
Pod B → Burstable
Pod C → BestEffort

A namespace can therefore contain Pods from different QoS classes.

QoS and Deployments

QoS applies to Pods created by controllers such as Deployments.

Example:

Deployment
    │
    ▼
ReplicaSet
    │
    ▼
Pods

If the Deployment template defines:

resources:
  requests:
    cpu: "500m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "256Mi"

the resulting Pods can have:

Guaranteed

QoS is ultimately determined from the Pod's containers.

QoS in a Deployment

Example:

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
          image: nginx:latest

          resources:
            requests:
              cpu: "250m"
              memory: "128Mi"

            limits:
              cpu: "250m"
              memory: "128Mi"

The Pods created by this template can be:

Guaranteed

Check:

kubectl get pods \
  -o custom-columns=NAME:.metadata.name,QOS:.status.qosClass

QoS and StatefulSets

The same resource concepts apply to StatefulSet Pods.

Example:

resources:
  requests:
    cpu: "500m"
    memory: "512Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"

The resulting Pod can be:

Guaranteed

QoS and Jobs

Jobs create Pods that can also receive QoS classifications.

Example:

resources:
  requests:
    cpu: "100m"
    memory: "128Mi"

  limits:
    cpu: "200m"
    memory: "256Mi"

The Job's Pods can be:

Burstable

QoS is a property of the Pod, regardless of whether the Pod came from a Deployment, Job, StatefulSet, or another controller.

Pod-Level QoS vs Container-Level Resources

This is one of the most important concepts.

Resources are configured at the container level:

spec:
  containers:
    - name: app
      resources:
        requests:
          cpu: "500m"
          memory: "256Mi"

But QoS classification is assigned to the Pod.

Therefore:

Container resources
       │
       ▼
Pod QoS classification

With multiple containers, Kubernetes considers the resource configuration of the containers when determining the Pod's QoS class.

Multiple Containers and QoS

Example:

spec:
  containers:

    - name: app
      image: nginx
      resources:
        requests:
          cpu: "500m"
          memory: "256Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"

    - name: sidecar
      image: busybox
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "200m"
          memory: "128Mi"

The second container has:

CPU request = 100m
CPU limit   = 200m

They are different.

Therefore the Pod does not satisfy the normal Guaranteed pattern.

Result:

Burstable

Init Containers and QoS

Init containers are part of the Pod specification and have their own resource configuration.

Example:

initContainers:
  - name: init
    image: busybox

    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"

      limits:
        cpu: "100m"
        memory: "64Mi"

Application containers also have resources:

containers:
  - name: app
    image: nginx

    resources:
      requests:
        cpu: "500m"
        memory: "256Mi"

      limits:
        cpu: "500m"
        memory: "256Mi"

QoS classification must be considered across the Pod's relevant containers, including init-container resource configuration.

Ephemeral Containers and QoS

Ephemeral containers are primarily intended for troubleshooting.

Example:

kubectl debug -it <pod> --image=busybox

They are not a mechanism for redesigning a Pod's normal resource-management strategy.

When troubleshooting QoS, inspect the actual Pod specification and status rather than assuming the original application container alone explains the current state.

QoS and Pod Overhead

Some workloads may include Pod overhead.

Pod overhead represents additional resource consumption associated with running a Pod beyond the application containers, particularly with runtimes such as sandboxed runtimes.

Example concept:

Pod
│
├── Application container resources
│
└── Pod overhead

This can matter when evaluating actual node resource consumption.

QoS and Node Capacity

Suppose:

Node Capacity:
CPU    = 4 cores
Memory = 8Gi

Pods request:

Pod A → 1 CPU, 2Gi
Pod B → 1 CPU, 2Gi
Pod C → 1 CPU, 2Gi

Total requests:

CPU    = 3 CPU
Memory = 6Gi

The scheduler can use these requests to determine whether another Pod can fit.

QoS does not replace resource requests.

Resource Requests and QoS Relationship

Think about the relationship as:

                 Pod
                  │
        ┌─────────┴─────────┐
        │                   │
    Requests             Limits
        │                   │
        └─────────┬─────────┘
                  │
                  ▼
             QoS Class

But remember:

Requests → scheduling
Limits   → runtime resource ceiling
QoS      → Pod classification

Guaranteed vs Burstable vs BestEffort

Guaranteed

Resource configuration:
Strict

Typical pattern:

request == limit

Use when:

workload needs predictable resource allocation

resource boundaries should be explicit

production-critical workloads need stronger protection from resource-pressure eviction

Burstable

Resource configuration:
Flexible

Typical pattern:

request < limit

Use when:

workload has a known baseline

workload may need additional resources

controlled bursting is useful

BestEffort

Resource configuration:
None

Use carefully.

Potential examples:

temporary development workloads

lightweight experiments

non-critical workloads

BestEffort should not automatically be considered appropriate for production applications.

Production Recommendation

For production workloads, define resource requests and limits deliberately.

Example:

resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"

This gives Kubernetes useful information for:

scheduling

resource accounting

resource enforcement

QoS classification

The exact values should come from workload measurements rather than arbitrary numbers.

How to Choose Requests

Requests should represent a realistic baseline requirement.

Bad example:

requests:
  cpu: "4"

when the application normally needs:

100m CPU

This can cause inefficient scheduling.

Another bad example:

requests:
  memory: "8Gi"

when the application normally requires:

256Mi

Overly large requests can make Pods harder to schedule.

How to Choose Limits

Limits should be based on workload behavior and operational requirements.

Example:

requests:
  cpu: "250m"
  memory: "256Mi"

limits:
  cpu: "1"
  memory: "512Mi"

This communicates:

Expected baseline:
250m CPU
256Mi memory

Configured ceiling:
1 CPU
512Mi memory

Do not blindly copy resource values from another workload.

Measuring Actual Resource Usage

Use:

kubectl top pods

Example:

NAME       CPU(cores)   MEMORY(bytes)
web        120m         180Mi
api        350m         420Mi
worker     80m          250Mi

For nodes:

kubectl top nodes

This requires the Kubernetes metrics pipeline, commonly Metrics Server.

Resource Usage vs Request

Suppose:

Request = 250m CPU
Actual usage = 100m CPU

The workload is using less than its request.

Another example:

Request = 250m CPU
Actual usage = 400m CPU
Limit = 500m CPU

The workload is using more than its request but remains below its limit.

This is a common Burstable pattern.

Resource Usage vs Limit

Suppose:

Request = 250m
Limit   = 500m
Usage   = 450m

The workload is:

Above request
Below limit

It can continue using CPU subject to node availability and CPU enforcement.

If memory usage reaches beyond the memory limit, the container may be terminated due to OOM.

CPU Throttling

For CPU:

Usage > CPU limit
        │
        ▼
CPU throttling

Example:

limits:
  cpu: "500m"

The application may experience CPU throttling if it continuously tries to consume more CPU than its configured limit.

This can contribute to:

increased latency

slower processing

lower throughput

Memory Limit Exceeded

For memory:

Usage > Memory limit
        │
        ▼
OOM handling
        │
        ▼
Container may be killed

Example:

limits:
  memory: "256Mi"

If the process requires substantially more memory than this limit allows, investigate for:

memory leaks

unexpectedly large workloads

incorrect application configuration

insufficient memory limit

excessive concurrency

Troubleshooting QoS

Start with:

kubectl get pods

Then inspect:

kubectl describe pod <pod>

Check resource configuration:

kubectl get pod <pod> -o yaml

Check QoS:

kubectl get pod <pod> \
  -o jsonpath='{.status.qosClass}'

Check current resource usage:

kubectl top pod <pod>

Check container status:

kubectl get pod <pod> \
  -o jsonpath='{.status.containerStatuses[*]}'

Troubleshooting OOMKilled

If you see:

OOMKilled

Check:

kubectl describe pod <pod>

Then:

kubectl logs <pod>

If the container restarted:

kubectl logs <pod> --previous

Check memory usage:

kubectl top pod <pod>

Inspect configured memory:

kubectl get pod <pod> -o yaml

Then compare:

Configured request
Configured limit
Observed usage
Application behavior

Troubleshooting CPU Throttling

If an application is slow:

Application latency increases
        │
        ▼
Check CPU usage
        │
        ▼
kubectl top pod
        │
        ▼
Inspect CPU limit
        │
        ▼
Check application metrics

Example:

resources:
  requests:
    cpu: "100m"

  limits:
    cpu: "200m"

If the application frequently requires more than:

200m CPU

it may experience CPU throttling.

Troubleshooting Unexpected BestEffort Pods

Check:

kubectl get pod <pod> -o yaml

Look for:

resources:

If containers have no CPU or memory resource configuration, the Pod may be:

BestEffort

Check:

kubectl get pod <pod> \
  -o jsonpath='{.status.qosClass}'

Troubleshooting Unexpected Burstable Pods

If you expected:

Guaranteed

but received:

Burstable

inspect every relevant container.

Check:

kubectl get pod <pod> -o yaml

Compare:

CPU request
CPU limit
Memory request
Memory limit

For a normal Guaranteed configuration, ensure the required request and limit values match for each relevant container.

Common Mistakes

Mistake 1: Setting Only Requests

Example:

resources:
  requests:
    cpu: "500m"
    memory: "256Mi"

This does not create the normal Guaranteed pattern.

It results in:

Burstable

Mistake 2: Setting Different Requests and Limits

Example:

resources:
  requests:
    cpu: "250m"
    memory: "128Mi"

  limits:
    cpu: "1"
    memory: "512Mi"

This is:

Burstable

not:

Guaranteed

Mistake 3: Configuring Only One Container

Consider:

Container A:
request == limit

Container B:
request != limit

The entire Pod does not become Guaranteed simply because one container has matching resources.

Always inspect all containers.

Mistake 4: Assuming QoS Controls Scheduling Priority

Incorrect:

Guaranteed → scheduled first

QoS is not a replacement for:

PriorityClass

The scheduler primarily considers resource requests and scheduling constraints.

Mistake 5: Assuming Guaranteed Means Unlimited

Guaranteed does not mean:

Unlimited resources

Example:

requests:
  memory: "256Mi"

limits:
  memory: "256Mi"

This is a Guaranteed-style configuration, but the container still has a configured memory limit.

Mistake 6: Ignoring Application Behavior

Setting:

memory:
  limit: "256Mi"

does not make the application efficient.

If the application genuinely requires:

500Mi

it may be repeatedly OOMKilled.

Resource configuration must match observed workload behavior.

QoS and Horizontal Pod Autoscaler

HPA can scale the number of Pod replicas based on metrics such as CPU utilization.

Example:

Deployment
    │
    ▼
HPA
    │
    ▼
More/Fewer Pods

Resource requests are especially important for CPU utilization calculations.

For example:

resources:
  requests:
    cpu: "500m"

A CPU utilization percentage can be evaluated relative to the CPU request.

Therefore resource requests can influence autoscaling behavior.

QoS and Vertical Pod Autoscaler

Vertical Pod Autoscaler (VPA) can recommend or adjust resource requests and limits depending on its configuration and mode.

Conceptually:

Observed usage
      │
      ▼
VPA
      │
      ▼
Resource recommendation
      │
      ▼
Requests/Limits
      │
      ▼
QoS may change

Changing requests and limits can change the Pod's QoS classification.

QoS and Cluster Autoscaler

Cluster Autoscaler primarily reacts to scheduling capacity and node utilization conditions.

QoS itself does not directly mean:

Guaranteed Pod → new node automatically created

However, resource requests affect whether Pods can be scheduled, and therefore can indirectly influence autoscaling decisions.

QoS and Node Pressure

The complete mental model:

             Pod
              │
      Requests + Limits
              │
              ▼
          QoS Class
              │
              ▼
      Runtime Resource Use
              │
              ▼
        Node Pressure
              │
              ▼
     Eviction / OOM Decisions

This is why QoS is important for DevOps and Kubernetes troubleshooting.

QoS Verification Commands

Get QoS

kubectl get pod <pod> \
  -o jsonpath='{.status.qosClass}'

Get Resources

kubectl get pod <pod> -o yaml

Describe Pod

kubectl describe pod <pod>

Check Usage

kubectl top pod <pod>

Check Node Usage

kubectl top node <node>

Check Node Conditions

kubectl describe node <node>

Check Events

kubectl get events --sort-by=.lastTimestamp

Useful JSONPath Commands

QoS

kubectl get pod <pod> \
  -o jsonpath='{.status.qosClass}'

CPU Requests

kubectl get pod <pod> \
  -o jsonpath='{.spec.containers[*].resources.requests.cpu}'

Memory Requests

kubectl get pod <pod> \
  -o jsonpath='{.spec.containers[*].resources.requests.memory}'

CPU Limits

kubectl get pod <pod> \
  -o jsonpath='{.spec.containers[*].resources.limits.cpu}'

Memory Limits

kubectl get pod <pod> \
  -o jsonpath='{.spec.containers[*].resources.limits.memory}'

One-Line QoS Inspection

For a single Pod:

kubectl get pod <pod> \
  -o custom-columns=NAME:.metadata.name,QOS:.status.qosClass

For all Pods:

kubectl get pods \
  -o custom-columns=NAME:.metadata.name,QOS:.status.qosClass

QoS Classification Examples

Example 1

resources:
  requests:
    cpu: "500m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "256Mi"

Result:

Guaranteed

Example 2

resources:
  requests:
    cpu: "250m"
    memory: "128Mi"

  limits:
    cpu: "500m"
    memory: "256Mi"

Result:

Burstable

Example 3

resources:
  requests:
    cpu: "500m"
    memory: "256Mi"

Result:

Burstable

Example 4

resources:
  limits:
    cpu: "500m"
    memory: "256Mi"

This is not automatically BestEffort; Kubernetes' effective resource configuration and defaulting behavior must be considered.

In the absence of namespace defaults, limits can contribute to the effective requests used by Kubernetes.

Example 5

containers:
  - name: app
    image: nginx

No CPU or memory resource configuration.

Result:

BestEffort

QoS Decision Table

CPU Request

CPU Limit

Memory Request

Memory Limit

Typical QoS

Equal

Equal

Equal

Equal

Guaranteed

Different

Different

Different

Different

Burstable

Configured

Not equal/missing

Configured

Not equal/missing

Burstable

None

None

None

None

BestEffort

Limit only

Limit only

Limit only

Limit only

Burstable under effective request/default semantics

For multi-container Pods, evaluate the complete Pod configuration rather than only one container.

QoS Mental Model

Remember:

NO resources
     │
     ▼
BestEffort

Resources configured
but not Guaranteed
     │
     ▼
Burstable

Required requests
match limits for
all relevant containers
     │
     ▼
Guaranteed

QoS in Production Architecture

A production Kubernetes environment may look like:

                         Kubernetes Cluster
                                │
             ┌──────────────────┴──────────────────┐
             │                                     │
          Node 1                                  Node 2
             │                                     │
       ┌─────┼─────┐                         ┌─────┼─────┐
       │     │     │                         │     │     │
      Pod   Pod   Pod                       Pod   Pod   Pod
       │     │     │                         │     │     │
       ▼     ▼     ▼                         ▼     ▼     ▼
    Guaranteed Burstable BestEffort       Guaranteed Burstable

Resource management exists at multiple layers:

Namespace
   │
   ├── ResourceQuota
   │
   ├── LimitRange
   │
   ▼
Pod
   │
   ├── Requests
   ├── Limits
   └── QoS

Best Practices

1. Always Understand Workload Requirements

Do not randomly choose:

CPU = 500m
Memory = 512Mi

Measure actual workload behavior.

2. Define Requests for Important Workloads

Requests help Kubernetes schedule workloads correctly.

Example:

requests:
  cpu: "250m"
  memory: "256Mi"

3. Define Memory Limits Carefully

Memory limits can protect a node from a runaway container, but setting them too low can cause:

OOMKilled

4. Be Careful With CPU Limits

CPU limits can introduce throttling.

For latency-sensitive applications, CPU limit strategy should be based on workload behavior and cluster policy rather than copied blindly.

5. Inspect Every Container

When troubleshooting QoS:

Application container
Sidecar
Init containers

should all be considered.

6. Use LimitRange Where Appropriate

Namespace defaults can help teams consistently define resource behavior.

7. Use ResourceQuota for Namespace Governance

ResourceQuota can prevent a namespace from consuming unlimited aggregate resources.

8. Monitor Actual Usage

Use:

kubectl top pods
kubectl top nodes

and, in production, use proper monitoring systems and application metrics.

Interview Questions

Q1. What is Kubernetes QoS?

Kubernetes QoS is a classification assigned to Pods based on their CPU and memory resource configuration.

The three classes are:

Guaranteed
Burstable
BestEffort

Q2. How does a Pod become Guaranteed?

For the standard Guaranteed classification, every relevant container must have the required CPU and memory requests and limits configured so that:

request == limit

for those resources.

Q3. How does a Pod become BestEffort?

A Pod is BestEffort when none of its containers has CPU or memory requests or limits.

Q4. What is Burstable QoS?

Burstable applies when the Pod has some CPU or memory resource configuration but does not meet the requirements for Guaranteed.

Q5. Does QoS apply to containers or Pods?

QoS is assigned to the:

Pod

Resource requests and limits are configured on:

Containers

Q6. Does QoS determine scheduling?

Not directly.

The scheduler primarily uses:

Resource requests

along with other scheduling constraints.

Q7. Which QoS class is most protected during resource pressure?

In general:

Guaranteed

receives stronger protection than Burstable and BestEffort, although actual eviction behavior depends on additional factors.

Q8. Which QoS class is most vulnerable?

Generally:

BestEffort

is the most vulnerable during resource pressure.

Q9. What happens when memory exceeds a container's limit?

The container may be terminated due to an out-of-memory condition.

You may see:

OOMKilled

Q10. What happens when CPU exceeds the limit?

CPU can be throttled rather than immediately killing the container.

Q11. Does Guaranteed mean the Pod can never be evicted?

No.

Guaranteed does not mean immortal.

Pods can still be terminated because of:

node failure

deletion

maintenance

controller actions

severe node conditions

other lifecycle events

Q12. What is the difference between QoS and PriorityClass?

QoS:

Guaranteed
Burstable
BestEffort

PriorityClass:

User-defined Pod priority

They are different mechanisms.

Q13. Why are requests important?

Requests are used heavily by the scheduler to determine whether a Pod can fit on a node.

Q14. Why are limits important?

Limits provide runtime resource boundaries.

CPU can be throttled at its limit, while memory overuse can lead to OOM termination.

Q15. How do you check a Pod's QoS class?

kubectl get pod <pod> \
  -o jsonpath='{.status.qosClass}'

Scenario-Based Interview Questions

Scenario 1: Pod is OOMKilled

You see:

Reason: OOMKilled

What do you check?

1. Memory limit
2. Actual memory usage
3. Application behavior
4. Previous container logs
5. Node memory pressure

Commands:

kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl top pod <pod>
kubectl describe node <node>

Scenario 2: Pod Is Burstable Instead of Guaranteed

Check:

kubectl get pod <pod> -o yaml

Inspect all relevant containers:

CPU request
CPU limit
Memory request
Memory limit

Find any mismatch such as:

CPU request = 250m
CPU limit   = 500m

That means the Pod does not satisfy the normal Guaranteed pattern.

Scenario 3: Application Is Slow

Check:

kubectl top pod <pod>

Then inspect:

resources:
  requests:
    cpu: ...
  limits:
    cpu: ...

Investigate CPU throttling, application metrics, node contention, and whether the configured limits are appropriate.

Scenario 4: Pod Is Pending

QoS itself is not usually the first thing to inspect.

Start with:

kubectl describe pod <pod>

Check:

Events

Then investigate:

CPU requests
Memory requests
Node capacity
Taints
Affinity
Selectors
Storage constraints

Scenario 5: Node Has MemoryPressure

Start with:

kubectl describe node <node>

Check:

MemoryPressure

Then inspect:

kubectl top node <node>
kubectl top pods -A

Identify high-memory workloads and investigate eviction/OOM events.

Important Production Mental Model

Do not think:

QoS = Performance Guarantee

Think:

QoS = Kubernetes Pod Resource Classification

And:

Requests = Scheduling signal
Limits   = Resource boundary
QoS      = Pod classification
Priority = Pod priority
Eviction = Resource-pressure response

These concepts work together but are not interchangeable.

End-to-End Resource Management Flow

Developer
   │
   ▼
Define Requests + Limits
   │
   ▼
Pod Specification
   │
   ▼
Kubernetes API
   │
   ▼
Scheduler uses Requests
   │
   ▼
Pod placed on Node
   │
   ▼
Container Runtime
   │
   ├── CPU enforcement
   └── Memory enforcement
   │
   ▼
Pod gets QoS classification
   │
   ▼
Node resource usage
   │
   ▼
Possible resource pressure
   │
   ▼
Eviction / OOM handling

Quick Revision

QoS Classes:
    Guaranteed
    Burstable
    BestEffort

Guaranteed:
    Required CPU request == CPU limit
    Required memory request == memory limit
    for all relevant containers

Burstable:
    Some CPU/memory resources configured
    but Pod does not qualify as Guaranteed

BestEffort:
    No CPU/memory requests or limits

Quick Command Cheat Sheet

# List Pods
kubectl get pods

# Describe Pod
kubectl describe pod <pod>

# Get Pod YAML
kubectl get pod <pod> -o yaml

# Get QoS
kubectl get pod <pod> \
  -o jsonpath='{.status.qosClass}'

# Get all Pod QoS classes
kubectl get pods \
  -o custom-columns=NAME:.metadata.name,QOS:.status.qosClass

# Check resource usage
kubectl top pod

# Check node usage
kubectl top nodes

# Check node details
kubectl describe node <node>

# Check events
kubectl get events --sort-by=.lastTimestamp

# Current logs
kubectl logs <pod>

# Previous container logs
kubectl logs <pod> --previous

QoS Cheat Sheet Table

Concept

Meaning

Request

Resource amount used heavily for scheduling

Limit

Maximum resource boundary for a container

Guaranteed

Strongest QoS classification

Burstable

Intermediate QoS classification

BestEffort

Lowest QoS classification

OOMKilled

Container terminated due to out-of-memory

CPU throttling

CPU usage restricted by CPU limit

ResourceQuota

Namespace-level aggregate resource control

LimitRange

Namespace-level resource defaults/constraints

PriorityClass

Pod priority mechanism

Eviction

Pod termination due to node resource pressure

Final Takeaways

Kubernetes assigns every Pod a QoS class.

The three QoS classes are Guaranteed, Burstable, and BestEffort.

QoS classification is based on CPU and memory resource configuration.

Requests are important for scheduling.

Limits define runtime resource boundaries.

CPU overuse can result in throttling.

Memory overuse can result in OOM termination.

Guaranteed normally requires matching CPU and memory requests and limits for all relevant containers.

Burstable is used when resource configuration exists but the Pod does not satisfy Guaranteed requirements.

BestEffort applies when no container has CPU or memory requests or limits.

QoS is assigned to Pods, while resources are configured on containers.

QoS is not the same as Pod priority.

QoS is not a guarantee that a Pod can never be terminated.

QoS class alone does not determine complete eviction behavior.

kubectl describe pod is useful for troubleshooting resource-related problems.

kubectl top pod helps inspect current resource usage.

OOMKilled should trigger investigation of memory usage, limits, application behavior, and node pressure.

Resource values should be based on measured workload behavior.

LimitRange can provide namespace-level resource defaults and constraints.

ResourceQuota controls aggregate resource consumption at the namespace level.

Production workloads should use deliberate resource requests and limits rather than relying blindly on defaults.

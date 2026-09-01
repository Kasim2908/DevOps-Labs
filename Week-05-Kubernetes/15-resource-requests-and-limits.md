# Kubernetes Resource Requests and Limits — Complete Notes

> Complete Kubernetes **Resource Requests and Limits** notes from **basics to advanced level**, including CPU, Memory, QoS Classes, scheduling, OOMKilled, LimitRange, ResourceQuota, best practices, troubleshooting, practical examples, and interview questions.

---

# 1. What are Resource Requests and Limits?

Kubernetes allows you to define how much **CPU and memory** a container needs and how much it is allowed to consume.

There are two important concepts:

```text
Requests → Minimum resources needed by the container

Limits   → Maximum resources the container can use
```

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

This means:

```text
CPU:
Request → 250m
Limit   → 500m

Memory:
Request → 256Mi
Limit   → 512Mi
```

---

# 2. Why Do We Need Requests and Limits?

Without resource management, one container could consume too many resources.

For example:

```text
Node
 |
 +-- Pod A → 100Mi
 |
 +-- Pod B → 200Mi
 |
 +-- Pod C → 7Gi
 |
 +-- Pod D → 500Mi
```

If Pod C suddenly consumes huge amounts of memory:

```text
Pod C
  ↓
High Memory Usage
  ↓
Node Memory Pressure
  ↓
Other Pods affected
```

Requests and limits help Kubernetes manage resources more predictably.

---

# 3. CPU and Memory Resources

Kubernetes primarily manages:

```text
CPU
Memory
```

Additional resources can also be managed, such as:

```text
Ephemeral Storage
Extended Resources
GPUs
Device Resources
```

For basic workloads, CPU and memory are the most important.

---

# 4. Resource Request

A **request** tells Kubernetes how much resource a container needs for scheduling.

Example:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
```

This means:

```text
CPU → 500 millicores
Memory → 512 MiB
```

Kubernetes uses the request when deciding:

> "Which node has enough available resources to run this Pod?"

---

# 5. Resource Limit

A **limit** defines the maximum amount of a resource a container is allowed to consume.

Example:

```yaml
resources:
  limits:
    cpu: "1"
    memory: "1Gi"
```

This means:

```text
CPU → Maximum 1 CPU
Memory → Maximum 1 GiB
```

---

# 6. Requests vs Limits

| Feature                       | Request                 | Limit                   |
| ----------------------------- | ----------------------- | ----------------------- |
| Purpose                       | Scheduling              | Maximum usage           |
| Used by Scheduler             | Yes                     | No, primarily           |
| Defines guaranteed allocation | Yes                     | No                      |
| CPU behavior                  | Reserved for scheduling | Throttled when exceeded |
| Memory behavior               | Used for scheduling     | Can cause OOM kill      |
| Can be equal                  | Yes                     | Yes                     |
| Can be different              | Yes                     | Yes                     |

### Easy way to remember

```text
Request → "I need this much."

Limit → "Don't let me use more than this."
```

---

# 7. Basic YAML Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: resource-demo

spec:
  containers:
    - name: app
      image: nginx

      resources:
        requests:
          cpu: "250m"
          memory: "256Mi"

        limits:
          cpu: "500m"
          memory: "512Mi"
```

---

# 8. CPU Units

Kubernetes CPU can be specified as:

```text
1
500m
250m
100m
50m
```

Where:

```text
1 CPU = 1000m
```

Therefore:

```text
1000m = 1 CPU
500m  = 0.5 CPU
250m  = 0.25 CPU
100m  = 0.1 CPU
50m   = 0.05 CPU
```

---

# 9. CPU Examples

```yaml
cpu: "1"
```

means:

```text
1 CPU
```

```yaml
cpu: "500m"
```

means:

```text
0.5 CPU
```

```yaml
cpu: "250m"
```

means:

```text
0.25 CPU
```

---

# 10. Memory Units

Memory can be specified using units such as:

```text
Mi
Gi
M
G
Ki
Ti
```

Common Kubernetes usage:

```text
128Mi
256Mi
512Mi
1Gi
2Gi
4Gi
```

---

# 11. Mi vs M

Be careful with:

```text
Mi
M
```

They use different unit conventions.

Similarly:

```text
Gi
G
```

are different.

For Kubernetes manifests, binary units such as:

```text
Mi
Gi
```

are commonly used for memory.

---

# 12. Request and Limit Together

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

Interpretation:

```text
              Request        Limit

CPU           250m           500m
Memory        256Mi          512Mi
```

---

# 13. What Happens During Scheduling?

Suppose a node has:

```text
CPU = 4
Memory = 8Gi
```

Pod requests:

```text
CPU = 1
Memory = 2Gi
```

Kubernetes checks whether the node has sufficient allocatable resources.

If yes:

```text
Pod
 ↓
Scheduler
 ↓
Node selected
```

The scheduler primarily considers **resource requests**, not actual current usage.

---

# 14. Example Scheduling

Node:

```text
CPU Capacity: 4
CPU Requested: 3
```

New Pod:

```text
CPU Request: 2
```

Available scheduling capacity:

```text
4 - 3 = 1 CPU
```

New Pod needs:

```text
2 CPU
```

Therefore:

```text
Pod → Cannot be scheduled on this node
```

Even if actual CPU usage happens to be low.

---

# 15. Requests Are Not the Same as Actual Usage

Suppose:

```yaml
requests:
  cpu: "500m"
```

The application may currently use:

```text
100m
```

or:

```text
400m
```

or:

```text
500m
```

The request is primarily a **scheduling reservation/expectation**, not a statement that the process always consumes exactly that amount.

---

# 16. What Happens When CPU Exceeds the Limit?

CPU is a **compressible resource**.

If a container reaches its CPU limit, Kubernetes/container runtime can throttle CPU usage.

Example:

```text
CPU Limit = 500m

Application tries to use:
800m
```

The container is generally:

```text
CPU throttled
```

rather than immediately killed.

---

# 17. What Happens When Memory Exceeds the Limit?

Memory is a **non-compressible resource**.

If a container exceeds its memory limit, it can be terminated by the kernel/container runtime.

Typical result:

```text
OOMKilled
```

OOM means:

```text
Out Of Memory
```

---

# 18. CPU vs Memory Behavior

| Resource | Exceed Limit            |
| -------- | ----------------------- |
| CPU      | Throttling              |
| Memory   | Container may be killed |
| CPU      | Compressible            |
| Memory   | Non-compressible        |

### Important

```text
CPU → Throttle

Memory → OOMKill
```

---

# 19. OOMKilled

If a container exceeds its memory limit:

```text
Application
   ↓
Memory usage increases
   ↓
Memory limit exceeded
   ↓
OOM
   ↓
Container killed
```

Check:

```bash
kubectl describe pod <pod-name>
```

You may see:

```text
Reason: OOMKilled
```

Also:

```bash
kubectl get pod <pod-name>
```

---

# 20. Restart After OOMKilled

If the Pod's container is managed by a controller such as a Deployment, Kubernetes may restart the container.

Example:

```text
Container
   ↓
OOMKilled
   ↓
Restart
   ↓
Application starts again
```

If the application repeatedly exceeds its memory limit:

```text
OOMKilled
   ↓
Restart
   ↓
OOMKilled
   ↓
Restart
```

This can result in:

```text
CrashLoopBackOff
```

---

# 21. Requests and Limits in Deployments

Example:

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
        - name: web
          image: nginx

          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"

            limits:
              cpu: "500m"
              memory: "512Mi"
```

Each Pod receives:

```text
CPU Request = 250m
Memory Request = 256Mi
```

For 3 replicas:

```text
Total CPU Request:
3 × 250m = 750m

Total Memory Request:
3 × 256Mi = 768Mi
```

---

# 22. Requests and Limits Are Per Container

This is an important interview concept.

Suppose a Pod has:

```text
Container A
Container B
```

Container A:

```text
CPU Request = 250m
```

Container B:

```text
CPU Request = 500m
```

Pod total:

```text
250m + 500m = 750m
```

Resource requests and limits are specified **per container**, while scheduling considers the aggregate resources required by the Pod.

---

# 23. Multi-Container Pod Example

```yaml
spec:
  containers:

    - name: app
      image: nginx

      resources:
        requests:
          cpu: "250m"
          memory: "256Mi"

        limits:
          cpu: "500m"
          memory: "512Mi"

    - name: sidecar
      image: busybox

      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"

        limits:
          cpu: "200m"
          memory: "256Mi"
```

Pod total request:

```text
CPU:
250m + 100m = 350m

Memory:
256Mi + 128Mi = 384Mi
```

---

# 24. QoS Classes

Kubernetes assigns Pods a Quality of Service class.

There are three main classes:

```text
Guaranteed
Burstable
BestEffort
```

QoS affects how Kubernetes treats Pods during resource pressure.

---

# 25. Guaranteed QoS

A Pod is generally **Guaranteed** when every container has CPU and memory requests and limits set, and for those resources:

```text
Request = Limit
```

Example:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

---

# 26. Guaranteed Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: guaranteed

spec:
  containers:
    - name: app
      image: nginx

      resources:
        requests:
          cpu: "500m"
          memory: "512Mi"

        limits:
          cpu: "500m"
          memory: "512Mi"
```

Check QoS:

```bash
kubectl get pod guaranteed \
-o jsonpath='{.status.qosClass}'
```

Expected:

```text
Guaranteed
```

---

# 27. Burstable QoS

A Pod is generally **Burstable** when it has resource requests/limits configured but does not meet the Guaranteed criteria.

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

Here:

```text
Request ≠ Limit
```

So the Pod can be Burstable.

---

# 28. BestEffort QoS

A Pod is generally **BestEffort** when none of its containers have CPU or memory requests or limits.

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: besteffort

spec:
  containers:
    - name: app
      image: nginx
```

No:

```yaml
resources:
```

Therefore:

```text
QoS = BestEffort
```

---

# 29. QoS Comparison

| QoS        | Requests                       | Limits                         | Typical Priority During Memory Pressure |
| ---------- | ------------------------------ | ------------------------------ | --------------------------------------- |
| Guaranteed | Set                            | Set                            | Lower eviction risk                     |
| Burstable  | Some/appropriate configuration | Some/appropriate configuration | Medium                                  |
| BestEffort | Not set                        | Not set                        | Higher eviction risk                    |

> QoS is one factor in eviction decisions; actual eviction behavior depends on resource pressure and other Kubernetes rules.

---

# 30. Check QoS Class

```bash
kubectl get pod <pod-name> \
-o jsonpath='{.status.qosClass}'
```

Example:

```text
Burstable
```

---

# 31. Resource Requests and Horizontal Pod Autoscaler

HPA often uses resource requests when calculating utilization.

For example:

```text
CPU Request = 500m
Current CPU = 250m
```

CPU utilization relative to request:

```text
250 / 500 × 100
= 50%
```

If HPA target is:

```text
50%
```

the utilization is approximately:

```text
50%
```

---

# 32. Why Requests Matter for HPA

Suppose:

```yaml
resources:
  requests:
    cpu: "500m"
```

and the Pod uses:

```text
250m
```

Then:

```text
CPU utilization ≈ 50%
```

If the CPU request is incorrectly set extremely low, utilization can appear extremely high.

Therefore:

> Proper resource requests are important for meaningful HPA behavior.

---

# 33. LimitRange

A **LimitRange** allows administrators to define default and boundary resource settings for containers in a namespace.

It can define:

```text
Default request
Default limit
Minimum request
Maximum request
```

Example:

```yaml
apiVersion: v1
kind: LimitRange

metadata:
  name: resource-limits

spec:
  limits:
    - type: Container

      default:
        cpu: "500m"
        memory: "512Mi"

      defaultRequest:
        cpu: "250m"
        memory: "256Mi"

      max:
        cpu: "2"
        memory: "2Gi"

      min:
        cpu: "100m"
        memory: "128Mi"
```

---

# 34. Why Use LimitRange?

Without a LimitRange, developers may create:

```text
Pod A → no limits
Pod B → huge memory limit
Pod C → tiny request
```

A LimitRange establishes namespace-level guardrails.

---

# 35. LimitRange Example

```text
Namespace: development

LimitRange:
-------------------------
Minimum CPU: 100m
Maximum CPU: 2
Default CPU: 500m
-------------------------
Minimum Memory: 128Mi
Maximum Memory: 2Gi
Default Memory: 512Mi
```

If a developer creates a container without resources, defaults can be applied according to the LimitRange configuration.

---

# 36. Check LimitRange

```bash
kubectl get limitrange
```

Detailed:

```bash
kubectl describe limitrange <name>
```

Namespace-specific:

```bash
kubectl get limitrange -n development
```

---

# 37. ResourceQuota

A **ResourceQuota** controls the total resource consumption allowed in a namespace.

Example:

```yaml
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
```

This applies to the namespace as a whole.

---

# 38. LimitRange vs ResourceQuota

| LimitRange                    | ResourceQuota                |
| ----------------------------- | ---------------------------- |
| Per container/Pod constraints | Namespace-wide quota         |
| Sets defaults                 | Controls total consumption   |
| Defines min/max               | Defines aggregate limits     |
| Protects individual workloads | Protects namespace resources |

### Easy Memory Trick

```text
LimitRange
    ↓
Individual Container

ResourceQuota
    ↓
Entire Namespace
```

---

# 39. ResourceQuota Example

Suppose:

```text
Namespace CPU quota = 4 CPUs
```

Existing workloads request:

```text
3 CPUs
```

New Pod requests:

```text
2 CPUs
```

Total would become:

```text
3 + 2 = 5 CPUs
```

This exceeds the quota.

The new workload may be rejected.

---

# 40. ResourceQuota with Object Counts

ResourceQuota can also limit object counts.

Example:

```yaml
spec:
  hard:
    pods: "20"
    services: "10"
    secrets: "30"
    configmaps: "30"
```

This helps prevent uncontrolled resource creation.

---

# 41. ResourceQuota Commands

Check:

```bash
kubectl get resourcequota
```

Detailed:

```bash
kubectl describe resourcequota compute-quota
```

Example output may show:

```text
Resource        Used      Hard
requests.cpu    2         4
requests.memory 4Gi       8Gi
```

---

# 42. Namespace Resource Management

A production namespace may use both:

```text
LimitRange
+
ResourceQuota
```

Architecture:

```text
Namespace
   |
   +── ResourceQuota
   |       ↓
   |   Total Resource Limit
   |
   +── LimitRange
           ↓
       Per Container Defaults/Bounds
```

---

# 43. Resource Requests and Node Allocatable

A node has:

```text
Capacity
Allocatable
```

Example:

```text
Capacity:
CPU = 8
Memory = 32Gi

Allocatable:
CPU = 7.5
Memory = 30Gi
```

Kubernetes schedules workloads against the resources available for Pods, generally reflected by **allocatable** resources.

---

# 44. Capacity vs Allocatable

```text
Capacity
   ↓
Total node resources

Allocatable
   ↓
Resources available for Kubernetes workloads
```

Some resources are reserved for:

```text
Kubernetes system components
Operating system
Node agents
Other reservations
```

---

# 45. Check Node Resources

```bash
kubectl describe node <node-name>
```

Look for:

```text
Capacity
Allocatable
Allocated resources
```

You can also use:

```bash
kubectl top node
```

if Metrics Server is installed.

---

# 46. Check Pod Resource Usage

```bash
kubectl top pod
```

For a namespace:

```bash
kubectl top pod -n production
```

For a specific Pod:

```bash
kubectl top pod <pod-name>
```

This shows current observed usage, not requests/limits.

---

# 47. Requests vs Actual Usage

Suppose:

```text
Request = 500m
Limit = 1 CPU
Actual Usage = 100m
```

Then:

```text
Request → 500m
Limit   → 1 CPU
Usage   → 100m
```

These are three different concepts.

---

# 48. Resource Overcommitment

Kubernetes can allow:

```text
Sum of limits > Node capacity
```

This is called **overcommitment**.

Example:

```text
Node memory = 8Gi

Pod A limit = 4Gi
Pod B limit = 4Gi
Pod C limit = 4Gi
```

Total limits:

```text
12Gi
```

while node has:

```text
8Gi
```

This can be valid from a scheduling perspective depending on requests, but creates risk if all workloads try to consume their limits simultaneously.

---

# 49. Why Requests Are Important for Overcommitment

Suppose:

```text
Node = 8Gi

Pod A:
Request = 1Gi
Limit = 4Gi

Pod B:
Request = 1Gi
Limit = 4Gi

Pod C:
Request = 1Gi
Limit = 4Gi
```

Requests total:

```text
3Gi
```

Limits total:

```text
12Gi
```

Kubernetes can schedule based on the requests, but actual usage could eventually exceed available memory.

This is why resource sizing matters.

---

# 50. CPU Overcommitment

CPU is different because it can be throttled.

Example:

```text
Node = 4 CPU

Pod A limit = 2 CPU
Pod B limit = 2 CPU
Pod C limit = 2 CPU
```

Total limits:

```text
6 CPU
```

During high demand, CPU time is shared and containers can be throttled according to their configured limits and runtime behavior.

---

# 51. Memory Pressure

If a node experiences memory pressure:

```text
Node
 ↓
Memory pressure
 ↓
Kubelet eviction logic
 ↓
Pods may be evicted
```

QoS class and Pod priority are among the factors involved in eviction behavior.

---

# 52. Requests and Eviction

A common misconception:

> "If I set a memory request, Kubernetes guarantees my Pod will never be killed."

Not necessarily.

Resource requests influence scheduling and QoS/eviction behavior, but they are **not an absolute guarantee against every type of failure or termination**.

Possible causes of termination include:

```text
Node failure
OOM
Eviction
Application crash
Manual deletion
Deployment update
```

---

# 53. Resource Starvation

Without proper requests and limits:

```text
Pod A → consumes huge CPU
Pod B → starved
Pod C → slow
Pod D → unstable
```

With proper resource management:

```text
Pod A → controlled
Pod B → predictable
Pod C → predictable
Pod D → predictable
```

---

# 54. Practical Example

Suppose your application normally uses:

```text
CPU = 200m
Memory = 300Mi
```

You might start with:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "384Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

These values should be based on actual workload measurements, not blindly copied.

---

# 55. How to Choose Requests and Limits

A good process:

```text
Deploy
   ↓
Measure
   ↓
Observe usage
   ↓
Set requests
   ↓
Set limits
   ↓
Monitor
   ↓
Tune
```

Useful data sources:

```text
kubectl top
Prometheus
Grafana
Cloud monitoring
Application metrics
Load testing
```

---

# 56. Avoid Setting Extremely High Requests

Example:

```yaml
requests:
  cpu: "8"
  memory: "16Gi"
```

for a tiny application.

This may cause:

```text
Poor bin packing
More unschedulable Pods
Higher infrastructure cost
Lower cluster utilization
```

---

# 57. Avoid Setting Extremely Low Requests

Example:

```yaml
requests:
  cpu: "1m"
  memory: "8Mi"
```

for an application that normally needs:

```text
500m CPU
1Gi memory
```

This can cause:

```text
Bad scheduling assumptions
Poor HPA calculations
Resource contention
Higher eviction risk under pressure
```

---

# 58. Memory Limit Best Practice

Do not blindly set memory limits.

If the limit is too low:

```text
Application
 ↓
Normal traffic spike
 ↓
Memory > limit
 ↓
OOMKilled
```

Measure actual application behavior first.

---

# 59. CPU Limit Best Practice

CPU limits can protect a node from one container consuming unlimited CPU.

However, overly restrictive CPU limits can cause:

```text
CPU throttling
Latency
Poor application performance
```

Therefore, CPU limits should be chosen based on workload behavior.

---

# 60. Resource Configuration in Production

A typical production container might look like:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"

  limits:
    cpu: "1"
    memory: "1Gi"
```

But these values are **examples**, not universal recommendations.

---

# 61. Init Container Resources

Init containers also support resources.

Example:

```yaml
initContainers:
  - name: init
    image: busybox

    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"

      limits:
        cpu: "200m"
        memory: "256Mi"
```

Init containers have special resource accounting rules because they run sequentially.

---

# 62. Ephemeral Storage Requests and Limits

Containers can also specify ephemeral storage.

Example:

```yaml
resources:
  requests:
    ephemeral-storage: "1Gi"

  limits:
    ephemeral-storage: "2Gi"
```

This relates to local writable storage used by the container and things such as:

```text
Container writable layer
emptyDir
Logs
```

depending on configuration and runtime behavior.

---

# 63. Ephemeral Storage Eviction

If a node runs out of local ephemeral storage:

```text
Disk pressure
   ↓
Kubelet eviction
   ↓
Pods may be evicted
```

Therefore production workloads should consider ephemeral storage as well.

---

# 64. Extended Resources

Kubernetes can schedule special resources such as GPUs.

Example concept:

```yaml
resources:
  limits:
    nvidia.com/gpu: "1"
```

The exact resource name depends on the device plugin.

---

# 65. Resource Names

Standard resources include:

```text
cpu
memory
ephemeral-storage
```

Extended resources can look like:

```text
example.com/my-resource
```

For example:

```yaml
resources:
  limits:
    example.com/fpga: "1"
```

---

# 66. Resource Requests Without Limits

You can specify:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
```

without explicit limits.

This makes scheduling expectations clear, but actual runtime behavior depends on namespace policies and Kubernetes configuration.

A LimitRange may also automatically provide defaults.

---

# 67. Limits Without Requests

You can specify limits without explicit requests.

In Kubernetes, if a limit is specified but the request is not, the request may be defaulted to the limit for that resource.

For example:

```yaml
resources:
  limits:
    cpu: "500m"
```

can result in an effective CPU request of:

```text
500m
```

depending on applicable Kubernetes defaults and LimitRange configuration.

---

# 68. ResourceQuota and Missing Requests

A namespace may require resource requests/limits through quota configuration.

For example, a ResourceQuota may specify:

```yaml
hard:
  requests.cpu: "4"
  requests.memory: "8Gi"
```

If a Pod does not provide required resource requests and no applicable defaults exist, creation may be rejected.

Always check:

```bash
kubectl describe resourcequota
kubectl get limitrange
```

when a Pod is unexpectedly rejected.

---

# 69. Troubleshooting Pod Pending

A Pod may remain:

```text
Pending
```

because its resource requests cannot fit on available nodes.

Check:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
Insufficient cpu
Insufficient memory
```

---

# 70. Example: Insufficient CPU

Node:

```text
Allocatable CPU = 4
```

Existing requests:

```text
3.5 CPU
```

New Pod:

```text
CPU request = 1 CPU
```

Required:

```text
3.5 + 1 = 4.5 CPU
```

Available:

```text
4 CPU
```

Result:

```text
Pod → Pending
```

---

# 71. Troubleshooting OOMKilled

Run:

```bash
kubectl describe pod <pod-name>
```

Check:

```text
Last State:
  Reason: OOMKilled
```

Then investigate:

```text
Current memory usage
Memory limit
Application memory behavior
Traffic/load
Memory leaks
JVM/runtime configuration
```

---

# 72. Check Resource Configuration

```bash
kubectl get pod <pod-name> -o yaml
```

Look for:

```yaml
resources:
  requests:
  limits:
```

---

# 73. Check Current Usage

```bash
kubectl top pod <pod-name>
```

For containers:

```bash
kubectl top pod <pod-name> --containers
```

This can help compare current usage against configured resources.

---

# 74. Troubleshooting Flow

```text
Pod Problem
     |
     v
kubectl describe pod
     |
     +------------------+
     |                  |
     v                  v
Pending              OOMKilled
     |                  |
     v                  v
Check Requests      Check Memory
     |              Limit/Usage
     v                  |
Insufficient CPU/       v
Memory               Increase/Tune
     |
     v
Check Node
Resources
```

---

# 75. Common Mistakes

## Mistake 1 — Thinking Request Is a Limit

```text
Request = 500m
```

does not mean:

```text
Container can never use more than 500m
```

It means the container requests that amount for scheduling.

---

## Mistake 2 — Thinking Limit Guarantees Usage

```text
limit:
  cpu: "1"
```

does not mean the container automatically receives 1 CPU.

It means it can use up to that configured limit, subject to node/runtime conditions.

---

## Mistake 3 — Setting Memory Limits Too Low

This can cause:

```text
OOMKilled
```

---

## Mistake 4 — Ignoring Requests

Bad requests can cause:

```text
Poor scheduling
HPA inaccuracies
Resource contention
```

---

## Mistake 5 — Using the Same Values Everywhere

Different workloads need different resources.

For example:

```text
Nginx
Redis
Java application
Python API
Database
```

may have completely different resource profiles.

---

# 76. Best Practices

### 1. Always Measure

Use:

```bash
kubectl top pod
```

and monitoring tools.

### 2. Set Reasonable Requests

Requests should represent realistic resource requirements.

### 3. Set Appropriate Limits

Limits should prevent runaway consumption without unnecessarily throttling or killing healthy workloads.

### 4. Use LimitRange

Set namespace-level defaults and boundaries.

### 5. Use ResourceQuota

Control total namespace resource consumption.

### 6. Monitor OOMKilled

Repeated OOM kills indicate that resource configuration or application behavior needs investigation.

### 7. Review Resources After Load Testing

Resource requirements can change significantly under production traffic.

---

# 77. Complete Production Example

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api

spec:
  replicas: 3

  selector:
    matchLabels:
      app: api

  template:
    metadata:
      labels:
        app: api

    spec:
      containers:
        - name: api
          image: my-api:v1

          ports:
            - containerPort: 8080

          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"

            limits:
              cpu: "1"
              memory: "1Gi"
```

Resource requirement for 3 replicas:

```text
CPU Requests:
3 × 250m = 750m

Memory Requests:
3 × 512Mi = 1536Mi
```

Maximum configured limits:

```text
CPU:
3 × 1 = 3 CPU

Memory:
3 × 1Gi = 3Gi
```

---

# 78. ResourceQuota Example

```yaml
apiVersion: v1
kind: ResourceQuota

metadata:
  name: production-quota

spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"

    limits.cpu: "20"
    limits.memory: "40Gi"

    pods: "20"
```

This controls the aggregate resource usage allowed in the namespace.

---

# 79. LimitRange Example

```yaml
apiVersion: v1
kind: LimitRange

metadata:
  name: production-defaults

spec:
  limits:
    - type: Container

      default:
        cpu: "1"
        memory: "1Gi"

      defaultRequest:
        cpu: "250m"
        memory: "256Mi"

      min:
        cpu: "100m"
        memory: "128Mi"

      max:
        cpu: "4"
        memory: "8Gi"
```

---

# 80. Resource Management Architecture

```text
                    Kubernetes Cluster
                           |
             +-------------+-------------+
             |                           |
             v                           v
           Node                      Namespace
             |                           |
             |                    +------+------+
             |                    |             |
             v                    v             v
      Allocatable Resources   LimitRange   ResourceQuota
             |                    |             |
             v                    |             |
          Scheduler               |             |
             |                    |             |
             v                    |             |
            Pod <-----------------+-------------+
             |
             v
         Container
             |
       +-----+------+
       |            |
       v            v
    Requests      Limits
       |            |
       v            v
  Scheduling   Runtime Control
                  |
          +-------+-------+
          |               |
          v               v
       CPU              Memory
    Throttling         OOMKill
```

---

# 81. Interview Questions

## Q1. What is a Resource Request?

**Answer:**

A resource request specifies the amount of CPU or memory a container needs for scheduling. The Kubernetes scheduler uses requests when selecting a suitable node.

---

## Q2. What is a Resource Limit?

**Answer:**

A resource limit defines the maximum amount of a resource a container is allowed to consume.

CPU exceeding the limit can result in throttling, while memory exceeding the limit can result in an OOM kill.

---

## Q3. What is the difference between Requests and Limits?

**Answer:**

```text
Request → Used primarily for scheduling
Limit   → Maximum resource usage boundary
```

Requests determine whether a Pod can fit on a node, while limits constrain runtime consumption.

---

## Q4. What happens when a container exceeds its CPU limit?

**Answer:**

CPU is a compressible resource, so the container can be **CPU throttled**.

It is not normally killed simply because it exceeds its CPU limit.

---

## Q5. What happens when a container exceeds its memory limit?

**Answer:**

Memory is a non-compressible resource. If the container exceeds its memory limit, it can be terminated with:

```text
OOMKilled
```

---

## Q6. What are Kubernetes QoS Classes?

**Answer:**

There are three main QoS classes:

```text
Guaranteed
Burstable
BestEffort
```

They influence Pod behavior during resource pressure.

---

## Q7. What is a Guaranteed Pod?

**Answer:**

A Pod is generally Guaranteed when every container has CPU and memory requests and limits configured, and the request equals the limit for those resources.

Example:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

---

## Q8. What is a Burstable Pod?

**Answer:**

A Pod is generally Burstable when it has resource requests/limits but does not satisfy the requirements for Guaranteed QoS.

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

---

## Q9. What is a BestEffort Pod?

**Answer:**

A Pod is BestEffort when none of its containers have CPU or memory requests or limits.

Example:

```yaml
containers:
  - name: app
    image: nginx
```

---

## Q10. What is LimitRange?

**Answer:**

LimitRange is a namespace-level policy that can define:

* Default requests
* Default limits
* Minimum resources
* Maximum resources

It helps enforce consistent resource configuration for containers.

---

## Q11. What is ResourceQuota?

**Answer:**

ResourceQuota limits the **aggregate resource consumption of a namespace**.

It can control:

```text
CPU
Memory
Pods
Services
Secrets
ConfigMaps
```

and other quota-supported resources.

---

## Q12. What is the difference between LimitRange and ResourceQuota?

**Answer:**

```text
LimitRange
    ↓
Per Container / Pod Defaults & Boundaries

ResourceQuota
    ↓
Total Namespace Consumption
```

---

## Q13. Why is a Pod stuck in Pending because of resources?

**Answer:**

The Pod may have resource requests that cannot fit on any available node.

Check:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
Insufficient cpu
Insufficient memory
```

---

## Q14. What is OOMKilled?

**Answer:**

OOMKilled means the container was terminated because it ran out of memory according to the kernel/container runtime's OOM handling.

In Kubernetes, exceeding a container's memory limit is a common cause.

---

## Q15. How do you troubleshoot OOMKilled?

**Answer:**

First:

```bash
kubectl describe pod <pod-name>
```

Then check:

```bash
kubectl top pod <pod-name>
```

Investigate:

```text
Memory limit
Actual memory usage
Application memory consumption
Traffic/load
Memory leaks
Runtime configuration
```

---

# 82. Scenario-Based Interview Questions

## Scenario 1: Pod is Pending

You created:

```yaml
resources:
  requests:
    cpu: "4"
    memory: "8Gi"
```

but the Pod remains Pending.

### What would you check?

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
Insufficient cpu
Insufficient memory
```

Then:

```bash
kubectl describe node <node-name>
```

Check:

```text
Capacity
Allocatable
Allocated resources
```

---

# 83. Scenario 2: Application Keeps Restarting

You see:

```text
CrashLoopBackOff
```

and:

```text
OOMKilled
```

### What could be the reason?

The container may be exceeding its memory limit.

Check:

```bash
kubectl describe pod <pod-name>
kubectl top pod <pod-name>
```

Then determine whether to:

```text
Increase memory limit
Increase request
Optimize application memory
Investigate memory leak
```

Do not simply increase memory without understanding the application's behavior.

---

# 84. Scenario 3: CPU Usage Is High

Your application uses:

```text
CPU Request = 250m
CPU Limit = 500m
```

and constantly reaches:

```text
500m
```

### What could happen?

The container may experience CPU throttling.

Investigate:

```text
CPU usage
CPU throttling
Application latency
Traffic
CPU request/limit sizing
```

---

# 85. Scenario 4: HPA Is Scaling Unexpectedly

Suppose:

```text
CPU Request = 50m
```

but actual usage is:

```text
100m
```

Utilization relative to request:

```text
100 / 50 × 100
= 200%
```

The HPA may consider CPU utilization very high.

Therefore:

> Incorrectly low CPU requests can lead to misleading HPA utilization.

---

# 86. Scenario 5: Developer Creates Pods Without Resources

The administrator wants every container to have sensible defaults.

### Solution

Use:

```text
LimitRange
```

For example:

```yaml
default:
  cpu: "500m"
  memory: "512Mi"

defaultRequest:
  cpu: "250m"
  memory: "256Mi"
```

---

# 87. Scenario 6: Namespace Consumes Too Many Resources

The platform team wants to limit the total namespace consumption.

### Solution

Use:

```text
ResourceQuota
```

Example:

```yaml
hard:
  requests.cpu: "10"
  requests.memory: "20Gi"
  limits.cpu: "20"
  limits.memory: "40Gi"
```

---

# 88. Quick Revision

```text
Requests
    ↓
Scheduling

Limits
    ↓
Maximum Runtime Usage

CPU
    ↓
Throttle

Memory
    ↓
OOMKill

QoS:
    Guaranteed
    Burstable
    BestEffort

LimitRange
    ↓
Per Container Defaults / Min / Max

ResourceQuota
    ↓
Namespace Total

kubectl top
    ↓
Current Usage

kubectl describe pod
    ↓
Troubleshooting
```

---

# 89. Important Commands

```bash
# View Pods
kubectl get pods

# Detailed Pod information
kubectl describe pod <pod-name>

# View Pod YAML
kubectl get pod <pod-name> -o yaml

# Current Pod resource usage
kubectl top pod

# Current Node resource usage
kubectl top node

# View node capacity and allocatable resources
kubectl describe node <node-name>

# View LimitRanges
kubectl get limitrange

# Describe LimitRange
kubectl describe limitrange <name>

# View ResourceQuotas
kubectl get resourcequota

# Describe ResourceQuota
kubectl describe resourcequota <name>

# Check Pod QoS
kubectl get pod <pod-name> \
-o jsonpath='{.status.qosClass}'
```

---

# 90. Resource Units Cheat Sheet

## CPU

```text
1 CPU  = 1000m
500m   = 0.5 CPU
250m   = 0.25 CPU
100m   = 0.1 CPU
50m    = 0.05 CPU
```

## Memory

```text
128Mi
256Mi
512Mi
1Gi
2Gi
4Gi
8Gi
```

---

# 91. Final Mental Model

```text
                    Container
                        |
                +-------+-------+
                |               |
                v               v
             Request           Limit
                |               |
                v               v
           Scheduler       Runtime Boundary
                |               |
                |          +----+----+
                |          |         |
                |          v         v
                |         CPU      Memory
                |          |         |
                |       Throttle   OOMKill
                |
                v
              Node
                |
                v
           Allocatable
```

---

# 92. Final Checklist

* [ ] Understand Resource Requests
* [ ] Understand Resource Limits
* [ ] Know CPU units
* [ ] Know Memory units
* [ ] Understand CPU throttling
* [ ] Understand OOMKilled
* [ ] Understand Pod scheduling
* [ ] Understand Requests vs Actual Usage
* [ ] Understand QoS Classes
* [ ] Understand Guaranteed
* [ ] Understand Burstable
* [ ] Understand BestEffort
* [ ] Understand LimitRange
* [ ] Understand ResourceQuota
* [ ] Understand Node Capacity vs Allocatable
* [ ] Understand Resource Overcommitment
* [ ] Understand HPA and CPU requests
* [ ] Understand Ephemeral Storage
* [ ] Understand Extended Resources
* [ ] Know resource troubleshooting commands
* [ ] Practice `kubectl top`
* [ ] Practice resource-constrained Pods
* [ ] Practice LimitRange
* [ ] Practice ResourceQuota
* [ ] Practice troubleshooting `Pending`
* [ ] Practice troubleshooting `OOMKilled`

---

# 93. Key Takeaway

> **Resource Requests tell Kubernetes how much resource a workload needs for scheduling, while Resource Limits define how much it is allowed to consume.**

The most important concepts to remember are:

```text
Request → Scheduling

Limit → Runtime Boundary

CPU → Throttling

Memory → OOMKilled

Guaranteed / Burstable / BestEffort
→ QoS Classes

LimitRange
→ Per-container defaults and boundaries

ResourceQuota
→ Namespace-wide resource limits
```

Mastering **Requests + Limits + QoS + LimitRange + ResourceQuota** is essential for production Kubernetes resource management.

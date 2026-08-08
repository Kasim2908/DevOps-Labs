# 🎯 Kubernetes Pods — Interview Questions

## Beginner

### 1. What is a Pod?

A Pod is the smallest deployable unit in Kubernetes and can contain one or more containers.

### 2. Why does Kubernetes use Pods instead of directly managing containers?

Pods provide a shared execution environment for tightly coupled containers, including shared networking and optional shared storage.

### 3. Can a Pod contain multiple containers?

Yes.

### 4. Do containers in the same Pod have different IP addresses?

Normally no. Containers in the same Pod share the Pod's network namespace and Pod IP.

### 5. What is the Pod IP?

The IP address assigned to the Pod's network namespace.

---

## Intermediate

### 6. What is an Init Container?

A container that runs before the application's regular containers and must successfully complete before they start.

### 7. What is a Sidecar Container?

A supporting container that runs alongside the main application container in the same Pod.

### 8. What is the difference between Init Container and Sidecar?

Init containers run before application containers and normally terminate. Sidecars run alongside the application.

### 9. What is CrashLoopBackOff?

It indicates that a container is repeatedly failing and Kubernetes is backing off between restart attempts.

### 10. Is CrashLoopBackOff the root cause?

No. It is a symptom. The root cause must be investigated using logs, describe output, and events.

---

## Troubleshooting

### 11. How would you troubleshoot a Pending Pod?

```bash
kubectl get pod <pod>
kubectl describe pod <pod>
```

Check the Events section for scheduling failures.

### 12. How would you troubleshoot ImagePullBackOff?

```bash
kubectl describe pod <pod>
```

Check image name, tag, registry access, and image-pull credentials.

### 13. How would you troubleshoot CrashLoopBackOff?

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

### 14. What does OOMKilled mean?

The container was terminated because it ran out of memory, commonly after exceeding its configured memory limit.

### 15. How do you see logs from a specific container?

```bash
kubectl logs <pod> -c <container>
```

### 16. How do you execute a command inside a Pod?

```bash
kubectl exec -it <pod> -- /bin/bash
```

### 17. How do you inspect Pod events?

```bash
kubectl describe pod <pod>
```

or:

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

## Advanced

### 18. What is a Static Pod?

A Pod managed directly by the kubelet from a local manifest rather than by the normal API/controller workflow.

### 19. Why aren't standalone Pods generally used for production applications?

Because a standalone Pod has no higher-level controller to maintain the desired number of replicas or recreate the Pod if it disappears.

### 20. What happens when a container in a Pod crashes?

The kubelet/container runtime can restart the container according to the Pod's restart policy.

### 21. What happens when a standalone Pod is deleted?

It is not automatically recreated because there is no controller managing it.

### 22. Why can two containers in the same Pod communicate through localhost?

Because they share the Pod's network namespace.

### 23. What resources can containers in the same Pod share?

They can share networking and, when configured, volumes.

### 24. What is the difference between Pod phase and container state?

Pod phase describes the overall lifecycle phase of the Pod. Container state describes the current state of an individual container.

---

# Scenario-Based Questions

### Scenario 1

A Pod is stuck in Pending.

What do you check first?

```bash
kubectl describe pod <pod>
```

Then inspect Events and scheduling-related information.

---

### Scenario 2

A Pod is in CrashLoopBackOff.

What commands do you run?

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

---

### Scenario 3

A Pod shows ImagePullBackOff.

What could be wrong?

- Incorrect image name
- Incorrect tag
- Private registry authentication
- Registry/network problem

---

### Scenario 4

A Pod shows OOMKilled.

What would you investigate?

- Memory limit
- Application memory usage
- Memory requests
- Application memory leak
- Node memory pressure

---

### Scenario 5

Two containers are inside the same Pod.

Can they communicate using localhost?

Yes, because they share the Pod's network namespace.

---

# Interview Rule

Do not simply answer:

> "Use kubectl describe."

Explain **why**:

> "`kubectl describe` provides detailed Pod information and, importantly, Events that can reveal scheduling, image-pull, startup, and other lifecycle failures."

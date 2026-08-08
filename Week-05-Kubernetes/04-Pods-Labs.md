# 🧪 Kubernetes Day 2 — Pod Labs

## Cluster Used

Hands-on labs were performed on a 3-node Kubernetes cluster:

```text
tws-cluster-control-plane
tws-cluster-worker
tws-cluster-worker2
```

---

# Lab 1 — Create a Pod

```bash
kubectl run nginx --image=nginx
```

Verify:

```bash
kubectl get pods
```

---

# Lab 2 — Get Pod Details

```bash
kubectl get pod nginx -o wide
```

Shows:

- Pod IP
- Node
- Status
- Restart count

---

# Lab 3 — Describe Pod

```bash
kubectl describe pod nginx
```

Important section:

```text
Events:
```

---

# Lab 4 — Create Pod Using YAML

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

Apply:

```bash
kubectl apply -f nginx-pod.yaml
```

---

# Lab 5 — Execute Commands

```bash
kubectl exec -it nginx-pod -- /bin/bash
```

Inside:

```bash
hostname
ls
```

Exit:

```bash
exit
```

---

# Lab 6 — Logs

```bash
kubectl logs nginx-pod
```

---

# Lab 7 — Port Forward

```bash
kubectl port-forward pod/nginx-pod 8080:80
```

Test:

```bash
curl http://localhost:8080
```

---

# Lab 8 — Multi-Container Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: multi-container-pod

spec:
  containers:

    - name: nginx
      image: nginx:latest

    - name: sidecar
      image: busybox:1.36
      command:
        - sh
        - -c
        - while true; do echo "Sidecar running"; sleep 5; done
```

Apply:

```bash
kubectl apply -f multi-container-pod.yaml
```

Check:

```bash
kubectl get pod multi-container-pod
```

Expected:

```text
2/2
```

---

# Lab 9 — Container-Specific Logs

```bash
kubectl logs multi-container-pod -c nginx
```

```bash
kubectl logs multi-container-pod -c sidecar
```

---

# Lab 10 — Shared Network

```bash
kubectl exec multi-container-pod -c nginx -- hostname -i
```

```bash
kubectl exec multi-container-pod -c sidecar -- hostname -i
```

Both containers should show the same Pod IP.

---

# Lab 11 — Init Container

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: init-demo

spec:
  initContainers:

    - name: init-message
      image: busybox:1.36
      command:
        - sh
        - -c
        - echo "Initializing application..."; sleep 5

  containers:

    - name: nginx
      image: nginx:latest
```

Apply:

```bash
kubectl apply -f init-container-pod.yaml
```

Check:

```bash
kubectl get pod init-demo
```

Logs:

```bash
kubectl logs init-demo -c init-message
```

---

# Lab 12 — Broken Init Container

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: broken-init

spec:
  initContainers:

    - name: broken-init
      image: busybox:1.36
      command:
        - sh
        - -c
        - echo "Initialization failed"; exit 1

  containers:

    - name: nginx
      image: nginx:latest
```

Apply:

```bash
kubectl apply -f broken-init.yaml
```

Troubleshoot:

```bash
kubectl describe pod broken-init
```

```bash
kubectl logs broken-init -c broken-init
```

---

# Lab 13 — Pending Pod

Create a Pod with a node selector that doesn't match any node.

```yaml
nodeSelector:
  kubernetes.io/hostname: node-that-does-not-exist
```

Troubleshoot:

```bash
kubectl get pod pending-pod
kubectl describe pod pending-pod
```

Root cause:

```text
No suitable node matches the selector.
```

---

# Lab 14 — Invalid Image

Use an invalid image:

```yaml
image: nginx:this-image-does-not-exist
```

Troubleshoot:

```bash
kubectl get pod bad-image
kubectl describe pod bad-image
```

Likely states:

```text
ErrImagePull
ImagePullBackOff
```

---

# Lab 15 — CrashLoopBackOff

Create a container that exits with status 1:

```yaml
command:
  - sh
  - -c
  - echo "Application starting"; exit 1
```

Troubleshoot:

```bash
kubectl get pod crash-pod
kubectl describe pod crash-pod
kubectl logs crash-pod
kubectl logs crash-pod --previous
```

Root cause:

```text
Application repeatedly exits unsuccessfully.
```

---

# Lab 16 — OOMKilled

Create a container with a small memory limit and make it allocate more memory.

```yaml
resources:
  limits:
    memory: "32Mi"
```

Troubleshoot:

```bash
kubectl describe pod oom-pod
```

Look for:

```text
Reason: OOMKilled
```

---

# Troubleshooting Workflow

```text
Pod Problem
     │
     ▼
kubectl get pod
     │
     ▼
Identify STATUS
     │
     ▼
kubectl describe pod
     │
     ▼
Check Events
     │
     ▼
kubectl logs
     │
     ▼
Check previous logs if required
     │
     ▼
Identify Root Cause
     │
     ▼
Fix
```

---

# Day 2 Cleanup

```bash
kubectl delete pod init-demo broken-init
kubectl delete pod multi-container-pod
```

Verify:

```bash
kubectl get pods
```

Practice YAML files can be retained under the Labs directory for future revision.

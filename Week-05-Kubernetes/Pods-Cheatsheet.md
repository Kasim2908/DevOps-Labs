# ⚡ Kubernetes Pods — Cheatsheet

## Create

```bash
kubectl run nginx --image=nginx
```

```bash
kubectl apply -f pod.yaml
```

## View

```bash
kubectl get pods
kubectl get pods -o wide
kubectl get pods -A
```

## Details

```bash
kubectl describe pod <pod>
kubectl get pod <pod> -o yaml
kubectl get pod <pod> -o json
```

## Logs

```bash
kubectl logs <pod>
kubectl logs <pod> -c <container>
kubectl logs <pod> --previous
kubectl logs -f <pod>
```

## Execute

```bash
kubectl exec -it <pod> -- /bin/bash
```

```bash
kubectl exec -it <pod> -c <container> -- /bin/bash
```

## Networking

```bash
kubectl get pod <pod> -o wide
```

```bash
kubectl port-forward pod/<pod> 8080:80
```

## JSONPath

```bash
kubectl get pod <pod> \
-o jsonpath='{.spec.containers[*].name}'
```

```bash
kubectl get pod <pod> \
-o jsonpath='{.status.podIP}'
```

## Events

```bash
kubectl get events --sort-by=.lastTimestamp
```

## Delete

```bash
kubectl delete pod <pod>
```

## Troubleshooting

| Problem | Command |
|---|---|
| Pending | `kubectl describe pod <pod>` |
| ImagePull | `kubectl describe pod <pod>` |
| CrashLoopBackOff | `kubectl logs <pod>` |
| Previous crash | `kubectl logs <pod> --previous` |
| OOMKilled | `kubectl describe pod <pod>` |
| Init failure | `kubectl logs <pod> -c <init-container>` |
| Scheduling failure | `kubectl describe pod <pod>` |
| Unknown behavior | `kubectl get events` |

## Pod Phases

```text
Pending
Running
Succeeded
Failed
Unknown
```

## Container States

```text
Waiting
Running
Terminated
```

# Kubernetes Cheatsheet

A practical Kubernetes command and concept reference for learning, interviews, labs, troubleshooting, and daily administration.

---

# 1. Kubernetes Architecture

```text
                    Kubernetes Cluster
                           |
              +------------+------------+
              |                         |
        Control Plane                 Worker Nodes
              |                         |
     +--------+--------+          +-----+------+
     |        |        |          |            |
  API Server Scheduler etcd    kubelet      kube-proxy
     |
     +---- Controllers
```

## Main Control Plane Components

| Component                | Purpose                       |
| ------------------------ | ----------------------------- |
| kube-apiserver           | Entry point to Kubernetes API |
| etcd                     | Stores cluster state          |
| kube-scheduler           | Assigns Pods to Nodes         |
| kube-controller-manager  | Runs controllers              |
| cloud-controller-manager | Cloud-provider integration    |

## Main Node Components

| Component         | Purpose                             |
| ----------------- | ----------------------------------- |
| kubelet           | Manages Pods on a Node              |
| kube-proxy        | Implements Service networking rules |
| Container runtime | Runs containers                     |

---

# 2. kubectl Basics

Check client version:

```bash
kubectl version --client
```

Check client and server:

```bash
kubectl version
```

Get cluster information:

```bash
kubectl cluster-info
```

Get Nodes:

```bash
kubectl get nodes
```

Get detailed Node information:

```bash
kubectl describe node <node-name>
```

Show API resources:

```bash
kubectl api-resources
```

Show API versions:

```bash
kubectl api-versions
```

Get kubectl help:

```bash
kubectl help
```

Command-specific help:

```bash
kubectl explain deployment
```

Nested field:

```bash
kubectl explain deployment.spec
```

---

# 3. Kubernetes Contexts

Show current context:

```bash
kubectl config current-context
```

List contexts:

```bash
kubectl config get-contexts
```

Switch context:

```bash
kubectl config use-context <context-name>
```

View configuration:

```bash
kubectl config view
```

Set namespace for current context:

```bash
kubectl config set-context --current --namespace=<namespace>
```

---

# 4. Namespaces

List Namespaces:

```bash
kubectl get namespaces
```

Short form:

```bash
kubectl get ns
```

Create Namespace:

```bash
kubectl create namespace dev
```

Delete Namespace:

```bash
kubectl delete namespace dev
```

Get resources from a Namespace:

```bash
kubectl get pods -n dev
```

Get all common resources:

```bash
kubectl get all -n dev
```

Use Namespace in YAML:

```yaml
metadata:
  name: my-app
  namespace: dev
```

---

# 5. YAML Basics

Typical Kubernetes manifest:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx

spec:
  replicas: 3
```

Main fields:

```text
apiVersion
kind
metadata
spec
status
```

`status` is normally populated by Kubernetes and should generally not be manually configured in application manifests.

---

# 6. Apply and Delete Resources

Create or update:

```bash
kubectl apply -f file.yaml
```

Apply an entire directory:

```bash
kubectl apply -f ./manifests/
```

Delete using YAML:

```bash
kubectl delete -f file.yaml
```

Delete resource:

```bash
kubectl delete pod <pod-name>
```

Delete multiple resources:

```bash
kubectl delete pod pod1 pod2
```

Delete by label:

```bash
kubectl delete pods -l app=nginx
```

---

# 7. Pods

## Create Pod

```bash
kubectl run nginx --image=nginx
```

Create Pod with port:

```bash
kubectl run nginx \
  --image=nginx \
  --port=80
```

List Pods:

```bash
kubectl get pods
```

Detailed Pods:

```bash
kubectl get pods -o wide
```

All Namespaces:

```bash
kubectl get pods -A
```

Watch Pods:

```bash
kubectl get pods -w
```

Describe Pod:

```bash
kubectl describe pod <pod-name>
```

Get Pod YAML:

```bash
kubectl get pod <pod-name> -o yaml
```

Get Pod JSON:

```bash
kubectl get pod <pod-name> -o json
```

Delete Pod:

```bash
kubectl delete pod <pod-name>
```

---

# 8. Pod Logs

Get logs:

```bash
kubectl logs <pod-name>
```

Follow logs:

```bash
kubectl logs -f <pod-name>
```

Previous container logs:

```bash
kubectl logs <pod-name> --previous
```

Specific container:

```bash
kubectl logs <pod-name> -c <container-name>
```

Follow specific container:

```bash
kubectl logs -f <pod-name> -c <container-name>
```

Previous logs from a specific container:

```bash
kubectl logs <pod-name> \
  -c <container-name> \
  --previous
```

---

# 9. Execute Commands Inside Pods

Open shell:

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

Bash:

```bash
kubectl exec -it <pod-name> -- /bin/bash
```

Run command:

```bash
kubectl exec <pod-name> -- ls
```

Specific container:

```bash
kubectl exec -it <pod-name> \
  -c <container-name> \
  -- /bin/sh
```

Check environment:

```bash
kubectl exec <pod-name> -- env
```

---

# 10. Copy Files

Copy from Pod:

```bash
kubectl cp <pod-name>:/path/file ./file
```

Copy to Pod:

```bash
kubectl cp ./file <pod-name>:/path/file
```

Specific container:

```bash
kubectl cp ./file \
  <pod-name>:/path/file \
  -c <container-name>
```

---

# 11. Pod Labels

Create Pod with label:

```bash
kubectl run nginx \
  --image=nginx \
  --labels="app=nginx"
```

Get Pods by label:

```bash
kubectl get pods -l app=nginx
```

Multiple labels:

```bash
kubectl get pods \
  -l app=web,env=prod
```

Show labels:

```bash
kubectl get pods --show-labels
```

Add label:

```bash
kubectl label pod <pod-name> env=dev
```

Update label:

```bash
kubectl label pod <pod-name> env=prod --overwrite
```

Remove label:

```bash
kubectl label pod <pod-name> env-
```

---

# 12. ReplicaSets

List ReplicaSets:

```bash
kubectl get replicasets
```

Short form:

```bash
kubectl get rs
```

Describe:

```bash
kubectl describe rs <name>
```

Get YAML:

```bash
kubectl get rs <name> -o yaml
```

Scale ReplicaSet:

```bash
kubectl scale rs <name> --replicas=5
```

Remember:

```text
ReplicaSet
    ↓
Maintains desired Pod count
```

---

# 13. Deployments

List Deployments:

```bash
kubectl get deployments
```

Short form:

```bash
kubectl get deploy
```

Create Deployment:

```bash
kubectl create deployment nginx \
  --image=nginx
```

Create with replicas:

```bash
kubectl create deployment nginx \
  --image=nginx \
  --replicas=3
```

Describe:

```bash
kubectl describe deployment <name>
```

Scale:

```bash
kubectl scale deployment <name> \
  --replicas=5
```

Update image:

```bash
kubectl set image deployment/<name> \
  <container>=<image>:<tag>
```

---

# 14. Deployment Rollouts

Rollout status:

```bash
kubectl rollout status deployment/<name>
```

Rollout history:

```bash
kubectl rollout history deployment/<name>
```

Specific revision:

```bash
kubectl rollout history deployment/<name> \
  --revision=2
```

Rollback:

```bash
kubectl rollout undo deployment/<name>
```

Rollback to specific revision:

```bash
kubectl rollout undo deployment/<name> \
  --to-revision=2
```

Restart:

```bash
kubectl rollout restart deployment/<name>
```

Pause:

```bash
kubectl rollout pause deployment/<name>
```

Resume:

```bash
kubectl rollout resume deployment/<name>
```

---

# 15. Deployment Strategies

Default:

```yaml
strategy:
  type: RollingUpdate
```

Example:

```yaml
strategy:
  type: RollingUpdate

  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

Recreate:

```yaml
strategy:
  type: Recreate
```

Concept:

```text
RollingUpdate:

Old → Old + New → New

Recreate:

Old → Nothing → New
```

---

# 16. DaemonSets

List:

```bash
kubectl get daemonsets
```

Short form:

```bash
kubectl get ds
```

Describe:

```bash
kubectl describe daemonset <name>
```

DaemonSet concept:

```text
Node 1 → Pod
Node 2 → Pod
Node 3 → Pod
```

Common use cases:

* Log collectors
* Node monitoring agents
* Networking agents
* Storage agents

---

# 17. StatefulSets

List:

```bash
kubectl get statefulsets
```

Short form:

```bash
kubectl get sts
```

Describe:

```bash
kubectl describe statefulset <name>
```

StatefulSet Pods generally have stable identities:

```text
database-0
database-1
database-2
```

Common use cases:

* Databases
* Distributed systems
* Stateful applications

---

# 18. Jobs

List Jobs:

```bash
kubectl get jobs
```

Create Job:

```bash
kubectl create job test \
  --image=busybox \
  -- echo "Hello"
```

Describe:

```bash
kubectl describe job <name>
```

Get Job Pods:

```bash
kubectl get pods
```

Delete Job:

```bash
kubectl delete job <name>
```

Concept:

```text
Job
 ↓
Runs task
 ↓
Completes
```

---

# 19. CronJobs

List:

```bash
kubectl get cronjobs
```

Short form:

```bash
kubectl get cj
```

Create:

```bash
kubectl create cronjob hello \
  --image=busybox \
  --schedule="*/5 * * * *" \
  -- echo "Hello"
```

Describe:

```bash
kubectl describe cronjob <name>
```

CronJob concept:

```text
CronJob
   ↓
Job
   ↓
Pod
   ↓
Container
```

---

# 20. Services

List Services:

```bash
kubectl get services
```

Short form:

```bash
kubectl get svc
```

Describe:

```bash
kubectl describe service <name>
```

Create ClusterIP Service:

```bash
kubectl expose deployment nginx \
  --port=80
```

Create NodePort:

```bash
kubectl expose deployment nginx \
  --type=NodePort \
  --port=80
```

Create LoadBalancer:

```bash
kubectl expose deployment nginx \
  --type=LoadBalancer \
  --port=80
```

---

# 21. Service Types

## ClusterIP

```yaml
type: ClusterIP
```

Default Service type.

Accessible inside the cluster.

```text
Pod → Service → Pods
```

---

## NodePort

```yaml
type: NodePort
```

Exposes the Service through a port on Nodes.

```text
External
   ↓
NodeIP:NodePort
   ↓
Service
   ↓
Pod
```

---

## LoadBalancer

```yaml
type: LoadBalancer
```

Typically provisions or integrates with an external load balancer in supported environments.

```text
Internet
   ↓
Load Balancer
   ↓
Service
   ↓
Pods
```

---

# 22. Service Endpoints

List EndpointSlices:

```bash
kubectl get endpointslices
```

Older environments may also expose:

```bash
kubectl get endpoints
```

Describe:

```bash
kubectl describe endpointslice <name>
```

If a Service has no endpoints, check:

```text
Service selector
        ↓
Pod labels
        ↓
Pod readiness
```

---

# 23. DNS

Typical Service DNS:

```text
<service>.<namespace>.svc.cluster.local
```

Example:

```text
web.default.svc.cluster.local
```

Within the same Namespace, often:

```text
web
```

Within another Namespace:

```text
web.production
```

---

# 24. ConfigMaps

Create:

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production
```

List:

```bash
kubectl get configmaps
```

Short form:

```bash
kubectl get cm
```

Describe:

```bash
kubectl describe configmap app-config
```

Get YAML:

```bash
kubectl get configmap app-config -o yaml
```

---

# 25. ConfigMap as Environment Variable

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```

---

# 26. ConfigMap as envFrom

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

---

# 27. ConfigMap as Volume

```yaml
volumes:
  - name: config
    configMap:
      name: app-config
```

Mount:

```yaml
volumeMounts:
  - name: config
    mountPath: /etc/config
```

---

# 28. Secrets

Create from literal:

```bash
kubectl create secret generic app-secret \
  --from-literal=PASSWORD=mysecret
```

List:

```bash
kubectl get secrets
```

Describe:

```bash
kubectl describe secret app-secret
```

Get YAML:

```bash
kubectl get secret app-secret -o yaml
```

Decode a value:

```bash
kubectl get secret app-secret \
  -o jsonpath='{.data.PASSWORD}' | base64 --decode
```

---

# 29. Secret as Environment Variable

```yaml
env:
  - name: PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: PASSWORD
```

---

# 30. Secret as Volume

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

Mount:

```yaml
volumeMounts:
  - name: secret-volume
    mountPath: /etc/secrets
    readOnly: true
```

---

# 31. ConfigMap vs Secret

| ConfigMap                   | Secret                  |
| --------------------------- | ----------------------- |
| Non-sensitive configuration | Sensitive configuration |
| Application settings        | Passwords               |
| URLs                        | Tokens                  |
| Feature flags               | Credentials             |
| Configuration files         | Certificates/keys       |

Important:

> Kubernetes Secrets are not automatically equivalent to encryption at rest. Cluster configuration determines how Secret data is protected in storage.

---

# 32. Nodes

List:

```bash
kubectl get nodes
```

Detailed:

```bash
kubectl get nodes -o wide
```

Describe:

```bash
kubectl describe node <node-name>
```

Labels:

```bash
kubectl get nodes --show-labels
```

Resource usage:

```bash
kubectl top nodes
```

---

# 33. Node Labels

Add:

```bash
kubectl label node <node-name> disk=ssd
```

Update:

```bash
kubectl label node <node-name> disk=hdd --overwrite
```

Remove:

```bash
kubectl label node <node-name> disk-
```

Use in Pod:

```yaml
nodeSelector:
  disk: ssd
```

---

# 34. Node Scheduling

## nodeSelector

```yaml
nodeSelector:
  disktype: ssd
```

Simple exact-match scheduling.

---

## Node Affinity

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

---

# 35. Taints and Tolerations

Add taint:

```bash
kubectl taint nodes <node-name> \
  dedicated=database:NoSchedule
```

Remove:

```bash
kubectl taint nodes <node-name> \
  dedicated=database:NoSchedule-
```

Pod toleration:

```yaml
tolerations:
  - key: dedicated
    operator: Equal
    value: database
    effect: NoSchedule
```

Common effects:

```text
NoSchedule
PreferNoSchedule
NoExecute
```

---

# 36. Cordon and Uncordon

Prevent new Pods from being scheduled:

```bash
kubectl cordon <node-name>
```

Allow scheduling again:

```bash
kubectl uncordon <node-name>
```

Check:

```bash
kubectl get nodes
```

---

# 37. Drain a Node

Safely evict workloads before maintenance:

```bash
kubectl drain <node-name>
```

Common practical options may include:

```bash
kubectl drain <node-name> \
  --ignore-daemonsets \
  --delete-emptydir-data
```

Use drain carefully in production.

---

# 38. Node Conditions

Inspect:

```bash
kubectl describe node <node-name>
```

Common conditions:

```text
Ready
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

---

# 39. Resource Requests

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
```

Requests tell Kubernetes approximately:

> Reserve this amount for scheduling purposes.

---

# 40. Resource Limits

Example:

```yaml
resources:
  limits:
    cpu: "500m"
    memory: "256Mi"
```

Limits constrain resource consumption according to Kubernetes/container-runtime behavior.

---

# 41. CPU Units

Examples:

```text
100m = 0.1 CPU
500m = 0.5 CPU
1000m = 1 CPU
2000m = 2 CPUs
```

---

# 42. Memory Units

Common values:

```text
128Mi
256Mi
512Mi
1Gi
2Gi
```

---

# 43. Resource Usage

Requires metrics support:

```bash
kubectl top nodes
```

Pods:

```bash
kubectl top pods
```

Specific Namespace:

```bash
kubectl top pods -n production
```

---

# 44. LimitRange

List:

```bash
kubectl get limitrange
```

Describe:

```bash
kubectl describe limitrange <name>
```

LimitRange can define default resource requests/limits and constraints within a Namespace.

---

# 45. ResourceQuota

List:

```bash
kubectl get resourcequota
```

Short form:

```bash
kubectl get quota
```

Describe:

```bash
kubectl describe resourcequota <name>
```

ResourceQuota limits aggregate resource consumption within a Namespace.

---

# 46. Persistent Volumes

List:

```bash
kubectl get pv
```

Describe:

```bash
kubectl describe pv <name>
```

Get YAML:

```bash
kubectl get pv <name> -o yaml
```

---

# 47. PersistentVolumeClaims

List:

```bash
kubectl get pvc
```

Describe:

```bash
kubectl describe pvc <name>
```

PVC concept:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage
```

---

# 48. Storage Classes

List:

```bash
kubectl get storageclass
```

Short form:

```bash
kubectl get sc
```

Describe:

```bash
kubectl describe storageclass <name>
```

StorageClass is commonly used for dynamic volume provisioning.

---

# 49. PersistentVolumeClaim Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-data

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi
```

---

# 50. Volume Mount

```yaml
volumeMounts:
  - name: data
    mountPath: /data

volumes:
  - name: data
    persistentVolumeClaim:
      claimName: app-data
```

---

# 51. Common Access Modes

```text
ReadWriteOnce
ReadOnlyMany
ReadWriteMany
```

Some storage systems support additional capabilities, so actual support depends on the storage implementation.

---

# 52. Probes

Three major probe types:

```text
Startup Probe
Readiness Probe
Liveness Probe
```

---

## Readiness

Question:

> Can this Pod receive traffic?

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

---

## Liveness

Question:

> Is this container still healthy enough to continue running?

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

---

## Startup

Question:

> Has this slow-starting application finished initialization?

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
```

---

# 53. Probe Types

HTTP:

```yaml
httpGet:
  path: /
  port: 8080
```

TCP:

```yaml
tcpSocket:
  port: 8080
```

Command:

```yaml
exec:
  command:
    - cat
    - /tmp/healthy
```

---

# 54. Service Discovery

Inside a cluster:

```bash
nslookup <service-name>
```

Or:

```bash
nslookup <service-name>.<namespace>
```

Example:

```bash
nslookup web.default
```

From a temporary debugging Pod:

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  -it --rm --restart=Never -- sh
```

Then:

```bash
nslookup kubernetes.default
```

---

# 55. Port Forwarding

Forward local port:

```bash
kubectl port-forward pod/<pod-name> 8080:80
```

Forward Service:

```bash
kubectl port-forward service/<service-name> 8080:80
```

Forward Deployment:

```bash
kubectl port-forward deployment/<deployment-name> 8080:80
```

Then access locally:

```text
http://localhost:8080
```

---

# 56. Networking Troubleshooting

Check Services:

```bash
kubectl get svc
```

Check EndpointSlices:

```bash
kubectl get endpointslices
```

Check Pod IPs:

```bash
kubectl get pods -o wide
```

Test DNS:

```bash
nslookup <service>
```

Test HTTP:

```bash
curl http://<service>
```

Test from a temporary Pod:

```bash
kubectl run curl \
  --image=curlimages/curl \
  -it --rm --restart=Never -- sh
```

Then:

```bash
curl http://<service>:<port>
```

---

# 57. NetworkPolicies

List:

```bash
kubectl get networkpolicies
```

Short form:

```bash
kubectl get netpol
```

Describe:

```bash
kubectl describe networkpolicy <name>
```

NetworkPolicy controls allowed network traffic for selected Pods when supported by the cluster's network plugin.

---

# 58. Ingress

List:

```bash
kubectl get ingress
```

Short form:

```bash
kubectl get ing
```

Describe:

```bash
kubectl describe ingress <name>
```

Ingress concept:

```text
Client
  ↓
Ingress
  ↓
Service
  ↓
Pods
```

Ingress requires an appropriate Ingress controller.

---

# 59. Gateway API

Modern Kubernetes environments may also use Gateway API resources such as:

```text
GatewayClass
Gateway
HTTPRoute
GRPCRoute
```

Check available resources:

```bash
kubectl api-resources | grep -i gateway
```

---

# 60. Events

List events:

```bash
kubectl get events
```

Sort chronologically:

```bash
kubectl get events \
  --sort-by=.lastTimestamp
```

Namespace:

```bash
kubectl get events -n production \
  --sort-by=.lastTimestamp
```

Events are extremely useful for troubleshooting:

* Scheduling failures
* Image pull errors
* Mount failures
* Probe failures
* Admission failures
* Deployment issues

---

# 61. Troubleshooting Workflow

When an application is not working:

```text
1. Check Deployment
       ↓
2. Check ReplicaSet
       ↓
3. Check Pods
       ↓
4. Describe Pod
       ↓
5. Check logs
       ↓
6. Check Services
       ↓
7. Check EndpointSlices
       ↓
8. Check DNS/networking
       ↓
9. Check Events
```

Commands:

```bash
kubectl get deployment
kubectl get rs
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get svc
kubectl get endpointslices
kubectl get events --sort-by=.lastTimestamp
```

---

# 62. Common Pod States

## Pending

Pod has not successfully started.

Possible causes:

* Insufficient resources
* Scheduling constraints
* PVC problems
* Taints
* Affinity rules

---

## ContainerCreating

Container is being prepared.

Possible causes:

* Image pull
* Volume mount
* Network setup

---

## Running

Pod has started and containers may be running.

Always check readiness separately.

---

## Succeeded

Pod completed successfully.

Common with Jobs.

---

## Failed

Pod terminated unsuccessfully.

---

# 63. Common Container Errors

## CrashLoopBackOff

Container repeatedly crashes.

Check:

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

---

## ImagePullBackOff

Image cannot be pulled.

Check:

```bash
kubectl describe pod <pod>
```

---

## ErrImagePull

Initial image pull failed.

---

## OOMKilled

Container exceeded its memory limit or was killed due to memory pressure.

Check:

```bash
kubectl describe pod <pod>
```

---

# 64. Get All Resources

Current Namespace:

```bash
kubectl get all
```

All Namespaces:

```bash
kubectl get all -A
```

Specific Namespace:

```bash
kubectl get all -n production
```

Note:

> `kubectl get all` does not literally mean every Kubernetes resource type. It returns a commonly useful set of workload/network resources.

---

# 65. Output Formats

Normal:

```bash
kubectl get pods
```

Wide:

```bash
kubectl get pods -o wide
```

YAML:

```bash
kubectl get pod <name> -o yaml
```

JSON:

```bash
kubectl get pod <name> -o json
```

Custom columns:

```bash
kubectl get pods \
  -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
```

---

# 66. JSONPath

Get Pod name:

```bash
kubectl get pod <name> \
  -o jsonpath='{.metadata.name}'
```

Get Pod IP:

```bash
kubectl get pod <name> \
  -o jsonpath='{.status.podIP}'
```

Get Node name:

```bash
kubectl get pod <name> \
  -o jsonpath='{.spec.nodeName}'
```

Get container image:

```bash
kubectl get pod <name> \
  -o jsonpath='{.spec.containers[0].image}'
```

Get Deployment replicas:

```bash
kubectl get deployment <name> \
  -o jsonpath='{.spec.replicas}'
```

---

# 67. Label Selectors

Exact match:

```bash
kubectl get pods -l app=web
```

Multiple selectors:

```bash
kubectl get pods \
  -l app=web,env=prod
```

Set-based selector:

```bash
kubectl get pods \
  -l 'environment in (dev,staging)'
```

Not equal:

```bash
kubectl get pods \
  -l 'environment!=production'
```

---

# 68. Annotations

Add annotation:

```bash
kubectl annotate pod <pod-name> \
  description="test pod"
```

Overwrite:

```bash
kubectl annotate pod <pod-name> \
  description="new value" \
  --overwrite
```

Remove:

```bash
kubectl annotate pod <pod-name> description-
```

---

# 69. Service Accounts

List:

```bash
kubectl get serviceaccounts
```

Short form:

```bash
kubectl get sa
```

Create:

```bash
kubectl create serviceaccount app-sa
```

Describe:

```bash
kubectl describe serviceaccount app-sa
```

Use in Pod:

```yaml
spec:
  serviceAccountName: app-sa
```

---

# 70. RBAC

Important resources:

```text
Role
ClusterRole
RoleBinding
ClusterRoleBinding
ServiceAccount
```

List:

```bash
kubectl get roles
kubectl get rolebindings
kubectl get clusterroles
kubectl get clusterrolebindings
```

---

# 71. Check Permissions

Can I create Pods?

```bash
kubectl auth can-i create pods
```

Can a ServiceAccount get Pods?

```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:default:app-sa
```

Check all permissions:

```bash
kubectl auth can-i --list
```

---

# 72. Role Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role

metadata:
  name: pod-reader

rules:
  - apiGroups: [""]
    resources:
      - pods
    verbs:
      - get
      - list
      - watch
```

---

# 73. RoleBinding Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding

metadata:
  name: pod-reader-binding

subjects:
  - kind: ServiceAccount
    name: app-sa

roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

# 74. Common RBAC Verbs

```text
get
list
watch
create
update
patch
delete
deletecollection
```

---

# 75. Security Context

Pod-level:

```yaml
securityContext:
  runAsNonRoot: true
```

Container-level:

```yaml
securityContext:
  allowPrivilegeEscalation: false
```

Other commonly used settings:

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 1000
  runAsNonRoot: true
  readOnlyRootFilesystem: true
```

---

# 76. Pod Security

Check labels on Namespace:

```bash
kubectl get namespace <namespace> --show-labels
```

Common Pod Security Admission labels include:

```text
pod-security.kubernetes.io/enforce
pod-security.kubernetes.io/audit
pod-security.kubernetes.io/warn
```

Typical profiles:

```text
privileged
baseline
restricted
```

---

# 77. Admission and Validation

Check available admission-related resources:

```bash
kubectl api-resources | grep -i admission
```

For validation failures, inspect the command output and Kubernetes Events.

---

# 78. Horizontal Pod Autoscaler

List:

```bash
kubectl get hpa
```

Create:

```bash
kubectl autoscale deployment nginx \
  --min=2 \
  --max=10 \
  --cpu-percent=70
```

Describe:

```bash
kubectl describe hpa <name>
```

HPA concept:

```text
Metrics
   ↓
HPA
   ↓
Deployment
   ↓
ReplicaSet
   ↓
Pods
```

---

# 79. Vertical Pod Autoscaler

VPA is not a built-in core `kubectl` command like HPA and is typically installed separately.

Concept:

```text
Application Metrics
       ↓
VPA
       ↓
Resource Recommendations
       ↓
CPU / Memory Requests
```

---

# 80. Cluster Autoscaling

Cluster Autoscaler can adjust the number of Nodes based on scheduling demand.

Concept:

```text
Pods Pending
     ↓
Cluster Autoscaler
     ↓
Add Nodes
```

Scale down:

```text
Unused Nodes
     ↓
Cluster Autoscaler
     ↓
Remove Nodes
```

Exact behavior depends on the Kubernetes environment and cloud/provider integration.

---

# 81. Pod Disruption Budget

List:

```bash
kubectl get pdb
```

Describe:

```bash
kubectl describe pdb <name>
```

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget

metadata:
  name: web-pdb

spec:
  minAvailable: 2

  selector:
    matchLabels:
      app: web
```

PDB helps limit voluntary disruptions.

---

# 82. Jobs and Parallelism

Example:

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: parallel-job

spec:
  completions: 5
  parallelism: 2

  template:
    spec:
      restartPolicy: Never

      containers:
        - name: worker
          image: busybox
          command:
            - sh
            - -c
            - echo "Processing"
```

Concept:

```text
5 total completions
2 Pods at a time
```

---

# 83. CronJob Schedule

Cron format:

```text
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of week
│ │ │ └──── Month
│ │ └────── Day of month
│ └──────── Hour
└────────── Minute
```

Examples:

Every minute:

```text
* * * * *
```

Every hour:

```text
0 * * * *
```

Every day at midnight:

```text
0 0 * * *
```

Every Sunday:

```text
0 0 * * 0
```

---

# 84. Resource Relationships

## Deployment

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

## Service

```text
Service
   ↓
EndpointSlices
   ↓
Pods
```

## StatefulSet

```text
StatefulSet
    ↓
Pods
    ↓
PVCs
```

## CronJob

```text
CronJob
    ↓
Job
    ↓
Pod
```

---

# 85. Common Kubernetes Resource Shortcuts

| Resource                 | Short Name |
| ------------------------ | ---------- |
| pods                     | po         |
| services                 | svc        |
| deployments              | deploy     |
| replicasets              | rs         |
| daemonsets               | ds         |
| statefulsets             | sts        |
| configmaps               | cm         |
| namespaces               | ns         |
| persistentvolumeclaims   | pvc        |
| persistentvolumes        | pv         |
| storageclasses           | sc         |
| serviceaccounts          | sa         |
| cronjobs                 | cj         |
| horizontalpodautoscalers | hpa        |
| networkpolicies          | netpol     |
| ingress                  | ing        |
| resourcequotas           | quota      |
| poddisruptionbudgets     | pdb        |

---

# 86. Useful Aliases

You can configure shell aliases:

```bash
alias k=kubectl
```

Then:

```bash
k get pods
```

Instead of:

```bash
kubectl get pods
```

Another useful alias:

```bash
alias kgp='kubectl get pods'
```

---

# 87. Kubernetes Object Discovery

List all API resources:

```bash
kubectl api-resources
```

Filter:

```bash
kubectl api-resources | grep -i deployment
```

Find resources with a specific API group:

```bash
kubectl api-resources | grep apps
```

---

# 88. kubectl Explain

Explain a resource:

```bash
kubectl explain deployment
```

Explain a field:

```bash
kubectl explain deployment.spec
```

Explain containers:

```bash
kubectl explain pod.spec.containers
```

Explain a specific field:

```bash
kubectl explain pod.spec.containers.resources
```

This is one of the most useful commands when writing YAML.

---

# 89. Generate YAML

Dry run:

```bash
kubectl create deployment nginx \
  --image=nginx \
  --dry-run=client \
  -o yaml
```

Save to file:

```bash
kubectl create deployment nginx \
  --image=nginx \
  --dry-run=client \
  -o yaml > deployment.yaml
```

Generate a Pod:

```bash
kubectl run nginx \
  --image=nginx \
  --dry-run=client \
  -o yaml
```

---

# 90. Dry Run

Client-side dry run:

```bash
kubectl apply \
  --dry-run=client \
  -f deployment.yaml
```

Server-side dry run:

```bash
kubectl apply \
  --dry-run=server \
  -f deployment.yaml
```

Dry runs are useful for validating intended changes before applying them.

---

# 91. Diff

Preview differences:

```bash
kubectl diff -f deployment.yaml
```

This helps compare the desired manifest with the current cluster state.

---

# 92. Replace vs Apply

Apply:

```bash
kubectl apply -f deployment.yaml
```

Declaratively creates or updates resources.

Replace:

```bash
kubectl replace -f deployment.yaml
```

Attempts to replace the existing resource.

For normal declarative workflows, `kubectl apply` is generally preferred.

---

# 93. Edit Resources

Edit Deployment:

```bash
kubectl edit deployment <name>
```

Edit Service:

```bash
kubectl edit service <name>
```

Edit Pod:

```bash
kubectl edit pod <name>
```

Be careful editing live resources manually because the change may not be reflected in your source manifests.

---

# 94. Patch Resources

Example:

```bash
kubectl patch deployment nginx \
  -p '{"spec":{"replicas":5}}'
```

Patch can be useful for targeted changes.

---

# 95. Field Selectors

Find Pods on a specific Node:

```bash
kubectl get pods \
  --field-selector spec.nodeName=<node-name>
```

Find failed Pods:

```bash
kubectl get pods \
  --field-selector status.phase=Failed
```

---

# 96. Sorting Resources

Sort Pods by creation time:

```bash
kubectl get pods \
  --sort-by=.metadata.creationTimestamp
```

Sort Nodes by CPU capacity:

```bash
kubectl get nodes \
  --sort-by=.status.capacity.cpu
```

Sort Events:

```bash
kubectl get events \
  --sort-by=.lastTimestamp
```

---

# 97. Watch Resources

Watch Pods:

```bash
kubectl get pods -w
```

Watch Deployments:

```bash
kubectl get deployments -w
```

Watch Nodes:

```bash
kubectl get nodes -w
```

---

# 98. Troubleshooting by Layer

## Layer 1: Pod

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
```

## Layer 2: ReplicaSet

```bash
kubectl get rs
kubectl describe rs <name>
```

## Layer 3: Deployment

```bash
kubectl get deployment
kubectl describe deployment <name>
kubectl rollout status deployment/<name>
```

## Layer 4: Service

```bash
kubectl get svc
kubectl describe svc <name>
```

## Layer 5: EndpointSlice

```bash
kubectl get endpointslices
```

## Layer 6: DNS

```bash
nslookup <service>
```

## Layer 7: NetworkPolicy

```bash
kubectl get netpol
```

## Layer 8: Node

```bash
kubectl get nodes
kubectl describe node <node>
```

---

# 99. Common Troubleshooting Commands

```bash
kubectl get pods -o wide

kubectl describe pod <pod>

kubectl logs <pod>

kubectl logs <pod> --previous

kubectl get events \
  --sort-by=.lastTimestamp

kubectl get svc

kubectl get endpointslices

kubectl get rs

kubectl get deployment

kubectl describe deployment <deployment>

kubectl rollout status deployment/<deployment>
```

---

# 100. Application Debugging Pod

Create temporary shell:

```bash
kubectl run debug \
  --image=busybox:1.36 \
  -it \
  --rm \
  --restart=Never \
  -- sh
```

For networking:

```bash
kubectl run curl \
  --image=curlimages/curl \
  -it \
  --rm \
  --restart=Never \
  -- sh
```

Inside:

```bash
curl http://service-name:80
```

DNS:

```bash
nslookup service-name
```

---

# 101. Check Service Connectivity

```text
Client Pod
    |
    ↓
Service DNS
    |
    ↓
Service ClusterIP
    |
    ↓
EndpointSlice
    |
    ↓
Pod IP
    |
    ↓
Container Port
```

If connection fails, inspect each layer.

---

# 102. Service Selector Troubleshooting

Service:

```yaml
selector:
  app: web
```

Pod:

```yaml
labels:
  app: frontend
```

Result:

```text
Service selector ≠ Pod label
             ↓
No matching Pods
             ↓
No Service endpoints
```

Correct:

```yaml
selector:
  app: web
```

Pod:

```yaml
labels:
  app: web
```

---

# 103. Deployment Selector Troubleshooting

Deployment:

```yaml
selector:
  matchLabels:
    app: web
```

Pod template:

```yaml
labels:
  app: backend
```

This is invalid because the selector does not match the Pod template labels.

Correct:

```yaml
selector:
  matchLabels:
    app: web

template:
  metadata:
    labels:
      app: web
```

---

# 104. Image Management

Check Pod image:

```bash
kubectl get pod <pod> \
  -o jsonpath='{.spec.containers[*].image}'
```

Update Deployment image:

```bash
kubectl set image deployment/app \
  app=repo/app:1.2.0
```

Check rollout:

```bash
kubectl rollout status deployment/app
```

---

# 105. ImagePullSecrets

Create registry Secret:

```bash
kubectl create secret docker-registry regcred \
  --docker-server=<registry> \
  --docker-username=<username> \
  --docker-password=<password>
```

Use it:

```yaml
imagePullSecrets:
  - name: regcred
```

---

# 106. Init Containers

Example:

```yaml
initContainers:
  - name: init
    image: busybox
    command:
      - sh
      - -c
      - echo "Initializing"
```

Concept:

```text
Init Container
      ↓
Completes
      ↓
Application Container
```

Useful for initialization tasks and prerequisite checks.

---

# 107. Sidecar Containers

A Pod can contain multiple containers:

```text
Pod
 |
 +-- Application Container
 |
 +-- Sidecar Container
```

Common sidecar uses:

* Log processing
* Proxies
* Configuration synchronization
* Supporting services

---

# 108. Multi-Container Pod

```yaml
spec:
  containers:

    - name: app
      image: nginx

    - name: sidecar
      image: busybox
```

Containers in the same Pod share the Pod network namespace and can communicate using `localhost`.

---

# 109. Pod Lifecycle

Typical lifecycle:

```text
Pending
   ↓
Running
   ↓
Succeeded / Failed
```

Container-level states can include:

```text
Waiting
Running
Terminated
```

---

# 110. Restart Policies

Pod-level restart policies:

```text
Always
OnFailure
Never
```

Deployments generally use:

```yaml
restartPolicy: Always
```

Jobs commonly use:

```yaml
restartPolicy: Never
```

or:

```yaml
restartPolicy: OnFailure
```

---

# 111. Init Container Troubleshooting

Check:

```bash
kubectl describe pod <pod>
```

Check logs:

```bash
kubectl logs <pod> -c <init-container>
```

If multiple init containers:

```bash
kubectl logs <pod> -c <init-container-name>
```

---

# 112. Container Restart Count

```bash
kubectl get pods
```

Example:

```text
NAME    READY   STATUS    RESTARTS   AGE
app     1/1     Running   3          5m
```

High restart count can indicate:

* Application crashes
* Liveness probe failures
* OOM kills
* Configuration problems

---

# 113. Check Pod Conditions

```bash
kubectl get pod <pod> \
  -o jsonpath='{.status.conditions[*]}'
```

Common conditions:

```text
PodScheduled
Initialized
ContainersReady
Ready
```

---

# 114. Check Container State

```bash
kubectl get pod <pod> -o json
```

Look under:

```text
status.containerStatuses
```

Useful fields include:

```text
ready
restartCount
state
lastState
```

---

# 115. Namespace Cleanup

Delete all Pods in a Namespace:

```bash
kubectl delete pods --all -n dev
```

Delete all Deployments:

```bash
kubectl delete deployments --all -n dev
```

Delete all Services:

```bash
kubectl delete services --all -n dev
```

Delete Namespace:

```bash
kubectl delete namespace dev
```

Be extremely careful with destructive commands.

---

# 116. Production Safety

Before deleting resources:

```bash
kubectl get <resource>
```

Describe:

```bash
kubectl describe <resource> <name>
```

Check Namespace:

```bash
kubectl config view --minify --output 'jsonpath={..namespace}'
```

Check context:

```bash
kubectl config current-context
```

A useful safety habit:

```text
Context
  ↓
Namespace
  ↓
Resource
  ↓
Command
```

---

# 117. Useful Production Checklist

Before deploying:

```text
□ Correct Namespace
□ Correct cluster/context
□ Correct image
□ Correct image tag
□ Resource requests
□ Resource limits
□ Readiness probe
□ Liveness/startup probes where appropriate
□ Correct replicas
□ Correct Service selector
□ Correct labels
□ ConfigMap/Secret available
□ Storage available
□ NetworkPolicy considered
□ Rollout strategy reviewed
```

---

# 118. Deployment Checklist

```text
Deployment
    ↓
Correct selector
    ↓
Correct Pod labels
    ↓
Correct image
    ↓
Correct replicas
    ↓
Resources configured
    ↓
Readiness configured
    ↓
Service selector matches
    ↓
Rollout monitored
```

---

# 119. Quick Command Reference

## Cluster

```bash
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node <node>
```

## Namespace

```bash
kubectl get ns
kubectl create ns <name>
kubectl delete ns <name>
```

## Pods

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl exec -it <pod> -- sh
kubectl delete pod <pod>
```

## Deployments

```bash
kubectl get deploy
kubectl describe deploy <name>
kubectl scale deploy <name> --replicas=5
kubectl set image deploy/<name> <container>=<image>
kubectl rollout status deploy/<name>
kubectl rollout history deploy/<name>
kubectl rollout undo deploy/<name>
kubectl rollout restart deploy/<name>
```

## ReplicaSets

```bash
kubectl get rs
kubectl describe rs <name>
kubectl scale rs <name> --replicas=5
```

## Services

```bash
kubectl get svc
kubectl describe svc <name>
kubectl expose deployment <name> --port=80
```

## ConfigMaps

```bash
kubectl get cm
kubectl describe cm <name>
kubectl get cm <name> -o yaml
```

## Secrets

```bash
kubectl get secrets
kubectl describe secret <name>
kubectl get secret <name> -o yaml
```

## Storage

```bash
kubectl get pv
kubectl get pvc
kubectl get sc
```

## Jobs

```bash
kubectl get jobs
kubectl describe job <name>
```

## CronJobs

```bash
kubectl get cronjobs
kubectl describe cronjob <name>
```

## Troubleshooting

```bash
kubectl get events --sort-by=.lastTimestamp
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl top nodes
kubectl top pods
```

---

# 120. Most Important Commands to Memorize

If you are preparing for Kubernetes interviews or daily administration, prioritize these:

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl exec -it <pod> -- sh

kubectl get deployments
kubectl describe deployment <name>
kubectl scale deployment <name> --replicas=5
kubectl set image deployment/<name> <container>=<image>:<tag>

kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout restart deployment/<name>

kubectl get rs
kubectl get svc
kubectl describe svc <name>

kubectl get nodes
kubectl describe node <node>

kubectl get namespaces
kubectl get events --sort-by=.lastTimestamp

kubectl apply -f file.yaml
kubectl delete -f file.yaml

kubectl explain <resource>
kubectl api-resources
kubectl auth can-i <verb> <resource>
```

---

# 121. Kubernetes Mental Model

The easiest way to understand Kubernetes is to think in layers:

```text
                    Kubernetes
                         |
          +--------------+--------------+
          |                             |
      Workloads                     Networking
          |                             |
    +-----+-----+                 +-----+-----+
    |     |     |                 |     |     |
 Deploy  DS   STS              Service Ingress
    |                       NetworkPolicy
    ↓
 ReplicaSet
    ↓
   Pods
    ↓
Containers
```

Storage:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
StorageClass / Storage Backend
```

Configuration:

```text
Pod
 ├── ConfigMap
 └── Secret
```

Security:

```text
Pod
 ↓
ServiceAccount
 ↓
Role / ClusterRole
 ↓
RoleBinding / ClusterRoleBinding
```

Autoscaling:

```text
Metrics
  ↓
HPA
  ↓
Deployment
  ↓
ReplicaSet
  ↓
Pods
```

---

# 122. Final Kubernetes Cheat Sheet

```text
kubectl
  |
  +-- get          → View resources
  +-- describe     → Detailed resource information
  +-- apply        → Create/update declaratively
  +-- delete       → Delete resources
  +-- create       → Create resources imperatively
  +-- edit         → Edit live resources
  +-- patch        → Partially update resources
  +-- logs         → Container logs
  +-- exec         → Execute commands in containers
  +-- cp           → Copy files
  +-- explain      → Learn resource fields
  +-- scale        → Change replica count
  +-- set          → Change selected resource fields
  +-- rollout      → Manage Deployment rollouts
  +-- expose       → Create Service
  +-- label        → Manage labels
  +-- annotate     → Manage annotations
  +-- top          → Resource usage
  +-- auth         → Check permissions
```

## Core Kubernetes Hierarchy

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container
```

## Networking

```text
Client
   ↓
Ingress / Gateway
   ↓
Service
   ↓
EndpointSlice
   ↓
Pod
```

## Storage

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage
```

## Scheduled Workloads

```text
CronJob
   ↓
Job
   ↓
Pod
```

## Security

```text
ServiceAccount
      ↓
Role / ClusterRole
      ↓
RoleBinding / ClusterRoleBinding
```

## Most Important Troubleshooting Flow

```text
Application Problem
       ↓
kubectl get pods
       ↓
kubectl describe pod
       ↓
kubectl logs
       ↓
kubectl get svc
       ↓
kubectl get endpointslices
       ↓
kubectl get events
       ↓
Check Node / Resources / Network / Storage
```

> **Core Kubernetes principle:** You declare the desired state, and Kubernetes controllers continuously reconcile the actual state toward that desired state.


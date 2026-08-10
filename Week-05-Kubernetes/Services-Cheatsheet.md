# ☸️ Kubernetes Services - Cheat Sheet

## 1. What is a Service?

A **Service** provides a stable network endpoint for accessing a group of Kubernetes Pods.

```text
Client
   │
   ▼
Service
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

### Why?

Pod IPs are temporary and can change.

```text
Pod IP → Changes
Service IP → Stable
```

---

# 2. Service Architecture

```text
                 Service
                    │
              Selector
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        Pod 1     Pod 2     Pod 3
```

A Service uses **labels and selectors** to find Pods.

---

# 3. Basic Service YAML

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f service.yaml
```

---

# 4. Important Service Fields

| Field           | Meaning                 |
| --------------- | ----------------------- |
| `apiVersion`    | API version             |
| `kind`          | Resource type           |
| `metadata.name` | Service name            |
| `selector`      | Selects backend Pods    |
| `port`          | Service port            |
| `targetPort`    | Pod/application port    |
| `nodePort`      | Node-level exposed port |
| `type`          | Service type            |

---

# 5. Port vs TargetPort

```text
Client
  │
  │ :80
  ▼
Service
  │
  │ :8080
  ▼
Pod
```

```yaml
ports:
  - port: 80
    targetPort: 8080
```

### Remember

```text
port       → Service port
targetPort → Pod port
```

---

# 6. Service Types

Kubernetes Service types:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

---

# 7. ClusterIP

**Default Service type.**

Used for internal communication inside the cluster.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  type: ClusterIP

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

Architecture:

```text
Pod
 │
 ▼
ClusterIP Service
 │
 ▼
Backend Pods
```

Check:

```bash
kubectl get svc
```

---

# 8. NodePort

Exposes the Service through a port on every Node.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-nodeport

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Traffic:

```text
NodeIP:30080
      │
      ▼
 Service :80
      │
      ▼
 Pod :80
```

Default NodePort range:

```text
30000-32767
```

---

# 9. LoadBalancer

Used to expose a Service through an external load balancer.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-lb

spec:
  type: LoadBalancer

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

Architecture:

```text
Internet
   │
   ▼
Load Balancer
   │
   ▼
Service
   │
   ├── Pod
   ├── Pod
   └── Pod
```

Commonly used in cloud environments.

---

# 10. ExternalName

Maps a Kubernetes Service to an external DNS name.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: external-service

spec:
  type: ExternalName
  externalName: example.com
```

Architecture:

```text
Application
     │
     ▼
external-service
     │
     ▼
example.com
```

---

# 11. Service Type Comparison

| Type           | Purpose                |
| -------------- | ---------------------- |
| `ClusterIP`    | Internal access        |
| `NodePort`     | Node IP + port         |
| `LoadBalancer` | External load balancer |
| `ExternalName` | External DNS mapping   |

### Quick Memory

```text
ClusterIP    → Internal
NodePort     → Node IP + Port
LoadBalancer → External LB
ExternalName → External DNS
```

---

# 12. Labels and Selectors

Pod:

```yaml
metadata:
  labels:
    app: nginx
```

Service:

```yaml
selector:
  app: nginx
```

They match:

```text
Service
   │
   │ app=nginx
   ▼
Pod
   │
   └── app=nginx ✓
```

If they don't match:

```text
Service
   │
   ▼
No matching Pods
   │
   ▼
No Endpoints
```

---

# 13. Check Services

List Services:

```bash
kubectl get svc
```

or:

```bash
kubectl get services
```

Detailed information:

```bash
kubectl describe svc <service>
```

Get YAML:

```bash
kubectl get svc <service> -o yaml
```

---

# 14. Service IP

Get ClusterIP:

```bash
kubectl get svc <service> \
  -o jsonpath='{.spec.clusterIP}'
```

Get Service type:

```bash
kubectl get svc <service> \
  -o jsonpath='{.spec.type}'
```

---

# 15. Endpoints

Check Service endpoints:

```bash
kubectl get endpoints <service>
```

Example:

```text
NAME            ENDPOINTS
nginx-service   10.244.0.5:80,10.244.0.6:80
```

If you see:

```text
<none>
```

check:

* Pod labels
* Service selector
* Pod readiness
* Pod existence
* Namespace

---

# 16. EndpointSlices

List EndpointSlices:

```bash
kubectl get endpointslice
```

For a specific Service:

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=<service>
```

EndpointSlices represent the backend endpoints of a Service.

---

# 17. Service DNS

Kubernetes provides DNS-based Service discovery.

Short name:

```text
nginx-service
```

Namespace-qualified:

```text
nginx-service.default
```

Fully qualified:

```text
nginx-service.default.svc.cluster.local
```

General format:

```text
<service>.<namespace>.svc.cluster.local
```

Example:

```bash
curl http://nginx-service
```

---

# 18. Service Discovery

```text
Application Pod
      │
      ▼
    CoreDNS
      │
      ▼
Service DNS
      │
      ▼
Service
      │
      ▼
Backend Pods
```

DNS is generally preferred over relying on automatically generated Service environment variables.

---

# 19. Headless Service

A Headless Service does not have a normal ClusterIP.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-headless

spec:
  clusterIP: None

  selector:
    app: nginx

  ports:
    - port: 80
```

Check:

```bash
kubectl get svc
```

Output:

```text
NAME             TYPE        CLUSTER-IP
nginx-headless   ClusterIP   None
```

### Common Uses

* Stateful applications
* Databases
* StatefulSets
* Direct Pod discovery
* Distributed systems

---

# 20. Service Without Selector

A Service does not always require a selector.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: external-api

spec:
  ports:
    - port: 80
```

Without a selector, Kubernetes does not automatically discover Pods for that Service.

Backend endpoints can be defined separately using EndpointSlices.

---

# 21. Multiple Service Ports

A Service can expose multiple ports.

```yaml
ports:

  - name: http
    port: 80
    targetPort: 8080

  - name: https
    port: 443
    targetPort: 8443
```

Traffic:

```text
Service :80
    │
    ▼
Pod :8080

Service :443
    │
    ▼
Pod :8443
```

---

# 22. Session Affinity

Default:

```yaml
sessionAffinity: None
```

Client IP affinity:

```yaml
sessionAffinity: ClientIP
```

Example:

```text
Client
  │
  ▼
Service
  │
  ▼
Same backend Pod
```

This can keep requests from the same client IP directed to the same backend for the configured session-affinity behavior.

---

# 23. Service + Deployment

Common Kubernetes architecture:

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ├── Pod 1
    ├── Pod 2
    └── Pod 3
          ▲
          │
       Service
```

### Responsibilities

```text
Deployment
    ↓
Manages Pods

Service
    ↓
Provides stable networking
```

---

# 24. Service + Ingress

```text
Internet
   │
   ▼
Ingress
   │
   ├── /api
   │     ↓
   │   API Service
   │
   └── /
         ↓
      Web Service
         │
       Pods
```

### Remember

```text
Service → Stable access to Pods

Ingress → HTTP/HTTPS routing to Services
```

---

# 25. Service Traffic Flow

Typical traffic flow:

```text
Client
  │
  ▼
Service
  │
  ▼
Virtual IP
  │
  ▼
EndpointSlice
  │
  ▼
Pod IP
  │
  ▼
Container
```

---

# 26. kube-proxy

`kube-proxy` is responsible for implementing Service networking rules on many Kubernetes installations.

It can use mechanisms such as:

```text
iptables
IPVS
```

depending on cluster configuration.

Conceptually:

```text
Service IP
    │
    ▼
kube-proxy rules
    │
    ▼
Pod IP
```

---

# 27. Troubleshooting Workflow

```text
kubectl get svc
       ↓
kubectl describe svc
       ↓
Check selector
       ↓
Check Pod labels
       ↓
Check Endpoints
       ↓
Check EndpointSlices
       ↓
Check targetPort
       ↓
Test DNS
       ↓
Test connectivity
```

---

# 28. Service Has No Endpoints

Check:

```bash
kubectl get endpoints <service>
```

If:

```text
<none>
```

Check Pods:

```bash
kubectl get pods --show-labels
```

Check Service:

```bash
kubectl describe svc <service>
```

Check EndpointSlices:

```bash
kubectl get endpointslice
```

### Common Causes

```text
Wrong selector
     ↓
Pod labels don't match

Pod not Ready
     ↓
Not selected as a ready backend

No Pods
     ↓
No endpoints
```

---

# 29. Wrong Selector

Service:

```yaml
selector:
  app: nginx
```

Pod:

```yaml
labels:
  app: web
```

Result:

```text
app=nginx
      ≠
app=web
```

Therefore:

```text
No matching Pods
       ↓
No Endpoints
```

---

# 30. Wrong targetPort

Service:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Application actually listens on:

```text
8081
```

Traffic:

```text
Service :80
    ↓
Pod :8080
    X
Application :8081
```

Verify the application's actual listening port.

---

# 31. Test Service From a Pod

Create a temporary test Pod:

```bash
kubectl run test-pod \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Test:

```bash
curl http://<service>
```

Test with DNS:

```bash
curl http://<service>.<namespace>.svc.cluster.local
```

---

# 32. Test DNS

Run:

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  -it --rm \
  -- sh
```

Then:

```bash
nslookup <service>
```

Or:

```bash
nslookup <service>.<namespace>.svc.cluster.local
```

---

# 33. Port Forward

Forward a local port to a Service:

```bash
kubectl port-forward service/<service> 8080:80
```

Then:

```text
http://localhost:8080
```

---

# 34. Useful Commands

### List Services

```bash
kubectl get svc
```

### Detailed Service Information

```bash
kubectl describe svc <service>
```

### Get Service YAML

```bash
kubectl get svc <service> -o yaml
```

### Get Endpoints

```bash
kubectl get endpoints <service>
```

### Get EndpointSlices

```bash
kubectl get endpointslice
```

### Get Pods With Labels

```bash
kubectl get pods --show-labels
```

### Get Pods

```bash
kubectl get pods -o wide
```

### Get Service IP

```bash
kubectl get svc <service> \
  -o jsonpath='{.spec.clusterIP}'
```

### Get Service Type

```bash
kubectl get svc <service> \
  -o jsonpath='{.spec.type}'
```

---

# 35. Quick Command Table

| Task               | Command                                      |
| ------------------ | -------------------------------------------- |
| List Services      | `kubectl get svc`                            |
| Describe Service   | `kubectl describe svc <svc>`                 |
| Service YAML       | `kubectl get svc <svc> -o yaml`              |
| Get Endpoints      | `kubectl get endpoints <svc>`                |
| Get EndpointSlices | `kubectl get endpointslice`                  |
| Show Pod Labels    | `kubectl get pods --show-labels`             |
| Test DNS           | `nslookup <svc>`                             |
| Test HTTP          | `curl http://<svc>`                          |
| Port Forward       | `kubectl port-forward service/<svc> 8080:80` |
| Apply Service      | `kubectl apply -f service.yaml`              |
| Delete Service     | `kubectl delete svc <svc>`                   |

---

# 36. Service vs Pod IP

| Pod IP                | Service IP                  |
| --------------------- | --------------------------- |
| Can change            | Stable                      |
| Belongs to Pod        | Belongs to Service          |
| Pod-specific          | Represents backend Pods     |
| Ephemeral             | Stable while Service exists |
| Not ideal for clients | Designed for stable access  |

---

# 37. Service vs Ingress

| Service                           | Ingress                                  |
| --------------------------------- | ---------------------------------------- |
| Provides access to Pods           | Routes HTTP/HTTPS                        |
| Works at Service networking layer | Works at HTTP/HTTPS layer                |
| Provides stable endpoint          | Provides routing rules                   |
| Can use ClusterIP                 | Routes to Services                       |
| Can expose NodePort/LoadBalancer  | Usually works with an Ingress Controller |

---

# 38. Common Interview Questions

### Q1. What is a Kubernetes Service?

A Kubernetes Service provides a stable network endpoint for accessing a group of Pods.

---

### Q2. Why do we need a Service?

Because Pod IPs are ephemeral and can change.

---

### Q3. What is the default Service type?

```text
ClusterIP
```

---

### Q4. What is ClusterIP?

A Service type used for internal communication within the cluster.

---

### Q5. What is NodePort?

A Service type that exposes the Service through a port on each Node.

---

### Q6. What is the default NodePort range?

```text
30000-32767
```

---

### Q7. What is LoadBalancer?

A Service type that exposes an application through an external load balancer when supported by the environment.

---

### Q8. What is targetPort?

The port on the backend Pod/application where Service traffic is sent.

---

### Q9. How does a Service find Pods?

Using a selector that matches Pod labels.

---

### Q10. What happens when a Service has no matching Pods?

The Service has no backend endpoints.

---

### Q11. What is a Headless Service?

A Service configured with:

```yaml
clusterIP: None
```

It does not provide a normal virtual ClusterIP and is useful for direct backend discovery.

---

### Q12. How do Pods discover Services?

Primarily through Kubernetes DNS.

Example:

```text
backend-service.default.svc.cluster.local
```

---

### Q13. How do you troubleshoot a Service?

```bash
kubectl get svc
kubectl describe svc <service>
kubectl get endpoints <service>
kubectl get endpointslice
kubectl get pods --show-labels
```

Then verify:

```text
Selector
   ↓
Pod Labels
   ↓
Endpoints
   ↓
targetPort
   ↓
Application
   ↓
DNS / Network
```

---

# 39. Quick Revision

```text
Service
│
├── Stable networking
│
├── Service Discovery
│
├── Load balancing
│
└── Access to Pods
```

### Service Types

```text
ClusterIP
   ↓
Internal

NodePort
   ↓
Node IP + Port

LoadBalancer
   ↓
External Load Balancer

ExternalName
   ↓
External DNS
```

### Important Fields

```text
selector
port
targetPort
nodePort
type
clusterIP
sessionAffinity
```

### Troubleshooting

```text
Service
   ↓
Selector
   ↓
Pod Labels
   ↓
Endpoints
   ↓
EndpointSlices
   ↓
targetPort
   ↓
Application
   ↓
DNS / Network
```

---

# 🔥 Must Remember

```text
Service = Stable network endpoint for Pods

ClusterIP = Internal access

NodePort = Node IP + Port

LoadBalancer = External Load Balancer

ExternalName = External DNS

port = Service port

targetPort = Pod port

nodePort = Node exposed port

selector = Finds Pods

Endpoints = Backend addresses

EndpointSlice = Modern endpoint representation

clusterIP: None = Headless Service

DNS = Service discovery

Ingress = HTTP/HTTPS routing
```

# 🎯 One-Line Interview Answer

> **A Kubernetes Service provides a stable network endpoint and service discovery mechanism for accessing a dynamic group of Pods selected using labels.**

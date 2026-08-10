# ☸️ Kubernetes Services - Interview Questions

## 1. What is a Service in Kubernetes?

A **Service** is a Kubernetes object that provides a stable network endpoint for accessing a group of Pods.

Pods are ephemeral, so their IP addresses can change when Pods are recreated.

A Service provides:

* Stable IP address
* Stable DNS name
* Service discovery
* Load balancing across backend Pods

Example:

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

---

## 2. Why do we need a Service in Kubernetes?

Pod IP addresses are not stable.

For example:

```text
Pod 1 → 10.244.0.5
Pod 2 → 10.244.0.6
```

If Pod 2 is deleted and recreated:

```text
Pod 2 → 10.244.0.15
```

The IP changed.

A Service provides a stable endpoint:

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

The client communicates with the Service instead of directly depending on Pod IPs.

---

## 3. What are the main responsibilities of a Service?

A Service mainly provides:

```text
Service
   │
   ├── Stable Networking
   ├── Service Discovery
   ├── Load Balancing
   └── Access to Pods
```

It allows applications to communicate without knowing the individual Pod IP addresses.

---

## 4. What are the different types of Kubernetes Services?

The main Service types are:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

| Type         | Purpose                       |
| ------------ | ----------------------------- |
| ClusterIP    | Internal cluster access       |
| NodePort     | Expose through Node IP + port |
| LoadBalancer | External load balancer        |
| ExternalName | Map to external DNS name      |

---

## 5. What is ClusterIP?

`ClusterIP` is the default Kubernetes Service type.

It provides an internal virtual IP that can be accessed from within the cluster.

Example:

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

Check:

```bash
kubectl get svc
```

Example:

```text
NAME            TYPE        CLUSTER-IP      PORT(S)
nginx-service   ClusterIP   10.96.20.10     80/TCP
```

---

## 6. What is the default Service type?

The default Service type is:

```text
ClusterIP
```

If you do not specify:

```yaml
type:
```

Kubernetes uses:

```yaml
type: ClusterIP
```

---

## 7. What is NodePort?

`NodePort` exposes a Service through a port on each Kubernetes Node.

Example:

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

Traffic flow:

```text
Client
   │
   │ NodeIP:30080
   ▼
NodePort
   │
   ▼
Service :80
   │
   ▼
Pod :80
```

---

## 8. What is the default NodePort range?

The default NodePort range is:

```text
30000-32767
```

Example:

```yaml
nodePort: 30080
```

The application can then be accessed using:

```text
<Node-IP>:30080
```

---

## 9. What is LoadBalancer?

`LoadBalancer` exposes a Service through an external load balancer.

Example:

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

Traffic:

```text
Internet
   │
   ▼
External Load Balancer
   │
   ▼
Service
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

It is commonly used in cloud environments.

---

## 10. What is ExternalName?

`ExternalName` maps a Kubernetes Service to an external DNS name.

Example:

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

It does not select Pods like a normal Service.

---

## 11. What is the difference between port and targetPort?

`port` is the port exposed by the Service.

`targetPort` is the port on the backend Pod/application.

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Traffic:

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

### Remember

```text
port       → Service port
targetPort → Pod/application port
```

---

## 12. What is nodePort?

`nodePort` is the port exposed on each Kubernetes Node when the Service type is `NodePort`.

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

Traffic:

```text
NodeIP:30080
      │
      ▼
Service:80
      │
      ▼
Pod:8080
```

---

## 13. How does a Service select Pods?

A Service uses a **selector** to match Pod labels.

Service:

```yaml
selector:
  app: nginx
```

Pod:

```yaml
metadata:
  labels:
    app: nginx
```

Because they match:

```text
Service
   │
   │ app=nginx
   ▼
Pod
   │
   └── app=nginx ✓
```

---

## 14. What happens if the Service selector does not match any Pod?

The Service will have no backend endpoints.

Example:

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

These do not match.

Check:

```bash
kubectl get endpoints <service>
```

You may see:

```text
<none>
```

This is one of the most common Service troubleshooting problems.

---

## 15. How can you check the Pods selected by a Service?

First check the Service selector:

```bash
kubectl describe svc <service>
```

Then check Pod labels:

```bash
kubectl get pods --show-labels
```

You can compare:

```text
Service Selector
      │
      ▼
Pod Labels
```

They must match.

---

## 16. What are Endpoints in Kubernetes?

Endpoints represent the backend addresses associated with a Service.

Example:

```bash
kubectl get endpoints nginx-service
```

Output:

```text
NAME            ENDPOINTS
nginx-service   10.244.0.5:80,10.244.0.6:80
```

The endpoints contain backend Pod IP addresses and ports.

---

## 17. What are EndpointSlices?

EndpointSlices are the modern Kubernetes representation of Service backend endpoints.

Check them using:

```bash
kubectl get endpointslice
```

For a specific Service:

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=nginx-service
```

They improve scalability when Services have many endpoints.

---

## 18. What happens when a Service has no endpoints?

If no Pods match the selector or no eligible backends are available:

```text
Service
   │
   ▼
No Endpoints
```

Check:

```bash
kubectl get endpoints <service>
```

Possible causes:

* Wrong selector
* Wrong Pod labels
* Pods do not exist
* Pods are not Ready
* Wrong namespace

---

## 19. What is a Headless Service?

A Headless Service is a Service that does not have a normal ClusterIP.

It uses:

```yaml
clusterIP: None
```

Example:

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

Example:

```text
NAME             TYPE        CLUSTER-IP
nginx-headless   ClusterIP   None
```

---

## 20. Why are Headless Services used?

Headless Services are useful when clients need to discover individual backend Pods instead of connecting through a single virtual ClusterIP.

Common use cases:

* Stateful applications
* Databases
* StatefulSets
* Distributed systems
* Direct Pod discovery

---

## 21. How does Kubernetes provide Service discovery?

Kubernetes provides Service discovery primarily through DNS.

Example:

```text
nginx-service
```

Fully qualified DNS name:

```text
nginx-service.default.svc.cluster.local
```

General format:

```text
<service-name>.<namespace>.svc.cluster.local
```

Example:

```bash
curl http://nginx-service
```

---

## 22. What is CoreDNS?

**CoreDNS** provides DNS-based service discovery inside a Kubernetes cluster.

Conceptually:

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
```

For example:

```text
backend-service.default.svc.cluster.local
```

can resolve to the Service's networking address.

---

## 23. Can a Service have multiple ports?

Yes.

A Service can expose multiple ports.

Example:

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
   ↓
Pod :8080

Service :443
   ↓
Pod :8443
```

---

## 24. What is sessionAffinity?

`sessionAffinity` controls whether connections from the same client can be directed to the same backend Pod.

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

---

## 25. What is a Service without a selector?

A Service does not always need a selector.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: external-api

spec:
  ports:
    - port: 80
```

Without a selector, Kubernetes does not automatically discover Pods for the Service.

Backend endpoints can be defined separately using EndpointSlices.

---

## 26. Can a Service exist without Pods?

Yes.

A Service can exist even when there are no matching backend Pods.

Example:

```text
Service
   │
   ▼
No matching Pods
   │
   ▼
No Endpoints
```

Check:

```bash
kubectl get svc
kubectl get endpoints <service>
```

---

## 27. Does a Service create Pods?

No.

A Service does **not** create or manage Pods.

Typically:

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
Pods
    ▲
    │
 Service
```

The Deployment manages Pods.

The Service provides networking to those Pods.

---

## 28. What is the relationship between Deployment and Service?

A Deployment manages the desired number of Pods.

A Service provides stable networking to those Pods.

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

---

## 29. What is the difference between Service and Ingress?

A **Service** provides networking access to Pods.

An **Ingress** provides HTTP/HTTPS routing to Services.

```text
Internet
   │
   ▼
Ingress
   │
   ├── /api → API Service
   │
   └── / → Web Service
```

### Simple Difference

```text
Service → Stable access to Pods

Ingress → HTTP/HTTPS routing
```

---

## 30. What is the difference between ClusterIP and NodePort?

| ClusterIP                            | NodePort                   |
| ------------------------------------ | -------------------------- |
| Internal access                      | External node-level access |
| Default type                         | Explicitly configured      |
| Uses ClusterIP                       | Uses Node IP + NodePort    |
| Not directly exposed outside cluster | Exposed through Node port  |

Architecture:

```text
ClusterIP:

Pod → Service → Pods
```

```text
NodePort:

Client → NodeIP:30080 → Service → Pods
```

---

## 31. What is the difference between NodePort and LoadBalancer?

| NodePort                    | LoadBalancer                           |
| --------------------------- | -------------------------------------- |
| Uses Node IP + port         | Uses external load balancer            |
| Manual node-level exposure  | Cloud/provider-managed exposure        |
| Port usually in 30000-32767 | External IP/DNS depends on environment |
| Common for testing          | Common for cloud deployments           |

---

## 32. What is the difference between Service and Pod IP?

| Pod IP             | Service IP                  |
| ------------------ | --------------------------- |
| Belongs to a Pod   | Belongs to a Service        |
| Can change         | Stable while Service exists |
| Represents one Pod | Represents backend Pods     |
| Ephemeral          | Stable endpoint             |

---

## 33. How does Service load balancing work?

When multiple Pods match a Service selector:

```text
              Service
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      Pod 1    Pod 2    Pod 3
```

Traffic can be distributed across available backend endpoints.

Example:

```text
Request 1 → Pod 1
Request 2 → Pod 2
Request 3 → Pod 3
```

The exact behavior depends on the Kubernetes networking implementation and configuration.

---

## 34. What is kube-proxy?

`kube-proxy` implements Service networking rules on many Kubernetes installations.

Depending on the cluster configuration, it can use mechanisms such as:

```text
iptables
IPVS
```

Conceptually:

```text
Service IP
    │
    ▼
kube-proxy networking rules
    │
    ▼
Pod IP
```

---

## 35. How do you troubleshoot a Service with no endpoints?

Follow this workflow:

```text
kubectl get svc
       ↓
kubectl describe svc <service>
       ↓
Check selector
       ↓
kubectl get pods --show-labels
       ↓
Check Pod labels
       ↓
kubectl get endpoints <service>
       ↓
kubectl get endpointslice
```

Look for:

* Selector mismatch
* Missing Pods
* Pods not Ready
* Wrong namespace
* Incorrect configuration

---

## 36. How do you troubleshoot a Service that cannot reach the application?

Start with:

```bash
kubectl get svc
```

Then:

```bash
kubectl describe svc <service>
```

Check endpoints:

```bash
kubectl get endpoints <service>
```

Check EndpointSlices:

```bash
kubectl get endpointslice
```

Check Pods:

```bash
kubectl get pods -o wide
```

Check labels:

```bash
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
Application Port
   ↓
DNS / Network
```

---

## 37. How do you test a Service from inside the cluster?

Run a temporary Pod:

```bash
kubectl run test-pod \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Then:

```bash
curl http://<service>
```

You can also test the fully qualified DNS name:

```bash
curl http://<service>.<namespace>.svc.cluster.local
```

---

## 38. How do you test Kubernetes Service DNS?

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

## 39. How do you expose a Service temporarily to your local machine?

Use port forwarding:

```bash
kubectl port-forward service/<service> 8080:80
```

Then access:

```text
http://localhost:8080
```

This is useful for testing and debugging.

---

## 40. How do you check a Service's ClusterIP?

Use:

```bash
kubectl get svc <service> \
  -o jsonpath='{.spec.clusterIP}'
```

---

## 41. How do you check the Service type?

Use:

```bash
kubectl get svc <service> \
  -o jsonpath='{.spec.type}'
```

---

## 42. How do you get the complete Service YAML?

Use:

```bash
kubectl get svc <service> -o yaml
```

This is useful when troubleshooting the actual configuration.

---

## 43. What happens if targetPort is wrong?

Suppose:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

But the application is listening on:

```text
8081
```

Traffic becomes:

```text
Service :80
    ↓
Pod :8080
    X
Application :8081
```

The Service cannot successfully connect to the application.

---

## 44. What is the difference between Endpoints and EndpointSlices?

Both represent backend endpoints for Services.

```text
Endpoints
    ↓
Traditional endpoint representation
```

```text
EndpointSlices
    ↓
Modern scalable endpoint representation
```

EndpointSlices are designed to scale better for Services with many endpoints.

---

## 45. What is a Service selector?

A Service selector specifies which Pods should receive traffic.

Example:

```yaml
selector:
  app: nginx
```

It matches Pods with:

```yaml
labels:
  app: nginx
```

---

## 46. Can a Service route traffic to Pods in another namespace?

A normal Service selects Pods within its own namespace.

Service DNS names include the namespace:

```text
service-name.namespace.svc.cluster.local
```

For example:

```text
backend-service.production.svc.cluster.local
```

This makes namespace boundaries explicit in Service discovery.

---

## 47. What happens when a backend Pod is deleted?

The Pod is removed from the Service's backend endpoints when it is no longer an eligible backend.

If a Deployment manages the application:

```text
Pod deleted
    ↓
Deployment notices desired replicas are not met
    ↓
Replacement Pod created
    ↓
Pod becomes Ready
    ↓
Service can send traffic to it
```

This is one reason Services work well with Deployments.

---

## 48. Why shouldn't applications directly use Pod IPs?

Pod IPs are ephemeral.

A Pod can be:

```text
Deleted
Recreated
Rescheduled
Replaced
```

Its IP may change.

Instead:

```text
Application
     │
     ▼
Service DNS
     │
     ▼
Backend Pods
```

This provides stable application-to-application communication.

---

## 49. Can a Service expose multiple Pods?

Yes.

A Service can select multiple Pods using labels.

```text
             Service
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     Pod 1    Pod 2    Pod 3
```

All matching backend Pods can receive Service traffic.

---

## 50. What are the most important Service troubleshooting commands?

The most useful commands are:

```bash
kubectl get svc
```

```bash
kubectl describe svc <service>
```

```bash
kubectl get endpoints <service>
```

```bash
kubectl get endpointslice
```

```bash
kubectl get pods --show-labels
```

```bash
kubectl get pods -o wide
```

```bash
kubectl get svc <service> -o yaml
```

For testing:

```bash
curl http://<service>
```

and:

```bash
nslookup <service>
```

---

# 🔥 Quick Interview Revision

```text
Service
   ↓
Stable network endpoint
   ↓
Selects Pods using labels
   ↓
Provides Service discovery
   ↓
Can distribute traffic
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

# 🎯 One-Line Interview Answer

> **A Kubernetes Service provides a stable network endpoint and service discovery mechanism for accessing a dynamic group of Pods selected using labels.**

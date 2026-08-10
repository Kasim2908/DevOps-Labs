# ☸️ Kubernetes Services

## Overview

A **Service** is a Kubernetes object that provides a stable network endpoint for accessing a group of Pods.

Pods are temporary and their IP addresses can change when Pods are recreated.

A Service provides:

* Stable IP address
* Stable DNS name
* Load balancing across Pods
* Service discovery
* Network access to applications

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

The Service selects Pods using **labels and selectors**.

---

# Why Do We Need Services?

Pod IP addresses are not reliable because Pods are ephemeral.

Example:

```text
Before:

Pod 1 → 10.244.0.5
Pod 2 → 10.244.0.6
Pod 3 → 10.244.0.7
```

If Pod 2 is deleted and recreated:

```text
Pod 2 → 10.244.0.12
```

The Pod IP changed.

Clients should not have to track changing Pod IP addresses.

A Service solves this problem:

```text
                Service
              10.96.20.10
                   │
          ┌────────┼────────┐
          │        │        │
        Pod 1    Pod 2    Pod 3
```

The Service provides a stable endpoint while Pods can be replaced.

---

# Service Architecture

```text
                         Client
                           │
                           ▼
                    ┌─────────────┐
                    │   Service   │
                    │ 10.96.x.x   │
                    └──────┬──────┘
                           │
                 Service Selector
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
          Pod 1          Pod 2          Pod 3
        10.244.0.5     10.244.0.6     10.244.0.7
```

The Service does not create Pods.

Instead, it provides networking for existing Pods.

---

# Service Components

A Service mainly uses:

```text
Service
   │
   ├── Selector
   │
   ├── Port
   │
   ├── TargetPort
   │
   └── Type
```

## Selector

The selector determines which Pods receive traffic.

Example:

```yaml
selector:
  app: nginx
```

The Service selects Pods having:

```yaml
labels:
  app: nginx
```

---

## Port

The `port` is the port exposed by the Service.

Example:

```yaml
port: 80
```

---

## TargetPort

The `targetPort` is the port on the Pod/container where traffic is forwarded.

Example:

```yaml
targetPort: 8080
```

Architecture:

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

---

# Port vs TargetPort

| Field        | Meaning                   |
| ------------ | ------------------------- |
| `port`       | Service port              |
| `targetPort` | Pod/container port        |
| `nodePort`   | Port exposed on each Node |

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Traffic flow:

```text
Service :80
     │
     ▼
Pod :8080
```

---

# Basic Service YAML

Example:

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

---

# Kubernetes Service YAML Structure

## apiVersion

Defines the Kubernetes API version.

```yaml
apiVersion: v1
```

---

## kind

Defines the Kubernetes resource type.

```yaml
kind: Service
```

---

## metadata

Contains identifying information.

```yaml
metadata:
  name: nginx-service
```

---

## spec

Defines the desired Service configuration.

```yaml
spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

---

# Creating a Service

Apply the YAML:

```bash
kubectl apply -f service.yaml
```

Check Services:

```bash
kubectl get services
```

Short form:

```bash
kubectl get svc
```

Example:

```text
NAME            TYPE        CLUSTER-IP      PORT(S)
nginx-service   ClusterIP   10.96.20.10     80/TCP
```

---

# Service Types

Kubernetes provides different Service types.

The commonly used types are:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

Architecture:

```text
ClusterIP
    │
    ▼
Internal access

NodePort
    │
    ▼
Node IP + Port

LoadBalancer
    │
    ▼
External Load Balancer

ExternalName
    │
    ▼
External DNS name
```

---

# ClusterIP

`ClusterIP` is the default Service type.

It provides an internal IP address that is accessible inside the Kubernetes cluster.

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

Traffic:

```text
Pod / Service inside cluster
          │
          ▼
     ClusterIP
          │
          ▼
      nginx Pods
```

---

# ClusterIP Example

Create the Service:

```bash
kubectl apply -f nginx-service.yaml
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

The Service is reachable from inside the cluster.

---

# Accessing ClusterIP

You can test a ClusterIP Service from another Pod.

Example:

```bash
kubectl run test-pod \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Inside the Pod:

```bash
curl http://nginx-service
```

Or:

```bash
curl http://10.96.20.10
```

The Service forwards traffic to one of the selected Pods.

---

# DNS Service Discovery

Kubernetes automatically provides DNS records for Services.

For example:

```text
nginx-service
```

can be resolved inside the same namespace.

Example:

```bash
curl http://nginx-service
```

Fully qualified DNS name:

```text
nginx-service.default.svc.cluster.local
```

Structure:

```text
<service-name>.<namespace>.svc.cluster.local
```

Example:

```text
nginx-service.default.svc.cluster.local
```

---

# Service DNS Architecture

```text
Application Pod
      │
      │ DNS Query
      ▼
CoreDNS
      │
      ▼
nginx-service.default.svc.cluster.local
      │
      ▼
Service
      │
      ▼
Pods
```

Kubernetes uses **CoreDNS** for cluster DNS service discovery.

---

# NodePort

`NodePort` exposes a Service on a port on each Kubernetes Node.

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

Traffic:

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

# NodePort Range

The default NodePort range is:

```text
30000-32767
```

Example:

```yaml
nodePort: 30080
```

The Service can then be accessed through:

```text
<Node-IP>:30080
```

---

# Check NodePort

Run:

```bash
kubectl get svc
```

Example:

```text
NAME             TYPE       CLUSTER-IP      EXTERNAL-IP
nginx-nodeport   NodePort   10.96.30.20     <none>
```

Check the port:

```bash
kubectl get svc nginx-nodeport
```

Example:

```text
80:30080/TCP
```

This means:

```text
Service Port = 80
NodePort      = 30080
```

---

# LoadBalancer

`LoadBalancer` exposes a Service using an external load balancer.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-loadbalancer

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
Kubernetes Service
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

In cloud environments, Kubernetes can integrate with a cloud provider's load balancer.

---

# LoadBalancer on Minikube

In Minikube, a LoadBalancer Service may remain:

```text
EXTERNAL-IP: <pending>
```

You can use:

```bash
minikube tunnel
```

Then check:

```bash
kubectl get svc
```

The LoadBalancer can receive an accessible IP depending on the Minikube environment.

---

# ExternalName

`ExternalName` maps a Service to an external DNS name.

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

The Service does not select Pods.

Instead, DNS resolution points to the external name.

Conceptually:

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

# Service Type Comparison

| Type         | Main Purpose             | Accessible From     |
| ------------ | ------------------------ | ------------------- |
| ClusterIP    | Internal service         | Cluster             |
| NodePort     | Expose through Node port | Outside cluster     |
| LoadBalancer | External load balancing  | Internet / External |
| ExternalName | External DNS mapping     | Cluster             |

---

# Service Selectors

Services use selectors to identify backend Pods.

Example Pod:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod
  labels:
    app: nginx

spec:
  containers:
    - name: nginx
      image: nginx
```

Service:

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

The selector:

```yaml
app: nginx
```

matches:

```yaml
labels:
  app: nginx
```

Therefore the Service sends traffic to the Pod.

---

# Labels and Selectors

```text
Service
   │
   │ selector:
   │ app=nginx
   ▼
Kubernetes
   │
   ├── Pod 1 → app=nginx ✓
   ├── Pod 2 → app=nginx ✓
   ├── Pod 3 → app=redis ✗
   └── Pod 4 → app=nginx ✓
```

The Service sends traffic only to matching Pods.

---

# Service Load Balancing

If multiple Pods match the Service selector:

```text
              Service
                 │
        ┌────────┼────────┐
        │        │        │
        ▼        ▼        ▼
      Pod 1    Pod 2    Pod 3
```

Traffic can be distributed across the available backend endpoints.

Example:

```text
Request 1 → Pod 1
Request 2 → Pod 2
Request 3 → Pod 3
Request 4 → Pod 1
```

The exact traffic behavior depends on the Kubernetes networking implementation and configuration.

---

# Endpoints

A Service needs backend endpoints.

Check them using:

```bash
kubectl get endpoints
```

Example:

```text
NAME            ENDPOINTS
nginx-service   10.244.0.5:80,10.244.0.6:80
```

These are the Pod IP addresses and ports selected by the Service.

---

# EndpointSlices

Modern Kubernetes uses **EndpointSlices** to represent Service endpoints.

Check:

```bash
kubectl get endpointslice
```

You can filter EndpointSlices for a Service:

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=nginx-service
```

Example:

```text
NAME                     ADDRESSTYPE   PORTS   ENDPOINTS
nginx-service-abc123     IPv4          80      10.244.0.5,10.244.0.6
```

EndpointSlices improve scalability compared with managing very large endpoint lists as a single object.

---

# Service Without a Selector

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

Without a selector, Kubernetes does not automatically discover Pods for this Service.

You can manually define backend endpoints using appropriate Kubernetes networking objects such as EndpointSlices.

Conceptually:

```text
Service
   │
   ▼
Manually defined Endpoint
   │
   ▼
External Application
```

---

# Headless Service

A **Headless Service** does not allocate a normal ClusterIP.

It is created using:

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

# Why Use Headless Services?

Headless Services are useful when clients need to discover individual backend Pods rather than connecting through a single virtual ClusterIP.

Common use cases include:

* Stateful applications
* Databases
* Distributed systems
* Direct Pod discovery
* StatefulSets

Architecture:

```text
DNS
 │
 ├── Pod 1 IP
 ├── Pod 2 IP
 └── Pod 3 IP
```

Instead of:

```text
DNS
 │
 ▼
ClusterIP
 │
 ├── Pod 1
 ├── Pod 2
 └── Pod 3
```

---

# Service and Deployment

Services are commonly used together with Deployments.

Example:

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

The Deployment manages the Pods.

The Service provides stable networking to those Pods.

---

# Service + Deployment Example

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:latest

          ports:
            - containerPort: 80
```

Service:

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

Architecture:

```text
                Service
                   │
          ┌────────┼────────┐
          │        │        │
          ▼        ▼        ▼
        Pod 1    Pod 2    Pod 3
          ▲        ▲        ▲
          └────────┼────────┘
                   │
               Deployment
```

---

# Service Discovery

Kubernetes provides two common ways to discover Services:

```text
1. DNS
2. Environment Variables
```

DNS is generally preferred for modern applications.

Example:

```bash
curl http://nginx-service
```

Fully qualified:

```text
nginx-service.default.svc.cluster.local
```

---

# Service Environment Variables

When a Pod is created, Kubernetes can provide environment variables for Services that already exist.

For example:

```text
NGINX_SERVICE_SERVICE_HOST
NGINX_SERVICE_SERVICE_PORT
```

However, applications generally use DNS-based service discovery rather than relying on automatically generated Service environment variables.

---

# Service Session Affinity

By default, traffic can be distributed among Service backends.

You can configure session affinity.

Example:

```yaml
spec:
  sessionAffinity: ClientIP
```

This can cause requests from the same client IP to be directed to the same backend Pod for the Service's configured session-affinity behavior.

Default:

```yaml
sessionAffinity: None
```

---

# Service Ports

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

Named ports make multi-port Services easier to understand and reference.

---

# Service Troubleshooting

Service troubleshooting is an important Kubernetes skill.

Basic workflow:

```text
kubectl get svc
       ↓
kubectl describe svc
       ↓
Check selector
       ↓
Check EndpointSlices
       ↓
Check Pod labels
       ↓
Test DNS
       ↓
Test connectivity
       ↓
Find root cause
```

---

# Common Service Problems

## 1. Service Has No Endpoints

Example:

```text
NAME            ENDPOINTS
nginx-service   <none>
```

Possible causes:

* Wrong selector
* Pod labels do not match
* Pods are not Ready
* Pods do not exist
* Incorrect namespace

Check Service:

```bash
kubectl describe svc nginx-service
```

Check Pods:

```bash
kubectl get pods --show-labels
```

Check EndpointSlices:

```bash
kubectl get endpointslice
```

---

# 2. Wrong Selector

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

Therefore:

```text
Service
   │
   │ app=nginx
   ▼
No matching Pods
```

Fix the selector or Pod labels so they match.

---

# 3. Wrong targetPort

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

But the application is actually listening on:

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

The Service cannot successfully reach the application.

Verify the application's listening port and configure `targetPort` correctly.

---

# 4. Service DNS Not Working

Test DNS from inside a Pod:

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  -it --rm \
  -- sh
```

Then:

```bash
nslookup nginx-service
```

Or:

```bash
nslookup nginx-service.default.svc.cluster.local
```

If DNS fails, investigate CoreDNS and cluster networking.

---

# 5. Service Exists but Application Is Not Reachable

Check:

```bash
kubectl get svc
```

Then:

```bash
kubectl describe svc <service>
```

Then:

```bash
kubectl get endpointslice
```

Then:

```bash
kubectl get pods -o wide
```

Finally test connectivity from another Pod.

Example:

```bash
curl http://nginx-service
```

---

# Important Service Commands

## List Services

```bash
kubectl get services
```

Short form:

```bash
kubectl get svc
```

---

## Detailed Service Information

```bash
kubectl describe service <service>
```

---

## Get Service YAML

```bash
kubectl get svc <service> -o yaml
```

---

## Get Service IP

```bash
kubectl get svc <service> \
  -o jsonpath='{.spec.clusterIP}'
```

---

## Get Service Type

```bash
kubectl get svc <service> \
  -o jsonpath='{.spec.type}'
```

---

## Get Service Ports

```bash
kubectl get svc <service>
```

---

## Get Endpoints

```bash
kubectl get endpoints <service>
```

---

## Get EndpointSlices

```bash
kubectl get endpointslice
```

---

## Get EndpointSlices for a Service

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=<service>
```

---

## Describe EndpointSlice

```bash
kubectl describe endpointslice <endpointslice>
```

---

## Test Service Using Port Forward

You can temporarily forward a local port to a Service:

```bash
kubectl port-forward service/<service> 8080:80
```

Then access:

```text
http://localhost:8080
```

---

# Service vs Pod IP

| Pod IP                | Service IP                  |
| --------------------- | --------------------------- |
| Can change            | Stable virtual IP           |
| Belongs to Pod        | Belongs to Service          |
| Pod-specific          | Represents a group of Pods  |
| Not ideal for clients | Designed for stable access  |
| Ephemeral             | Stable while Service exists |

---

# Service vs Ingress

A Service provides network access to Pods.

Ingress provides HTTP/HTTPS routing to Services.

Architecture:

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
        ┌──┴──┐
        ▼     ▼
      Pod   Pod
```

In simple terms:

```text
Service → Exposes application
Ingress → Routes HTTP/HTTPS traffic
```

---

# Service vs NodePort vs LoadBalancer

```text
                    Service
                       │
          ┌────────────┼────────────┐
          │            │            │
       ClusterIP     NodePort    LoadBalancer
          │            │            │
       Internal     Node IP      External LB
        access       + Port        access
```

---

# Service Traffic Flow

For a typical Service:

```text
Client
  │
  ▼
Service
  │
  ▼
Service Virtual IP
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

Example:

```text
Client
  │
  │ nginx-service:80
  ▼
ClusterIP
10.96.20.10:80
  │
  ▼
Pod
10.244.0.5:80
```

---

# Service and kube-proxy

Kubernetes Services require networking rules that direct Service traffic to backend endpoints.

On many Kubernetes installations, **kube-proxy** implements these Service networking rules using mechanisms such as:

```text
iptables
IPVS
```

depending on the cluster configuration.

Conceptually:

```text
Client
   │
   ▼
Service IP
   │
   ▼
kube-proxy networking rules
   │
   ▼
Pod IP
```

The exact implementation depends on the Kubernetes environment.

---

# Service Without Pods

A Service can exist even when no matching Pods are available.

Example:

```text
Service
   │
   ▼
No matching Pods
```

Check:

```bash
kubectl get endpoints <service>
```

You may see:

```text
<none>
```

This is a common troubleshooting scenario.

---

# Production Insight

In real-world Kubernetes applications, Services are commonly combined with Deployments.

Typical architecture:

```text
                    Internet
                       │
                       ▼
                    Ingress
                       │
                       ▼
                  Web Service
                       │
              ┌────────┼────────┐
              │        │        │
              ▼        ▼        ▼
            Pod 1    Pod 2    Pod 3
              ▲        ▲        ▲
              └────────┼────────┘
                       │
                  Deployment
```

The responsibilities are separated:

```text
Deployment
    ↓
Manages Pods

Service
    ↓
Provides stable networking

Ingress
    ↓
Provides HTTP/HTTPS routing
```

---

# Real-World Example

Imagine an application with:

```text
Frontend
Backend
Database
```

The architecture could be:

```text
                    Internet
                       │
                       ▼
                    Ingress
                       │
                       ▼
                Frontend Service
                       │
                 Frontend Pods
                       │
                       ▼
                 Backend Service
                       │
                 Backend Pods
                       │
                       ▼
                Database Service
                       │
                 Database Pods
```

Each application component can communicate using a stable Service DNS name.

For example:

```text
backend-service
```

instead of directly using a Pod IP.

---

# Service Best Practices

* Use Services instead of directly depending on Pod IPs.
* Use meaningful Service names.
* Keep selectors consistent with Pod labels.
* Prefer DNS-based Service discovery.
* Verify EndpointSlices during troubleshooting.
* Use `ClusterIP` for internal communication.
* Use `NodePort` when direct node-level exposure is appropriate.
* Use `LoadBalancer` for cloud-based external exposure.
* Use Ingress for HTTP/HTTPS routing when appropriate.
* Avoid exposing databases publicly unless required.
* Use namespaces to organize applications and Services.

---

# Important Interview Questions

## 1. What is a Service in Kubernetes?

A Service is a Kubernetes object that provides a stable network endpoint for accessing a group of Pods.

---

## 2. Why do we need Services?

Because Pod IP addresses are ephemeral and can change when Pods are recreated.

A Service provides stable networking and service discovery.

---

## 3. What is the default Service type?

The default Service type is:

```text
ClusterIP
```

---

## 4. What is ClusterIP?

ClusterIP exposes a Service internally within the Kubernetes cluster.

---

## 5. What is NodePort?

NodePort exposes a Service through a port on each Kubernetes Node.

The default NodePort range is:

```text
30000-32767
```

---

## 6. What is LoadBalancer?

LoadBalancer exposes a Service through an external load balancer, typically when supported by the cloud environment.

---

## 7. What is targetPort?

`targetPort` specifies the port on the backend Pod where traffic should be sent.

Example:

```yaml
port: 80
targetPort: 8080
```

---

## 8. What is the difference between port and targetPort?

```text
port
 ↓
Service port

targetPort
 ↓
Pod/application port
```

---

## 9. How does a Service find Pods?

A Service uses a **selector** to match Pod labels.

Example:

```yaml
selector:
  app: nginx
```

---

## 10. What happens if the selector does not match any Pods?

The Service has no backend endpoints.

Check:

```bash
kubectl get endpoints <service>
```

or:

```bash
kubectl get endpointslice
```

---

## 11. What is a Headless Service?

A Headless Service is a Service with:

```yaml
clusterIP: None
```

It does not provide a normal virtual ClusterIP and is commonly used for direct Pod discovery.

---

## 12. How do Pods discover Services?

Primarily through Kubernetes DNS.

Example:

```text
nginx-service.default.svc.cluster.local
```

---

## 13. How do you troubleshoot a Service?

Use:

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
Network/DNS
```

---

# Key Takeaways

* A Service provides stable networking for Kubernetes Pods.
* Pod IP addresses can change.
* Services provide a stable endpoint.
* Services use selectors to identify backend Pods.
* `ClusterIP` is the default Service type.
* `NodePort` exposes a Service through a node port.
* `LoadBalancer` provides external load-balancer access when supported.
* `ExternalName` maps a Service to an external DNS name.
* `targetPort` is the backend Pod/application port.
* `port` is the Service port.
* Services can have multiple ports.
* Kubernetes provides DNS-based Service discovery.
* EndpointSlices represent Service backend endpoints.
* Headless Services use `clusterIP: None`.
* Services are commonly used with Deployments.
* Services do not create or manage Pods.
* `kubectl describe svc` is useful for troubleshooting.
* Empty Endpoints often indicate a selector, label, readiness, or Pod problem.
* Ingress can route HTTP/HTTPS traffic to Services.
* Services are a fundamental component of Kubernetes networking.


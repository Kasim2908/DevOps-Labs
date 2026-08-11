# ☸️ Kubernetes Ingress

## Overview

An **Ingress** is a Kubernetes API object that manages external HTTP and HTTPS access to applications running inside a Kubernetes cluster.

Ingress typically provides:

* HTTP/HTTPS routing
* Host-based routing
* Path-based routing
* TLS/SSL termination
* Routing traffic to Services
* Centralized external access

Example:

```text
                    Internet
                       │
                       ▼
                    Ingress
                       │
              ┌────────┴────────┐
              │                 │
           /api               /
              │                 │
              ▼                 ▼
        Backend Service    Frontend Service
              │                 │
           Pods              Pods
```

---

# Why Do We Need Ingress?

Suppose we have multiple applications:

```text
Frontend
Backend
Admin Panel
```

Without Ingress, we could expose each application separately:

```text
frontend.example.com → LoadBalancer
api.example.com      → LoadBalancer
admin.example.com    → LoadBalancer
```

This can require multiple external load balancers.

Ingress allows centralized routing:

```text
                         Internet
                            │
                            ▼
                         Ingress
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
          Frontend        Backend        Admin
          Service         Service        Service
```

One external entry point can route traffic to multiple Services.

---

# Ingress Architecture

```text
                         Internet
                            │
                            ▼
                   ┌─────────────────┐
                   │     Ingress     │
                   │ Routing Rules   │
                   └────────┬────────┘
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
        Frontend        Backend          Admin
         Service         Service         Service
             │              │              │
             ▼              ▼              ▼
           Pods            Pods            Pods
```

Ingress does not normally route directly to Pods.

It routes traffic to **Services**.

---

# Ingress Components

A typical Ingress architecture contains:

```text
Ingress
   │
   ├── Ingress Rules
   │
   ├── Host
   │
   ├── Path
   │
   ├── Backend Service
   │
   └── TLS Configuration
```

An **Ingress Controller** watches these rules and implements the actual routing.

---

# Ingress vs Ingress Controller

These are different concepts.

## Ingress

Ingress is a Kubernetes API object containing routing rules.

Example:

```yaml
kind: Ingress
```

It describes:

```text
Which host?
Which path?
Which Service?
Which port?
```

---

## Ingress Controller

An **Ingress Controller** is the component that actually implements those routing rules.

Examples include controllers based on:

* NGINX
* HAProxy
* Traefik
* Kong
* Cloud-provider load balancers

Conceptually:

```text
Ingress Resource
       │
       ▼
Ingress Controller
       │
       ▼
Services
       │
       ▼
Pods
```

Creating an Ingress resource alone does not automatically provide routing unless an appropriate Ingress Controller is running.

---

# Ingress API Version

A modern Ingress resource uses:

```yaml
apiVersion: networking.k8s.io/v1
```

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
```

---

# Basic Ingress YAML

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: nginx-ingress

spec:
  ingressClassName: nginx

  rules:
    - host: example.com

      http:
        paths:
          - path: /
            pathType: Prefix

            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```

Architecture:

```text
example.com
     │
     ▼
Ingress
     │
     ▼
nginx-service:80
     │
     ▼
NGINX Pods
```

---

# Kubernetes YAML Structure

## apiVersion

Defines the API version.

```yaml
apiVersion: networking.k8s.io/v1
```

---

## kind

Defines the Kubernetes resource type.

```yaml
kind: Ingress
```

---

## metadata

Contains identifying information.

```yaml
metadata:
  name: nginx-ingress
```

---

## spec

Defines the desired Ingress configuration.

```yaml
spec:
  ingressClassName: nginx

  rules:
    - host: example.com
```

---

# IngressClass

`IngressClass` identifies which Ingress Controller should implement an Ingress.

Example:

```yaml
spec:
  ingressClassName: nginx
```

Check available IngressClasses:

```bash
kubectl get ingressclass
```

Example:

```text
NAME    CONTROLLER
nginx   k8s.io/ingress-nginx
```

Conceptually:

```text
Ingress
   │
   │ ingressClassName: nginx
   ▼
NGINX Ingress Controller
```

---

# Create an Ingress

Apply the configuration:

```bash
kubectl apply -f ingress.yaml
```

Check:

```bash
kubectl get ingress
```

Detailed information:

```bash
kubectl describe ingress <ingress>
```

---

# Ingress Rules

Ingress rules determine how incoming traffic should be routed.

A rule can contain:

```text
Host
  ↓
HTTP
  ↓
Path
  ↓
Backend Service
```

Example:

```yaml
rules:
  - host: example.com

    http:
      paths:
        - path: /
          pathType: Prefix

          backend:
            service:
              name: frontend-service
              port:
                number: 80
```

---

# Host-Based Routing

Host-based routing sends traffic to different Services based on the hostname.

Example:

```text
app.example.com
api.example.com
admin.example.com
```

Ingress:

```yaml
spec:
  rules:

    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80

    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80
```

Traffic:

```text
app.example.com
      │
      ▼
Frontend Service

api.example.com
      │
      ▼
Backend Service
```

---

# Path-Based Routing

Path-based routing sends traffic to different Services based on the URL path.

Example:

```text
example.com/
example.com/api
example.com/admin
```

Ingress:

```yaml
spec:
  rules:
    - host: example.com

      http:
        paths:

          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80

          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80

          - path: /admin
            pathType: Prefix
            backend:
              service:
                name: admin-service
                port:
                  number: 80
```

Traffic:

```text
example.com/
      ↓
Frontend Service

example.com/api
      ↓
Backend Service

example.com/admin
      ↓
Admin Service
```

---

# Host-Based vs Path-Based Routing

| Routing    | Example           | Purpose           |
| ---------- | ----------------- | ----------------- |
| Host-based | `api.example.com` | Route by hostname |
| Path-based | `example.com/api` | Route by URL path |

---

# Path Types

Ingress supports different path matching types.

The important ones are:

```text
Prefix
Exact
ImplementationSpecific
```

---

# Prefix Path Type

`Prefix` matches URL paths based on path prefixes.

Example:

```yaml
path: /api
pathType: Prefix
```

It can match:

```text
/api
/api/
/api/users
/api/products
```

Conceptually:

```text
/api
 ├── /api/users
 ├── /api/products
 └── /api/orders
```

---

# Exact Path Type

`Exact` matches the exact URL path.

Example:

```yaml
path: /login
pathType: Exact
```

It matches:

```text
/login
```

It does not generally match:

```text
/login/
/login/user
```

---

# ImplementationSpecific

`ImplementationSpecific` allows the Ingress Controller to determine how the path is interpreted.

Example:

```yaml
path: /api
pathType: ImplementationSpecific
```

Behavior can vary depending on the Ingress Controller.

For predictable behavior, `Prefix` or `Exact` is generally preferable when they fit the requirement.

---

# Path Type Comparison

| Path Type                | Example | Meaning                      |
| ------------------------ | ------- | ---------------------------- |
| `Exact`                  | `/api`  | Exact path                   |
| `Prefix`                 | `/api`  | Path prefix                  |
| `ImplementationSpecific` | `/api`  | Controller-specific behavior |

---

# Default Backend

An Ingress can define a default backend for requests that do not match a configured rule.

Example:

```yaml
spec:
  defaultBackend:
    service:
      name: default-service
      port:
        number: 80
```

Conceptually:

```text
Request
   │
   ├── Matches rule → Matching Service
   │
   └── No match → Default Backend
```

---

# Ingress TLS

Ingress can terminate HTTPS traffic using TLS certificates.

Example:

```yaml
spec:
  tls:
    - hosts:
        - example.com

      secretName: example-tls
```

The Secret contains the TLS certificate and private key.

---

# TLS Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: secure-ingress

spec:
  ingressClassName: nginx

  tls:
    - hosts:
        - example.com
      secretName: example-tls

  rules:
    - host: example.com

      http:
        paths:
          - path: /
            pathType: Prefix

            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

Traffic:

```text
Client
  │
  │ HTTPS
  ▼
Ingress Controller
  │
  │ TLS Termination
  ▼
Service
  │
  ▼
Pods
```

---

# TLS Secret

A TLS Secret can be created using:

```bash
kubectl create secret tls example-tls \
  --cert=tls.crt \
  --key=tls.key
```

Check:

```bash
kubectl get secret example-tls
```

---

# TLS Architecture

```text
                    HTTPS
Client ─────────────────────────► Ingress
                                  │
                                  │ TLS Termination
                                  ▼
                              Service
                                  │
                                  ▼
                                Pods
```

TLS termination at the Ingress means the Ingress Controller handles the external TLS connection.

---

# Ingress With Multiple Hosts

Example:

```yaml
spec:
  rules:

    - host: frontend.example.com
      http:
        paths:
          - path: /
            pathType: Prefix

            backend:
              service:
                name: frontend-service
                port:
                  number: 80

    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix

            backend:
              service:
                name: backend-service
                port:
                  number: 80
```

Architecture:

```text
                    Ingress
                       │
          ┌────────────┴────────────┐
          │                         │
 frontend.example.com          api.example.com
          │                         │
          ▼                         ▼
 Frontend Service             Backend Service
```

---

# Ingress With Multiple Paths

```yaml
spec:
  rules:
    - host: example.com

      http:
        paths:

          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80

          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80

          - path: /admin
            pathType: Prefix
            backend:
              service:
                name: admin-service
                port:
                  number: 80
```

Architecture:

```text
                 example.com
                      │
                   Ingress
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
          /          /api       /admin
          │           │           │
          ▼           ▼           ▼
       Frontend    Backend     Admin
       Service     Service     Service
```

---

# Ingress and Service

Ingress normally routes traffic to Services.

```text
Internet
   │
   ▼
Ingress
   │
   ▼
Service
   │
   ▼
Pods
```

Responsibilities:

```text
Ingress
   ↓
External HTTP/HTTPS routing

Service
   ↓
Stable access to Pods
```

---

# Ingress and Deployment

A common Kubernetes architecture is:

```text
                    Internet
                       │
                       ▼
                    Ingress
                       │
                       ▼
                    Service
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
            Pod 1    Pod 2    Pod 3
              ▲        ▲        ▲
              └────────┼────────┘
                       │
                   Deployment
```

Responsibilities:

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

# Ingress Traffic Flow

Typical request:

```text
Client
  │
  │ HTTPS request
  ▼
Ingress Controller
  │
  │ Host/Path matching
  ▼
Ingress Rule
  │
  ▼
Service
  │
  ▼
Pod
  │
  ▼
Application
```

Example:

```text
https://api.example.com/users
             │
             ▼
          Ingress
             │
       Host = api.example.com
       Path = /users
             │
             ▼
      Backend Service
             │
             ▼
        Backend Pod
```

---

# Ingress Controller

An Ingress Controller continuously watches Kubernetes resources.

Conceptually:

```text
Kubernetes API
      │
      ▼
Ingress Resource
      │
      ▼
Ingress Controller
      │
      ▼
Routing Configuration
      │
      ▼
Incoming Traffic
```

The controller translates Kubernetes Ingress rules into configuration understood by the underlying proxy/load balancer.

---

# Popular Ingress Controllers

Common examples include:

```text
NGINX Ingress Controller
Traefik
HAProxy
Kong
Cloud-provider Ingress Controllers
```

The exact features and annotations depend on the controller.

---

# Installing Ingress on Minikube

Minikube provides an Ingress addon.

Check addons:

```bash
minikube addons list
```

Enable Ingress:

```bash
minikube addons enable ingress
```

Check:

```bash
kubectl get pods -n ingress-nginx
```

You should see the Ingress Controller components.

---

# Check Ingress Controller

For the NGINX Ingress Controller namespace:

```bash
kubectl get pods -n ingress-nginx
```

Check Services:

```bash
kubectl get svc -n ingress-nginx
```

Check IngressClass:

```bash
kubectl get ingressclass
```

---

# Creating a Test Ingress

Example Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 2

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

Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: nginx-ingress

spec:
  ingressClassName: nginx

  rules:
    - host: nginx.local

      http:
        paths:
          - path: /
            pathType: Prefix

            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```

Apply:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

---

# Check Ingress

Run:

```bash
kubectl get ingress
```

Example:

```text
NAME            CLASS   HOSTS         ADDRESS
nginx-ingress   nginx   nginx.local   ...
```

Detailed information:

```bash
kubectl describe ingress nginx-ingress
```

---

# Host Resolution

If you use a hostname such as:

```text
nginx.local
```

your machine must resolve that hostname to the appropriate Ingress endpoint.

For local testing, this may involve:

```text
/etc/hosts
```

or another DNS mechanism.

Conceptually:

```text
nginx.local
     │
     ▼
Ingress IP
     │
     ▼
Ingress Controller
```

---

# Ingress Annotations

Some Ingress Controllers support additional configuration through annotations.

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```

Annotations are controller-specific.

Important:

```text
Not every annotation works with every Ingress Controller.
```

Always verify the annotations supported by the controller being used.

---

# Path Rewriting

Some Ingress Controllers support path rewriting.

For example:

```text
Incoming:

example.com/api/users
```

could be rewritten before reaching the backend.

Controller-specific configuration may look like:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```

The exact behavior depends on the Ingress Controller and its configuration.

---

# Ingress Troubleshooting

Ingress troubleshooting workflow:

```text
kubectl get ingress
        ↓
kubectl describe ingress
        ↓
Check IngressClass
        ↓
Check Ingress Controller
        ↓
Check Service
        ↓
Check Endpoints
        ↓
Check Pods
        ↓
Check DNS / Host
        ↓
Check TLS
        ↓
Test request
```

---

# Common Ingress Problems

## 1. Ingress Has No Address

Example:

```text
ADDRESS: <none>
```

Possible causes:

* Ingress Controller is not running
* Wrong IngressClass
* Controller is not watching the resource
* Load balancer is not available
* Local cluster configuration issue

Check:

```bash
kubectl get ingress
```

Then:

```bash
kubectl describe ingress <ingress>
```

Check:

```bash
kubectl get ingressclass
```

---

# 2. Ingress Controller Is Not Running

Check:

```bash
kubectl get pods -n ingress-nginx
```

If Pods are not running:

```bash
kubectl describe pod <pod> -n ingress-nginx
```

Check logs:

```bash
kubectl logs <pod> -n ingress-nginx
```

---

# 3. Wrong IngressClass

Check:

```bash
kubectl get ingressclass
```

Example:

```text
NAME
nginx
```

Ingress:

```yaml
spec:
  ingressClassName: nginx
```

If the class does not match the controller configuration, the Ingress may not be processed.

---

# 4. Service Has No Endpoints

Ingress can be configured correctly but still fail if the backend Service has no endpoints.

Check:

```bash
kubectl get endpoints <service>
```

And:

```bash
kubectl get endpointslice
```

If there are no endpoints, troubleshoot the Service.

Check:

```bash
kubectl get pods --show-labels
```

---

# 5. Wrong Service Name

Ingress:

```yaml
backend:
  service:
    name: backend-service
```

But actual Service:

```text
backend
```

The Ingress cannot route correctly.

Check:

```bash
kubectl get svc
```

Make sure the names match exactly.

---

# 6. Wrong Service Port

Ingress:

```yaml
backend:
  service:
    name: nginx-service
    port:
      number: 8080
```

But Service exposes:

```yaml
ports:
  - port: 80
```

The Ingress backend configuration is incorrect.

Check:

```bash
kubectl get svc nginx-service
```

---

# 7. Wrong Host

Ingress:

```yaml
host: app.example.com
```

But request is sent to:

```text
api.example.com
```

The rule may not match.

Check the configured hosts:

```bash
kubectl describe ingress <ingress>
```

---

# 8. DNS Problem

If:

```text
app.example.com
```

does not resolve to the Ingress endpoint, the request may never reach the Ingress Controller.

Check DNS resolution:

```bash
nslookup app.example.com
```

or:

```bash
dig app.example.com
```

For local testing, verify your hosts/DNS configuration.

---

# 9. TLS Certificate Problem

Check TLS configuration:

```bash
kubectl describe ingress <ingress>
```

Check Secret:

```bash
kubectl get secret
```

Check the specific Secret:

```bash
kubectl describe secret <tls-secret>
```

Verify:

```text
Host
  ↓
TLS Secret
  ↓
Certificate
```

---

# 10. 404 Not Found

Possible causes:

* Wrong host
* Wrong path
* Incorrect path type
* No matching Ingress rule
* Controller-specific routing behavior
* Backend configuration issue

Check:

```bash
kubectl describe ingress <ingress>
```

Verify:

```text
Host
Path
Service
Port
```

---

# 11. 502 / 503 Errors

These can indicate that the Ingress Controller cannot successfully reach the backend.

Check:

```bash
kubectl get svc
```

Then:

```bash
kubectl get endpoints
```

Then:

```bash
kubectl get pods
```

Also check Ingress Controller logs:

```bash
kubectl logs -n ingress-nginx <controller-pod>
```

---

# Important Ingress Commands

## List Ingresses

```bash
kubectl get ingress
```

---

## Detailed Ingress Information

```bash
kubectl describe ingress <ingress>
```

---

## Get Ingress YAML

```bash
kubectl get ingress <ingress> -o yaml
```

---

## List IngressClasses

```bash
kubectl get ingressclass
```

---

## Describe IngressClass

```bash
kubectl describe ingressclass <class>
```

---

## Check Ingress Controller Pods

```bash
kubectl get pods -n ingress-nginx
```

---

## Check Controller Services

```bash
kubectl get svc -n ingress-nginx
```

---

## Controller Logs

```bash
kubectl logs -n ingress-nginx <controller-pod>
```

---

## Check Backend Services

```bash
kubectl get svc
```

---

## Check Backend Endpoints

```bash
kubectl get endpoints
```

---

## Check EndpointSlices

```bash
kubectl get endpointslice
```

---

## Get All Resources

```bash
kubectl get all
```

---

# Ingress vs Service vs LoadBalancer

```text
Internet
   │
   ▼
Ingress
   │
   ▼
Service
   │
   ▼
Pods
```

Responsibilities:

```text
Ingress
   ↓
HTTP/HTTPS routing

Service
   ↓
Stable networking

Pods
   ↓
Run application
```

---

# Ingress vs LoadBalancer

| Ingress                        | LoadBalancer Service          |
| ------------------------------ | ----------------------------- |
| HTTP/HTTPS routing             | External Service exposure     |
| Host-based routing             | Basic external exposure       |
| Path-based routing             | Usually one Service           |
| TLS termination                | Depends on environment        |
| Can route to multiple Services | Primarily exposes a Service   |
| Requires Ingress Controller    | Uses LoadBalancer integration |

---

# Ingress vs NodePort

| Ingress                         | NodePort                                          |
| ------------------------------- | ------------------------------------------------- |
| HTTP/HTTPS routing              | Node IP + port                                    |
| Uses host/path rules            | Simple port exposure                              |
| Requires Ingress Controller     | Does not require Ingress Controller               |
| Can route many Services         | Exposes a Service                                 |
| Supports TLS through controller | TLS handling depends on application/configuration |

---

# Production Architecture

A typical production Kubernetes web architecture:

```text
                         Internet
                            │
                            ▼
                    External Load Balancer
                            │
                            ▼
                    Ingress Controller
                            │
                     Ingress Rules
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
        Frontend         Backend         Admin
         Service          Service         Service
             │              │              │
             ▼              ▼              ▼
           Pods            Pods            Pods
```

---

# Real-World Example

Suppose an application has:

```text
Frontend
Backend API
Admin Dashboard
```

Ingress can route:

```text
example.com
     ↓
Frontend

example.com/api
     ↓
Backend API

example.com/admin
     ↓
Admin Dashboard
```

Architecture:

```text
                         Internet
                            │
                            ▼
                         Ingress
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
          /                /api             /admin
          │                 │                 │
          ▼                 ▼                 ▼
      Frontend          Backend API        Admin
       Service            Service          Service
          │                 │                 │
         Pods              Pods              Pods
```

---

# Ingress Best Practices

* Always use an appropriate Ingress Controller.
* Use `networking.k8s.io/v1`.
* Explicitly specify `ingressClassName` when appropriate.
* Use `Prefix` or `Exact` path types when possible.
* Use TLS for production HTTPS traffic.
* Use meaningful hostnames and paths.
* Verify backend Services and endpoints.
* Avoid relying on controller-specific annotations without understanding them.
* Monitor Ingress Controller logs and metrics.
* Use DNS correctly.
* Keep TLS Secrets protected.
* Test host-based and path-based routing before production deployment.

---

# Important Interview Questions

## 1. What is Ingress?

Ingress is a Kubernetes API object that defines rules for HTTP and HTTPS traffic entering the cluster.

---

## 2. Does Ingress directly expose Pods?

No.

Ingress normally routes traffic to Services.

```text
Ingress
   ↓
Service
   ↓
Pods
```

---

## 3. What is an Ingress Controller?

An Ingress Controller is the component that implements Ingress routing rules.

Examples:

```text
NGINX
Traefik
HAProxy
Kong
Cloud-provider controllers
```

---

## 4. What happens if you create an Ingress without an Ingress Controller?

The Ingress object may exist, but there is no controller to implement its routing rules.

Therefore, traffic will not automatically be routed just because the Ingress resource exists.

---

## 5. What is IngressClass?

IngressClass identifies the Ingress Controller responsible for an Ingress.

Example:

```yaml
spec:
  ingressClassName: nginx
```

---

## 6. What is host-based routing?

Host-based routing sends traffic to different Services depending on the hostname.

Example:

```text
app.example.com → Frontend Service

api.example.com → Backend Service
```

---

## 7. What is path-based routing?

Path-based routing sends traffic based on the URL path.

Example:

```text
example.com/       → Frontend

example.com/api    → Backend

example.com/admin  → Admin
```

---

## 8. What is the difference between Exact and Prefix path types?

```text
Exact
 ↓
Exact URL path

Prefix
 ↓
Path and matching subpaths
```

Example:

```yaml
path: /api
pathType: Prefix
```

Can match:

```text
/api
/api/users
/api/products
```

---

## 9. How does Ingress support HTTPS?

Ingress can use TLS configuration with a Kubernetes Secret containing the certificate and private key.

Example:

```yaml
tls:
  - hosts:
      - example.com
    secretName: example-tls
```

---

## 10. What is TLS termination?

TLS termination means the Ingress Controller handles the external HTTPS connection and decrypts the traffic before forwarding it to the backend according to its configuration.

```text
Client
  │
  │ HTTPS
  ▼
Ingress Controller
  │
  │ TLS Termination
  ▼
Service
  │
  ▼
Pod
```

---

## 11. Can one Ingress route to multiple Services?

Yes.

Example:

```text
Ingress
  │
  ├── /      → Frontend Service
  ├── /api   → Backend Service
  └── /admin → Admin Service
```

---

## 12. What is the difference between Ingress and Service?

```text
Service
   ↓
Provides stable networking to Pods

Ingress
   ↓
Provides HTTP/HTTPS routing to Services
```

---

## 13. What is the difference between Ingress and LoadBalancer?

A LoadBalancer Service exposes a Service externally.

Ingress provides HTTP/HTTPS routing and can route requests to multiple Services.

```text
LoadBalancer
     ↓
Service

Ingress
     ↓
Multiple Services
```

---

## 14. How do you troubleshoot an Ingress?

Use:

```bash
kubectl get ingress
kubectl describe ingress <ingress>
kubectl get ingressclass
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <controller-pod>
kubectl get svc
kubectl get endpoints
kubectl get endpointslice
```

Then verify:

```text
IngressClass
     ↓
Ingress Controller
     ↓
Host
     ↓
Path
     ↓
Service
     ↓
Service Port
     ↓
Endpoints
     ↓
Pods
```

---

# Production Insight

Ingress is commonly used as the HTTP/HTTPS entry point for Kubernetes applications.

A typical architecture is:

```text
                           Internet
                              │
                              ▼
                    External Load Balancer
                              │
                              ▼
                    Ingress Controller
                              │
                              ▼
                       Ingress Rules
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
        Frontend           Backend           Admin
         Service            Service           Service
             │                │                │
             ▼                ▼                ▼
           Pods              Pods              Pods
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
Routes HTTP/HTTPS traffic

Ingress Controller
    ↓
Actually implements routing
```

---

# Key Takeaways

* Ingress manages external HTTP/HTTPS access to Kubernetes applications.
* Ingress normally routes traffic to Services.
* An Ingress resource contains routing rules.
* An Ingress Controller implements those rules.
* `IngressClass` identifies the appropriate controller.
* Host-based routing uses domain names.
* Path-based routing uses URL paths.
* `Prefix` matches a path prefix.
* `Exact` matches an exact path.
* `ImplementationSpecific` depends on the controller.
* Ingress can route traffic to multiple Services.
* Ingress supports TLS configuration.
* TLS certificates are commonly stored in Kubernetes Secrets.
* Ingress can provide centralized external routing.
* A Service provides stable access to Pods.
* Ingress provides HTTP/HTTPS routing to Services.
* Creating an Ingress resource alone does not provide routing without an appropriate controller.
* `kubectl describe ingress` is important for troubleshooting.
* Ingress Controller logs are useful when debugging routing problems.
* DNS and hostname configuration are important for host-based routing.
* Ingress is commonly placed in front of Services in production architectures.

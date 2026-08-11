# ☸️ Kubernetes Ingress - Interview Questions

## 1. What is Ingress in Kubernetes?

An **Ingress** is a Kubernetes API object that manages external HTTP and HTTPS access to applications running inside a Kubernetes cluster.

Ingress can provide:

* Host-based routing
* Path-based routing
* TLS/HTTPS
* Centralized external access
* Routing traffic to Services

Architecture:

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

---

## 2. Why do we need Ingress?

Suppose a cluster contains multiple applications:

```text
Frontend
Backend
Admin Panel
```

Without Ingress, each application could require separate external exposure.

```text
frontend.example.com → LoadBalancer
api.example.com      → LoadBalancer
admin.example.com    → LoadBalancer
```

Ingress can provide a centralized entry point:

```text
                    Internet
                       │
                       ▼
                    Ingress
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Frontend       Backend       Admin
       Service        Service      Service
```

---

## 3. Does Ingress directly route traffic to Pods?

Normally, no.

Ingress routes traffic to **Services**, and Services route traffic to Pods.

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

This separates responsibilities:

```text
Ingress  → HTTP/HTTPS routing

Service  → Stable networking

Pod      → Runs application
```

---

## 4. What is an Ingress Controller?

An **Ingress Controller** is the component that actually implements the routing rules defined by an Ingress resource.

Examples include:

```text
NGINX
Traefik
HAProxy
Kong
Cloud-provider controllers
```

Architecture:

```text
Ingress Resource
       │
       ▼
Ingress Controller
       │
       ▼
Service
       │
       ▼
Pods
```

---

## 5. What is the difference between Ingress and Ingress Controller?

### Ingress

Ingress is a Kubernetes API object that defines routing rules.

```yaml
kind: Ingress
```

It specifies things such as:

```text
Host
Path
Backend Service
TLS
```

### Ingress Controller

The Ingress Controller watches the Ingress resources and implements the routing behavior.

```text
Ingress
   ↓
Defines rules

Ingress Controller
   ↓
Implements rules
```

---

## 6. What happens if you create an Ingress without an Ingress Controller?

The Ingress resource can exist in Kubernetes, but there is no controller to implement its routing rules.

Therefore:

```text
Ingress Resource
      │
      ▼
No Controller
      │
      ▼
No actual routing
```

An appropriate Ingress Controller must be running.

---

## 7. What API version is used for modern Ingress resources?

Modern Kubernetes Ingress uses:

```yaml
apiVersion: networking.k8s.io/v1
```

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
```

---

## 8. What is IngressClass?

`IngressClass` identifies which Ingress Controller should process an Ingress.

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

---

## 9. What is the purpose of ingressClassName?

`ingressClassName` tells Kubernetes which Ingress Controller is responsible for the Ingress.

Example:

```yaml
spec:
  ingressClassName: nginx
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

## 10. What is host-based routing?

Host-based routing routes traffic based on the hostname.

Example:

```text
app.example.com
api.example.com
admin.example.com
```

Ingress:

```yaml
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
      ↓
Frontend Service

api.example.com
      ↓
Backend Service
```

---

## 11. What is path-based routing?

Path-based routing routes requests based on the URL path.

Example:

```text
example.com/
example.com/api
example.com/admin
```

Ingress:

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

        - path: /api
          pathType: Prefix
          backend:
            service:
              name: backend-service
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
```

---

## 12. What is the difference between host-based and path-based routing?

| Host-Based             | Path-Based           |
| ---------------------- | -------------------- |
| Uses hostname          | Uses URL path        |
| `api.example.com`      | `example.com/api`    |
| Routes based on domain | Routes based on path |

Example:

```text
Host-based:

api.example.com → Backend
```

```text
Path-based:

example.com/api → Backend
```

---

## 13. What are Ingress path types?

Ingress supports:

```text
Prefix
Exact
ImplementationSpecific
```

---

## 14. What is Prefix path type?

`Prefix` matches URL paths based on a path prefix.

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

Architecture:

```text
/api
 ├── /api/users
 ├── /api/products
 └── /api/orders
```

---

## 15. What is Exact path type?

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

## 16. What is ImplementationSpecific?

`ImplementationSpecific` allows the Ingress Controller to determine how the path should be interpreted.

Example:

```yaml
path: /api
pathType: ImplementationSpecific
```

The exact behavior depends on the Ingress Controller.

For predictable routing, `Prefix` or `Exact` is generally preferable when appropriate.

---

## 17. What is the difference between Prefix and Exact?

| Prefix                        | Exact                         |
| ----------------------------- | ----------------------------- |
| Matches a path prefix         | Matches exact path            |
| `/api` can match `/api/users` | `/api` matches only `/api`    |
| Useful for API routes         | Useful for specific endpoints |

Example:

```text
Prefix /api

/api
/api/users
/api/products
```

Example:

```text
Exact /login

/login
```

---

## 18. Can one Ingress route traffic to multiple Services?

Yes.

Example:

```text
Ingress
   │
   ├── /        → Frontend Service
   ├── /api     → Backend Service
   └── /admin   → Admin Service
```

This allows multiple applications to share one external entry point.

---

## 19. What is a default backend?

A default backend handles requests that do not match a configured Ingress rule.

Example:

```yaml
spec:
  defaultBackend:
    service:
      name: default-service
      port:
        number: 80
```

Traffic:

```text
Request
   │
   ├── Rule matches
   │       ↓
   │   Matching Service
   │
   └── No match
           ↓
     Default Backend
```

---

## 20. How does Ingress support HTTPS?

Ingress can configure TLS using a Kubernetes Secret containing a certificate and private key.

Example:

```yaml
spec:
  tls:
    - hosts:
        - example.com
      secretName: example-tls
```

Architecture:

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

## 21. What is TLS termination?

TLS termination means the Ingress Controller handles the external HTTPS connection and decrypts the traffic before forwarding it to the backend according to its configuration.

```text
HTTPS
Client
  │
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

## 22. Where is the TLS certificate stored?

The TLS certificate and private key are commonly stored in a Kubernetes Secret.

Example:

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

## 23. What is a TLS Secret?

A TLS Secret stores TLS certificate data used by components such as an Ingress Controller.

Example:

```text
Secret
  │
  ├── tls.crt
  └── tls.key
```

Ingress references it using:

```yaml
tls:
  - hosts:
      - example.com
    secretName: example-tls
```

---

## 24. Can Ingress support multiple hosts?

Yes.

Example:

```yaml
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

Architecture:

```text
                    Ingress
                       │
          ┌────────────┴────────────┐
          │                         │
 app.example.com              api.example.com
          │                         │
          ▼                         ▼
Frontend Service              Backend Service
```

---

## 25. Can Ingress support multiple paths?

Yes.

Example:

```yaml
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

---

## 26. What is the relationship between Ingress and Service?

Ingress provides external HTTP/HTTPS routing.

Service provides stable networking to Pods.

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

---

## 27. What is the relationship between Ingress and Deployment?

A Deployment manages Pods.

A Service provides stable networking.

Ingress provides external HTTP/HTTPS routing.

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
Deployment
   │
   ▼
Pods
```

More precisely:

```text
Ingress
   ↓
Service
   ↓
Pods
   ↑
Deployment manages Pods
```

---

## 28. What is the difference between Ingress and Service?

```text
Service
   ↓
Stable networking and service discovery

Ingress
   ↓
HTTP/HTTPS routing
```

| Service                                  | Ingress                           |
| ---------------------------------------- | --------------------------------- |
| Provides stable access to Pods           | Provides HTTP/HTTPS routing       |
| Works with Pod networking                | Works at HTTP/HTTPS routing layer |
| Can be ClusterIP, NodePort, LoadBalancer | Uses Ingress Controller           |
| Can expose one application Service       | Can route to multiple Services    |

---

## 29. What is the difference between Ingress and LoadBalancer?

A `LoadBalancer` Service exposes a Service externally.

Ingress provides HTTP/HTTPS routing and can route traffic to multiple Services.

```text
LoadBalancer:

Internet
   ↓
LoadBalancer
   ↓
Service
   ↓
Pods
```

```text
Ingress:

Internet
   ↓
Ingress
   ↓
Multiple Services
   ↓
Pods
```

---

## 30. What is the difference between Ingress and NodePort?

`NodePort` exposes a Service through a port on each Node.

Ingress provides HTTP/HTTPS routing using hosts and paths.

```text
NodePort:

NodeIP:30080
      ↓
Service
      ↓
Pods
```

```text
Ingress:

example.com/api
      ↓
Ingress
      ↓
Backend Service
      ↓
Pods
```

---

## 31. Does Ingress replace a Service?

No.

Ingress normally routes traffic to Services.

```text
Ingress
   ↓
Service
   ↓
Pods
```

They have different responsibilities.

---

## 32. What happens if the backend Service has no endpoints?

Ingress may be configured correctly, but it cannot successfully route traffic to an unavailable backend.

Check:

```bash
kubectl get endpoints <service>
```

And:

```bash
kubectl get endpointslice
```

If no endpoints exist, check:

```text
Service selector
       ↓
Pod labels
       ↓
Pod readiness
```

---

## 33. How do you troubleshoot an Ingress?

Use the following workflow:

```text
kubectl get ingress
        ↓
kubectl describe ingress
        ↓
Check IngressClass
        ↓
Check Ingress Controller
        ↓
Check Host
        ↓
Check Path
        ↓
Check Service
        ↓
Check Service Port
        ↓
Check Endpoints
        ↓
Check Pods
        ↓
Check DNS
        ↓
Check TLS
```

Important commands:

```bash
kubectl get ingress
kubectl describe ingress <ingress>
kubectl get ingressclass
kubectl get svc
kubectl get endpoints
kubectl get endpointslice
kubectl get pods
```

---

## 34. How do you check whether the Ingress Controller is running?

For the NGINX Ingress Controller on Minikube:

```bash
kubectl get pods -n ingress-nginx
```

Example:

```text
NAME                                        READY   STATUS
ingress-nginx-controller-xxxxx              1/1     Running
```

If the Controller is not running, investigate:

```bash
kubectl describe pod <pod> -n ingress-nginx
```

And:

```bash
kubectl logs <pod> -n ingress-nginx
```

---

## 35. How do you check Ingress Controller logs?

First find the Controller Pod:

```bash
kubectl get pods -n ingress-nginx
```

Then:

```bash
kubectl logs \
  -n ingress-nginx \
  <controller-pod>
```

Follow logs:

```bash
kubectl logs \
  -n ingress-nginx \
  -f <controller-pod>
```

Logs can help identify:

* Routing errors
* Backend connection problems
* Configuration problems
* TLS problems

---

## 36. What causes an Ingress to have no address?

Example:

```text
ADDRESS
<none>
```

Possible causes:

* Ingress Controller is not running
* Wrong IngressClass
* Controller is not watching the Ingress
* Load balancer is unavailable
* Local cluster configuration issue

Check:

```bash
kubectl get ingress
kubectl describe ingress <ingress>
kubectl get ingressclass
kubectl get pods -n ingress-nginx
```

---

## 37. What happens if the IngressClass is incorrect?

The intended Ingress Controller may not process the Ingress.

Example:

```yaml
spec:
  ingressClassName: nginx
```

Check available classes:

```bash
kubectl get ingressclass
```

If the configured class does not correspond to the appropriate controller, routing may not work as expected.

---

## 38. How do you troubleshoot a 404 error from Ingress?

Possible causes:

* Wrong hostname
* Wrong path
* Incorrect path type
* No matching Ingress rule
* Controller-specific routing behavior
* Incorrect backend configuration

Check:

```bash
kubectl describe ingress <ingress>
```

Verify:

```text
Host
Path
PathType
Service
Port
```

---

## 39. How do you troubleshoot 502 or 503 errors?

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

Also inspect Controller logs:

```bash
kubectl logs \
  -n ingress-nginx \
  <controller-pod>
```

---

## 40. How do you troubleshoot an Ingress with no backend endpoints?

Check the Service:

```bash
kubectl get svc
```

Check endpoints:

```bash
kubectl get endpoints <service>
```

Check EndpointSlices:

```bash
kubectl get endpointslice
```

Check Pod labels:

```bash
kubectl get pods --show-labels
```

Verify:

```text
Service selector
       ↓
Pod labels
       ↓
Endpoints
```

If the selector and labels do not match, the Service will not select the expected Pods.

---

## 41. How do you test an Ingress using curl?

You can send a specific Host header:

```bash
curl -H "Host: nginx.local" \
  http://$(minikube ip)
```

For HTTPS:

```bash
curl -k \
  -H "Host: secure.local" \
  https://$(minikube ip)
```

This is useful for testing host-based routing without configuring DNS first.

---

## 42. How do you test Ingress DNS?

Use:

```bash
nslookup example.com
```

Or:

```bash
dig example.com
```

For local development, you may configure:

```text
/etc/hosts
```

Linux/WSL or:

```text
C:\Windows\System32\drivers\etc\hosts
```

on Windows.

Example:

```text
192.168.49.2 example.local
```

---

## 43. How do you enable Ingress on Minikube?

Check addons:

```bash
minikube addons list
```

Enable:

```bash
minikube addons enable ingress
```

Check Controller:

```bash
kubectl get pods -n ingress-nginx
```

---

## 44. How do you check all Ingress resources?

Use:

```bash
kubectl get ingress
```

For all namespaces:

```bash
kubectl get ingress -A
```

---

## 45. How do you get the complete Ingress YAML?

Use:

```bash
kubectl get ingress <ingress> -o yaml
```

This is useful for inspecting:

* Rules
* Hosts
* Paths
* Backend Services
* TLS
* IngressClass

---

## 46. How do you describe an Ingress?

Use:

```bash
kubectl describe ingress <ingress>
```

This is one of the most useful commands for troubleshooting.

It shows information such as:

```text
Rules
Hosts
Paths
Backends
TLS
Events
```

---

## 47. What are Ingress annotations?

Annotations provide additional configuration for some Ingress Controllers.

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```

Important:

```text
Annotations are controller-specific.
```

An annotation supported by NGINX may not be supported by another Ingress Controller.

---

## 48. What is path rewriting?

Path rewriting changes the path sent to the backend.

Some Ingress Controllers support rewrite behavior through annotations or other configuration.

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```

The exact behavior depends on the Ingress Controller.

---

## 49. What is TLS termination in a production architecture?

A common architecture is:

```text
                    HTTPS
Internet ─────────────────────► Ingress
                                │
                                │ TLS Termination
                                ▼
                              Service
                                │
                                ▼
                               Pods
```

The Ingress Controller handles the external TLS connection.

---

## 50. What are common Ingress use cases?

Ingress is commonly used for:

* Host-based routing
* Path-based routing
* HTTPS/TLS
* Centralized external access
* Routing multiple applications
* HTTP/HTTPS entry point for Kubernetes applications

Example:

```text
example.com/
      ↓
Frontend

example.com/api
      ↓
Backend

example.com/admin
      ↓
Admin
```

---

# 🔥 Scenario-Based Interview Questions

## 51. Your Ingress exists, but traffic is not reaching the application. What do you check?

Follow this order:

```text
1. Ingress Controller
        ↓
2. IngressClass
        ↓
3. Ingress rules
        ↓
4. Host
        ↓
5. Path
        ↓
6. Service
        ↓
7. Service port
        ↓
8. Endpoints
        ↓
9. Pods
        ↓
10. DNS
```

Commands:

```bash
kubectl get ingress
kubectl describe ingress <ingress>
kubectl get ingressclass
kubectl get pods -n ingress-nginx
kubectl get svc
kubectl get endpoints
kubectl get endpointslice
kubectl get pods --show-labels
```

---

## 52. Your Ingress returns 404. What could be wrong?

Check:

```text
Host
Path
PathType
Ingress rules
Controller configuration
```

For example:

```yaml
host: app.example.com
```

But request:

```text
api.example.com
```

The rule may not match.

---

## 53. Your Ingress returns 503. What would you check?

Check backend availability:

```bash
kubectl get svc
kubectl get endpoints
kubectl get endpointslice
kubectl get pods
```

Then verify:

```text
Ingress
   ↓
Correct Service?
   ↓
Correct Service Port?
   ↓
Endpoints available?
   ↓
Pods Ready?
```

Also inspect Ingress Controller logs.

---

## 54. Your Ingress has no address. What would you check?

Check:

```bash
kubectl get ingress
kubectl describe ingress <ingress>
kubectl get ingressclass
kubectl get pods -n ingress-nginx
```

Possible causes:

```text
Ingress Controller unavailable
Wrong IngressClass
No external address
Local cluster configuration
```

---

## 55. Your Service works directly, but Ingress does not. What would you check?

If the Service works:

```text
Service ✓
Pods ✓
Endpoints ✓
```

Then focus on:

```text
IngressClass
Ingress Controller
Host
Path
Ingress backend Service
Service port
DNS
TLS
```

Useful commands:

```bash
kubectl describe ingress <ingress>
kubectl get ingressclass
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <controller-pod>
```

---

## 56. Two applications use the same Ingress. How can you route traffic to them?

Use host-based or path-based routing.

### Host-based

```text
app.example.com → Frontend
api.example.com → Backend
```

### Path-based

```text
example.com/    → Frontend
example.com/api → Backend
```

---

## 57. How would you expose frontend and backend using one domain?

Use path-based routing:

```text
example.com/
      ↓
Frontend Service

example.com/api
      ↓
Backend Service
```

Example:

```yaml
rules:
  - host: example.com

    http:
      paths:

        - path: /api
          pathType: Prefix
          backend:
            service:
              name: backend-service
              port:
                number: 80

        - path: /
          pathType: Prefix
          backend:
            service:
              name: frontend-service
              port:
                number: 80
```

---

## 58. How would you expose frontend and backend using different domains?

Use host-based routing:

```text
app.example.com → Frontend

api.example.com → Backend
```

Example:

```yaml
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

---

## 59. What is a typical production Kubernetes web architecture?

A common architecture is:

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
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
          Frontend        Backend        Admin
           Service         Service       Service
             │              │              │
             ▼              ▼              ▼
           Pods            Pods            Pods
             ▲              ▲              ▲
             └──────────────┼──────────────┘
                            │
                       Deployments
```

Responsibilities:

```text
Deployment
    ↓
Manages Pods

Service
    ↓
Stable networking

Ingress
    ↓
HTTP/HTTPS routing

Ingress Controller
    ↓
Implements routing
```

---

# 🎯 Quick Interview Revision

## Ingress

```text
Ingress
   ↓
HTTP/HTTPS routing
   ↓
Services
   ↓
Pods
```

## Ingress Controller

```text
Ingress Resource
       ↓
Ingress Controller
       ↓
Actually implements routing
```

## Routing

```text
Host-Based

api.example.com
       ↓
Backend
```

```text
Path-Based

example.com/api
       ↓
Backend
```

## Path Types

```text
Exact
   ↓
Exact path

Prefix
   ↓
Path + subpaths

ImplementationSpecific
   ↓
Controller-specific
```

## TLS

```text
HTTPS
  ↓
Ingress Controller
  ↓
TLS Termination
  ↓
Service
  ↓
Pods
```

## Troubleshooting

```text
Ingress
   ↓
IngressClass
   ↓
Controller
   ↓
Host
   ↓
Path
   ↓
Service
   ↓
Port
   ↓
Endpoints
   ↓
Pods
   ↓
DNS / TLS
```

# 🔥 Must Remember

```text
Ingress = HTTP/HTTPS routing

Ingress Controller = Implements Ingress rules

IngressClass = Identifies the Controller

Service = Stable networking to Pods

Host-based = Route using hostname

Path-based = Route using URL path

Prefix = Matches path prefix

Exact = Matches exact path

TLS = HTTPS configuration

TLS Secret = Certificate + Private Key

Ingress does NOT normally route directly to Pods

Ingress → Service → Pods
```

# 🎯 One-Line Interview Answer

> **Kubernetes Ingress is an API object that defines HTTP/HTTPS routing rules for exposing applications through Services, while an Ingress Controller implements those rules and handles the actual traffic routing.**

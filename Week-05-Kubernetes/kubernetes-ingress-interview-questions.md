# Kubernetes Ingress - Interview Questions & Answers

> Interview preparation notes for **Kubernetes Ingress, Host-Based Routing, Multiple Services, Annotations, TLS/HTTPS, and Troubleshooting**.

---

# 📌 Basic Ingress Questions

## 1. What is Kubernetes Ingress?

**Answer:**

Kubernetes Ingress is an API object that manages external **HTTP and HTTPS access** to services inside a Kubernetes cluster.

It provides features such as:

* Host-based routing
* Path-based routing
* TLS/HTTPS
* HTTP → HTTPS redirects
* Load balancing
* Traffic routing to multiple Services

Example:

```text
Internet
   |
   v
Ingress
   |
   +----> frontend-service
   |
   +----> backend-service
```

---

## 2. Is Ingress a Service?

**Answer:**

No.

Ingress and Service are different Kubernetes resources.

A **Service** provides stable network access to Pods.

An **Ingress** provides HTTP/HTTPS routing from outside the cluster to Services.

```text
Internet
   |
   v
Ingress
   |
   v
Service
   |
   v
Pods
```

---

## 3. What is an Ingress Controller?

**Answer:**

An Ingress Controller is the component that actually implements the routing rules defined by an Ingress resource.

Examples:

* NGINX Ingress Controller
* Traefik
* HAProxy
* Kong
* AWS Load Balancer Controller

Without an appropriate Ingress Controller, creating an Ingress resource alone does not provide the expected routing behavior.

---

## 4. What is the difference between Ingress and Ingress Controller?

**Answer:**

| Ingress                 | Ingress Controller      |
| ----------------------- | ----------------------- |
| Kubernetes API object   | Actual implementation   |
| Defines routing rules   | Processes routing rules |
| YAML configuration      | Software/controller     |
| Describes desired state | Makes routing happen    |

Example:

```text
Ingress YAML
     |
     v
Ingress Controller
     |
     v
Service
```

---

## 5. What is `ingressClassName`?

**Answer:**

`ingressClassName` specifies which Ingress Controller should handle an Ingress resource.

Example:

```yaml
spec:
  ingressClassName: nginx
```

This tells Kubernetes that the Ingress should be handled by the controller associated with the `nginx` IngressClass.

---

# 🌐 Host-Based Routing

## 6. What is host-based routing?

**Answer:**

Host-based routing routes requests to different Services based on the **hostname** in the HTTP request.

Example:

```text
app.example.com
      |
      v
frontend-service

api.example.com
      |
      v
backend-service
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

## 7. Why would you use host-based routing?

**Answer:**

It allows multiple applications to share the same Ingress endpoint while using different domain names.

For example:

```text
app.example.com
api.example.com
admin.example.com
```

can all point to the same Ingress Controller while routing to different Services.

---

## 8. Can multiple domains use the same Ingress?

**Answer:**

Yes.

One Ingress resource can contain multiple host rules.

Example:

```yaml
rules:
  - host: app.example.com
  - host: api.example.com
  - host: admin.example.com
```

Each host can route to a different Service.

---

# 🔀 Multiple Services

## 9. Can one Ingress route traffic to multiple Services?

**Answer:**

Yes.

For example:

```text
                    Ingress
                       |
          +------------+------------+
          |            |            |
          v            v            v
      frontend      backend       admin
       Service       Service       Service
```

Routing can be based on:

* Host
* Path
* Host + Path

---

## 10. What is path-based routing?

**Answer:**

Path-based routing routes traffic based on the URL path.

Example:

```text
example.com/
      |
      v
frontend-service

example.com/api
      |
      v
backend-service
```

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
          number: 8080
```

---

## 11. What is the difference between host-based and path-based routing?

**Answer:**

### Host-based

Routes based on the domain:

```text
app.example.com
api.example.com
```

### Path-based

Routes based on the URL path:

```text
example.com/app
example.com/api
```

---

# 🏷️ Ingress Annotations

## 12. What are Ingress annotations?

**Answer:**

Annotations are key-value pairs added to an Ingress resource to customize the behavior of an Ingress Controller.

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```

Annotations are often **controller-specific**.

---

## 13. Are all Ingress annotations standard Kubernetes features?

**Answer:**

No.

Many annotations are specific to a particular Ingress Controller.

For example:

```text
nginx.ingress.kubernetes.io/*
```

are NGINX Ingress Controller annotations.

Therefore, the supported annotations depend on the controller being used.

---

## 14. What does `rewrite-target` do?

**Answer:**

It changes the request path before forwarding the request to the backend.

Example:

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /
```

Suppose the client requests:

```text
/api/users
```

The Ingress can rewrite the request before sending it to the backend.

This is useful when the external URL structure differs from the backend application's expected URL structure.

---

## 15. What does `use-regex` do?

**Answer:**

It enables regular-expression path matching in NGINX Ingress.

Example:

```yaml
nginx.ingress.kubernetes.io/use-regex: "true"
```

A regex path can be used with `ImplementationSpecific`.

---

## 16. What does `ssl-redirect` do?

**Answer:**

It redirects HTTP requests to HTTPS when TLS is configured.

Example:

```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

Flow:

```text
http://example.com
       |
       v
   Redirect
       |
       v
https://example.com
```

---

## 17. What is the difference between `ssl-redirect` and `force-ssl-redirect`?

**Answer:**

`ssl-redirect` is used to redirect HTTP to HTTPS when TLS is configured for the Ingress.

```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

`force-ssl-redirect` forces HTTPS redirection even when TLS is not configured directly in the Ingress, which can be useful when TLS is terminated by an external load balancer.

```yaml
nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

---

## 18. What does `backend-protocol` do?

**Answer:**

It tells NGINX which protocol to use when communicating with the backend.

Example:

```yaml
nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
```

Traffic becomes:

```text
Client
  |
 HTTPS
  v
Ingress
  |
 HTTPS
  v
Backend
```

---

## 19. What does `proxy-body-size` do?

**Answer:**

It controls the maximum request body size accepted by NGINX.

Example:

```yaml
nginx.ingress.kubernetes.io/proxy-body-size: "50m"
```

This is useful for applications that accept large:

* File uploads
* Images
* Videos
* Documents

---

## 20. What does `proxy-read-timeout` do?

**Answer:**

It specifies how long NGINX waits for a response from the backend.

Example:

```yaml
nginx.ingress.kubernetes.io/proxy-read-timeout: "120"
```

It can be useful for:

* Long-running APIs
* Streaming
* Large downloads
* Slow backend operations

---

## 21. How can you enable CORS using NGINX Ingress?

**Answer:**

Use:

```yaml
nginx.ingress.kubernetes.io/enable-cors: "true"
```

You can also configure allowed origins:

```yaml
nginx.ingress.kubernetes.io/cors-allow-origin: "https://example.com"
```

---

# 🔐 TLS / HTTPS

## 22. What is TLS?

**Answer:**

TLS stands for **Transport Layer Security**.

It provides:

* Encryption
* Data integrity
* Authentication

HTTPS is essentially HTTP running over TLS.

---

## 23. How does HTTPS work with Kubernetes Ingress?

**Answer:**

The client sends an HTTPS request to the Ingress Controller.

The Ingress Controller uses a TLS certificate stored in a Kubernetes Secret.

Typical flow:

```text
Client
  |
 HTTPS :443
  |
  v
Ingress Controller
  |
 TLS Termination
  |
  v
Service
  |
  v
Pod
```

---

## 24. Where is the TLS certificate stored?

**Answer:**

The certificate and private key are normally stored in a Kubernetes Secret of type:

```yaml
type: kubernetes.io/tls
```

It contains:

```text
tls.crt
tls.key
```

---

## 25. How do you create a TLS Secret?

**Answer:**

Use:

```bash
kubectl create secret tls app-tls \
  --cert=tls.crt \
  --key=tls.key
```

Then reference it from the Ingress:

```yaml
tls:
  - hosts:
      - app.example.com
    secretName: app-tls
```

---

## 26. What is TLS termination?

**Answer:**

TLS termination means the Ingress Controller decrypts HTTPS traffic.

```text
Client
  |
 HTTPS
  v
Ingress
  |
 HTTP
  v
Service
```

The connection from Ingress to the backend is HTTP unless separately configured for HTTPS.

---

## 27. What is TLS passthrough?

**Answer:**

TLS passthrough means the Ingress Controller forwards encrypted TLS traffic to the backend without terminating TLS.

```text
Client
  |
 HTTPS
  v
Ingress
  |
 HTTPS
  v
Service
  |
 HTTPS
  v
Pod
```

NGINX Ingress can use:

```yaml
nginx.ingress.kubernetes.io/ssl-passthrough: "true"
```

---

## 28. TLS termination vs TLS passthrough?

**Answer:**

| TLS Termination               | TLS Passthrough               |
| ----------------------------- | ----------------------------- |
| Ingress decrypts TLS          | Backend decrypts TLS          |
| Backend usually receives HTTP | Backend receives HTTPS        |
| Easier to manage              | More complex                  |
| HTTP-level routing available  | Limited HTTP-level processing |
| Certificate at Ingress        | Certificate at backend        |

---

## 29. Which port is normally used for HTTPS?

**Answer:**

HTTPS normally uses:

```text
443
```

HTTP normally uses:

```text
80
```

---

## 30. Why should HTTPS be used in production?

**Answer:**

HTTPS protects data while it travels across the network.

Without HTTPS:

```text
Client -------- Plaintext --------> Server
```

With HTTPS:

```text
Client ===== Encrypted TLS =====> Server
```

This protects sensitive information such as:

* Passwords
* Tokens
* Cookies
* Personal information
* API requests

---

# 🧪 Troubleshooting Questions

## 31. How do you check whether an Ingress exists?

**Answer:**

```bash
kubectl get ingress
```

or:

```bash
kubectl get ing
```

---

## 32. How do you troubleshoot an Ingress?

**Answer:**

I would troubleshoot it layer by layer:

```bash
kubectl get ingress
kubectl describe ingress <name>

kubectl get svc
kubectl get endpoints
kubectl get pods

kubectl get pods -n ingress-nginx

kubectl logs -n ingress-nginx \
  deployment/ingress-nginx-controller
```

I would verify:

1. Ingress exists.
2. Correct IngressClass is configured.
3. Host/path rules are correct.
4. Service exists.
5. Service port is correct.
6. Service has healthy endpoints.
7. Pods are running.
8. TLS Secret exists.
9. DNS/hosts configuration is correct.
10. Ingress Controller logs show no errors.

---

## 33. What would you check if Ingress returns 404?

**Answer:**

I would check:

```bash
kubectl describe ingress <name>
```

Then verify:

* Hostname
* Path
* `pathType`
* IngressClass
* Backend Service
* Service port

For host-based routing, I would also verify that the request contains the expected `Host` header.

Example:

```bash
curl -H "Host: app.example.com" http://<INGRESS-IP>
```

---

## 34. What does a 502 Bad Gateway usually indicate?

**Answer:**

A `502 Bad Gateway` often means the Ingress Controller cannot successfully communicate with the backend.

I would check:

```bash
kubectl get svc
kubectl get endpoints
kubectl get pods
```

If the Service has no endpoints, I would check the Service selector and Pod labels.

---

## 35. What would you check if the Service has no endpoints?

**Answer:**

I would compare:

```yaml
Service:
  selector:
    app: nginx
```

with the Pod labels:

```yaml
Pod:
  labels:
    app: nginx
```

The labels must match the Service selector.

Then:

```bash
kubectl get endpoints <service-name>
```

---

## 36. What would you check if HTTPS is not working?

**Answer:**

I would check:

```bash
kubectl get ingress
kubectl describe ingress <name>
kubectl get secret
```

Then verify:

* TLS Secret exists.
* Secret is in the same namespace as the Ingress.
* Secret type is `kubernetes.io/tls`.
* `tls.crt` and `tls.key` exist.
* Hostname matches the certificate.
* Ingress Controller is running.
* Port 443 is reachable.

---

## 37. What happens if the TLS Secret doesn't exist?

**Answer:**

The Ingress Controller cannot load the configured certificate for that Ingress.

I would check:

```bash
kubectl get secret app-tls
```

and:

```bash
kubectl describe ingress <name>
```

Then create or correct the TLS Secret.

---

## 38. Why might a certificate show a hostname mismatch?

**Answer:**

The hostname requested by the client may not match the hostname contained in the certificate's SANs.

For example:

```text
Requested:
api.example.com

Certificate:
app.example.com
```

This causes a certificate validation error.

The certificate should include the hostname being accessed.

---

# 🛠️ Scenario-Based Questions

## 39. You have `app.example.com` and `api.example.com`. How would you route them?

**Answer:**

I would use host-based routing.

```text
app.example.com
       |
       v
frontend-service

api.example.com
       |
       v
backend-service
```

The Ingress would contain two host rules.

---

## 40. Your frontend is available at `/` and backend at `/api`. How would you configure it?

**Answer:**

I would use path-based routing:

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
          number: 8080
```

---

## 41. Users should never access the application through HTTP. What would you configure?

**Answer:**

I would configure TLS and enable HTTP-to-HTTPS redirection:

```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

Then:

```text
HTTP :80
   |
   v
Redirect
   |
   v
HTTPS :443
```

---

## 42. Your backend itself requires HTTPS. What would you configure?

**Answer:**

I would configure:

```yaml
nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
```

Then the traffic flow becomes:

```text
Client
  |
 HTTPS
  v
Ingress
  |
 HTTPS
  v
Backend
```

---

## 43. Your application accepts 100 MB file uploads, but uploads fail. What would you check?

**Answer:**

I would check the NGINX request body limit.

For example:

```yaml
nginx.ingress.kubernetes.io/proxy-body-size: "100m"
```

I would also verify that the application and any other proxy/load balancer in front of it allow the same or larger request size.

---

## 44. Your API takes 2 minutes to respond and Ingress returns a timeout. What would you check?

**Answer:**

I would check the NGINX proxy timeout.

For example:

```yaml
nginx.ingress.kubernetes.io/proxy-read-timeout: "180"
```

I would also check whether there are other timeout limits in:

* Load balancer
* Application
* Service mesh
* Network infrastructure

---

## 45. Your Ingress works with the IP address but not with the domain. What would you check?

**Answer:**

I would check DNS first.

```text
app.example.com
       |
       v
Ingress IP
```

For a local lab, I can temporarily configure `/etc/hosts`:

```text
192.168.49.2 app.example.com
```

Then test:

```bash
curl -H "Host: app.example.com" http://192.168.49.2
```

---

# 🚀 Advanced Questions

## 46. Can Ingress route TCP or UDP traffic?

**Answer:**

Standard Kubernetes Ingress is designed primarily for **HTTP and HTTPS** traffic.

It is not a general-purpose TCP/UDP load balancer.

For non-HTTP protocols, other Kubernetes networking/load-balancing mechanisms or controller-specific features may be required.

---

## 47. Does Ingress automatically create a LoadBalancer?

**Answer:**

Not necessarily.

Ingress is an API object. Whether an externally reachable load balancer is created depends on the Ingress Controller and its deployment architecture.

For example:

```text
Cloud Load Balancer
       |
       v
Ingress Controller
       |
       v
Services
```

In other environments, the controller may be exposed using `NodePort`, `LoadBalancer`, host networking, or another mechanism.

---

## 48. Can multiple Ingress resources exist in a cluster?

**Answer:**

Yes.

You can have multiple Ingress resources, even in different namespaces.

For example:

```text
Namespace: production
  └── production-ingress

Namespace: staging
  └── staging-ingress
```

They are processed according to the Ingress Controller and IngressClass configuration.

---

## 49. Can an Ingress reference a Service in another namespace?

**Answer:**

Normally, an Ingress backend references a Service in the **same namespace** as the Ingress.

For cross-namespace routing, you generally need a different architecture or controller-specific mechanism rather than simply specifying another namespace in the standard Ingress backend reference.

---

## 50. What is `pathType`?

**Answer:**

`pathType` defines how an Ingress path should be matched.

The main Kubernetes path types are:

```text
Exact
Prefix
ImplementationSpecific
```

Example:

```yaml
path: /api
pathType: Prefix
```

---

## 51. Difference between `Exact` and `Prefix`?

**Answer:**

### Exact

```yaml
path: /api
pathType: Exact
```

Matches:

```text
/api
```

but not generally:

```text
/api/users
```

### Prefix

```yaml
path: /api
pathType: Prefix
```

Matches paths such as:

```text
/api
/api/
/api/users
/api/products
```

---

## 52. What is cert-manager?

**Answer:**

cert-manager is a Kubernetes add-on that automates certificate management.

It can:

* Request certificates
* Store certificates in Secrets
* Renew certificates
* Work with certificate authorities such as Let's Encrypt

Typical flow:

```text
Ingress
   |
   v
cert-manager
   |
   v
Certificate Authority
   |
   v
TLS Certificate
   |
   v
Kubernetes Secret
```

---

# 🎯 Real Interview Scenario

## 53. Explain a complete production Ingress architecture.

**Answer:**

A typical architecture could be:

```text
                         Internet
                            |
                            v
                     DNS / Domain
                            |
                            v
                 Cloud Load Balancer
                            |
                            v
                  NGINX Ingress Controller
                            |
                    TLS Termination
                            |
             +--------------+--------------+
             |                             |
             v                             v
       /frontend                       /api
             |                             |
             v                             v
      frontend-service              backend-service
             |                             |
             v                             v
        Frontend Pods                  Backend Pods
```

I would use:

* Host/path-based routing
* TLS certificates
* HTTPS redirect
* Appropriate annotations
* Health checks
* Service selectors
* Monitoring and logging
* Certificate automation where appropriate

---

# 🔥 Rapid-Fire Interview Revision

| Question                   | Short Answer                                      |
| -------------------------- | ------------------------------------------------- |
| What is Ingress?           | HTTP/HTTPS routing into a Kubernetes cluster      |
| What implements Ingress?   | Ingress Controller                                |
| Popular controller?        | NGINX                                             |
| HTTP port?                 | 80                                                |
| HTTPS port?                | 443                                               |
| Host-based routing?        | Routing based on hostname                         |
| Path-based routing?        | Routing based on URL path                         |
| TLS Secret type?           | `kubernetes.io/tls`                               |
| TLS certificate key?       | `tls.crt`                                         |
| TLS private key?           | `tls.key`                                         |
| HTTPS redirect annotation? | `ssl-redirect`                                    |
| Force HTTPS annotation?    | `force-ssl-redirect`                              |
| Rewrite annotation?        | `rewrite-target`                                  |
| Regex annotation?          | `use-regex`                                       |
| Backend HTTPS annotation?  | `backend-protocol: "HTTPS"`                       |
| Large upload annotation?   | `proxy-body-size`                                 |
| Backend timeout?           | `proxy-read-timeout`                              |
| TLS termination?           | Ingress decrypts TLS                              |
| TLS passthrough?           | Backend decrypts TLS                              |
| Certificate automation?    | cert-manager                                      |
| Ingress debugging?         | `get`, `describe`, logs, Service/endpoints checks |

---

# 🧠 Top 10 Questions to Prepare First

If you're short on interview time, **master these 10**:

1. **What is Kubernetes Ingress?**
2. **What is an Ingress Controller?**
3. **Ingress vs Service?**
4. **Host-based vs path-based routing?**
5. **What are Ingress annotations?**
6. **What does `rewrite-target` do?**
7. **How does TLS/HTTPS work with Ingress?**
8. **TLS termination vs TLS passthrough?**
9. **How do you troubleshoot a 404/502 from Ingress?**
10. **Explain your complete Ingress project architecture.**

### ⭐ Best interview answer pattern

For scenario questions, answer in this order:

```text
1. Identify the problem
        ↓
2. Check Ingress
        ↓
3. Check Service
        ↓
4. Check Endpoints
        ↓
5. Check Pods
        ↓
6. Check Ingress Controller
        ↓
7. Check Logs
        ↓
8. Fix and retest
```

This demonstrates **practical troubleshooting ability**, rather than just memorizing Kubernetes commands.

# Kubernetes Ingress - Annotations, TLS & HTTP/HTTPS

> Complete notes for understanding and practicing **Kubernetes Ingress Annotations, HTTP/HTTPS routing, TLS certificates, HTTPS redirection, and common NGINX Ingress configurations**.

---

# 1. What is Kubernetes Ingress?

Kubernetes **Ingress** is an API object used to manage external HTTP and HTTPS access to services inside a Kubernetes cluster.

Instead of exposing every application using `NodePort` or `LoadBalancer`, Ingress provides a single entry point and routes traffic based on:

* Hostnames
* URL paths
* HTTP/HTTPS
* TLS certificates
* Controller-specific configuration

### Basic architecture

```text
                    Internet
                       |
                       v
              +----------------+
              |    Ingress     |
              |   Controller   |
              +----------------+
                 /          \
                /            \
               v              v
        +-----------+   +-----------+
        | frontend  |   | backend   |
        | Service   |   | Service   |
        +-----------+   +-----------+
             |               |
             v               v
          Frontend Pods    Backend Pods
```

---

# 2. What is an Ingress Controller?

An Ingress resource only defines the desired routing rules.

The **Ingress Controller** actually implements those rules.

Popular controllers include:

* NGINX Ingress Controller
* Traefik
* HAProxy
* Kong
* AWS Load Balancer Controller

For these notes, we mainly use **NGINX Ingress Controller**.

```text
Ingress YAML
     |
     v
Ingress Controller
     |
     v
Reverse Proxy / Load Balancer
     |
     +----> Service A
     |
     +----> Service B
```

---

# 3. Basic Ingress Structure

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: my-ingress

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
                name: web-service
                port:
                  number: 80
```

### Important fields

| Field              | Purpose                        |
| ------------------ | ------------------------------ |
| `apiVersion`       | Kubernetes API version         |
| `kind`             | Resource type                  |
| `metadata.name`    | Ingress name                   |
| `ingressClassName` | Selects the Ingress Controller |
| `rules`            | Routing rules                  |
| `host`             | Domain name                    |
| `path`             | URL path                       |
| `pathType`         | Path matching behavior         |
| `backend`          | Destination Service            |

---

# 4. What are Ingress Annotations?

Ingress annotations are **key-value pairs attached to an Ingress resource** that allow an Ingress Controller to customize its behavior.

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```

Annotations are controller-specific.

For NGINX Ingress, the default annotation prefix is:

```text
nginx.ingress.kubernetes.io/
```

Annotation values are strings, so boolean and numeric values should normally be quoted.

Example:

```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "true"
nginx.ingress.kubernetes.io/proxy-body-size: "50m"
```

---

# 5. Common NGINX Ingress Annotations

## 5.1 Rewrite Target

Used to rewrite the URL path before sending the request to the backend.

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /
```

Example:

```text
Client:
GET /api/users

Ingress:
rewrite /api/users -> /users

Backend:
GET /users
```

---

# 6. Use Regex

Enable regular-expression path matching:

```yaml
nginx.ingress.kubernetes.io/use-regex: "true"
```

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
```

Path:

```yaml
paths:
  - path: /api(/|$)(.*)
    pathType: ImplementationSpecific
```

Request:

```text
/api/users
```

Backend receives:

```text
/users
```

NGINX documents `use-regex` as the annotation that enables regex paths.

---

# 7. SSL Redirect

The annotation:

```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

forces HTTP requests to redirect to HTTPS when TLS is configured.

Example:

```text
http://example.com
        |
        v
     308 Redirect
        |
        v
https://example.com
```

NGINX Ingress normally redirects HTTP clients to HTTPS when TLS is enabled for that Ingress.

---

# 8. Force SSL Redirect

You can force HTTPS even when TLS is not configured directly in the Ingress:

```yaml
nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

This can be useful when TLS termination happens outside the Kubernetes cluster, such as at an external load balancer.

---

# 9. Disable SSL Redirect

If HTTP should remain available:

```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "false"
```

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
```

---

# 10. Backend Protocol

By default, NGINX communicates with the backend using HTTP.

To communicate with a backend over HTTPS:

```yaml
nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
```

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
```

Traffic flow:

```text
Client
  |
 HTTPS
  |
  v
Ingress
  |
 HTTPS
  |
  v
Service
  |
 HTTPS
  |
  v
Pod
```

The NGINX Ingress documentation supports backend protocols such as `HTTP`, `HTTPS`, `GRPC`, and `GRPCS`.

---

# 11. Proxy Body Size

Controls the maximum allowed request body size.

Example:

```yaml
nginx.ingress.kubernetes.io/proxy-body-size: "50m"
```

Useful when uploading:

* Images
* Videos
* Documents
* Large JSON payloads

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "100m"
```

---

# 12. Proxy Read Timeout

Controls how long NGINX waits for a response from the backend.

Example:

```yaml
nginx.ingress.kubernetes.io/proxy-read-timeout: "120"
```

Useful for:

* Long-running APIs
* File downloads
* Streaming
* Slow backend operations

---

# 13. Proxy Connect Timeout

Controls the timeout for establishing a connection to the backend.

```yaml
nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
```

---

# 14. CORS

CORS can be enabled using:

```yaml
nginx.ingress.kubernetes.io/enable-cors: "true"
```

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://example.com"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT, DELETE, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-headers: "Content-Type, Authorization"
```

---

# 15. Client IP Restrictions

You can allow or deny traffic based on source IP ranges.

Allowlist:

```yaml
nginx.ingress.kubernetes.io/whitelist-source-range: "10.0.0.0/24"
```

Denylist:

```yaml
nginx.ingress.kubernetes.io/denylist-source-range: "10.0.0.0/24"
```

NGINX Ingress supports CIDR-based source restrictions through these annotations.

---

# 16. Session Affinity

Session affinity can be configured using cookies.

```yaml
nginx.ingress.kubernetes.io/affinity: "cookie"
```

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "route"
```

Useful when an application needs users to remain connected to the same backend Pod.

---

# 17. TLS / HTTPS

TLS provides encrypted communication between the client and the Ingress endpoint.

```text
HTTP:

Client ----------- Ingress
        Plaintext


HTTPS:

Client ===== TLS ===== Ingress
       Encrypted
```

Kubernetes Ingress supports TLS configuration using a Kubernetes Secret containing:

```text
tls.crt
tls.key
```

The Secret should have:

```yaml
type: kubernetes.io/tls
```

---

# 18. TLS Secret

Example TLS Secret:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: example-tls

type: kubernetes.io/tls

data:
  tls.crt: BASE64_CERTIFICATE
  tls.key: BASE64_PRIVATE_KEY
```

Usually, you should avoid manually writing base64 values.

Instead, create the TLS Secret using:

```bash
kubectl create secret tls example-tls \
  --cert=tls.crt \
  --key=tls.key
```

---

# 19. Generate a Self-Signed Certificate

For local labs:

```bash
openssl req -x509 \
  -nodes \
  -days 365 \
  -newkey rsa:2048 \
  -keyout tls.key \
  -out tls.crt \
  -subj "/CN=example.com/O=example.com" \
  -addext "subjectAltName = DNS:example.com"
```

Then:

```bash
kubectl create secret tls example-tls \
  --cert=tls.crt \
  --key=tls.key
```

NGINX Ingress documents this approach for generating a self-signed certificate for testing.

---

# 20. TLS Ingress Configuration

Example:

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
                name: web-service
                port:
                  number: 80
```

Traffic:

```text
https://example.com
        |
        v
   TLS Certificate
        |
        v
      Ingress
        |
        v
   web-service:80
```

---

# 21. Important TLS Rule

The hostname in:

```yaml
spec:
  tls:
    - hosts:
        - example.com
```

should match the hostname in:

```yaml
spec:
  rules:
    - host: example.com
```

Example:

```yaml
tls:
  - hosts:
      - app.example.com
    secretName: app-tls

rules:
  - host: app.example.com
```

Kubernetes specifically notes that TLS hosts should explicitly match the corresponding rule hosts.

---

# 22. TLS Termination

In the common Ingress TLS model:

```text
Client
  |
  | HTTPS
  v
Ingress Controller
  |
  | HTTP
  v
Service
  |
  v
Pod
```

The Ingress Controller decrypts HTTPS traffic.

This is called **TLS termination**.

Kubernetes documents this as the standard Ingress TLS model, where TLS terminates at the Ingress point and traffic to the Service/Pods is plaintext unless separately configured.

---

# 23. TLS Passthrough

With TLS passthrough:

```text
Client
  |
 HTTPS
  |
  v
Ingress Controller
  |
 HTTPS
  |
  v
Service
  |
 HTTPS
  |
  v
Pod
```

The Ingress Controller does not terminate TLS.

NGINX forwards the encrypted TLS connection to the backend.

Annotation:

```yaml
nginx.ingress.kubernetes.io/ssl-passthrough: "true"
```

NGINX SSL passthrough must be enabled on the controller and works at Layer 4, bypassing normal NGINX Layer 7 processing.

### Important

SSL passthrough can prevent other HTTP-level annotations from working because NGINX cannot inspect the decrypted HTTP request.

---

# 24. TLS Termination vs TLS Passthrough

| Feature                  | TLS Termination | TLS Passthrough |
| ------------------------ | --------------- | --------------- |
| TLS ends at              | Ingress         | Backend         |
| Ingress decrypts traffic | Yes             | No              |
| Backend receives HTTPS   | Usually no      | Yes             |
| HTTP annotations         | Available       | Limited         |
| Certificate location     | Ingress Secret  | Backend         |
| Complexity               | Lower           | Higher          |

---

# 25. HTTP to HTTPS Redirect

Recommended production flow:

```text
                  HTTP
Client --------------------------+
                                  |
                                  v
                            +-----------+
                            |  Ingress  |
                            +-----------+
                                  |
                            308 Redirect
                                  |
                                  v
                           HTTPS :443
                                  |
                                  v
                             Service
```

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

---

# 26. Complete HTTP + HTTPS Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: web-ingress

  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"

spec:
  ingressClassName: nginx

  tls:
    - hosts:
        - app.example.com
      secretName: app-tls

  rules:
    - host: app.example.com

      http:
        paths:
          - path: /
            pathType: Prefix

            backend:
              service:
                name: web-service
                port:
                  number: 80
```

---

# 27. HTTP vs HTTPS

## HTTP

```text
Protocol: HTTP
Port: 80
Encryption: No
```

Example:

```text
http://example.com
```

## HTTPS

```text
Protocol: HTTPS
Port: 443
Encryption: TLS
```

Example:

```text
https://example.com
```

---

# 28. HTTPS Request Flow

```text
                    Internet
                       |
                       |
                HTTPS :443
                       |
                       v
             +------------------+
             | Ingress Controller|
             +------------------+
                       |
                 TLS Termination
                       |
                       v
                HTTP :80
                       |
                       v
                 Kubernetes
                   Service
                       |
                       v
                     Pods
```

---

# 29. Path-Based Routing

Example:

```yaml
rules:
  - host: example.com

    http:
      paths:

        - path: /app
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

Traffic:

```text
https://example.com/app
        |
        v
frontend-service


https://example.com/api
        |
        v
backend-service
```

---

# 30. Host-Based Routing

You can route different domains to different Services.

```yaml
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
                number: 8080
```

Traffic:

```text
frontend.example.com
        |
        v
frontend-service


api.example.com
        |
        v
backend-service
```

---

# 31. Ingress with Multiple Annotations

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: production-ingress

  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "120"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    nginx.ingress.kubernetes.io/enable-cors: "true"

spec:
  ingressClassName: nginx

  tls:
    - hosts:
        - app.example.com
      secretName: app-tls

  rules:
    - host: app.example.com

      http:
        paths:
          - path: /
            pathType: Prefix

            backend:
              service:
                name: web-service
                port:
                  number: 80
```

---

# 32. Basic Ingress Commands

Check Ingress:

```bash
kubectl get ingress
```

Short form:

```bash
kubectl get ing
```

Detailed information:

```bash
kubectl describe ingress <ingress-name>
```

Get YAML:

```bash
kubectl get ingress <ingress-name> -o yaml
```

List all namespaces:

```bash
kubectl get ingress -A
```

---

# 33. Check Ingress Controller

```bash
kubectl get pods -n ingress-nginx
```

Check Services:

```bash
kubectl get svc -n ingress-nginx
```

Check controller logs:

```bash
kubectl logs -n ingress-nginx \
  deployment/ingress-nginx-controller
```

---

# 34. Check TLS Secret

```bash
kubectl get secret
```

```bash
kubectl get secret example-tls
```

Check Secret type:

```bash
kubectl get secret example-tls -o yaml
```

Expected:

```yaml
type: kubernetes.io/tls
```

---

# 35. Verify Certificate

You can inspect a certificate with:

```bash
openssl x509 \
  -in tls.crt \
  -text \
  -noout
```

Check certificate expiration:

```bash
openssl x509 \
  -in tls.crt \
  -noout \
  -dates
```

---

# 36. Test HTTPS

Using curl:

```bash
curl -k https://example.com
```

`-k` allows testing with self-signed certificates.

For a trusted production certificate:

```bash
curl https://example.com
```

---

# 37. Test HTTP Redirect

```bash
curl -I http://example.com
```

Expected result may look like:

```text
HTTP/1.1 308 Permanent Redirect
Location: https://example.com/
```

The exact response can depend on the controller configuration.

---

# 38. Debugging Ingress

If Ingress is not working, check in this order.

### Step 1 - Check Ingress

```bash
kubectl get ingress
```

### Step 2 - Describe Ingress

```bash
kubectl describe ingress <name>
```

### Step 3 - Check Service

```bash
kubectl get svc
```

### Step 4 - Check Endpoints

```bash
kubectl get endpoints
```

Or:

```bash
kubectl get endpointslices
```

### Step 5 - Check Pods

```bash
kubectl get pods -o wide
```

### Step 6 - Check Controller

```bash
kubectl get pods -n ingress-nginx
```

### Step 7 - Check Logs

```bash
kubectl logs -n ingress-nginx \
  deployment/ingress-nginx-controller
```

---

# 39. Common Ingress Problems

## Problem 1: 404 Not Found

Possible causes:

* Wrong host
* Wrong path
* Wrong Service
* Wrong Ingress Controller
* Incorrect rewrite rule

Check:

```bash
kubectl describe ingress <name>
```

---

## Problem 2: 502 Bad Gateway

Usually indicates that NGINX cannot successfully communicate with the backend.

Check:

```bash
kubectl get svc
```

```bash
kubectl get endpoints
```

Make sure the Service has healthy endpoints.

---

## Problem 3: TLS Secret Not Found

Check:

```bash
kubectl get secret
```

The Secret must exist in the **same namespace** as the Ingress.

Example:

```yaml
tls:
  - hosts:
      - app.example.com
    secretName: app-tls
```

---

## Problem 4: Certificate Mismatch

Make sure:

```text
Ingress host
      =
Certificate hostname
```

For example:

```text
Ingress:
app.example.com

Certificate:
app.example.com
```

---

## Problem 5: HTTPS Not Working

Check:

```bash
kubectl describe ingress <name>
```

Then:

```bash
kubectl get secret app-tls
```

Verify:

```yaml
type: kubernetes.io/tls
```

Also verify that the certificate and private key match.

---

# 40. Important Annotation Examples

### Rewrite

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /
```

### Regex

```yaml
nginx.ingress.kubernetes.io/use-regex: "true"
```

### HTTPS Redirect

```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

### Force HTTPS

```yaml
nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

### Backend HTTPS

```yaml
nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
```

### SSL Passthrough

```yaml
nginx.ingress.kubernetes.io/ssl-passthrough: "true"
```

### CORS

```yaml
nginx.ingress.kubernetes.io/enable-cors: "true"
```

### Body Size

```yaml
nginx.ingress.kubernetes.io/proxy-body-size: "50m"
```

### Read Timeout

```yaml
nginx.ingress.kubernetes.io/proxy-read-timeout: "120"
```

### IP Allowlist

```yaml
nginx.ingress.kubernetes.io/whitelist-source-range: "10.0.0.0/24"
```

---

# 41. Ingress Annotation Cheat Sheet

| Annotation               | Purpose                        |
| ------------------------ | ------------------------------ |
| `rewrite-target`         | Rewrite request URL            |
| `use-regex`              | Enable regex paths             |
| `ssl-redirect`           | Redirect HTTP to HTTPS         |
| `force-ssl-redirect`     | Force HTTPS redirect           |
| `ssl-passthrough`        | Pass encrypted TLS to backend  |
| `backend-protocol`       | Configure backend protocol     |
| `proxy-body-size`        | Maximum request body           |
| `proxy-read-timeout`     | Backend response timeout       |
| `proxy-connect-timeout`  | Backend connection timeout     |
| `enable-cors`            | Enable CORS                    |
| `cors-allow-origin`      | Configure allowed origins      |
| `whitelist-source-range` | Allow specific IP ranges       |
| `denylist-source-range`  | Block specific IP ranges       |
| `affinity`               | Enable session affinity        |
| `configuration-snippet`  | Add custom NGINX configuration |

NGINX maintains the controller-specific annotation reference; available annotations and behavior can change with controller versions.

---

# 42. Important Security Considerations

### 1. Use HTTPS in production

Prefer:

```text
HTTPS :443
```

instead of exposing sensitive application traffic over plain HTTP.

### 2. Protect TLS private keys

The:

```text
tls.key
```

file contains the private key and must be protected.

### 3. Avoid unnecessary configuration snippets

Annotations such as:

```yaml
nginx.ingress.kubernetes.io/configuration-snippet
```

can introduce security risks in multi-tenant environments because they allow custom NGINX configuration.

### 4. Restrict access where required

Use source-range restrictions, authentication, network policies, or other security controls where appropriate.

---

# 43. cert-manager

For production Kubernetes clusters, certificates are commonly automated using **cert-manager**.

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
   |
   v
Ingress
```

NGINX Ingress supports integration with cert-manager for automated certificate management.

Example annotation:

```yaml
metadata:
  annotations:
    cert-manager.io/issuer: "letsencrypt-staging"
```

TLS:

```yaml
spec:
  tls:
    - hosts:
        - app.example.com
      secretName: app-example-tls
```

---

# 44. Production HTTPS Architecture

```text
                         Internet
                            |
                            |
                     HTTPS :443
                            |
                            v
                 +-------------------+
                 | Load Balancer /   |
                 | Ingress Controller|
                 +-------------------+
                            |
                     TLS Termination
                            |
                            v
                     Kubernetes
                        Service
                            |
                  +---------+---------+
                  |         |         |
                  v         v         v
                Pod       Pod       Pod
```

Benefits:

* Centralized routing
* TLS termination
* Load balancing
* Host-based routing
* Path-based routing
* HTTPS enforcement
* Centralized configuration

---

# 45. HTTP vs HTTPS Request Flow

## HTTP

```text
Browser
   |
   | HTTP :80
   v
Ingress
   |
   v
Service
   |
   v
Pod
```

## HTTPS

```text
Browser
   |
   | HTTPS :443
   v
Ingress
   |
   | TLS termination
   v
Service
   |
   v
Pod
```

## HTTPS Passthrough

```text
Browser
   |
   | HTTPS
   v
Ingress
   |
   | Encrypted HTTPS
   v
Service
   |
   v
Pod
```

---

# 46. Interview Questions

### Q1. What are Ingress annotations?

Annotations are key-value configuration entries attached to an Ingress resource to customize the behavior of an Ingress Controller.

---

### Q2. Are Ingress annotations part of standard Kubernetes?

Some annotations are controller-specific.

For example:

```text
nginx.ingress.kubernetes.io/*
```

are specific to the NGINX Ingress Controller.

---

### Q3. What is TLS termination?

TLS termination means the Ingress Controller decrypts HTTPS traffic and forwards the request to the backend.

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

---

### Q4. What is TLS passthrough?

TLS passthrough forwards encrypted TLS traffic to the backend without terminating TLS at the Ingress Controller.

---

### Q5. What is the difference between `ssl-redirect` and `force-ssl-redirect`?

```text
ssl-redirect
    |
    +-- Redirect HTTP when TLS is configured


force-ssl-redirect
    |
    +-- Force HTTPS even when TLS is terminated elsewhere
```

---

### Q6. What is `backend-protocol`?

It tells NGINX which protocol to use when communicating with the backend.

Example:

```yaml
nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
```

---

### Q7. Where is the TLS certificate stored?

In a Kubernetes Secret of type:

```yaml
kubernetes.io/tls
```

with:

```text
tls.crt
tls.key
```

---

### Q8. What port does Ingress TLS use?

Ingress TLS uses port:

```text
443
```

---

### Q9. Why is HTTPS preferred?

HTTPS provides:

* Encryption
* Data integrity
* Server authentication
* Protection against network interception

---

### Q10. How do you troubleshoot an Ingress?

Use:

```bash
kubectl get ingress
kubectl describe ingress <name>
kubectl get svc
kubectl get endpoints
kubectl get pods
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
```

---

# 47. Quick Revision

```text
Ingress
   |
   +-- Routes external HTTP/HTTPS traffic
   |
   +-- Host-based routing
   |
   +-- Path-based routing
   |
   +-- TLS termination
   |
   +-- HTTPS redirect
   |
   +-- Controller-specific annotations
```

### Most important annotations

```text
rewrite-target
use-regex
ssl-redirect
force-ssl-redirect
backend-protocol
ssl-passthrough
proxy-body-size
proxy-read-timeout
enable-cors
whitelist-source-range
denylist-source-range
affinity
```

### TLS essentials

```text
Secret Type:
kubernetes.io/tls

Keys:
tls.crt
tls.key

HTTPS:
443

HTTP:
80
```

---

# 48. Key Takeaways

1. **Ingress** manages external HTTP/HTTPS access to Kubernetes Services.
2. An **Ingress Controller** implements the routing rules.
3. **Annotations** customize controller-specific behavior.
4. NGINX annotations normally use the prefix:

   ```text
   nginx.ingress.kubernetes.io/
   ```
5. TLS certificates are stored in Kubernetes Secrets.
6. TLS Secrets contain:

   ```text
   tls.crt
   tls.key
   ```
7. HTTPS normally uses port `443`.
8. TLS termination decrypts traffic at the Ingress.
9. TLS passthrough keeps traffic encrypted until the backend.
10. `ssl-redirect` can redirect HTTP traffic to HTTPS.
11. `backend-protocol: "HTTPS"` enables HTTPS communication from NGINX to the backend.
12. `cert-manager` can automate certificate management.
13. Always verify the hostname in the TLS configuration matches the Ingress rule.
14. Annotation behavior is controller-specific, so always check the documentation for the controller/version being used.

---

# Official References

* Kubernetes Ingress documentation: [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/?utm_source=chatgpt.com)
* Kubernetes Ingress API reference: [Kubernetes Ingress API](https://kubernetes.io/docs/reference/kubernetes-api/networking/ingress-v1/?utm_source=chatgpt.com)
* NGINX Ingress Annotations: [NGINX Ingress Annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/?utm_source=chatgpt.com)
* NGINX Ingress TLS/HTTPS: [NGINX Ingress TLS/HTTPS](https://kubernetes.github.io/ingress-nginx/user-guide/tls/?utm_source=chatgpt.com)

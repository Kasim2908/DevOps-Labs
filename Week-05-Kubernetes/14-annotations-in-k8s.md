# Kubernetes Annotations — Complete Notes (Basics to Advanced)

> Complete Kubernetes **Annotations** notes from **basics to advanced level**, including concepts, syntax, YAML examples, Ingress annotations, Service annotations, Pod annotations, Helm usage, best practices, troubleshooting, and interview questions.

---

# 1. What are Annotations?

**Annotations** are key-value metadata attached to Kubernetes objects.

Unlike **Labels**, annotations are **not used for selecting or grouping resources**. They are used to store additional information for tools, controllers, and humans.

### Simple Definition

> **Annotations = Extra metadata that helps Kubernetes tools and applications, but not resource selection.**

---

# 2. Why Do We Need Annotations?

Kubernetes resources often need additional information such as:

* Load balancer configuration
* Ingress behavior
* Monitoring settings
* Deployment history
* Helm metadata
* Custom controller configuration

Without annotations, this metadata would have nowhere to live.

---

# 3. Labels vs Annotations

| Feature             | Labels            | Annotations                                    |
| ------------------- | ----------------- | ---------------------------------------------- |
| Purpose             | Identify & Select | Store Metadata                                 |
| Used by Selectors   | ✅ Yes             | ❌ No                                           |
| Query Resources     | ✅ Yes             | ❌ No                                           |
| Size Limit          | Small             | Large metadata                                 |
| Used by Controllers | Sometimes         | Very often                                     |
| Example             | `app=nginx`       | `nginx.ingress.kubernetes.io/rewrite-target=/` |

### Easy Memory Trick

```text
Labels      = WHO AM I?
Annotations = HOW SHOULD I BEHAVE?
```

---

# 4. Annotation Syntax

Every Kubernetes object has a metadata section.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

  annotations:
    owner: devops-team
    project: ecommerce
```

---

# 5. Basic Annotation Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: web-app

  annotations:
    description: Frontend application
    owner: Kasim
    created-by: Kubernetes
```

Check annotations:

```bash
kubectl describe pod web-app
```

or

```bash
kubectl get pod web-app -o yaml
```

---

# 6. Where Can We Use Annotations?

Annotations can be added to almost every Kubernetes resource.

| Resource    | Supports Annotations |
| ----------- | -------------------- |
| Pod         | ✅                    |
| Deployment  | ✅                    |
| Service     | ✅                    |
| Ingress     | ✅                    |
| ConfigMap   | ✅                    |
| Secret      | ✅                    |
| Namespace   | ✅                    |
| StatefulSet | ✅                    |
| Job         | ✅                    |
| PVC         | ✅                    |

---

# 7. Annotation Structure

```yaml
metadata:
  annotations:
    key: value
```

Example:

```yaml
metadata:
  annotations:
    company.com/environment: production
    company.com/owner: devops
```

---

# 8. Annotation Naming Convention

Annotations should use a **domain prefix**.

### Recommended

```text
example.com/key
```

Example:

```yaml
annotations:
  devops.company.com/owner: kasim
```

### Avoid

```yaml
annotations:
  owner: kasim
```

Using a domain avoids naming conflicts.

---

# 9. Built-in Kubernetes Annotations

Example:

```yaml
metadata:
  annotations:
    kubernetes.io/change-cause: Updated image to v2
```

This stores deployment history.

Check rollout history:

```bash
kubectl rollout history deployment nginx
```

---

# 10. Add Annotation Using Kubectl

Create annotation:

```bash
kubectl annotate pod nginx owner=kasim
```

Check:

```bash
kubectl describe pod nginx
```

---

# 11. Update Annotation

```bash
kubectl annotate pod nginx owner=devops --overwrite
```

Without `--overwrite`, Kubernetes will reject the update.

---

# 12. Remove Annotation

```bash
kubectl annotate pod nginx owner-
```

Notice the **hyphen (-)**.

---

# 13. View Only Annotations

```bash
kubectl get pod nginx \
-o jsonpath='{.metadata.annotations}'
```

Pretty output:

```bash
kubectl get pod nginx -o yaml
```

---

# 14. Multiple Annotations

```yaml
metadata:
  annotations:
    owner: kasim
    team: platform
    environment: production
    monitoring: enabled
```

---

# 15. Pod Annotations

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: app

  annotations:
    monitoring: "true"
    backup: daily
    owner: kasim
```

These annotations may be consumed by external tools.

---

# 16. Deployment Annotations

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: web

  annotations:
    kubernetes.io/change-cause: Initial deployment
```

Useful for deployment history.

---

# 17. Service Annotations

Service annotations configure cloud load balancers.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: web

  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
```

Different cloud providers use different annotation keys.

---

# 18. Ingress Annotations

Ingress is where annotations are most commonly used.

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```

This changes request routing behavior.

---

# 19. Why Ingress Uses Annotations

Ingress resource defines **routing**.

Annotations define **behavior**.

```text
Ingress Rule
      |
      v
Annotations
      |
      v
NGINX Controller
      |
      v
Traffic Behavior
```

---

# 20. Most Common NGINX Ingress Annotations

| Annotation             | Purpose            |
| ---------------------- | ------------------ |
| rewrite-target         | Rewrite URL        |
| ssl-redirect           | Force HTTPS        |
| enable-cors            | Enable CORS        |
| proxy-body-size        | Upload size        |
| proxy-read-timeout     | Backend timeout    |
| rate-limit             | Limit requests     |
| whitelist-source-range | Allow specific IPs |

---

# 21. Rewrite Target Annotation

```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /
```

Example request:

```text
/api/users
```

Backend receives:

```text
/users
```

---

# 22. SSL Redirect

Force HTTPS.

```yaml
annotations:
  nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

Flow:

```text
HTTP
  |
  v
301 Redirect
  |
  v
HTTPS
```

---

# 23. Enable CORS

```yaml
annotations:
  nginx.ingress.kubernetes.io/enable-cors: "true"
```

Useful for frontend applications consuming APIs.

---

# 24. Proxy Body Size

Default upload size may be too small.

Increase it:

```yaml
annotations:
  nginx.ingress.kubernetes.io/proxy-body-size: "100m"
```

Useful for file uploads.

---

# 25. Read Timeout

```yaml
annotations:
  nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
```

Prevents timeout for long-running requests.

---

# 26. Rate Limiting

```yaml
annotations:
  nginx.ingress.kubernetes.io/limit-rps: "10"
```

Limits requests per second.

Useful for API protection.

---

# 27. Whitelist IP Range

```yaml
annotations:
  nginx.ingress.kubernetes.io/whitelist-source-range: 10.0.0.0/24
```

Only allows requests from specified IP ranges.

---

# 28. Complete Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: web

  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"

spec:
  ingressClassName: nginx

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

# 29. Service Cloud Annotations

### AWS

```yaml
annotations:
  service.beta.kubernetes.io/aws-load-balancer-type: nlb
```

### Azure

```yaml
annotations:
  service.beta.kubernetes.io/azure-load-balancer-internal: "true"
```

### GCP

```yaml
annotations:
  networking.gke.io/load-balancer-type: Internal
```

---

# 30. Prometheus Annotations

Prometheus automatically discovers Pods using annotations.

```yaml
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/path: /metrics
  prometheus.io/port: "8080"
```

Architecture:

```text
Pod
 |
 | annotations
 v
Prometheus
 |
 v
Metrics Collection
```

---

# 31. Helm Annotations

Helm stores release metadata.

Example:

```yaml
annotations:
  meta.helm.sh/release-name: ecommerce
  meta.helm.sh/release-namespace: production
```

---

# 32. ArgoCD Annotations

ArgoCD uses annotations for sync behavior.

Example:

```yaml
annotations:
  argocd.argoproj.io/sync-wave: "5"
```

Higher sync wave means later deployment.

---

# 33. Istio Annotations

Istio sidecar injection.

```yaml
annotations:
  sidecar.istio.io/inject: "true"
```

Pod startup:

```text
Pod Created
      |
      v
Istio Injector
      |
      v
Envoy Sidecar Added
```

---

# 34. Vault Annotations

HashiCorp Vault injects secrets.

```yaml
annotations:
  vault.hashicorp.com/agent-inject: "true"
```

No secrets need to be stored inside YAML.

---

# 35. Cert Manager Annotations

Automatically issue TLS certificates.

```yaml
annotations:
  cert-manager.io/cluster-issuer: letsencrypt-prod
```

---

# 36. External DNS Annotation

Automatically create DNS records.

```yaml
annotations:
  external-dns.alpha.kubernetes.io/hostname: app.example.com
```

Flow:

```text
Ingress
   |
   v
ExternalDNS
   |
   v
DNS Record
```

---

# 37. Annotation Size Limit

Annotations are stored inside object metadata.

Best practices:

* Keep values reasonably small
* Avoid storing large files
* Use ConfigMaps for large configuration

---

# 38. Labels vs Annotations Decision Tree

```text
Need to Select Resource?
        |
      Yes -------> Label
        |
       No
        |
Need Extra Metadata?
        |
      Yes -------> Annotation
```

---

# 39. Best Practices

### Use Domain Prefix

```yaml
company.com/owner: kasim
```

### Keep Values Meaningful

```yaml
deployment-version: v1.2.0
```

### Avoid Sensitive Data

❌ Never

```yaml
annotations:
  password: admin123
```

Use **Secrets** instead.

---

# 40. Common Annotation Categories

| Category   | Example      |
| ---------- | ------------ |
| Monitoring | Prometheus   |
| Networking | Ingress      |
| Cloud      | LoadBalancer |
| Security   | Vault        |
| TLS        | Cert Manager |
| Deployment | Helm         |
| GitOps     | ArgoCD       |

---

# 41. Real Production Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: ecommerce

  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "100m"
    external-dns.alpha.kubernetes.io/hostname: shop.example.com

spec:
  ingressClassName: nginx

  tls:
    - hosts:
        - shop.example.com

      secretName: ecommerce-tls

  rules:
    - host: shop.example.com

      http:
        paths:
          - path: /

            pathType: Prefix

            backend:
              service:
                name: frontend

                port:
                  number: 80
```

---

# 42. Troubleshooting Annotations

### Check Resource

```bash
kubectl describe ingress ecommerce
```

### View YAML

```bash
kubectl get ingress ecommerce -o yaml
```

### Verify Controller

```bash
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller
```

---

# 43. Common Mistakes

### Mistake 1

Using annotation instead of label.

❌ Wrong

```yaml
annotations:
  app: nginx
```

Should be:

```yaml
labels:
  app: nginx
```

### Mistake 2

Missing quotes around boolean.

Prefer:

```yaml
ssl-redirect: "true"
```

instead of

```yaml
ssl-redirect: true
```

Many controllers expect strings.

---

# 44. Interview Questions

### Q1. What is an annotation?

Metadata stored as key-value pairs that is **not used for resource selection**.

### Q2. Difference between labels and annotations?

Labels identify resources.

Annotations configure behavior and store metadata.

### Q3. Can annotations be queried with selectors?

**No.**

### Q4. Which resource uses annotations the most?

**Ingress**

### Q5. Give examples of annotation use cases.

* NGINX Ingress
* Prometheus scraping
* Cert Manager
* External DNS
* Helm
* ArgoCD
* Vault

---

# 45. One-Minute Revision

```text
Annotations = Extra Metadata

Used For:
- Ingress
- Services
- Monitoring
- TLS
- Cloud
- GitOps

NOT Used For:
- Selectors
- Scheduling
- Grouping

Common Commands:

kubectl annotate pod nginx owner=kasim
kubectl annotate pod nginx owner-
kubectl describe pod nginx

Most Popular Annotation:

nginx.ingress.kubernetes.io/rewrite-target: /

Remember:

Labels → Selection
Annotations → Configuration
```

---

# 46. Final Architecture

```text
                Kubernetes Object
                       |
                 metadata section
                       |
          +------------+------------+
          |                         |
          v                         v
        Labels                 Annotations
          |                         |
   Resource Selection        Extra Metadata
          |                         |
          |                 Controllers / Tools
          |                         |
          +------------+------------+
                       |
                 Application Behavior
```

---

# 47. Annotation Cheat Sheet

| Annotation                       | Purpose        |
| -------------------------------- | -------------- |
| rewrite-target                   | Rewrite URL    |
| ssl-redirect                     | Force HTTPS    |
| enable-cors                      | Enable CORS    |
| proxy-body-size                  | Upload Limit   |
| proxy-read-timeout               | Timeout        |
| limit-rps                        | Rate Limiting  |
| whitelist-source-range           | IP Restriction |
| prometheus.io/scrape             | Metrics        |
| cert-manager.io/cluster-issuer   | TLS            |
| external-dns hostname            | DNS            |
| sidecar.istio.io/inject          | Sidecar        |
| vault.hashicorp.com/agent-inject | Secrets        |

---

# 48. Final Checklist

* [ ] Understand Labels vs Annotations
* [ ] Know Annotation Syntax
* [ ] Add & Remove Annotations
* [ ] Pod Annotations
* [ ] Deployment Annotations
* [ ] Service Annotations
* [ ] Ingress Annotations
* [ ] Prometheus Annotations
* [ ] Helm Metadata
* [ ] ArgoCD Annotations
* [ ] Istio Sidecar Injection
* [ ] Vault Secret Injection
* [ ] Cert Manager Integration
* [ ] External DNS Automation
* [ ] Troubleshooting
* [ ] Best Practices
* [ ] Interview Questions

---

## Key Takeaway

> **Labels identify resources, while Annotations define behavior and store additional metadata for Kubernetes controllers, cloud providers, and DevOps tools.**

```text
Kubernetes Resource
        |
        v
   metadata
   ├── labels      → Selection
   └── annotations → Configuration & Metadata
```

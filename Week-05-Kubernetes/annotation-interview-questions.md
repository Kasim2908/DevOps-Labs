# Kubernetes Annotations — Interview Questions & Answers

## 1. What are Annotations in Kubernetes?

**Answer:**

Annotations are **key-value pairs** attached to Kubernetes objects to store additional metadata or configuration information.

They are mainly used by:

* Kubernetes controllers
* Operators
* Ingress controllers
* Cloud providers
* Monitoring tools
* DevOps tools

### Example

```yaml
metadata:
  annotations:
    company.com/owner: devops
    company.com/environment: production
```

---

## 2. What is the difference between Labels and Annotations?

**Answer:**

| Labels                       | Annotations                                              |
| ---------------------------- | -------------------------------------------------------- |
| Used to identify resources   | Used to store additional metadata                        |
| Used by selectors            | Not used by selectors                                    |
| Used for grouping/filtering  | Used for configuration/integration                       |
| Usually contain small values | Can contain larger metadata                              |
| Example: `app: nginx`        | Example: `nginx.ingress.kubernetes.io/rewrite-target: /` |

### Easy way to remember

```text
Labels      → Select / Identify
Annotations → Configure / Describe
```

---

## 3. Can Annotations be used with Kubernetes Selectors?

**Answer:**

No. Kubernetes selectors work with **Labels**, not Annotations.

### Correct

```yaml
selector:
  matchLabels:
    app: nginx
```

### Incorrect

```yaml
selector:
  matchAnnotations:
    app: nginx
```

Kubernetes does not provide a standard `matchAnnotations` selector.

---

## 4. Where are Annotations commonly used?

**Answer:**

Annotations are commonly used by:

* Ingress controllers
* Cloud load balancers
* Prometheus
* Cert-Manager
* ExternalDNS
* Helm
* Argo CD
* Istio
* Vault
* Custom Kubernetes controllers

### Example

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```

Here, the NGINX Ingress controller reads the annotation and changes its behavior accordingly.

---

## 5. How do you add an Annotation using `kubectl`?

**Answer:**

Use the `kubectl annotate` command.

```bash
kubectl annotate pod nginx owner=devops
```

Verify the annotation:

```bash
kubectl describe pod nginx
```

Or:

```bash
kubectl get pod nginx -o yaml
```

---

## 6. How do you update an existing Annotation?

**Answer:**

Use the `--overwrite` option.

```bash
kubectl annotate pod nginx owner=platform --overwrite
```

Without `--overwrite`, Kubernetes will normally reject an attempt to replace an existing annotation.

---

## 7. How do you delete an Annotation?

**Answer:**

Add `-` after the annotation key.

```bash
kubectl annotate pod nginx owner-
```

For example:

```bash
kubectl annotate pod nginx environment-
```

This removes the `environment` annotation.

---

## 8. Can Annotations contain sensitive information such as passwords?

**Answer:**

No. Annotations should **not** be used to store sensitive information such as:

* Passwords
* API keys
* Tokens
* Private keys
* Database credentials

### ❌ Bad Practice

```yaml
metadata:
  annotations:
    database-password: mypassword123
```

### ✅ Better Approach

Use a Kubernetes Secret:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: database-secret
```

Annotations are metadata, not a secure secret-management mechanism.

---

## 9. Give a real-world example of an Annotation.

**Answer:**

A common real-world example is an NGINX Ingress annotation:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "100m"
```

This configures the request body size handled by the NGINX Ingress controller.

Another example is Prometheus-related metadata:

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
```

The important point is that many annotations are interpreted by the **specific controller or tool** that supports them.

---

## 10. What happens if you add an Annotation that no controller understands?

**Answer:**

Kubernetes can store the annotation as metadata, but it will have **no operational effect** unless some controller, operator, or application understands it.

### Example

```yaml
metadata:
  annotations:
    mycompany.com/test: "hello"
```

Kubernetes can store this annotation.

However, if no component is watching:

```text
mycompany.com/test
```

nothing will happen.

---

# Quick Revision

```text
Annotation
    ↓
Key-Value Metadata
    ↓
Used by Controllers / Tools
```

### Remember

```text
Labels
  ↓
Identify / Select / Group

Annotations
  ↓
Configure / Describe / Integrate
```

### Important Commands

```bash
# Add annotation
kubectl annotate pod nginx owner=devops

# Update annotation
kubectl annotate pod nginx owner=platform --overwrite

# Delete annotation
kubectl annotate pod nginx owner-

# View annotations
kubectl get pod nginx -o yaml

# Describe resource
kubectl describe pod nginx
```

### Important Interview Point

> **Labels are primarily used for identifying and selecting Kubernetes resources, while Annotations are used to store additional metadata or controller-specific configuration.**

# Kubernetes Ingress - pathType: Prefix vs Exact

## 1. What is `pathType`?

`pathType` defines **how Kubernetes should match the URL path** in an Ingress rule.

Example:

```yaml
paths:
  - path: /api
    pathType: Prefix
```

The two important path types are:

```text
Prefix
Exact
```

Kubernetes also supports:

```text
ImplementationSpecific
```

but `Prefix` and `Exact` are the main path types we need to understand.

---

# 2. Prefix Path Matching

With:

```yaml
path: /api
pathType: Prefix
```

the Ingress matches `/api` and paths underneath `/api`.

Example:

```text
/api
/api/
/api/users
/api/products
/api/users/123
```

All can match:

```text
/api → backend-service
```

### Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80
```

The routing behavior is conceptually:

```text
/api             → backend-service
/api/users       → backend-service
/api/products    → backend-service
/api/orders/123  → backend-service
```

---

# 3. Important Prefix Detail

Prefix matching is based on **path segments**, not simply string characters.

For example:

```text
/api
```

with:

```text
pathType: Prefix
```

matches:

```text
/api
/api/
/api/users
/api/products
```

But it does **not** match:

```text
/apiv2
/api-test
```

because those are not separate path segments under `/api`.

This is an important detail.

---

# 4. Exact Path Matching

With:

```yaml
path: /api
pathType: Exact
```

the path must match exactly.

Example:

```text
/api       → Match
/api/      → No match
/api/users → No match
/api/test  → No match
```

### Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: exact-ingress
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /api
            pathType: Exact
            backend:
              service:
                name: backend-service
                port:
                  number: 80
```

This means:

```text
/api → backend-service
```

but:

```text
/api/users → not matched by this rule
```

---

# 5. Prefix vs Exact

| Request          | Prefix `/api` | Exact `/api` |
| ---------------- | ------------: | -----------: |
| `/api`           |             ✅ |            ✅ |
| `/api/`          |             ✅ |            ❌ |
| `/api/users`     |             ✅ |            ❌ |
| `/api/products`  |             ✅ |            ❌ |
| `/api/users/123` |             ✅ |            ❌ |
| `/apiv2`         |             ❌ |            ❌ |
| `/api-test`      |             ❌ |            ❌ |

---

# 6. Real-World Example

Suppose you have an API application:

```text
/api
/api/users
/api/products
/api/orders
```

You normally want all these requests to go to the backend.

Use:

```yaml
path: /api
pathType: Prefix
```

Architecture:

```text
/api
/api/users
/api/products
/api/orders
       │
       ▼
backend-service
       │
       ▼
backend Pods
```

---

# 7. When to Use `Exact`

Suppose you have a specific endpoint:

```text
/health
```

and you only want the exact path to match.

You can use:

```yaml
path: /health
pathType: Exact
```

Then:

```text
/health        → Match
/health/check  → No match
/health/abc    → No match
```

This can be useful when you want very specific routing behavior.

---

# 8. Prefix Example with Multiple Services

Consider:

```yaml
rules:
  - http:
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

The routing becomes:

```text
/                    → frontend-service

/api                 → backend-service
/api/users           → backend-service
/api/products        → backend-service

/admin               → admin-service
/admin/users          → admin-service
/admin/dashboard     → admin-service
```

This is one of the most common uses of Prefix routing.

---

# 9. Prefix Matching and `/`

A very common Ingress rule is:

```yaml
path: /
pathType: Prefix
```

This effectively acts as a catch-all for normal URL paths.

For example:

```text
/             → frontend-service
/about        → frontend-service
/contact      → frontend-service
/products     → frontend-service
```

This is why our current rule:

```yaml
path: /
pathType: Prefix
```

can act as the frontend/default application route.

---

# 10. Prefix vs Exact - Visual Explanation

### Prefix

```text
/api
 │
 ├── /api
 ├── /api/
 ├── /api/users
 ├── /api/products
 └── /api/orders
```

All belong to the `/api` path hierarchy.

### Exact

```text
/api
 │
 └── ONLY /api
```

Only the exact path is matched.

---

# 11. Our Hands-On Example

In our lab, we created:

```text
/       → frontend-service
/api    → backend-service
/admin  → admin-service
```

using:

```yaml
pathType: Prefix
```

So:

```text
/              → frontend-service
/api            → backend-service
/api/users      → backend-service
/admin          → admin-service
/admin/users    → admin-service
```

However, we observed:

```text
/       → FRONTEND SERVICE
/api    → 404 Not Found
/admin  → 404 Not Found
```

This is important.

The `404` does **not necessarily mean Ingress routing failed**.

The Ingress correctly routed:

```text
/api
 ↓
backend-service
```

But the backend NGINX received:

```text
GET /api
```

and did not have a resource for `/api`.

Therefore it returned:

```text
404 Not Found
```

The same happened with `/admin`.

---

# 12. Prefix Matching vs URL Rewriting

These are two different concepts.

### Prefix matching

Answers:

> **Which Service should receive the request?**

Example:

```text
/api/users
     ↓
backend-service
```

### URL rewriting

Answers:

> **What path should the backend receive?**

For example:

```text
Client requests:

/api/users

        ↓

Ingress matches:

/api

        ↓

Rewrite:

/api/users → /users

        ↓

backend-service
```

So:

```text
Path Matching
    ↓
Select Service

URL Rewrite
    ↓
Modify request path
```

We will study URL rewriting separately.

---

# 13. `ImplementationSpecific`

Kubernetes also supports:

```yaml
pathType: ImplementationSpecific
```

Example:

```yaml
path: /api
pathType: ImplementationSpecific
```

Its behavior depends on the specific Ingress Controller.

For example, NGINX may interpret the path according to its own implementation.

Because behavior can vary between controllers, prefer:

```text
Prefix
```

or:

```text
Exact
```

when their behavior matches your requirements.

---

# 14. Quick Comparison

```text
Prefix
------
/api
/api/
/api/users
/api/products
```

Matches the `/api` path hierarchy.

```text
Exact
-----
/api
```

Matches only `/api`.

---

# 15. Interview Questions

## 1. What is `pathType` in Kubernetes Ingress?

`pathType` defines how Kubernetes should match an incoming URL path against an Ingress path rule.

---

## 2. What is the difference between Prefix and Exact?

`Prefix` matches a path and its child paths, while `Exact` matches only the exact path.

Example:

```text
Prefix /api:

/api
/api/users
/api/products
```

Exact `/api`:

```text
/api
```

only.

---

## 3. Does Prefix `/api` match `/apiv2`?

No.

Ingress Prefix matching is based on path segments, so `/api` does not match `/apiv2`.

---

## 4. What happens with `path: /` and `pathType: Prefix`?

It can match the root path and normal paths under `/`, making it useful as a catch-all/frontend route.

---

## 5. What is `ImplementationSpecific`?

It allows the Ingress Controller to determine the path matching behavior. Because behavior can differ between controllers, `Prefix` or `Exact` is generally preferable when possible.

---

## 6. Does `pathType` rewrite the URL?

No.

`pathType` determines **which path rule matches**.

URL rewriting is a separate feature, commonly configured with controller-specific annotations.

---

# 16. Important Mental Model

Remember:

```text
Incoming Request
       │
       ▼
   pathType
       │
       ▼
Which Ingress rule matches?
       │
       ▼
Which Service?
       │
       ▼
      Pods
```

And:

```text
Prefix
  =
Path hierarchy

Exact
  =
Exact URL path
```

---

# 17. Commands

Check Ingress:

```bash
kubectl get ingress
```

Detailed rules:

```bash
kubectl describe ingress <ingress-name>
```

View complete YAML:

```bash
kubectl get ingress <ingress-name> -o yaml
```

Test Prefix routing:

```bash
curl http://<NODE-IP>:<NODEPORT>/api
```

Test a child path:

```bash
curl http://<NODE-IP>:<NODEPORT>/api/users
```

---

# 18. Summary

| Concept                  | Meaning                              |
| ------------------------ | ------------------------------------ |
| `path`                   | URL path to match                    |
| `Prefix`                 | Matches the path hierarchy           |
| `Exact`                  | Matches only the exact path          |
| `ImplementationSpecific` | Controller-specific behavior         |
| URL Rewrite              | Changes the path sent to the backend |

### Final takeaway

> **`pathType` controls how Ingress matches a URL; it does not change the URL sent to the backend.**

```text
Client
  │
  │ /api/users
  ▼
Ingress
  │
  │ Prefix /api matches
  ▼
backend-service
  │
  ▼
Backend Pod
```

For URL modification:

```text
/api/users
     │
     ▼
Rewrite
     │
     ▼
/users
```

That is the next concept: **Ingress URL Rewrite**.

# Kubernetes ConfigMaps and Secrets — 20 Interview Questions with Answers

## 1. What is a ConfigMap in Kubernetes?

**Answer:**

A ConfigMap is a Kubernetes API object used to store **non-sensitive configuration data** separately from application code.

For example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
  DB_HOST: mysql-service
```

The application can consume this configuration as:

* Environment variables
* Files through volume mounts
* Individual configuration values

**Interview point:**

> ConfigMap is mainly used for non-confidential configuration.

---

# 2. What is a Secret in Kubernetes?

**Answer:**

A Secret is a Kubernetes API object designed to store **sensitive information**.

Examples include:

* Passwords
* API tokens
* Database credentials
* TLS certificates
* Private keys
* Registry credentials

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  username: admin
  password: password123
```

**Interview point:**

> Secret is intended for sensitive data, while ConfigMap is intended for non-sensitive data.

---

# 3. What is the difference between ConfigMap and Secret?

**Answer:**

| ConfigMap                                  | Secret                                     |
| ------------------------------------------ | ------------------------------------------ |
| Non-sensitive configuration                | Sensitive configuration                    |
| Example: `LOG_LEVEL`                       | Example: `DB_PASSWORD`                     |
| No special encoding requirement for `data` | `data` values are represented using Base64 |
| Can be environment variables               | Can be environment variables               |
| Can be mounted as files                    | Can be mounted as files                    |
| Not intended for credentials               | Intended for credentials                   |

Example:

```text
ConfigMap:
    APP_ENV=production
    DB_HOST=mysql

Secret:
    DB_USERNAME=admin
    DB_PASSWORD=******
```

**Interview point:**

> Never put passwords or API keys in a ConfigMap just because Kubernetes allows arbitrary data there.

---

# 4. Is data inside a Kubernetes Secret encrypted?

**Answer:**

Not automatically.

This is a very common interview trick.

Kubernetes Secret values in the normal `data` representation are **Base64-encoded**, but Base64 is not encryption.

For example:

```text
password123
```

can become:

```text
cGFzc3dvcmQxMjM=
```

Anyone who can obtain the value can decode it.

For stronger protection, Kubernetes can be configured with **encryption at rest** for data stored in the cluster's backing store.

**Interview point:**

```text
Base64 ≠ Encryption
```

---

# 5. What is the difference between `data` and `stringData` in a Secret?

**Answer:**

`data` expects Base64-encoded values:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=
```

`stringData` allows you to provide string values directly:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  username: admin
```

Kubernetes handles the conversion for `stringData`.

### Important

`stringData` does **not** mean the Secret is encrypted.

It is simply a convenient input mechanism.

---

# 6. How do you create a ConfigMap using `kubectl`?

**Answer:**

You can create one from literal values:

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info
```

Check it:

```bash
kubectl get configmap app-config
```

Get YAML:

```bash
kubectl get configmap app-config -o yaml
```

Describe it:

```bash
kubectl describe configmap app-config
```

You can also create ConfigMaps from files:

```bash
kubectl create configmap app-config \
  --from-file=application.properties
```

---

# 7. How do you create a Secret using `kubectl`?

**Answer:**

For a generic Secret:

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=password123
```

Check it:

```bash
kubectl get secret db-secret
```

Get its YAML:

```bash
kubectl get secret db-secret -o yaml
```

For TLS:

```bash
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key
```

**Interview point:**

> `kubectl create secret generic` is commonly used for application credentials, while specific Secret types exist for things such as TLS and registry credentials.

---

# 8. How can you use a ConfigMap as an environment variable?

**Answer:**

Suppose the ConfigMap is:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
```

A Pod can reference it:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
```

The container receives:

```text
APP_ENV=production
```

**Important terms:**

```text
configMapKeyRef
    ↓
specific key from ConfigMap
```

---

# 9. How can you use a Secret as an environment variable?

**Answer:**

Suppose:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  password: password123
```

Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

The application receives:

```text
DB_PASSWORD=password123
```

The important field is:

```yaml
secretKeyRef:
```

---

# 10. What is `envFrom`?

**Answer:**

`envFrom` allows you to import multiple suitable keys from a ConfigMap or Secret as environment variables.

For a ConfigMap:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

For a Secret:

```yaml
envFrom:
  - secretRef:
      name: app-secret
```

Suppose ConfigMap contains:

```yaml
data:
  APP_ENV: production
  LOG_LEVEL: info
  DB_HOST: mysql
```

The container receives corresponding environment variables.

### Difference

```text
configMapKeyRef
    ↓
One specific key

envFrom + configMapRef
    ↓
Import all suitable keys
```

The same concept applies to Secrets.

---

# 11. Can ConfigMaps and Secrets be mounted as files?

**Answer:**

Yes.

A ConfigMap can be mounted:

```yaml
volumes:
  - name: config-volume
    configMap:
      name: app-config
```

and then:

```yaml
volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```

A Secret can be mounted:

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

and:

```yaml
volumeMounts:
  - name: secret-volume
    mountPath: /etc/secrets
    readOnly: true
```

The keys generally appear as files inside the mounted directory.

---

# 12. What happens when a ConfigMap or Secret is updated?

**Answer:**

The answer depends on how the application consumes it.

### Environment variable

If a Pod gets a value through:

```yaml
env:
```

or:

```yaml
envFrom:
```

the already-running process does not automatically receive a new environment-variable value.

A workload restart/rollout is normally needed.

For example:

```bash
kubectl rollout restart deployment my-app
```

### Volume mount

ConfigMaps and Secrets mounted as volumes can be updated by Kubernetes after the underlying object changes, subject to Kubernetes' normal propagation mechanism.

However, the application must actually read/reload the changed file.

### Important interview answer

> Environment-variable consumers generally require a Pod/container restart to get changed values, while mounted ConfigMap/Secret volumes can receive updates.

---

# 13. What is `configMapKeyRef` and `secretKeyRef`?

**Answer:**

They are used when you want to reference a **specific key**.

ConfigMap:

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```

Secret:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

Difference:

```text
configMapKeyRef
    ↓
ConfigMap

secretKeyRef
    ↓
Secret
```

---

# 14. What are the common Kubernetes Secret types?

**Answer:**

Some important built-in Secret types are:

### 1. Opaque

Generic application Secret:

```yaml
type: Opaque
```

### 2. TLS

```yaml
type: kubernetes.io/tls
```

Common keys:

```text
tls.crt
tls.key
```

### 3. Docker registry credentials

```yaml
type: kubernetes.io/dockerconfigjson
```

Used for private container registries.

### 4. Basic authentication

```text
kubernetes.io/basic-auth
```

### 5. SSH authentication

```text
kubernetes.io/ssh-auth
```

### 6. Bootstrap token

```text
bootstrap.kubernetes.io/token
```

**Interview point:**

> `Opaque` is commonly used for generic application secrets.

---

# 15. Are ConfigMaps and Secrets namespace-scoped?

**Answer:**

Yes.

Both ConfigMaps and Secrets are **namespaced resources**.

For example:

```text
development namespace
    └── app-config

production namespace
    └── app-config
```

Both can have the same name because they exist in different namespaces.

A Pod normally references resources in its own namespace.

Therefore, if:

```text
Pod → production
ConfigMap → development
```

the Pod cannot simply reference that ConfigMap as though it were in the same namespace.

**Interview point:**

> Always check the namespace when a ConfigMap or Secret cannot be found.

---

# 16. How would you troubleshoot a Pod that cannot find a ConfigMap or Secret?

**Answer:**

I would check the following systematically.

### Step 1: Check the Pod

```bash
kubectl get pods
```

### Step 2: Describe the Pod

```bash
kubectl describe pod <pod-name>
```

Look at the **Events** section.

### Step 3: Check the ConfigMap

```bash
kubectl get configmap <configmap-name>
```

### Step 4: Check the Secret

```bash
kubectl get secret <secret-name>
```

### Step 5: Verify namespace

```bash
kubectl get cm <name> -n <namespace>
kubectl get secret <name> -n <namespace>
```

### Step 6: Verify key names

For example, if Secret contains:

```yaml
data:
  password: ...
```

but the Pod references:

```yaml
key: db-password
```

the reference is incorrect.

### Common causes

```text
Wrong resource name
Wrong namespace
Wrong key
Resource does not exist
Typo in YAML
Pod references an optional/non-optional resource incorrectly
```

---

# 17. What are immutable ConfigMaps and Secrets?

**Answer:**

Kubernetes allows ConfigMaps and Secrets to be marked immutable:

```yaml
immutable: true
```

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
immutable: true
data:
  APP_ENV: production
```

An immutable ConfigMap cannot have its data changed.

Similarly:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
immutable: true
type: Opaque
stringData:
  username: admin
```

### Benefits

* Prevents accidental modification.
* Helps make configuration predictable.
* Can reduce unnecessary update/watch activity in large clusters.
* Useful for configuration that should be replaced rather than modified.

If the value needs to change, an operational pattern is to create a new resource and update the workload reference.

---

# 18. How would you securely manage Secrets in production?

**Answer:**

I would use multiple layers of security.

### 1. Enable encryption at rest

Protect Secret data stored by the Kubernetes control plane.

### 2. Use RBAC

Only allow users/service accounts to access the Secrets they actually need.

### 3. Follow least privilege

Avoid permissions such as:

```text
get/list/watch all Secrets
```

unless genuinely required.

### 4. Don't commit Secrets to Git

Avoid both:

```yaml
password: password123
```

and:

```yaml
password: cGFzc3dvcmQxMjM=
```

because Base64 is reversible.

### 5. Rotate credentials

Regularly replace passwords, tokens, and keys.

### 6. Avoid logging Secret values

Don't print:

```bash
env
```

or application configuration if it contains credentials.

### 7. Consider external secret management

Depending on the organization, use systems such as:

* Cloud secret managers
* HashiCorp Vault
* External Secrets Operator
* Secrets Store CSI Driver
* Other approved enterprise secret-management solutions

### Interview answer

> I would protect Secrets using RBAC, encryption at rest, least privilege, rotation, auditing, secure Git practices, and, where appropriate, an external secret-management solution.

---

# 19. What is the difference between using a Secret as an environment variable and mounting it as a volume?

**Answer:**

## Environment variable

Example:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

The application receives:

```text
DB_PASSWORD=...
```

### Advantages

* Simple.
* Most application frameworks support environment variables.
* Easy to configure.

### Considerations

* Existing processes don't automatically receive changed values.
* Applications or debugging tools may accidentally expose environment variables.
* Environment values can appear in diagnostic output depending on the application/runtime.

---

## Volume mount

Example:

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

The application reads:

```text
/etc/secrets/password
```

### Advantages

* Useful for files such as certificates and keys.
* Kubernetes can update mounted Secret contents.
* Applications can implement file-based reload behavior.

### Considerations

* Application must know how to read files.
* Application may need reload logic.
* `subPath` mounts have special update behavior.

---

# 20. How would you design ConfigMap and Secret management for a production application?

**Answer:**

I would separate configuration into two categories.

### Non-sensitive configuration

Use a ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
  DB_HOST: mysql-service
  DB_PORT: "3306"
```

### Sensitive configuration

Use a Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_USERNAME: appuser
  DB_PASSWORD: <managed-securely>
```

### Application

```yaml
containers:
  - name: app
    image: myapp:1.0

    envFrom:
      - configMapRef:
          name: app-config

      - secretRef:
          name: app-secret
```

### Production security

I would additionally:

1. Use RBAC with least privilege.
2. Enable encryption at rest.
3. Avoid committing Secrets to Git.
4. Use an approved external secret manager when appropriate.
5. Rotate credentials.
6. Avoid logging sensitive values.
7. Restrict which Pods can access Secrets.
8. Use immutable resources where appropriate.
9. Use versioning/checksum-based rollout strategies when configuration changes should trigger a new Deployment rollout.
10. Monitor and audit access to sensitive resources.

### Strong interview answer

> I would keep non-sensitive configuration in ConfigMaps and sensitive credentials in Secrets. I would consume them through environment variables or volume mounts depending on application requirements. For production, I would protect Secrets with RBAC, encryption at rest, least privilege, rotation, auditing, secure Git practices, and potentially an external secret manager. I would also design the deployment so configuration changes are rolled out predictably rather than relying on already-running processes to notice changed environment variables.

---

# Quick Interview Revision

| Question                            | One-Line Answer                                                                                                    |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| What is ConfigMap?                  | Stores non-sensitive configuration.                                                                                |
| What is Secret?                     | Stores sensitive configuration.                                                                                    |
| ConfigMap vs Secret?                | Configuration vs sensitive configuration.                                                                          |
| Is Base64 encryption?               | No, Base64 is encoding.                                                                                            |
| `data` vs `stringData`?             | `data` uses Base64 representation; `stringData` accepts strings.                                                   |
| `configMapKeyRef`?                  | References one ConfigMap key.                                                                                      |
| `secretKeyRef`?                     | References one Secret key.                                                                                         |
| `envFrom`?                          | Imports multiple/all suitable keys as environment variables.                                                       |
| Can they be mounted as files?       | Yes.                                                                                                               |
| Are they namespace-scoped?          | Yes.                                                                                                               |
| Do env vars update automatically?   | No, running processes need a restart/reload mechanism.                                                             |
| Can mounted values update?          | Yes, mounted volumes can receive updates subject to Kubernetes propagation behavior.                               |
| TLS Secret type?                    | `kubernetes.io/tls`.                                                                                               |
| Registry Secret type?               | `kubernetes.io/dockerconfigjson`.                                                                                  |
| Immutable resource?                 | A resource whose data cannot be modified after being made immutable.                                               |
| How to secure Secrets?              | RBAC + encryption at rest + least privilege + rotation + auditing.                                                 |
| Should Secrets be committed to Git? | Generally no.                                                                                                      |
| Where should passwords go?          | Secret/external secret manager.                                                                                    |
| How troubleshoot missing Secret?    | Check Pod events, name, namespace, and key.                                                                        |
| Production approach?                | Separate ConfigMap/Secret, secure access, rotate credentials, and use external secret management when appropriate. |

---

# Important Interview Traps

### Trap 1

**Interviewer:** "Secrets are encrypted because they are Base64 encoded, right?"

**Correct answer:**

> No. Base64 is encoding, not encryption. Kubernetes can be configured to encrypt data at rest, but that is a separate security mechanism.

---

### Trap 2

**Interviewer:** "Can I put a database password in a ConfigMap?"

**Correct answer:**

> Kubernetes technically allows arbitrary data in a ConfigMap, but passwords are sensitive and should be stored in a Secret or an appropriate external secret-management system.

---

### Trap 3

**Interviewer:** "I updated the ConfigMap. Why didn't my application's environment variable change?"

**Correct answer:**

> Environment variables are established when the container starts. Updating the ConfigMap doesn't modify the environment of an already-running process. The workload normally needs to restart or otherwise reload the configuration.

---

### Trap 4

**Interviewer:** "Are Secrets completely secure by default?"

**Correct answer:**

> No. Proper Secret security depends on controls such as RBAC, encryption at rest, node security, auditing, least privilege, and secure secret-management practices.

---

### Trap 5

**Interviewer:** "Can a Pod in namespace A directly use a ConfigMap from namespace B?"

**Correct answer:**

> ConfigMaps and Secrets are namespaced resources. A normal Pod reference does not cross namespaces; the resource needs to be available in the Pod's namespace or be made available through an appropriate controller/application mechanism.

---

# Final 20-Question Cheat Sheet

```text
1.  ConfigMap?
    → Non-sensitive configuration.

2.  Secret?
    → Sensitive configuration.

3.  Difference?
    → ConfigMap = normal config; Secret = sensitive config.

4.  Secret encrypted?
    → Not merely by being Base64 encoded.

5.  Base64?
    → Encoding, not encryption.

6.  stringData?
    → Allows string values to be supplied conveniently.

7.  data?
    → Base64-encoded Secret representation.

8.  configMapKeyRef?
    → One ConfigMap key.

9.  secretKeyRef?
    → One Secret key.

10. envFrom?
    → Imports all suitable keys from ConfigMap/Secret.

11. Volume mount?
    → Makes configuration available as files.

12. Namespace?
    → Both ConfigMaps and Secrets are namespaced.

13. ConfigMap update + env?
    → Running process does not automatically get the new value.

14. Mounted ConfigMap update?
    → Kubernetes can propagate changes to the volume.

15. TLS Secret?
    → kubernetes.io/tls.

16. Registry Secret?
    → kubernetes.io/dockerconfigjson.

17. Immutable?
    → Prevents data modification after creation.

18. Secure Secret?
    → RBAC + encryption at rest + least privilege + rotation.

19. Git?
    → Don't commit plaintext or merely Base64-encoded Secrets.

20. Production?
    → Separate config from code, protect credentials,
       use controlled access, rotation, auditing,
       and external secret management when appropriate.
```

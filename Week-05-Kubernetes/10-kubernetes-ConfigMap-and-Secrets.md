# Kubernetes ConfigMaps and Secrets — Complete Notes

> **Topic:** Kubernetes ConfigMap and Secret
> **Level:** Beginner → Advanced
> **Format:** Markdown notes
> **Last updated:** August 2026

---

## Table of Contents

1. [Configuration in Kubernetes](#1-configuration-in-kubernetes)
2. [ConfigMap](#2-configmap)
3. [Why ConfigMaps Are Needed](#3-why-configmaps-are-needed)
4. [ConfigMap Structure](#4-configmap-structure)
5. [Creating a ConfigMap](#5-creating-a-configmap)
6. [Using ConfigMap as Environment Variables](#6-using-configmap-as-environment-variables)
7. [Using ConfigMap as Volumes](#7-using-configmap-as-volumes)
8. [ConfigMap from Files](#8-configmap-from-files)
9. [ConfigMap from Literals](#9-configmap-from-literals)
10. [ConfigMap YAML Examples](#10-configmap-yaml-examples)
11. [ConfigMap Important Fields](#11-configmap-important-fields)
12. [ConfigMap Updates](#12-configmap-updates)
13. [ConfigMap Limitations](#13-configmap-limitations)
14. [Secret](#14-secret)
15. [Why Secrets Are Needed](#15-why-secrets-are-needed)
16. [Secret Types](#16-secret-types)
17. [Creating Secrets](#17-creating-secrets)
18. [Base64 Encoding and Decoding](#18-base64-encoding-and-decoding)
19. [Using Secrets as Environment Variables](#19-using-secrets-as-environment-variables)
20. [Using Secrets as Volumes](#20-using-secrets-as-volumes)
21. [Secret YAML Examples](#21-secret-yaml-examples)
22. [Secret Updates](#22-secret-updates)
23. [Secret Security](#23-secret-security)
24. [ConfigMap vs Secret](#24-configmap-vs-secret)
25. [Common Commands](#25-common-commands)
26. [Common Mistakes](#26-common-mistakes)
27. [Best Practices](#27-best-practices)
28. [Advanced Concepts](#28-advanced-concepts)
29. [Troubleshooting](#29-troubleshooting)
30. [Interview Questions](#30-interview-questions)
31. [Quick Revision](#31-quick-revision)

---

# 1. Configuration in Kubernetes

Applications usually need configuration values such as:

* Application environment
* Database hostname
* Database port
* API URLs
* Feature flags
* Log levels
* Application names
* Usernames
* Passwords
* API tokens
* TLS certificates
* SSH keys

Kubernetes provides two important objects for storing configuration:

* **ConfigMap** → non-sensitive configuration
* **Secret** → sensitive configuration

Example:

```text
Application
    |
    +-- ConfigMap
    |     |
    |     +-- APP_ENV=production
    |     +-- LOG_LEVEL=info
    |     +-- DB_HOST=mysql
    |
    +-- Secret
          |
          +-- DB_USERNAME=admin
          +-- DB_PASSWORD=******
```

---

# 2. ConfigMap

A **ConfigMap** is a Kubernetes API object used to store **non-confidential configuration data**.

Typical examples:

```text
APP_ENV=production
LOG_LEVEL=info
DB_HOST=mysql-service
DB_PORT=3306
FEATURE_X_ENABLED=true
```

A ConfigMap separates configuration from application code.

Instead of hardcoding:

```text
DB_HOST=mysql-service
```

inside the application image, the application can read it from a ConfigMap.

---

# 3. Why ConfigMaps Are Needed

Without ConfigMaps, configuration may be hardcoded inside:

* Docker images
* Application source code
* Startup scripts
* Deployment manifests

This creates problems.

## Benefits

### 3.1 Separation of configuration and code

Application code:

```text
app.py
```

Configuration:

```text
ConfigMap
```

They can be changed independently.

### 3.2 Same image, different environments

The same Docker image can be used in:

```text
Development
       ↓
Testing
       ↓
Staging
       ↓
Production
```

while each environment has different ConfigMaps.

### 3.3 Easier configuration management

Configuration can be managed using Kubernetes resources.

### 3.4 Avoid rebuilding images

Changing:

```text
LOG_LEVEL=debug
```

does not necessarily require building a new Docker image.

---

# 4. ConfigMap Structure

Basic ConfigMap:

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

Important sections:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  KEY: VALUE
```

---

# 5. Creating a ConfigMap

## 5.1 Create from literal values

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info
```

Check:

```bash
kubectl get configmap
```

or:

```bash
kubectl get cm
```

Describe:

```bash
kubectl describe configmap app-config
```

Get YAML:

```bash
kubectl get configmap app-config -o yaml
```

---

# 6. Using ConfigMap as Environment Variables

A ConfigMap can provide environment variables to a container.

## ConfigMap

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

## Pod

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

---

# 7. Import All ConfigMap Values

Instead of importing variables individually, use:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

Complete example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app
      image: nginx
      envFrom:
        - configMapRef:
            name: app-config
```

Every key in the ConfigMap becomes an environment variable.

For example:

```yaml
data:
  APP_ENV: production
  LOG_LEVEL: info
```

becomes:

```text
APP_ENV=production
LOG_LEVEL=info
```

---

# 8. Using ConfigMap as a Volume

A ConfigMap can be mounted as files.

ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  application.properties: |
    app.name=myapp
    app.environment=production
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
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config

  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

The container can see:

```text
/etc/config/application.properties
```

---

# 9. ConfigMap from Files

Create a ConfigMap from a file:

```bash
kubectl create configmap app-config \
  --from-file=application.properties
```

Suppose:

```text
application.properties
```

contains:

```properties
app.name=myapp
app.env=production
log.level=info
```

Kubernetes stores the file content under the key:

```text
application.properties
```

---

## 9.1 Custom Key Name

```bash
kubectl create configmap app-config \
  --from-file=config=application.properties
```

Now:

```text
config
```

is the key.

---

# 10. ConfigMap from Literals

Example:

```bash
kubectl create configmap app-config \
  --from-literal=APP_NAME=myapp \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info
```

Check:

```bash
kubectl get cm app-config -o yaml
```

---

# 11. ConfigMap YAML Examples

## Simple ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_NAME: myapp
  APP_ENV: production
  LOG_LEVEL: info
```

## Multi-line Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.conf: |
    server {
      port = 8080
      host = 0.0.0.0
    }
```

The `|` syntax preserves a multi-line value.

---

# 12. ConfigMap Important Fields

## `apiVersion`

For ConfigMap:

```yaml
apiVersion: v1
```

## `kind`

```yaml
kind: ConfigMap
```

## `metadata.name`

Name of the ConfigMap:

```yaml
metadata:
  name: app-config
```

## `metadata.namespace`

Optional namespace:

```yaml
metadata:
  namespace: production
```

## `data`

Stores normal configuration:

```yaml
data:
  APP_ENV: production
```

## `binaryData`

Used for binary data encoded as base64.

Example:

```yaml
binaryData:
  example.bin: <base64-data>
```

For normal text configuration, use `data`.

---

# 13. ConfigMap Updates

You can update a ConfigMap:

```bash
kubectl edit configmap app-config
```

Or apply a modified YAML:

```bash
kubectl apply -f configmap.yaml
```

Important behavior:

### Environment variables

If a ConfigMap is consumed as environment variables, an already-running container does **not** automatically receive the changed environment variables.

The Pod/container generally needs to be recreated.

Example:

```bash
kubectl rollout restart deployment my-app
```

### Mounted ConfigMap volumes

ConfigMaps mounted as volumes can be updated by Kubernetes eventually after the ConfigMap changes.

There can be a propagation delay.

Applications also need to actually reread the files if they are expected to react to changes.

---

# 14. ConfigMap Limitations

Important points:

* ConfigMaps are intended for non-sensitive data.
* They are not designed for passwords.
* They are namespace-scoped.
* A Pod can consume a ConfigMap from its own namespace.
* ConfigMaps have a size limitation.
* The commonly relevant total size limit is around **1 MiB** per ConfigMap object.
* Large application configuration should generally be stored elsewhere.
* Environment-variable consumers do not automatically get updated values in running containers.
* Mounted files have update behavior, but applications must handle/reload changed files appropriately.
* ConfigMaps do not provide encryption simply because they are ConfigMaps.

---

# 15. Secret

A **Secret** is a Kubernetes API object intended to hold sensitive data.

Examples:

* Passwords
* API tokens
* Access keys
* Database credentials
* TLS certificates
* SSH keys
* Registry credentials

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

---

# 16. Why Secrets Are Needed

Secrets provide a Kubernetes-native mechanism for separating sensitive configuration from application code and ordinary configuration.

Instead of:

```yaml
env:
  - name: DB_PASSWORD
    value: password123
```

you can use:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

This is better operationally, although Kubernetes Secrets are **not automatically equivalent to a secure external secrets manager**.

---

# 17. Secret Types

Kubernetes supports several Secret types.

## 17.1 `Opaque`

Generic Secret:

```yaml
type: Opaque
```

This is the most common custom Secret type.

---

## 17.2 `kubernetes.io/tls`

Used for TLS certificates.

Typical keys:

```text
tls.crt
tls.key
```

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-certificate>
  tls.key: <base64-private-key>
```

---

## 17.3 `kubernetes.io/dockerconfigjson`

Used for container registry authentication.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: registry-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-data>
```

Used by Pods through:

```yaml
imagePullSecrets:
  - name: registry-secret
```

---

## 17.4 `kubernetes.io/basic-auth`

Used for basic authentication credentials.

Typical keys:

```text
username
password
```

---

## 17.5 `kubernetes.io/ssh-auth`

Used for SSH authentication.

Typical key:

```text
ssh-privatekey
```

---

## 17.6 `bootstrap.kubernetes.io/token`

Used for bootstrap tokens.

---

# 18. Base64 Encoding and Decoding

A very important concept:

**Base64 is encoding, not encryption.**

For example:

```text
admin
```

encoded using Base64 becomes:

```text
YWRtaW4=
```

Decode:

```bash
echo 'YWRtaW4=' | base64 --decode
```

Output:

```text
admin
```

Therefore, this:

```yaml
data:
  password: cGFzc3dvcmQ=
```

does **not** mean the password is encrypted.

Anyone with sufficient access to the Secret can potentially retrieve and decode the value.

---

# 19. `data` vs `stringData`

Secrets support both:

```yaml
data:
```

and:

```yaml
stringData:
```

## Using `data`

Values must be Base64 encoded:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

## Using `stringData`

You can write plain strings:

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

Kubernetes handles conversion into the Secret representation.

### Important

Do not confuse:

```text
stringData
```

with encryption.

It is simply a convenient way to specify Secret values in manifests.

---

# 20. Creating Secrets

## 20.1 Create from literals

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=password123
```

Check:

```bash
kubectl get secret
```

Describe:

```bash
kubectl describe secret db-secret
```

Get YAML:

```bash
kubectl get secret db-secret -o yaml
```

---

## 20.2 Create from a file

```bash
kubectl create secret generic app-secret \
  --from-file=secret.txt
```

---

## 20.3 Create TLS Secret

```bash
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key
```

---

## 20.4 Create Docker Registry Secret

```bash
kubectl create secret docker-registry registry-secret \
  --docker-server=REGISTRY \
  --docker-username=USERNAME \
  --docker-password=PASSWORD
```

The exact command options can vary by use case and Kubernetes tooling.

---

# 21. Using Secrets as Environment Variables

## One key

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
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username

        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

The application receives:

```text
DB_USERNAME=admin
DB_PASSWORD=password123
```

---

# 22. Import All Secret Values

You can import all keys:

```yaml
envFrom:
  - secretRef:
      name: db-secret
```

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app
      image: nginx
      envFrom:
        - secretRef:
            name: db-secret
```

---

# 23. Using Secrets as Volumes

Secrets can be mounted as files.

Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  username: admin
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
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true

  volumes:
    - name: secret-volume
      secret:
        secretName: app-secret
```

Files become:

```text
/etc/secrets/username
/etc/secrets/password
```

---

# 24. Secret File Permissions

You can specify a default file mode:

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
      defaultMode: 0400
```

Example permissions:

```text
0400
```

means:

```text
Owner: read
Group: no permission
Others: no permission
```

Be careful with YAML numeric interpretation and use the documented representation expected by your Kubernetes version.

---

# 25. Secret Updates

Secrets can be updated:

```bash
kubectl edit secret db-secret
```

or:

```bash
kubectl apply -f secret.yaml
```

### Secret used as environment variable

Running containers do not automatically receive changed environment variables.

A rollout/restart is normally required:

```bash
kubectl rollout restart deployment my-app
```

### Secret mounted as a volume

Kubernetes can update the mounted Secret contents after the Secret changes, subject to the normal projected-volume update mechanism.

Applications must still reload/read the updated files.

---

# 26. Secret Security

This is one of the most important sections.

## 26.1 Secrets are not automatically encrypted everywhere

By default, Kubernetes Secret values are Base64-encoded in the API representation.

Base64 is **not encryption**.

For stronger protection, configure **encryption at rest** for Kubernetes API data.

---

## 26.2 RBAC is extremely important

Control who can:

```text
get
list
watch
create
update
patch
delete
```

Secrets.

Example RBAC permissions:

```yaml
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
```

Grant the minimum permissions required.

---

## 26.3 Avoid putting Secrets in Git

Avoid committing:

```yaml
password: password123
```

or even:

```yaml
password: cGFzc3dvcmQxMjM=
```

Base64 does not make a Secret safe to commit.

---

## 26.4 Use external secret-management systems when appropriate

For production environments, organizations often use:

* Cloud secret managers
* HashiCorp Vault
* External Secrets Operator
* Secrets Store CSI Driver
* GitOps secret-management solutions

These can reduce the need to keep plaintext secrets in Git repositories.

---

# 27. ConfigMap vs Secret

| Feature               | ConfigMap                   | Secret                                   |
| --------------------- | --------------------------- | ---------------------------------------- |
| Purpose               | Non-sensitive configuration | Sensitive configuration                  |
| Example               | `LOG_LEVEL=info`            | Database password                        |
| Namespace scoped      | Yes                         | Yes                                      |
| Base64 required       | No                          | `data` values use Base64                 |
| Encryption            | Not inherently              | Can be encrypted at rest when configured |
| Environment variables | Yes                         | Yes                                      |
| Volume mount          | Yes                         | Yes                                      |
| TLS support           | No special type             | Yes                                      |
| Registry credentials  | No                          | Yes                                      |
| Password storage      | No                          | Yes                                      |
| Size considerations   | Around 1 MiB object limit   | Around 1 MiB object limit                |

### Simple rule

```text
Normal configuration → ConfigMap

Sensitive configuration → Secret
```

---

# 28. Using Both Together

A real application commonly uses both.

ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
  DB_HOST: mysql
  DB_PORT: "3306"
```

Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_USERNAME: admin
  DB_PASSWORD: password123
```

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app

  template:
    metadata:
      labels:
        app: app

    spec:
      containers:
        - name: app
          image: myapp:1.0

          envFrom:
            - configMapRef:
                name: app-config

            - secretRef:
                name: app-secret
```

The application receives both types of configuration.

---

# 29. Common Commands

## ConfigMap commands

Create:

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production
```

List:

```bash
kubectl get configmaps
```

Short form:

```bash
kubectl get cm
```

Get:

```bash
kubectl get cm app-config
```

Get YAML:

```bash
kubectl get cm app-config -o yaml
```

Describe:

```bash
kubectl describe cm app-config
```

Edit:

```bash
kubectl edit cm app-config
```

Delete:

```bash
kubectl delete cm app-config
```

---

## Secret commands

Create:

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=password123
```

List:

```bash
kubectl get secrets
```

Get:

```bash
kubectl get secret db-secret
```

Get YAML:

```bash
kubectl get secret db-secret -o yaml
```

Describe:

```bash
kubectl describe secret db-secret
```

Edit:

```bash
kubectl edit secret db-secret
```

Delete:

```bash
kubectl delete secret db-secret
```

---

# 30. Decoding a Secret

Get a specific key:

```bash
kubectl get secret db-secret \
  -o jsonpath='{.data.password}'
```

Decode:

```bash
kubectl get secret db-secret \
  -o jsonpath='{.data.password}' | base64 --decode
```

Another approach:

```bash
kubectl get secret db-secret -o yaml
```

Then decode the Base64 value manually.

---

# 31. ConfigMap and Secret Namespace Behavior

ConfigMaps and Secrets are **namespaced resources**.

For example:

```text
namespace: development
    |
    +-- ConfigMap: app-config
    +-- Secret: db-secret
```

and:

```text
namespace: production
    |
    +-- ConfigMap: app-config
    +-- Secret: db-secret
```

The same name can exist in different namespaces.

A Pod normally references resources in its own namespace.

---

# 32. `optional` References

You can make references optional.

Example:

```yaml
env:
  - name: OPTIONAL_VALUE
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: OPTIONAL_VALUE
        optional: true
```

For a Secret:

```yaml
env:
  - name: OPTIONAL_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: password
        optional: true
```

This can prevent the Pod from failing because the referenced resource/key is missing.

Use this only when the application can actually operate without the value.

---

# 33. ConfigMap Key Restrictions

When ConfigMap values are imported into environment variables, the keys need to be valid for the intended environment-variable usage.

For example:

```yaml
data:
  APP_NAME: myapp
```

works naturally.

But arbitrary configuration keys may not map cleanly to environment variable names.

For arbitrary keys/files, mounting the ConfigMap as a volume can be more appropriate.

---

# 34. `items` for Selective Mounting

You can mount only selected keys.

Example:

```yaml
volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
        - key: application.properties
          path: application.properties
```

Similarly for Secrets:

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
      items:
        - key: password
          path: db-password
```

---

# 35. Changing File Names

Suppose Secret contains:

```yaml
data:
  password: <encoded-value>
```

You can mount it as:

```yaml
items:
  - key: password
    path: database-password
```

The container sees:

```text
database-password
```

instead of:

```text
password
```

---

# 36. `subPath` Important Behavior

If a ConfigMap or Secret is mounted using a `subPath`, updates to the underlying ConfigMap/Secret are not propagated to that mounted file in the same way as a normal volume mount.

Example:

```yaml
volumeMounts:
  - name: config
    mountPath: /app/config.yaml
    subPath: config.yaml
```

This is an important operational detail when designing applications that need live configuration updates.

---

# 37. Immutable ConfigMaps

Kubernetes supports immutable ConfigMaps.

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

Once immutable, the data cannot be changed.

To change it, you generally create a new ConfigMap.

Benefits:

* Prevents accidental changes.
* Helps protect applications from unexpected configuration changes.
* Can reduce unnecessary watches in large clusters.

---

# 38. Immutable Secrets

Secrets can also be immutable.

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

An immutable Secret cannot have its data changed.

You can delete/recreate it if your operational process requires a new value.

---

# 39. ConfigMap `data` vs `binaryData`

ConfigMap:

```yaml
data:
```

is intended for UTF-8 text.

Example:

```yaml
data:
  config.txt: |
    hello world
```

`binaryData` is intended for binary content encoded as Base64.

```yaml
binaryData:
  file.bin: <base64-data>
```

---

# 40. Secret `data` vs `stringData`

Remember:

```text
Secret data
    |
    +-- data       → Base64-encoded values
    |
    +-- stringData → plaintext input; Kubernetes converts it
```

Example:

```yaml
stringData:
  username: admin
```

is easier to write than:

```yaml
data:
  username: YWRtaW4=
```

But neither should be confused with encryption.

---

# 41. Secret Volume Security Considerations

Secrets mounted into Pods are generally presented as files from a volume.

Important considerations:

* Use `readOnly: true` where appropriate.
* Limit access to the Pod/container.
* Avoid unnecessary Secret mounts.
* Use minimal RBAC permissions.
* Avoid logging Secret contents.
* Avoid printing environment variables containing secrets.
* Be careful with debugging commands such as `env`.
* Protect node access because a highly privileged attacker on a node can potentially access sensitive workload information.

---

# 42. Environment Variables vs Volume Mounts

## Environment variables

Advantages:

* Easy for applications to consume.
* Commonly supported by application frameworks.
* Simple configuration.

Disadvantages:

* Existing processes do not automatically receive changed environment variables.
* Values can accidentally appear in debugging output.
* Environment variables may be exposed through application/process diagnostics.

Example:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

---

## Volume mounts

Advantages:

* Configuration appears as files.
* Mounted ConfigMaps/Secrets can receive updates.
* Good for certificates and configuration files.

Disadvantages:

* Application must read files.
* Application may need reload logic.
* `subPath` has special update behavior.

---

# 43. ConfigMap and Secret in a Deployment

Recommended general structure:

```text
Deployment
    |
    +-- ConfigMap
    |     |
    |     +-- application configuration
    |
    +-- Secret
          |
          +-- credentials
          +-- tokens
          +-- certificates
```

Example:

```yaml
envFrom:
  - configMapRef:
      name: app-config

  - secretRef:
      name: app-secret
```

---

# 44. Common Mistakes

## Mistake 1: Storing passwords in ConfigMap

Bad:

```yaml
kind: ConfigMap
data:
  DB_PASSWORD: password123
```

Use a Secret instead.

---

## Mistake 2: Thinking Base64 means encryption

Incorrect:

```text
Base64 = encryption
```

Correct:

```text
Base64 = encoding
```

---

## Mistake 3: Committing Secret YAML to Git

Avoid:

```yaml
stringData:
  password: password123
```

in a normal source repository.

Even Base64-encoded values are not safe merely because they are encoded.

---

## Mistake 4: Expecting environment variables to update automatically

Changing:

```text
ConfigMap
```

does not update the environment variables inside already-running processes.

A restart/rollout is normally required.

---

## Mistake 5: Forgetting namespace

You may have:

```text
app-config
```

in:

```text
development
```

but your Pod is in:

```text
production
```

The Pod will not automatically use the ConfigMap from the other namespace.

---

## Mistake 6: Wrong key name

ConfigMap:

```yaml
data:
  DB_HOST: mysql
```

Pod:

```yaml
key: DATABASE_HOST
```

This causes a missing-key/reference problem.

---

## Mistake 7: Secret exists but Pod still fails

Check:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
Failed
NotFound
secret not found
configmap not found
```

---

# 45. Best Practices

## ConfigMap best practices

* Store only non-sensitive configuration.
* Use meaningful names.
* Keep configuration organized.
* Use namespaces appropriately.
* Avoid huge ConfigMaps.
* Use immutable ConfigMaps when configuration should never change.
* Use versioned names when appropriate.
* Use GitOps/configuration management for reproducibility.

---

## Secret best practices

* Never treat Base64 as encryption.
* Enable encryption at rest.
* Use RBAC to restrict access.
* Follow least privilege.
* Avoid storing Secrets directly in Git.
* Rotate credentials.
* Avoid logging Secret values.
* Mount Secrets only into workloads that need them.
* Use read-only mounts where possible.
* Consider external secret-management solutions for sensitive production workloads.
* Use immutable Secrets when appropriate.
* Audit access to Secrets.

---

# 46. Secret Rotation

Secret rotation means replacing an old credential with a new one.

Example:

```text
Old password
     ↓
Rotate database password
     ↓
Update Kubernetes Secret
     ↓
Restart/reload application
     ↓
Application uses new password
```

A mature production system should have a defined rotation strategy.

---

# 47. Secret Rotation with Versioned Names

A common pattern is:

```text
db-secret-v1
db-secret-v2
db-secret-v3
```

Deployment changes from:

```yaml
secretRef:
  name: db-secret-v1
```

to:

```yaml
secretRef:
  name: db-secret-v2
```

This can make rollouts and GitOps changes easier to track.

---

# 48. ConfigMap Versioning

Similarly:

```text
app-config-v1
app-config-v2
app-config-v3
```

can be used when you want configuration changes to create explicit Deployment revisions.

Another common approach is to calculate a checksum of the ConfigMap/Secret and place it in a Pod-template annotation so a Deployment rollout occurs when the referenced data changes.

Conceptually:

```yaml
annotations:
  config-checksum: "<hash>"
```

When configuration changes:

```text
ConfigMap changes
       ↓
checksum changes
       ↓
Pod template changes
       ↓
Deployment rollout
```

---

# 49. ConfigMap and Secret Are Kubernetes API Objects

They are not simply files.

Kubernetes stores them as resources in the Kubernetes control plane.

You can interact with them using:

```bash
kubectl
```

API clients can also read/manage them subject to authorization.

This is why:

```text
RBAC
Encryption at rest
Audit logging
Access control
```

matter.

---

# 50. How a Pod Gets Configuration

Typical flow:

```text
kubectl apply
     |
     v
Kubernetes API Server
     |
     v
ConfigMap / Secret
     |
     v
Pod specification references it
     |
     v
Kubelet prepares container
     |
     +---- Environment variables
     |
     +---- Mounted files
     |
     v
Application consumes configuration
```

---

# 51. Debugging ConfigMaps

Check resource:

```bash
kubectl get cm app-config -o yaml
```

Check Pod:

```bash
kubectl describe pod app-pod
```

Check environment:

```bash
kubectl exec app-pod -- env
```

Check mounted files:

```bash
kubectl exec app-pod -- ls -la /etc/config
```

Read a file:

```bash
kubectl exec app-pod -- cat /etc/config/application.properties
```

Be careful not to expose sensitive values when debugging production workloads.

---

# 52. Debugging Secrets

Check existence:

```bash
kubectl get secret db-secret
```

Check keys without unnecessarily exposing values:

```bash
kubectl describe secret db-secret
```

Check Secret YAML:

```bash
kubectl get secret db-secret -o yaml
```

Only decode values when you are authorized and genuinely need to inspect them:

```bash
kubectl get secret db-secret \
  -o jsonpath='{.data.password}' | base64 --decode
```

Check Pod events:

```bash
kubectl describe pod <pod-name>
```

---

# 53. Important `kubectl` Short Names

ConfigMap:

```text
configmaps
cm
```

Secret:

```text
secrets
secret
```

Examples:

```bash
kubectl get cm
```

```bash
kubectl get secret
```

---

# 54. Example: Complete Application Setup

## ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
data:
  APP_ENV: production
  LOG_LEVEL: info
  DB_HOST: mysql-service
  DB_PORT: "3306"
```

## Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: webapp-secret
type: Opaque
stringData:
  DB_USERNAME: appuser
  DB_PASSWORD: change-me
```

## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3

  selector:
    matchLabels:
      app: webapp

  template:
    metadata:
      labels:
        app: webapp

    spec:
      containers:
        - name: webapp
          image: my-webapp:1.0

          envFrom:
            - configMapRef:
                name: webapp-config

            - secretRef:
                name: webapp-secret
```

Apply:

```bash
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get pods
```

---

# 55. Important Difference: Secret Security Is Not Automatic

It is easy to misunderstand Kubernetes Secrets.

The word "Secret" does not mean:

```text
automatically encrypted
automatically hidden from administrators
automatically safe in Git
automatically inaccessible to Pods
```

A Secret is a Kubernetes resource specifically intended for sensitive data.

Security depends on configuration such as:

```text
RBAC
+
Encryption at rest
+
Network/control-plane security
+
Node security
+
Audit controls
+
Secret rotation
+
External secret management
```

---

# 56. Admission and Validation Considerations

Kubernetes validates resource structure and referenced fields, but application-specific configuration is generally the application's responsibility.

For example:

```yaml
data:
  DB_PORT: "hello"
```

may be structurally valid as a ConfigMap.

The application may later fail because it expects a number.

Therefore:

```text
Kubernetes validates the resource
        ≠
application configuration is necessarily correct
```

---

# 57. ConfigMap/Secret and Helm

Helm charts commonly template ConfigMaps and Secrets.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "myapp.fullname" . }}-config
data:
  APP_ENV: {{ .Values.app.environment | quote }}
```

Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "myapp.fullname" . }}-secret
type: Opaque
stringData:
  username: {{ .Values.database.username | quote }}
```

Be careful:

**Putting a plaintext password in Helm values does not automatically make it secure.**

Your Git repository and rendered manifests still need appropriate protection.

---

# 58. ConfigMap/Secret and Kubernetes Operators

Operators/controllers can dynamically create and manage ConfigMaps and Secrets.

Example flow:

```text
Custom Resource
      ↓
Operator
      ↓
Secret / ConfigMap
      ↓
Deployment
```

This is common in larger Kubernetes platforms.

---

# 59. ConfigMap/Secret and GitOps

In GitOps environments:

```text
Git repository
      ↓
GitOps controller
      ↓
Kubernetes
      ↓
ConfigMap / Secret
```

Plaintext Secrets should generally not be committed to Git.

Common approaches include:

* SOPS
* Sealed Secrets
* External Secrets
* Cloud secret managers
* Vault-based solutions

The correct solution depends on the organization's security model.

---

# 60. External Secret Management

For highly sensitive environments, Kubernetes Secrets can be synchronized from an external secret manager.

Conceptual architecture:

```text
External Secret Manager
          |
          v
External Secrets Controller
          |
          v
Kubernetes Secret
          |
          v
Pod
```

Advantages can include:

* Centralized secret management
* Rotation support
* Access auditing
* Cloud IAM integration
* Reduced plaintext secret storage in Git

---

# 61. TLS Secret Example

Generate/use a TLS Secret:

```bash
kubectl create secret tls my-tls \
  --cert=tls.crt \
  --key=tls.key
```

Check:

```bash
kubectl get secret my-tls
```

Typical structure:

```yaml
type: kubernetes.io/tls
data:
  tls.crt: <base64>
  tls.key: <base64>
```

The private key is sensitive and should be protected carefully.

---

# 62. Image Pull Secret Example

A private registry may require authentication.

Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: registry-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-data>
```

Pod:

```yaml
spec:
  imagePullSecrets:
    - name: registry-secret
```

Deployment:

```yaml
spec:
  template:
    spec:
      imagePullSecrets:
        - name: registry-secret
      containers:
        - name: app
          image: private-registry.example/app:1.0
```

---

# 63. What Happens if a Referenced ConfigMap Does Not Exist?

For a required reference, the Pod may fail to start correctly because the required configuration resource cannot be obtained.

Check:

```bash
kubectl describe pod <pod-name>
```

Look at:

```text
Events
```

Typical causes:

```text
ConfigMap not found
Wrong namespace
Wrong ConfigMap name
Wrong key
```

---

# 64. What Happens if a Referenced Secret Does Not Exist?

Similar problem.

Check:

```bash
kubectl describe pod <pod-name>
```

Verify:

```bash
kubectl get secret <secret-name>
```

Then verify the namespace:

```bash
kubectl get secret <secret-name> -n <namespace>
```

---

# 65. ConfigMap/Secret and Pod Lifecycle

Important mental model:

```text
ConfigMap/Secret
       |
       | referenced by
       v
Pod
       |
       v
Container process
```

The application does not directly query Kubernetes every time it reads an environment variable.

For environment variables, values are populated when the container starts.

For volume mounts, Kubernetes manages the mounted representation.

---

# 66. Security Checklist

Before deploying Secrets to production, ask:

* [ ] Is this value actually sensitive?
* [ ] If yes, is it stored in a Secret rather than ConfigMap?
* [ ] Is encryption at rest configured?
* [ ] Are RBAC permissions least-privilege?
* [ ] Is access to Secrets audited?
* [ ] Are Secrets excluded from Git?
* [ ] Are logs free of sensitive values?
* [ ] Are Secret mounts limited to required containers?
* [ ] Are credentials rotated?
* [ ] Is `readOnly` used where appropriate?
* [ ] Is an external secret manager appropriate?
* [ ] Are old credentials revoked after rotation?
* [ ] Are production and development Secrets separated?
* [ ] Are namespaces correctly configured?

---

# 67. Quick Command Cheat Sheet

## ConfigMap

```bash
# Create
kubectl create configmap app-config --from-literal=ENV=prod

# List
kubectl get cm

# Inspect
kubectl describe cm app-config

# YAML
kubectl get cm app-config -o yaml

# Edit
kubectl edit cm app-config

# Delete
kubectl delete cm app-config
```

## Secret

```bash
# Create
kubectl create secret generic app-secret \
  --from-literal=username=admin \
  --from-literal=password=password123

# List
kubectl get secrets

# Inspect metadata/keys
kubectl describe secret app-secret

# YAML
kubectl get secret app-secret -o yaml

# Edit
kubectl edit secret app-secret

# Delete
kubectl delete secret app-secret
```

## Restart Deployment

```bash
kubectl rollout restart deployment <deployment-name>
```

## Check rollout

```bash
kubectl rollout status deployment <deployment-name>
```

---

# 68. Interview Questions and Answers

## Q1. What is a ConfigMap?

A ConfigMap stores non-sensitive configuration data in Kubernetes.

---

## Q2. What is a Secret?

A Secret is a Kubernetes resource designed to hold sensitive information such as passwords, tokens, keys, and certificates.

---

## Q3. Is Kubernetes Secret encrypted?

Not merely because it is a Secret. Secret values are Base64-encoded in the normal API representation. Encryption at rest must be configured for stronger protection.

---

## Q4. Is Base64 encryption?

No.

```text
Base64 = encoding
Encryption = cryptographic protection
```

---

## Q5. Can ConfigMap store passwords?

Technically Kubernetes can store arbitrary data in a ConfigMap, but passwords and other sensitive values should not be stored there. Use Secrets or an appropriate external secret-management system.

---

## Q6. Can Secrets be used as environment variables?

Yes.

```yaml
env:
  - name: PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: password
```

---

## Q7. Can ConfigMaps be mounted as files?

Yes.

---

## Q8. Can Secrets be mounted as files?

Yes.

---

## Q9. What happens when a ConfigMap used as an environment variable changes?

Existing containers do not automatically receive the new environment-variable value. A restart/recreation is normally needed.

---

## Q10. What happens when a mounted ConfigMap changes?

Kubernetes can update the mounted volume contents after the resource changes, subject to the normal update mechanism. The application must read/reload the changed files if it needs the new values.

---

## Q11. What is `stringData`?

`stringData` allows Secret values to be supplied as strings instead of manually Base64-encoding them.

---

## Q12. What is `data` in a Secret?

`data` contains Secret values represented as Base64-encoded data.

---

## Q13. What is `envFrom`?

`envFrom` imports all appropriate keys from a ConfigMap or Secret as environment variables.

Example:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

---

## Q14. What is `configMapKeyRef`?

It references one specific key from a ConfigMap.

---

## Q15. What is `secretKeyRef`?

It references one specific key from a Secret.

---

## Q16. Are ConfigMaps namespaced?

Yes.

---

## Q17. Are Secrets namespaced?

Yes.

---

## Q18. What is an immutable ConfigMap?

A ConfigMap with:

```yaml
immutable: true
```

Its data cannot be changed after creation.

---

## Q19. What is an immutable Secret?

A Secret with:

```yaml
immutable: true
```

Its data cannot be changed after creation.

---

## Q20. What is `kubernetes.io/tls`?

It is the Secret type commonly used for TLS certificates and private keys.

---

## Q21. What is `kubernetes.io/dockerconfigjson`?

It is the Secret type commonly used for Docker/container registry authentication configuration.

---

## Q22. How do you decode a Secret?

For example:

```bash
kubectl get secret app-secret \
  -o jsonpath='{.data.password}' | base64 --decode
```

Only do this when authorized because it exposes sensitive data.

---

# 69. Final Mental Model

Remember these four concepts:

```text
                Kubernetes Configuration
                         |
             +-----------+-----------+
             |                       |
             v                       v
        ConfigMap                 Secret
             |                       |
             v                       v
     Non-sensitive data       Sensitive data
             |                       |
             +-----------+-----------+
                         |
                         v
                       Pod
                         |
             +-----------+-----------+
             |                       |
             v                       v
       Environment             Volume/File
       Variables                Mount
```

### ConfigMap

```text
Non-sensitive configuration
```

Examples:

```text
APP_ENV
LOG_LEVEL
DB_HOST
DB_PORT
FEATURE_FLAG
```

### Secret

```text
Sensitive configuration
```

Examples:

```text
PASSWORD
TOKEN
API_KEY
PRIVATE_KEY
TLS_CERTIFICATE
REGISTRY_CREDENTIALS
```

### Most important security rule

```text
Base64 ≠ Encryption
```

### Most important operational rule

```text
Environment-variable changes
do not automatically update
already-running container processes.
```

### Most important design rule

```text
Separate application code
from configuration,
and separate ordinary configuration
from sensitive credentials.
```

---

# 70. One-Page Revision

```text
CONFIGMAP
---------
Purpose:
    Non-sensitive configuration

Create:
    kubectl create configmap ...

Use as:
    Environment variables
    Volume/files

Fields:
    data
    binaryData

Namespace:
    Yes

Typical examples:
    APP_ENV
    LOG_LEVEL
    DB_HOST
    DB_PORT


SECRET
------
Purpose:
    Sensitive configuration

Create:
    kubectl create secret generic ...

Use as:
    Environment variables
    Volume/files

Fields:
    data
    stringData
    type

Namespace:
    Yes

Typical examples:
    Password
    API token
    TLS key
    Registry credentials


SECURITY
--------
Base64:
    Encoding, NOT encryption

Important:
    RBAC
    Encryption at rest
    Least privilege
    Rotation
    Auditing
    Avoid Git commits
    Avoid logging secrets


UPDATES
-------
Environment variable:
    Restart/recreate workload to get new value

Mounted volume:
    Kubernetes can propagate changes;
    application must reload/read them


KEY COMMANDS
------------
kubectl get cm
kubectl describe cm <name>
kubectl get cm <name> -o yaml

kubectl get secrets
kubectl describe secret <name>
kubectl get secret <name> -o yaml

kubectl rollout restart deployment <name>
```

---

# 71. Final Exam/Interview Summary

If you remember only these points, remember:

1. **ConfigMap stores non-sensitive configuration.**
2. **Secret is intended for sensitive configuration.**
3. **Both are namespaced Kubernetes resources.**
4. **Both can be consumed as environment variables.**
5. **Both can be mounted as volumes.**
6. **`configMapKeyRef` selects one ConfigMap key.**
7. **`secretKeyRef` selects one Secret key.**
8. **`envFrom` imports multiple/all suitable keys.**
9. **Secret `data` uses Base64 representation.**
10. **Base64 is not encryption.**
11. **`stringData` lets you specify Secret values as strings.**
12. **ConfigMap/Secret environment variables do not automatically update inside running processes.**
13. **Mounted resources can receive updates, but applications need appropriate reload behavior.**
14. **Secrets require proper RBAC and encryption-at-rest configuration for stronger security.**
15. **Do not commit plaintext or merely Base64-encoded Secrets to Git.**
16. **Use external secret-management solutions when they better fit production security requirements.**
17. **Use immutable ConfigMaps/Secrets when preventing accidental modification is valuable.**
18. **Use `kubernetes.io/tls` for TLS Secrets.**
19. **Use `kubernetes.io/dockerconfigjson` for registry authentication.**
20. **Always follow least privilege when granting access to Secrets.**

---

**End of Notes**

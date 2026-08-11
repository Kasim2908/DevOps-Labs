# ☸️ Kubernetes Ingress - Labs

This file contains hands-on Kubernetes Ingress labs for practicing:

* Ingress Controller
* IngressClass
* Host-based routing
* Path-based routing
* Multiple Services
* TLS/HTTPS
* Ingress troubleshooting
* Minikube Ingress

---

# Lab Environment

These labs use:

```text
Kubernetes
Minikube
kubectl
NGINX Ingress Controller
```

Check Kubernetes:

```bash
kubectl version --client
```

Check Minikube:

```bash
minikube version
```

Check cluster:

```bash
kubectl get nodes
```

Expected:

```text
NAME       STATUS   ROLES           AGE
minikube   Ready    control-plane   ...
```

---

# Lab 1 - Enable Ingress on Minikube

Minikube provides an Ingress addon.

Check available addons:

```bash
minikube addons list
```

Enable Ingress:

```bash
minikube addons enable ingress
```

Check Ingress Controller:

```bash
kubectl get pods -n ingress-nginx
```

Expected output should show the Ingress Controller Pod running.

Example:

```text
NAME                                        READY   STATUS
ingress-nginx-controller-xxxxx              1/1     Running
```

---

# Lab 2 - Verify IngressClass

Check available IngressClasses:

```bash
kubectl get ingressclass
```

Example:

```text
NAME    CONTROLLER
nginx   k8s.io/ingress-nginx
```

Describe the IngressClass:

```bash
kubectl describe ingressclass nginx
```

The IngressClass tells Kubernetes which Ingress Controller should process an Ingress resource.

---

# Lab 3 - Deploy NGINX Application

Create:

```text
nginx-deployment.yaml
```

Content:

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

Apply:

```bash
kubectl apply -f nginx-deployment.yaml
```

Check:

```bash
kubectl get deployment
```

Check Pods:

```bash
kubectl get pods -o wide
```

Expected:

```text
nginx-deployment-xxxxx   1/1   Running
nginx-deployment-yyyyy   1/1   Running
```

---

# Lab 4 - Create ClusterIP Service

Create:

```text
nginx-service.yaml
```

Content:

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

Apply:

```bash
kubectl apply -f nginx-service.yaml
```

Check:

```bash
kubectl get svc
```

Expected:

```text
NAME            TYPE        CLUSTER-IP      PORT(S)
nginx-service   ClusterIP   10.96.x.x       80/TCP
```

---

# Lab 5 - Verify Service Endpoints

Check:

```bash
kubectl get endpoints nginx-service
```

You should see the backend Pod IPs.

Example:

```text
NAME            ENDPOINTS
nginx-service   10.244.0.5:80,10.244.0.6:80
```

Also check EndpointSlices:

```bash
kubectl get endpointslice
```

---

# Lab 6 - Create Basic Ingress

Create:

```text
nginx-ingress.yaml
```

Content:

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
kubectl apply -f nginx-ingress.yaml
```

Check:

```bash
kubectl get ingress
```

Example:

```text
NAME            CLASS   HOSTS         ADDRESS
nginx-ingress   nginx   nginx.local   ...
```

---

# Lab 7 - Inspect the Ingress

Run:

```bash
kubectl describe ingress nginx-ingress
```

Check:

```text
Rules
Host
Path
Backend
Address
IngressClass
```

You should see:

```text
Host: nginx.local
Path: /
Backend: nginx-service:80
```

---

# Lab 8 - Test Ingress Using Minikube IP

Get Minikube IP:

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

You can test the Ingress by sending the correct Host header:

```bash
curl -H "Host: nginx.local" http://$(minikube ip)
```

Expected response:

```text
<!DOCTYPE html>
<html>
...
<title>Welcome to nginx!</title>
...
</html>
```

---

# Lab 9 - Configure Local Hostname

For convenient browser testing, map:

```text
nginx.local
```

to the Minikube IP.

First:

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

Then add an entry to your hosts file.

Linux/WSL:

```text
/etc/hosts
```

Windows:

```text
C:\Windows\System32\drivers\etc\hosts
```

Add:

```text
192.168.49.2 nginx.local
```

Now test:

```bash
curl http://nginx.local
```

Or open:

```text
http://nginx.local
```

in your browser.

---

# Lab 10 - Host-Based Routing

Now create two applications:

```text
frontend.local
api.local
```

Architecture:

```text
                  Ingress
                     │
          ┌──────────┴──────────┐
          │                     │
   frontend.local            api.local
          │                     │
          ▼                     ▼
Frontend Service           Backend Service
          │                     │
        Pods                   Pods
```

---

# Lab 11 - Deploy Frontend

Create:

```text
frontend.yaml
```

Content:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: frontend

spec:
  replicas: 2

  selector:
    matchLabels:
      app: frontend

  template:
    metadata:
      labels:
        app: frontend

    spec:
      containers:
        - name: frontend
          image: nginx:latest

          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service

metadata:
  name: frontend-service

spec:
  selector:
    app: frontend

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f frontend.yaml
```

Check:

```bash
kubectl get pods
kubectl get svc
```

---

# Lab 12 - Deploy Backend

Create:

```text
backend.yaml
```

Content:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend

spec:
  replicas: 2

  selector:
    matchLabels:
      app: backend

  template:
    metadata:
      labels:
        app: backend

    spec:
      containers:
        - name: backend
          image: hashicorp/http-echo:1.0

          args:
            - "-text=Backend Application"

          ports:
            - containerPort: 5678

---
apiVersion: v1
kind: Service

metadata:
  name: backend-service

spec:
  selector:
    app: backend

  ports:
    - port: 80
      targetPort: 5678
```

Apply:

```bash
kubectl apply -f backend.yaml
```

Check:

```bash
kubectl get pods
kubectl get svc
```

---

# Lab 13 - Host-Based Ingress

Create:

```text
host-routing.yaml
```

Content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: host-routing

spec:
  ingressClassName: nginx

  rules:

    - host: frontend.local

      http:
        paths:
          - path: /
            pathType: Prefix

            backend:
              service:
                name: frontend-service
                port:
                  number: 80

    - host: api.local

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

Apply:

```bash
kubectl apply -f host-routing.yaml
```

Check:

```bash
kubectl get ingress
```

---

# Lab 14 - Test Host-Based Routing

Get Minikube IP:

```bash
minikube ip
```

Test frontend:

```bash
curl -H "Host: frontend.local" http://$(minikube ip)
```

Test backend:

```bash
curl -H "Host: api.local" http://$(minikube ip)
```

Expected:

```text
frontend.local
      ↓
Frontend Service
```

```text
api.local
      ↓
Backend Service
      ↓
Backend Application
```

---

# Lab 15 - Path-Based Routing

Now route multiple applications using one hostname.

We will use:

```text
example.local/
example.local/api
```

Architecture:

```text
                    example.local
                          │
                       Ingress
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
             /                        /api
             │                         │
             ▼                         ▼
      Frontend Service          Backend Service
```

---

# Lab 16 - Create Path-Based Ingress

Create:

```text
path-routing.yaml
```

Content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: path-routing

spec:
  ingressClassName: nginx

  rules:

    - host: example.local

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

Apply:

```bash
kubectl apply -f path-routing.yaml
```

Check:

```bash
kubectl get ingress
```

---

# Lab 17 - Test Path-Based Routing

Test frontend:

```bash
curl -H "Host: example.local" http://$(minikube ip)/
```

Test backend:

```bash
curl -H "Host: example.local" http://$(minikube ip)/api
```

Traffic:

```text
example.local/
      ↓
Frontend Service
```

```text
example.local/api
      ↓
Backend Service
```

---

# Lab 18 - Exact Path Matching

Create an Ingress rule:

```yaml
paths:
  - path: /login
    pathType: Exact

    backend:
      service:
        name: frontend-service
        port:
          number: 80
```

This matches:

```text
/login
```

But does not generally match:

```text
/login/
/login/user
```

Apply:

```bash
kubectl apply -f ingress.yaml
```

Test:

```bash
curl -H "Host: example.local" \
  http://$(minikube ip)/login
```

---

# Lab 19 - Prefix Path Matching

Create:

```yaml
paths:
  - path: /api
    pathType: Prefix

    backend:
      service:
        name: backend-service
        port:
          number: 80
```

This can match:

```text
/api
/api/
/api/users
/api/products
/api/orders
```

Test:

```bash
curl -H "Host: example.local" \
  http://$(minikube ip)/api/users
```

---

# Lab 20 - TLS/HTTPS Ingress

For this lab we will create a self-signed certificate for testing.

Create a directory:

```bash
mkdir ingress-tls
cd ingress-tls
```

Generate a private key:

```bash
openssl genrsa -out tls.key 2048
```

Generate a certificate:

```bash
openssl req \
  -x509 \
  -new \
  -nodes \
  -key tls.key \
  -sha256 \
  -days 365 \
  -out tls.crt \
  -subj "/CN=secure.local"
```

---

# Lab 21 - Create TLS Secret

Create a Kubernetes TLS Secret:

```bash
kubectl create secret tls secure-tls \
  --cert=tls.crt \
  --key=tls.key
```

Check:

```bash
kubectl get secret secure-tls
```

Expected:

```text
NAME         TYPE                DATA
secure-tls   kubernetes.io/tls   2
```

---

# Lab 22 - Create HTTPS Ingress

Create:

```text
tls-ingress.yaml
```

Content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: secure-ingress

spec:
  ingressClassName: nginx

  tls:
    - hosts:
        - secure.local

      secretName: secure-tls

  rules:

    - host: secure.local

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

Apply:

```bash
kubectl apply -f tls-ingress.yaml
```

Check:

```bash
kubectl get ingress
```

---

# Lab 23 - Test HTTPS

For a self-signed certificate, use:

```bash
curl -k \
  -H "Host: secure.local" \
  https://$(minikube ip)/
```

`-k` tells curl to allow the self-signed certificate.

Expected:

```text
NGINX response
```

---

# Lab 24 - Inspect TLS Configuration

Run:

```bash
kubectl describe ingress secure-ingress
```

Look for:

```text
TLS:
  secure.local
  secure-tls
```

Check Secret:

```bash
kubectl describe secret secure-tls
```

---

# Lab 25 - Test Ingress From Browser

Add the hostname to your hosts file:

```text
<MINIKUBE-IP> secure.local
```

For example:

```text
192.168.49.2 secure.local
```

Then open:

```text
https://secure.local
```

Because the certificate is self-signed, the browser may display a certificate warning.

This is expected for this lab.

---

# Lab 26 - Ingress Troubleshooting

Check Ingress:

```bash
kubectl get ingress
```

Detailed information:

```bash
kubectl describe ingress <ingress>
```

Check IngressClass:

```bash
kubectl get ingressclass
```

Check Controller:

```bash
kubectl get pods -n ingress-nginx
```

Check Controller logs:

```bash
kubectl logs -n ingress-nginx \
  <controller-pod>
```

---

# Lab 27 - Troubleshoot No Ingress Address

If:

```text
ADDRESS
<none>
```

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

Check Controller:

```bash
kubectl get pods -n ingress-nginx
```

Check Controller logs:

```bash
kubectl logs -n ingress-nginx \
  <controller-pod>
```

---

# Lab 28 - Troubleshoot No Backend Endpoints

Check Service:

```bash
kubectl get svc
```

Check Endpoints:

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

If they do not match, the Service will not have the expected backend endpoints.

---

# Lab 29 - Troubleshoot Wrong Service Port

Check:

```bash
kubectl get svc <service>
```

Example:

```text
PORT(S)
80/TCP
```

Ingress must reference the Service port:

```yaml
backend:
  service:
    name: backend-service
    port:
      number: 80
```

Remember:

```text
Ingress
   ↓
Service port
   ↓
Service targetPort
   ↓
Pod port
```

---

# Lab 30 - Troubleshoot Wrong Host

Suppose the Ingress has:

```yaml
host: app.local
```

But you send:

```bash
curl -H "Host: api.local" \
  http://$(minikube ip)
```

The rule may not match.

Check configured hosts:

```bash
kubectl describe ingress <ingress>
```

Test using the correct Host header:

```bash
curl -H "Host: app.local" \
  http://$(minikube ip)
```

---

# Lab 31 - Troubleshoot DNS

Test hostname resolution:

```bash
nslookup app.local
```

For local testing, verify your hosts file.

Linux/WSL:

```text
/etc/hosts
```

Windows:

```text
C:\Windows\System32\drivers\etc\hosts
```

Example:

```text
192.168.49.2 app.local
```

---

# Lab 32 - Check Ingress Controller Logs

Find Controller Pod:

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

Logs are useful for identifying:

* Configuration errors
* Backend connection failures
* TLS problems
* Routing issues

---

# Lab 33 - Complete Ingress Architecture

At this point, your architecture should look like:

```text
                         Internet
                            │
                            ▼
                    Ingress Controller
                            │
                            ▼
                         Ingress
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
          Frontend        Backend         Admin
           Service         Service        Service
             │              │              │
             ▼              ▼              ▼
           Pods            Pods            Pods
             ▲              ▲              ▲
             └──────────────┼──────────────┘
                            │
                       Deployments
```

---

# Lab 34 - Complete Host + Path Routing

Use:

```text
app.local
api.local
```

And:

```text
app.local/
app.local/admin
api.local/
api.local/users
```

Architecture:

```text
                         Internet
                            │
                            ▼
                         Ingress
                            │
             ┌──────────────┴──────────────┐
             │                             │
          app.local                     api.local
             │                             │
        ┌────┴────┐                   ┌────┴────┐
        │         │                   │         │
        ▼         ▼                   ▼         ▼
        /       /admin                /       /users
        │         │                   │         │
        ▼         ▼                   ▼         ▼
    Frontend    Admin              Backend    API
    Service     Service            Service    Service
```

This demonstrates both:

```text
Host-based routing
+
Path-based routing
```

---

# Lab 35 - Verify Complete Setup

Run:

```bash
kubectl get deployments
```

```bash
kubectl get pods -o wide
```

```bash
kubectl get svc
```

```bash
kubectl get endpoints
```

```bash
kubectl get endpointslice
```

```bash
kubectl get ingress
```

```bash
kubectl get ingressclass
```

Finally:

```bash
kubectl get all
```

---

# Lab 36 - Troubleshooting Checklist

When Ingress is not working, follow this order:

```text
1. Is the Ingress Controller running?
             ↓
2. Does the IngressClass exist?
             ↓
3. Is ingressClassName correct?
             ↓
4. Does the Ingress rule have the correct host?
             ↓
5. Is the path correct?
             ↓
6. Does the Service exist?
             ↓
7. Is the Service port correct?
             ↓
8. Does the Service have endpoints?
             ↓
9. Are Pods Running and Ready?
             ↓
10. Does DNS/hosts configuration work?
             ↓
11. Is TLS configured correctly?
             ↓
12. Check Ingress Controller logs
```

---

# Lab 37 - Useful Commands Cheat Sheet

## Ingress

```bash
kubectl get ingress
```

```bash
kubectl describe ingress <ingress>
```

```bash
kubectl get ingress <ingress> -o yaml
```

---

## IngressClass

```bash
kubectl get ingressclass
```

```bash
kubectl describe ingressclass <class>
```

---

## Ingress Controller

```bash
kubectl get pods -n ingress-nginx
```

```bash
kubectl get svc -n ingress-nginx
```

```bash
kubectl logs -n ingress-nginx <pod>
```

---

## Services

```bash
kubectl get svc
```

```bash
kubectl describe svc <service>
```

```bash
kubectl get endpoints <service>
```

```bash
kubectl get endpointslice
```

---

## Pods

```bash
kubectl get pods
```

```bash
kubectl get pods --show-labels
```

```bash
kubectl get pods -o wide
```

---

## Testing

```bash
curl -H "Host: example.local" \
  http://$(minikube ip)
```

HTTPS:

```bash
curl -k \
  -H "Host: secure.local" \
  https://$(minikube ip)
```

DNS:

```bash
nslookup example.local
```

---

# Lab 38 - Cleanup

Delete Ingress:

```bash
kubectl delete ingress --all
```

Delete Services:

```bash
kubectl delete svc --all
```

Delete Deployments:

```bash
kubectl delete deployment --all
```

Delete TLS Secret:

```bash
kubectl delete secret secure-tls
```

Disable Minikube Ingress addon if you no longer need it:

```bash
minikube addons disable ingress
```

---

# 🎯 Final Ingress Lab Architecture

```text
                              Internet
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ Ingress         │
                         │ Controller      │
                         └────────┬────────┘
                                  │
                         Ingress Rules
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
          Host Rules          Path Rules            TLS
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  │
                                  ▼
                             Services
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
                    ▼             ▼             ▼
                 Frontend      Backend        Admin
                   Pods          Pods           Pods
                    ▲             ▲             ▲
                    └─────────────┼─────────────┘
                                  │
                             Deployments
```

# 🔥 What You Should Be Able to Do After These Labs

By completing these labs, you should be able to:

* Enable an Ingress Controller on Minikube.
* Understand `Ingress` vs `Ingress Controller`.
* Create an `IngressClass`.
* Create a basic Ingress.
* Route traffic to a Service.
* Configure host-based routing.
* Configure path-based routing.
* Understand `Prefix` and `Exact`.
* Configure multiple hosts.
* Configure multiple paths.
* Configure TLS.
* Create TLS Secrets.
* Test HTTPS traffic.
* Configure local hostname resolution.
* Troubleshoot Ingress routing.
* Troubleshoot Services behind Ingress.
* Read Ingress Controller logs.
* Build a production-style Ingress architecture.

# ☸️ Kubernetes Services - Labs

## Lab Overview

In this lab, we will work with Kubernetes Services and learn how to expose Pods using different Service types.

We will practice:

* Creating Pods
* Creating a Deployment
* Creating a ClusterIP Service
* Understanding `port` and `targetPort`
* Testing Service connectivity
* Service DNS
* Creating a NodePort Service
* Accessing NodePort Services
* Creating a LoadBalancer Service
* Using `minikube tunnel`
* Inspecting Endpoints
* Inspecting EndpointSlices
* Troubleshooting Services
* Scaling Pods and observing Service load balancing
* Deleting and recreating Pods
* Understanding why Services provide stable networking

---

# Lab Environment

We will use:

```text
Kubernetes
kubectl
Minikube
Docker
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

# Lab 1 - Create an NGINX Pod

First, create a simple NGINX Pod.

Create:

```text
nginx-pod.yaml
```

Add:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod

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
kubectl apply -f nginx-pod.yaml
```

Check the Pod:

```bash
kubectl get pods
```

Expected:

```text
NAME         READY   STATUS    RESTARTS   AGE
nginx-pod    1/1     Running   0          ...
```

---

# Lab 2 - Verify Pod Labels

Check the labels:

```bash
kubectl get pods --show-labels
```

Expected:

```text
NAME         READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod    1/1     Running   0          ...   app=nginx
```

The Service will use:

```yaml
selector:
  app: nginx
```

to find this Pod.

---

# Lab 3 - Check the Pod IP

Run:

```bash
kubectl get pod nginx-pod -o wide
```

Example:

```text
NAME         READY   STATUS    IP            NODE
nginx-pod    1/1     Running   10.244.0.5    minikube
```

The Pod IP is:

```text
10.244.0.5
```

Remember:

> Pod IP addresses are temporary and can change when Pods are recreated.

This is one of the main reasons we use Services.

---

# Lab 4 - Create a ClusterIP Service

Create:

```text
nginx-service.yaml
```

Add:

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
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
nginx-service   ClusterIP   10.96.20.10     <none>        80/TCP
```

---

# Lab 5 - Understand the Service Configuration

Our Service contains:

```yaml
ports:
  - port: 80
    targetPort: 80
```

This means:

```text
Client
   │
   │ :80
   ▼
Service :80
   │
   │ :80
   ▼
Pod :80
```

The Service selector is:

```yaml
selector:
  app: nginx
```

The Pod has:

```yaml
labels:
  app: nginx
```

Therefore the Service selects the Pod.

---

# Lab 6 - Describe the Service

Run:

```bash
kubectl describe svc nginx-service
```

Look for:

```text
Name:
Type:
IP:
Port:
TargetPort:
Endpoints:
Selector:
```

Example:

```text
Name:              nginx-service
Type:              ClusterIP
IP:                10.96.20.10
Port:              80
TargetPort:        80
Endpoints:         10.244.0.5:80
Selector:          app=nginx
```

The important part is:

```text
Endpoints:
10.244.0.5:80
```

This confirms that the Service has discovered the Pod.

---

# Lab 7 - Check Endpoints

Run:

```bash
kubectl get endpoints nginx-service
```

Expected:

```text
NAME            ENDPOINTS
nginx-service   10.244.0.5:80
```

The endpoint represents the backend Pod.

Architecture:

```text
Service
   │
   ▼
10.244.0.5:80
   │
   ▼
NGINX Pod
```

---

# Lab 8 - Check EndpointSlices

Run:

```bash
kubectl get endpointslice
```

You should see an EndpointSlice associated with the Service.

Filter it:

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=nginx-service
```

Example:

```text
NAME                    ADDRESSTYPE   PORTS   ENDPOINTS
nginx-service-abc123    IPv4          80      10.244.0.5
```

---

# Lab 9 - Test the Service from Inside the Cluster

A ClusterIP Service is normally accessible from inside the cluster.

Create a temporary test Pod:

```bash
kubectl run curl-pod \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Inside the Pod:

```bash
curl http://nginx-service
```

Expected:

```text
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

The request flow is:

```text
curl-pod
    │
    ▼
nginx-service
    │
    ▼
nginx-pod
```

Exit:

```bash
exit
```

---

# Lab 10 - Test Using Service DNS

Kubernetes automatically creates DNS records for Services.

From inside the test Pod:

```bash
kubectl run curl-pod \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Test:

```bash
curl http://nginx-service
```

You can also use the full DNS name:

```bash
curl http://nginx-service.default.svc.cluster.local
```

The general format is:

```text
<service-name>.<namespace>.svc.cluster.local
```

For our Service:

```text
nginx-service.default.svc.cluster.local
```

Exit:

```bash
exit
```

---

# Lab 11 - Test Service DNS Resolution

Create a temporary BusyBox Pod:

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  -it --rm \
  -- sh
```

Run:

```bash
nslookup nginx-service
```

Expected output will show that the Service name resolves to its ClusterIP.

You can also test:

```bash
nslookup nginx-service.default.svc.cluster.local
```

Exit:

```bash
exit
```

---

# Lab 12 - Port Forward the Service

You can access the Service from your local machine using port forwarding.

Run:

```bash
kubectl port-forward service/nginx-service 8080:80
```

Expected:

```text
Forwarding from 127.0.0.1:8080 -> 80
```

Open:

```text
http://localhost:8080
```

You should see the NGINX welcome page.

Stop port forwarding:

```text
Ctrl + C
```

---

# Lab 13 - Create a Deployment

Services are commonly used with Deployments.

Create:

```text
nginx-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

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
kubectl get pods
```

Expected:

```text
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-xxxxx-xxxxx        1/1     Running   0          ...
nginx-deployment-xxxxx-yyyyy        1/1     Running   0          ...
nginx-deployment-xxxxx-zzzzz        1/1     Running   0          ...
```

---

# Lab 14 - Update the Service

The existing Service selector:

```yaml
selector:
  app: nginx
```

matches all three Deployment Pods.

Check:

```bash
kubectl get endpoints nginx-service
```

You should now see multiple endpoints.

Example:

```text
NAME            ENDPOINTS
nginx-service   10.244.0.5:80,10.244.0.6:80,10.244.0.7:80
```

Architecture:

```text
                  nginx-service
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
        Pod 1        Pod 2        Pod 3
       :80           :80           :80
```

---

# Lab 15 - Test Service Load Distribution

Run a temporary test Pod:

```bash
kubectl run curl-pod \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Test the Service:

```bash
curl http://nginx-service
```

The Service can distribute traffic among its available backend endpoints.

Exit:

```bash
exit
```

---

# Lab 16 - Scale the Deployment

Scale from 3 Pods to 5:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Check:

```bash
kubectl get pods
```

Expected:

```text
5 Pods
```

Now check:

```bash
kubectl get endpoints nginx-service
```

You should see more backend endpoints.

Architecture:

```text
                 Service
                    │
       ┌────────────┼────────────┐
       │            │            │
       ▼            ▼            ▼
     Pod 1        Pod 2        Pod 3
       │
       ├── Pod 4
       │
       └── Pod 5
```

---

# Lab 17 - Delete a Pod

List Pods:

```bash
kubectl get pods
```

Choose one Pod and delete it:

```bash
kubectl delete pod <pod-name>
```

Immediately check:

```bash
kubectl get pods
```

Because the Deployment manages the Pods, Kubernetes creates a replacement Pod.

Check:

```bash
kubectl get pods
```

You should again have:

```text
5 Pods
```

---

# Lab 18 - Observe Service Endpoints After Pod Replacement

Before deleting a Pod:

```bash
kubectl get endpoints nginx-service
```

Note the Pod IPs.

Delete one Pod:

```bash
kubectl delete pod <pod-name>
```

Check:

```bash
kubectl get endpoints nginx-service
```

The old Pod endpoint disappears and the replacement Pod eventually appears.

This demonstrates why applications should use the Service instead of directly using Pod IP addresses.

---

# Lab 19 - Create a NodePort Service

Create:

```text
nginx-nodeport.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-nodeport

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Apply:

```bash
kubectl apply -f nginx-nodeport.yaml
```

Check:

```bash
kubectl get svc
```

Expected:

```text
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
nginx-nodeport   NodePort    10.96.x.x       <none>        80:30080/TCP
```

---

# Lab 20 - Access NodePort Using Minikube

Get the Minikube IP:

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

The NodePort is:

```text
30080
```

Therefore:

```text
http://192.168.49.2:30080
```

You can also use:

```bash
minikube service nginx-nodeport
```

This can open the Service through the Minikube environment.

---

# Lab 21 - Check NodePort Details

Run:

```bash
kubectl describe svc nginx-nodeport
```

Look for:

```text
Type:
NodePort

Port:
80

TargetPort:
80

NodePort:
30080

Endpoints:
...
```

Traffic flow:

```text
Client
   │
   │ NodeIP:30080
   ▼
NodePort
   │
   ▼
Service :80
   │
   ▼
NGINX Pod :80
```

---

# Lab 22 - Create a LoadBalancer Service

Create:

```text
nginx-loadbalancer.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-loadbalancer

spec:
  type: LoadBalancer

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f nginx-loadbalancer.yaml
```

Check:

```bash
kubectl get svc nginx-loadbalancer
```

In some Minikube configurations you may initially see:

```text
EXTERNAL-IP   <pending>
```

---

# Lab 23 - Use Minikube Tunnel

For a Minikube LoadBalancer Service, run:

```bash
minikube tunnel
```

Keep this terminal running.

Open another terminal and run:

```bash
kubectl get svc nginx-loadbalancer
```

Depending on the Minikube environment, the Service may receive an external IP.

Example:

```text
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP
nginx-loadbalancer  LoadBalancer   10.96.x.x       10.96.x.x
```

---

# Lab 24 - Inspect Service Endpoints

Run:

```bash
kubectl get endpoints nginx-loadbalancer
```

Example:

```text
NAME                ENDPOINTS
nginx-loadbalancer  10.244.0.5:80,10.244.0.6:80
```

Also check EndpointSlices:

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=nginx-loadbalancer
```

---

# Lab 25 - Service Selector Troubleshooting

Now intentionally create a broken Service.

Create:

```text
broken-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: broken-service

spec:
  selector:
    app: apache

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f broken-service.yaml
```

Check:

```bash
kubectl get svc
```

Now check:

```bash
kubectl get endpoints broken-service
```

Expected:

```text
NAME             ENDPOINTS
broken-service   <none>
```

---

# Lab 26 - Find the Problem

Check Pod labels:

```bash
kubectl get pods --show-labels
```

You will see:

```text
app=nginx
```

But the broken Service expects:

```yaml
selector:
  app: apache
```

Therefore:

```text
Service Selector
app=apache
      │
      X
      │
Pod Label
app=nginx
```

No Pods match.

---

# Lab 27 - Fix the Selector

Edit the Service:

```bash
kubectl edit svc broken-service
```

Change:

```yaml
selector:
  app: apache
```

to:

```yaml
selector:
  app: nginx
```

Check:

```bash
kubectl get endpoints broken-service
```

Now backend endpoints should appear.

---

# Lab 28 - Wrong targetPort Troubleshooting

Create a Service with an incorrect `targetPort`.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: wrong-port-service

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 8080
```

Apply:

```bash
kubectl apply -f wrong-port-service.yaml
```

The NGINX container is listening on:

```text
80
```

But the Service sends traffic to:

```text
8080
```

Traffic:

```text
Service :80
    │
    ▼
Pod :8080
    X
NGINX :80
```

The Service may have endpoints, but the application will not respond correctly on the wrong target port.

---

# Lab 29 - Fix targetPort

Edit:

```bash
kubectl edit svc wrong-port-service
```

Change:

```yaml
targetPort: 8080
```

to:

```yaml
targetPort: 80
```

Test:

```bash
kubectl run curl-pod \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Then:

```bash
curl http://wrong-port-service
```

You should receive the NGINX response.

Exit:

```bash
exit
```

---

# Lab 30 - Service Without Matching Pods

Create:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: test-service

spec:
  selector:
    app: does-not-exist

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f test-service.yaml
```

Check:

```bash
kubectl get endpoints test-service
```

Expected:

```text
NAME           ENDPOINTS
test-service   <none>
```

This demonstrates:

> A Service can exist without having any backend Pods.

---

# Lab 31 - Observe Service During Deployment Scaling

Scale the Deployment down:

```bash
kubectl scale deployment nginx-deployment --replicas=1
```

Check:

```bash
kubectl get pods
```

Then:

```bash
kubectl get endpoints nginx-service
```

You should see one backend endpoint.

Now scale back:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Check:

```bash
kubectl get endpoints nginx-service
```

The number of backend endpoints should increase.

---

# Lab 32 - Create a Headless Service

Create:

```text
nginx-headless.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-headless

spec:
  clusterIP: None

  selector:
    app: nginx

  ports:
    - port: 80
```

Apply:

```bash
kubectl apply -f nginx-headless.yaml
```

Check:

```bash
kubectl get svc nginx-headless
```

Expected:

```text
NAME             TYPE        CLUSTER-IP
nginx-headless   ClusterIP   None
```

---

# Lab 33 - Compare ClusterIP and Headless Service

Normal Service:

```text
Service
   │
   ▼
ClusterIP
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

Headless Service:

```text
DNS
 │
 ├── Pod 1 IP
 ├── Pod 2 IP
 └── Pod 3 IP
```

Check both:

```bash
kubectl get svc
```

Example:

```text
NAME             TYPE        CLUSTER-IP
nginx-service    ClusterIP   10.96.x.x
nginx-headless   ClusterIP   None
```

---

# Lab 34 - Inspect Service YAML

Run:

```bash
kubectl get svc nginx-service -o yaml
```

Important fields:

```yaml
spec:
  type: ClusterIP

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

This is useful when troubleshooting or understanding an existing Service.

---

# Lab 35 - Get Service ClusterIP

Run:

```bash
kubectl get svc nginx-service \
  -o jsonpath='{.spec.clusterIP}'
```

Example:

```text
10.96.20.10
```

Test from inside the cluster:

```bash
kubectl run curl-pod \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Then:

```bash
curl http://10.96.20.10
```

Exit:

```bash
exit
```

---

# Lab 36 - Get Service Port

Run:

```bash
kubectl get svc nginx-service \
  -o jsonpath='{.spec.ports[0].port}'
```

Expected:

```text
80
```

Get targetPort:

```bash
kubectl get svc nginx-service \
  -o jsonpath='{.spec.ports[0].targetPort}'
```

Expected:

```text
80
```

---

# Lab 37 - Get Service Selector

Run:

```bash
kubectl get svc nginx-service \
  -o jsonpath='{.spec.selector}'
```

Expected:

```text
map[app:nginx]
```

The selector is responsible for identifying backend Pods.

---

# Lab 38 - Complete Service Architecture

At this point, our environment looks like:

```text
                         Client
                           │
                           ▼
                    nginx-service
                     ClusterIP
                           │
                 ┌─────────┼─────────┐
                 │         │         │
                 ▼         ▼         ▼
               Pod 1     Pod 2     Pod 3
                 │         │         │
                 └─────────┼─────────┘
                           │
                       Deployment
```

The responsibilities are:

```text
Deployment
    ↓
Creates and manages Pods

Service
    ↓
Provides stable networking

EndpointSlice
    ↓
Tracks backend endpoints

CoreDNS
    ↓
Provides Service DNS
```

---

# Lab 39 - Complete Troubleshooting Exercise

We will intentionally create a broken Service.

Create:

```text
troubleshoot-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: troubleshoot-service

spec:
  selector:
    app: backend

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f troubleshoot-service.yaml
```

Check:

```bash
kubectl get svc troubleshoot-service
```

Then:

```bash
kubectl get endpoints troubleshoot-service
```

You should find:

```text
<none>
```

Now troubleshoot systematically.

---

# Troubleshooting Step 1 - Check Pods

```bash
kubectl get pods --show-labels
```

Find the actual Pod label.

Example:

```text
app=nginx
```

---

# Troubleshooting Step 2 - Check Service Selector

```bash
kubectl describe svc troubleshoot-service
```

Look for:

```text
Selector:
```

You may find:

```text
app=backend
```

---

# Troubleshooting Step 3 - Compare Labels

Pod:

```text
app=nginx
```

Service:

```text
app=backend
```

They do not match.

---

# Troubleshooting Step 4 - Fix Selector

Edit:

```bash
kubectl edit svc troubleshoot-service
```

Change:

```yaml
selector:
  app: backend
```

to:

```yaml
selector:
  app: nginx
```

---

# Troubleshooting Step 5 - Verify Endpoints

Run:

```bash
kubectl get endpoints troubleshoot-service
```

You should now see backend Pod IPs.

---

# Troubleshooting Step 6 - Test Connectivity

Run:

```bash
kubectl run curl-pod \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Then:

```bash
curl http://troubleshoot-service
```

If the Service is correctly configured, NGINX should respond.

Exit:

```bash
exit
```

---

# Lab 40 - Service Debugging Checklist

When a Service is not working, check the following:

```text
1. Does the Service exist?
        ↓
2. Is the Service selector correct?
        ↓
3. Do Pod labels match?
        ↓
4. Are Pods Running?
        ↓
5. Are Pods Ready?
        ↓
6. Does the Service have Endpoints?
        ↓
7. Does the EndpointSlice contain backend addresses?
        ↓
8. Is targetPort correct?
        ↓
9. Is the application listening on that port?
        ↓
10. Is DNS working?
        ↓
11. Is cluster networking working?
```

Useful commands:

```bash
kubectl get svc
kubectl describe svc <service>
kubectl get pods --show-labels
kubectl get endpoints <service>
kubectl get endpointslice
kubectl get pods -o wide
```

---

# Lab 41 - Cleanup

Delete the Services:

```bash
kubectl delete svc nginx-service
kubectl delete svc nginx-nodeport
kubectl delete svc nginx-loadbalancer
kubectl delete svc nginx-headless
kubectl delete svc broken-service
kubectl delete svc wrong-port-service
kubectl delete svc test-service
kubectl delete svc troubleshoot-service
```

Delete the Deployment:

```bash
kubectl delete deployment nginx-deployment
```

Delete the standalone Pod:

```bash
kubectl delete pod nginx-pod
```

Check:

```bash
kubectl get pods
kubectl get svc
```

---

# Lab 42 - Quick Revision

## Create Deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

## Create Service

```bash
kubectl apply -f nginx-service.yaml
```

## List Services

```bash
kubectl get svc
```

## Describe Service

```bash
kubectl describe svc nginx-service
```

## Check Endpoints

```bash
kubectl get endpoints nginx-service
```

## Check EndpointSlices

```bash
kubectl get endpointslice
```

## Check Pods

```bash
kubectl get pods -o wide
```

## Check Pod Labels

```bash
kubectl get pods --show-labels
```

## Test Service

```bash
kubectl run curl-pod \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Then:

```bash
curl http://nginx-service
```

## Create NodePort

```bash
kubectl apply -f nginx-nodeport.yaml
```

## Access NodePort

```bash
minikube service nginx-nodeport
```

## Create LoadBalancer

```bash
kubectl apply -f nginx-loadbalancer.yaml
```

## Start Minikube Tunnel

```bash
minikube tunnel
```

## Scale Deployment

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

## Delete Pod

```bash
kubectl delete pod <pod-name>
```

---

# Practical Exercises

## Exercise 1 - ClusterIP

Create an NGINX Deployment with:

```text
3 replicas
```

Create a ClusterIP Service:

```text
Service name: nginx-service
Port: 80
TargetPort: 80
```

Verify:

```bash
kubectl get svc
kubectl get endpoints nginx-service
```

Test:

```bash
curl http://nginx-service
```

---

## Exercise 2 - NodePort

Create a NodePort Service:

```text
Service name: nginx-nodeport
Port: 80
TargetPort: 80
NodePort: 30080
```

Verify:

```bash
kubectl get svc
```

Access using:

```bash
minikube service nginx-nodeport
```

---

## Exercise 3 - Scaling

Scale the Deployment:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Check:

```bash
kubectl get pods
kubectl get endpoints nginx-service
```

Observe how the number of backend endpoints changes.

---

## Exercise 4 - Pod Replacement

Delete one Pod:

```bash
kubectl delete pod <pod-name>
```

Observe:

```bash
kubectl get pods
```

Then:

```bash
kubectl get endpoints nginx-service
```

Understand how the Service continues working even when individual Pods are replaced.

---

## Exercise 5 - Break the Selector

Change the Service selector from:

```yaml
selector:
  app: nginx
```

to:

```yaml
selector:
  app: apache
```

Check:

```bash
kubectl get endpoints nginx-service
```

Observe:

```text
<none>
```

Fix the selector and verify the endpoints return.

---

# Final Service Architecture

```text
                         Internet
                            │
                            ▼
                     LoadBalancer
                            │
                            ▼
                        NodePort
                            │
                            ▼
                        ClusterIP
                            │
                  ┌─────────┼─────────┐
                  │         │         │
                  ▼         ▼         ▼
                Pod 1     Pod 2     Pod 3
                  │         │         │
                  └─────────┼─────────┘
                            │
                       EndpointSlice
                            │
                       Pod IP + Port
```

Service discovery:

```text
Application
     │
     ▼
Service DNS
     │
     ▼
Service IP
     │
     ▼
EndpointSlice
     │
     ▼
Pod IP
     │
     ▼
Container
```

---

# Key Takeaways

* A Service provides stable networking for Pods.
* Pod IP addresses can change.
* Services use selectors to find Pods.
* Pod labels must match Service selectors.
* `ClusterIP` is used for internal communication.
* `NodePort` exposes a Service through a Node port.
* `LoadBalancer` provides external load-balancer access.
* `targetPort` must match the application's listening port.
* Services can have multiple backend Pods.
* EndpointSlices show the backend endpoints associated with a Service.
* Kubernetes provides DNS-based Service discovery.
* Services continue providing a stable endpoint even when Pods are recreated.
* Empty Endpoints usually indicate a selector, label, readiness, or Pod problem.
* `kubectl describe svc` is useful for Service troubleshooting.
* `kubectl get endpoints` and `kubectl get endpointslice` are important debugging commands.
* Services are commonly used with Deployments.
* Ingress can be placed in front of Services for HTTP/HTTPS routing.

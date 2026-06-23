# Kubernetes Service Types Demo

This project demonstrates the practical implementation of different **Kubernetes Service Types** and how they are used to expose applications **inside and outside** a Kubernetes cluster.

The goal of this project is to understand not just the YAML syntax, but also **how Kubernetes Services work**, **why they are important**, **how traffic reaches pods**, and **where each service type is used in real-world deployments**.

---

## Project Objective

In Kubernetes, pods are **ephemeral** — they can restart, get rescheduled, and their IP addresses can change. Because of this, directly connecting to pod IPs is not a reliable way to access applications.

**Kubernetes Services** solve this problem by providing:

* A **stable virtual IP / endpoint**
* **Service discovery** inside the cluster
* **Load balancing** across multiple pods
* Controlled **internal or external access** to applications

This project focuses on the four major Kubernetes service types:

* **ClusterIP**
* **NodePort**
* **LoadBalancer**
* **ExternalName**

---

# Service Types Covered

## 1) ClusterIP

**ClusterIP** is the default Kubernetes Service type. It exposes an application **only within the cluster**.

### Why it is used

It is used when an application should be accessible only by other applications running inside Kubernetes.

### Common use cases

* Frontend calling a backend API
* Backend connecting to a database
* Communication between microservices
* Internal monitoring or logging components

### Example scenario

A `frontend` pod calls a `backend` service, and the backend service talks to a `mysql` service — all inside the cluster.

---

## 2) NodePort

**NodePort** exposes an application on a **fixed port on each worker node**.

The application can be accessed from outside the cluster using:

```bash
<NodeIP>:<NodePort>
```

### Why it is used

It provides a simple way to expose an application externally without needing a cloud load balancer.

### Common use cases

* Lab environments
* Kubernetes practice projects
* Demo applications
* Temporary testing from outside the cluster

### Example scenario

A sample web app is exposed on port `30080`, and users can access it using the node IP and port.

---

## 3) LoadBalancer

**LoadBalancer** exposes the application externally through a **cloud provider load balancer**.

### Why it is used

It is commonly used for **production-facing applications** that need a public endpoint.

### Common use cases

* Public web applications
* Production APIs
* Applications accessed by users over the internet
* Services behind cloud load balancers in EKS / AKS / GKE

### Example scenario

A production web app is deployed in Kubernetes and exposed through a cloud load balancer so that users can access it via an external IP or DNS.

> Note: On local clusters such as Docker Desktop, Minikube, or kind, `LoadBalancer` may remain in **pending** state unless a load balancer implementation like **MetalLB** is configured.

---

## 4) ExternalName

**ExternalName** does not expose pods directly. Instead, it maps a Kubernetes Service to an **external DNS name**.

### Why it is used

It allows applications running inside Kubernetes to connect to an external service using a Kubernetes service abstraction.

### Common use cases

* Connecting to external APIs
* Accessing SaaS endpoints
* Using an external database or external service from within the cluster
* Avoiding hardcoded external URLs in application configuration

### Example scenario

An application inside Kubernetes needs to connect to `api.example.com`. Instead of hardcoding the external hostname in the application, it can use an `ExternalName` service.

---

# Project Structure

```bash
Service-type/
│
├── namespace.yaml
├── clusterip.yaml
├── nodeport.yaml
├── loadbalancer.yaml
├── externalname.yaml
 ├── app1.yaml
 ├── app2.yaml
 └─ app3.yaml
─ README.md
```

> If your folder structure is different, update this section accordingly.

---

# How Kubernetes Service Flow Works

The general traffic flow in Kubernetes is:

```text
User / Application Request
        ↓
Kubernetes Service
        ↓
Service Selector matches labels
        ↓
Traffic routed to matching Pod(s)
        ↓
Application response returned
```

### Internal flow example

For **ClusterIP**:

```text
Frontend Pod → ClusterIP Service → Backend Pods
```

### External flow example

For **NodePort**:

```text
Browser/User → NodeIP:NodePort → Service → Pod
```

### Production flow example

For **LoadBalancer**:

```text
User → External Load Balancer → Service → Pod
```

### External dependency flow example

For **ExternalName**:

```text
Application Pod → ExternalName Service → External DNS / External Service
```

---

# Prerequisites

Before running this project, ensure you have:

* A working **Kubernetes cluster**

  * Docker Desktop Kubernetes / Minikube / kind / EKS / AKS / GKE
* `kubectl` installed and configured
* YAML manifests for applications and services

Check cluster connectivity:

```bash
kubectl cluster-info
kubectl get nodes
```

---

# Deployment Steps

## Step 1: Create Namespace

Create a namespace for the project:

```bash
kubectl apply -f namespace.yaml
```

Verify:

```bash
kubectl get ns
```

---

## Step 2: Deploy Applications

Deploy the application manifests used by the services:

```bash
kubectl apply -f app/app1.yaml
kubectl apply -f app/app2.yaml
kubectl apply -f app/app3.yaml
```

Verify pods:

```bash
kubectl get pods
kubectl get pods -o wide
```

If your apps are deployed in a custom namespace:

```bash
kubectl get pods -n <namespace-name>
```

---

## Step 3: Create Kubernetes Services

Apply the service manifests:

```bash
kubectl apply -f clusterip.yaml
kubectl apply -f nodeport.yaml
kubectl apply -f loadbalancer.yaml
kubectl apply -f externalname.yaml
```

Verify services:

```bash
kubectl get svc
```

For a specific namespace:

```bash
kubectl get svc -n <namespace-name>
```

---

# Service Verification Commands

## View all services

```bash
kubectl get svc
```

## Describe a service

```bash
kubectl describe svc <service-name>
```

## View endpoints created for a service

```bash
kubectl get endpoints
```

## View pods with labels

```bash
kubectl get pods --show-labels
```

## Check deployment details

```bash
kubectl get deploy
kubectl describe deploy <deployment-name>
```

---

# Practical Testing of Each Service Type

# 1) Testing ClusterIP

## Purpose

ClusterIP is **internal only**, so it cannot be accessed directly from outside the cluster.

## Verify service

```bash
kubectl get svc
kubectl describe svc <clusterip-service-name>
```

## Test from inside the cluster

Run a temporary pod:

```bash
kubectl run testpod --rm -it --image=busybox -- /bin/sh
```

Inside the pod, test the service:

```bash
wget -qO- http://<clusterip-service-name>:<port>
```

or if DNS is not configured in the image, use the ClusterIP directly:

```bash
wget -qO- http://<cluster-ip>:<port>
```

### Expected result

The request should reach the target application pod and return the application response.

---

# 2) Testing NodePort

## Purpose

NodePort exposes the service externally through a port on each worker node.

## Verify service

```bash
kubectl get svc
kubectl describe svc <nodeport-service-name>
```

You will see a port mapping like:

```text
80:30007/TCP
```

Where:

* `80` = service port
* `30007` = NodePort

## Access the application

Use:

```bash
http://<NodeIP>:<NodePort>
```

Example:

```bash
http://192.168.1.10:30007
```

## Find node IP

```bash
kubectl get nodes -o wide
```

### Expected result

The application should be reachable from outside the cluster using the node IP and NodePort.

---

# 3) Testing LoadBalancer

## Purpose

LoadBalancer exposes the service externally using a cloud load balancer.

## Verify service

```bash
kubectl get svc
kubectl describe svc <loadbalancer-service-name>
```

If supported, you will see an **EXTERNAL-IP** assigned.

Example:

```text
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)
my-loadbalancer  LoadBalancer   10.96.10.20    34.100.20.10     80:31500/TCP
```

## Access the application

Use:

```bash
http://<EXTERNAL-IP>
```

### Expected result

Traffic should flow from the external load balancer to the service and then to the target pods.

> If you are using Docker Desktop / Minikube / kind without MetalLB, the external IP may remain **pending**.

---

# 4) Testing ExternalName

## Purpose

ExternalName maps a Kubernetes service to an external DNS name.

## Verify service

```bash
kubectl get svc
kubectl describe svc <externalname-service-name>
```

You should see something like:

```text
Type: ExternalName
External Name: example.com
```

## Test from inside the cluster

Run a temporary pod:

```bash
kubectl run dns-test --rm -it --image=busybox -- /bin/sh
```

Inside the pod, test DNS resolution or connectivity:

```bash
nslookup <externalname-service-name>
```

or

```bash
wget -qO- http://<externalname-service-name>
```

### Expected result

The service name should resolve to the configured external DNS name.

---

# Useful Debugging Commands

## Check pods and services together

```bash
kubectl get pods,svc
```

## Check labels on pods

```bash
kubectl get pods --show-labels
```

## Check endpoints

```bash
kubectl get endpoints
```

## Describe pods

```bash
kubectl describe pod <pod-name>
```

## View logs

```bash
kubectl logs <pod-name>
```

## View namespace resources

```bash
kubectl get all -n <namespace-name>
```

---

# Key Learnings from This Project

Through this project, I got practical understanding of:

* Why **Kubernetes Services** are required when pod IPs are dynamic
* How **ClusterIP** enables internal service-to-service communication
* How **NodePort** exposes applications externally using node ports
* Why **LoadBalancer** is the preferred model for production-facing apps
* How **ExternalName** helps in integrating external services cleanly
* How service selectors map traffic to the correct pods
* The difference between **internal**, **external**, and **external dependency** service models

---

# Real-World Use Cases Summary

| Service Type | Access Scope                     | Best Use Cases                                    |
| ------------ | -------------------------------- | ------------------------------------------------- |
| ClusterIP    | Internal only                    | Backend APIs, microservices, databases            |
| NodePort     | External via node port           | Labs, testing, demos, temporary external access   |
| LoadBalancer | External public/private endpoint | Production apps, APIs, public services            |
| ExternalName | DNS mapping to external service  | External APIs, SaaS endpoints, external databases |

---

# Cleanup Commands

Delete all resources created by this project:

```bash
kubectl delete -f clusterip.yaml
kubectl delete -f nodeport.yaml
kubectl delete -f loadbalancer.yaml
kubectl delete -f externalname.yaml
kubectl delete -f app/app1.yaml
kubectl delete -f app/app2.yaml
kubectl delete -f app/app3.yaml
kubectl delete -f namespace.yaml
```

Or delete the namespace directly if everything is deployed inside it:

```bash
kubectl delete ns <namespace-name>
```

---

# Conclusion

This project is a practical walkthrough of **Kubernetes Service Types** and how they are used to expose applications in different ways based on the requirement.

It highlights:

* **why Services are important**
* **how traffic reaches pods**
* **when to use ClusterIP, NodePort, LoadBalancer, and ExternalName**
* **how Kubernetes abstracts networking and service discovery**

Understanding these service types is a core part of working with Kubernetes because they directly affect **application accessibility, networking design, and production architecture**.

---

# GitHub Repository

**Repo:** https://github.com/gauravrai11/Service-type

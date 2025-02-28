# OpenShift Deployment: Migrating Microservices from UAT to Production

## Table of Contents
1. [Introduction](#1-introduction)
2. [Key Concepts](#2-key-concepts)
   - [What are Microservices?](#what-are-microservices)
   - [What is User Acceptance Testing (UAT)?](#what-is-user-acceptance-testing-uat)
   - [What is a Production Server?](#what-is-a-production-server)
   - [OpenShift vs Kubernetes–Whats the Difference?](#openshift-vs-kubernetes--whats-the-difference)
   - [What is a Namespace?](#what-is-a-namespace)
   - [What is a ConfigMap?](#what-is-a-configmap)
   - [What is a Deployment?](#what-is-a-deployment)
   - [What is a Service?](#what-is-a-service)
   - [What is Istio?](#what-is-istio)
3. [Migration Process: UAT to Production](#3-migration-process-uat-to-production)
   - [Step 1: Retrieve ConfigMap and Deployment from UAT](#31-step-1-retrieve-configmap-and-deployment-from-uat)
   - [Step 2: Modify Configurations for Production](#32-step-2-modify-configurations-for-production)
   - [Step 3: Apply ConfigMap and Deployment to Production](#33-step-3-apply-configmap-and-deployment-to-production)
   - [Step 4: Expose the Service and Verify](#34-step-4-expose-the-service-and-verify)
4. [Practical Example: Deploying NGINX-based Microservice](#4-practical-example-deploying-nginx-based-microservice)
   - [Step 1: Create a ConfigMap](#41-step-1-create-a-configmap)
   - [Step 2: Create a Deployment](#42-step-2-create-a-deployment)
   - [Step 3: Expose the Service](#43-step-3-expose-the-service)
   - [Step 4: Get the Service URL and Test](#44-step-4-get-the-service-url-and-test)


---

## 1. Introduction
This document provides a comprehensive and step-by-step guide for migrating microservices from the User Acceptance Testing (UAT) environment to the Production environment in OpenShift. Additionally, it offers a practical example of deploying an NGINX-based microservice, covering key processes such as configuration management, deployment, and service exposure.

The sections include:
- Key Concepts and Definitions
- Migration Process: UAT to Production
- Practical Example: Deploying NGINX-based Microservice
- Testing the Deployment using cURL

---

## 2. Key Concepts
Understanding the core components involved in OpenShift deployment is crucial for managing the migration process effectively. Below are key concepts and terms used in OpenShift and Kubernetes environments.

## **What are Microservices?**
### **Definition**
Microservices is a way of designing an application by breaking it into **small, independent services**. Each service runs separately and communicates with others through APIs (Application Programming Interfaces).

### **Example**
Think about an **online shopping app** like Amazon. It has different features, such as:
- **User Login Service** – Handles account logins
- **Product Listing Service** – Displays available products
- **Order Service** – Manages purchases
- **Payment Service** – Processes payments

Each service works independently, meaning if there is a problem with the **payment service**, the **product listing service** will not be affected.

---

## **What is User Acceptance Testing (UAT)?**
### **Definition**
UAT is a **testing phase** where real users check if the application works correctly **before** it is made available to the public.

### **Example**
- Imagine you are **opening a restaurant**.
- Before allowing real customers, you **invite family and friends** for a **trial run** to see if the kitchen, menu, and service are working smoothly.
- If something is wrong, you fix it before opening the restaurant officially.
- This **trial run** is similar to UAT in software development.

---

## **What is a Production Server?**
### **Definition**
The **production server** is the **live environment** where real users access the application.

### **Example**
- After successful UAT testing, an **e-commerce website** is made available for real customers to **place orders, make payments, and receive products**.
- This live website is running on the **production server**.

---

## **OpenShift vs Kubernetes – What’s the Difference?**

| Feature  | **OpenShift** | **Kubernetes** |
|------------|--------------|---------------|
| **Ownership & Support** | Red Hat maintains it with enterprise-level support | CNCF (Cloud Native Computing Foundation), community-driven |
| **Installation & Setup** | Simple but requires Red Hat ecosystem (RHEL, RHCOS) | Manual setup, flexible but complex |
| **Web UI (Console)** | Advanced built-in UI (Developer & Admin views) | Basic Dashboard, needs extra configuration |
| **Security** | **More secure** (strict security policies, built-in RBAC) | Secure but requires manual security hardening |
| **CI/CD Integration** | Built-in **OpenShift Pipelines (Tekton)** & **ArgoCD** | Separate tools required (Jenkins, ArgoCD, Tekton) |
| **Networking** | **OpenShift SDN** + Built-in **Istio Service Mesh** | Flannel, Calico, Cilium (Networking plugin required) |
| **Registry (Image Management)** | Built-in **Red Hat OpenShift Container Registry (OCR)** | Requires setting up a registry (Harbor, Docker Hub, etc.) |
| **Pricing** | Paid (Red Hat Subscription) | Free & Open-Source |


### **Simple Explanation**
1. **Kubernetes is like an Android phone** – basic but powerful.
2. **OpenShift is like an iPhone** – the same core features but with more security, support, and automation.

---

## **What is a Namespace?**
### **Definition**
A **namespace** in OpenShift helps organize and separate different projects, teams, or environments inside a Kubernetes cluster.

### **Example**
Imagine a **university**:
- The university has different **departments** like **IT, HR, and Sales**.
- Each department has its **own office space** and **works independently** but still belongs to the same university.
- In OpenShift, these **departments are like namespaces**, keeping things organized and preventing conflicts.

---

## **What is a ConfigMap?**
### **Definition**
A **ConfigMap** stores configuration settings for an application so that developers **don’t have to change the application code** whenever settings are updated.

### **Example**
- A **restaurant menu** is a ConfigMap.
- The **chef (application)** reads the menu to know what dishes to prepare.
- If the restaurant wants to add a new dish, they **update the menu (ConfigMap)** instead of changing how the chef cooks everything.

---

## **What is a Deployment?**
### **Definition**
A **Deployment** ensures that the correct number of application instances (**pods**) are running. If a pod crashes, OpenShift will **automatically start a new one**.

### **Example**
- A company has **10 employees** in customer support.
- If one employee **leaves**, the company **hires a new one** to keep the total at **10**.
- OpenShift does the same with applications – if a pod stops working, it replaces it with a new one.

---

## **What is a Service?**
### **Definition**
A **Service** in OpenShift allows applications inside the cluster to **communicate with each other** and lets users access them from outside.

### **Example**
- Think of a **receptionist in an office**.
- A visitor (request) arrives and asks for the **HR department (application pod)**.
- The **receptionist (Service)** directs them to the correct office.
- Similarly, a **Service in OpenShift** directs network traffic to the correct application pod.

---

## **What is Istio?**
### **Definition**
**Istio is a service mesh** – a technology that manages how different microservices talk to each other in OpenShift. It adds **security, monitoring, and traffic management** between services.

### **Example**
- Imagine a **city traffic control system**.
- It manages how cars **(requests)** move between different roads **(microservices)** to avoid accidents and congestion.
- Istio does the same for microservices, **ensuring smooth communication** between them.

---

## 3. Migration Process: UAT to Production
Follow these steps to migrate microservices from UAT to the Production environment in OpenShift.

### 3.1 Step 1: Retrieve ConfigMap and Deployment from UAT
First, export the configuration and deployment definitions from the UAT namespace:

```bash
oc get configmap my-service-config -n uat -o yaml > configmap.yaml
oc get deployment my-service -n uat -o yaml > deployment.yaml
```
### 3.2 Step 2: Modify Configurations for Production
Update the namespace field in the YAML files to reflect the production environment. This ensures that the resources are applied in the correct namespace.

```bash
metadata:
  namespace: production
```
### 3.3 Step 3: Apply ConfigMap and Deployment to Production
Now, apply the updated files to the production namespace:

```bash
oc apply -f configmap.yaml -n production
oc apply -f deployment.yaml -n production
```

### 3.4 Step 4: Expose the Service and Verify
Expose the service to make it accessible externally and verify its functionality using cURL.

```bash
oc expose deployment my-service --type=LoadBalancer --name=my-service-lb -n production
curl http://my-service-lb.production.example.com
```

## 4. Practical Example: Deploying NGINX-based Microservice
Let’s deploy a simple NGINX application in Minikube using ConfigMap, Deployment, and Service.

### 4.1 Step 1: Create a ConfigMap
First, create a ConfigMap that holds the HTML content for NGINX.

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: production
data:
  index.html: |
    <html><body><h1>Hello, OpenShift!</h1></body></html>
```

Apply the ConfigMap:

```bash
kubectl apply -f configmap.yaml
```
Output:-
```
configmap/nginx-config created
```

### 4.2 Step 2: Create a Deployment
Create a Deployment that uses the NGINX image and mounts the ConfigMap to serve the HTML content.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  namespace: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-app  
  template:
    metadata:
      labels:
        app: nginx-app  
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        volumeMounts:
        - name: nginx-html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-html
        configMap:
          name: nginx-config

```


Apply the Deployment:

```bash
kubectl apply -f deployment.yaml
```
Output:-
```
deployment.apps/nginx-app created
```


### 4.3 Step 3: Expose the Service
Expose the NGINX service to the network to make it accessible externally.

```bash
kubectl expose deployment nginx-app --port=80 --target-port=80 --type=NodePort -n production
```

Output:-
```
service/nginx-app exposed
```

### 4.4 Step 4: Get the Service URL and Test
Retrieve the external IP or URL assigned to the service and test it using cURL:

```bash
praveshkumar@fedora:~/newone$ kubectl get svc -n production
NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-app   NodePort   10.106.75.47   <none>        80:32142/TCP   11s
```

```bash
praveshkumar@fedora:~/newone$ curl 192.168.39.169:32142
```
Expected Output:
```bash
<html><body><h1>Hello, OpenShift!</h1></body></html>
```



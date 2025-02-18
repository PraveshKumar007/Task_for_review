# OpenShift Deployment: Migrating Microservices from UAT to Production

## Table of Contents
1. [Introduction](#1-introduction)
2. [Key Concepts](#2-key-concepts)
   - [Microservices](#21-microservices)
   - [User Acceptance Testing (UAT)](#22-user-acceptance-testing-uat)
   - [Production Server](#23-production-server)
   - [OpenShift vs Kubernetes](#24-openshift-vs-kubernetes)
   - [Namespace](#25-namespace)
   - [ConfigMap](#26-configmap)
   - [Deployment](#27-deployment)
   - [Service](#28-service)
   - [Istio](#29-istio)
   - [cURL](#210-curl)
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

### 2.1 Microservices
**Definition:** Microservices refer to an architectural design where an application is broken down into small, self-contained services that can be developed, deployed, and scaled independently.

**Example:** In an online shopping platform, the product catalog, user authentication, and payment processing are each handled by separate microservices. Each service communicates via APIs to deliver the full functionality.

### 2.2 User Acceptance Testing (UAT)
**Definition:** UAT is the phase of testing where end-users validate the software in an environment that simulates production to ensure it meets business requirements.

**Example:** A restaurant’s trial service before opening allows chefs, servers, and managers to ensure everything functions smoothly before officially serving customers.

### 2.3 Production Server
**Definition:** The live environment that hosts applications and services used by real customers.

**Example:** After successful testing, an e-commerce website is deployed to a production server where real customers can place orders.

### 2.4 OpenShift vs Kubernetes
**Kubernetes:** An open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

**OpenShift:** A Red Hat enterprise version of Kubernetes with enhanced security features, automated installation, and management tools.

**Example:** Think of Kubernetes as a basic smartphone and OpenShift as a premium version with more features designed for enterprise needs.

### 2.5 Namespace
**Definition:** A Namespace in OpenShift is a logical division within a cluster that helps group resources and isolate them from other groups.

**Example:** A university might have separate namespaces for departments like IT, HR, and Sales, each with its own resources and access control.

### 2.6 ConfigMap
**Definition:** A ConfigMap in Kubernetes/OpenShift stores non-sensitive configuration data for applications. Containers access this data at runtime.

**Example:** A restaurant’s menu is stored as a ConfigMap—chefs refer to it to prepare specific dishes for customers.

### 2.7 Deployment
**Definition:** A Deployment manages the lifecycle of application replicas, ensuring that the desired number of application instances are running.

**Example:** A mobile app update rolled out via the Play Store automatically updates the user’s version, ensuring consistency across devices.

### 2.8 Service
**Definition:** A Service in OpenShift exposes an application to a network, allowing other services or end-users to access it.

**Example:** A receptionist in an office acts like a Service—directing visitors (requests) to the correct department (pod).

### 2.9 Istio
**Definition:** Istio is a service mesh that provides advanced features like traffic management, security, and monitoring for microservices.

**Example:** Similar to a traffic control system that manages the movement of vehicles between different routes, Istio manages traffic between microservices.

### 2.10 cURL
**Definition:** cURL is a command-line tool used to make HTTP requests for interacting with services or testing endpoints.

**Example:** Like calling a phone number to check if a service is available, you use cURL to check if your application is running by hitting an endpoint.

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
  namespace: uat
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
praveshkumar@fedora:~/newone$ kubectl get svc -n uat
NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-app   NodePort   10.106.75.47   <none>        80:32142/TCP   11s
```

```bash
praveshkumar@fedora:~/newone$ curl 192.168.39.169:32142
```
Expected Output:

<html><body><h1>Hello, OpenShift!</h1></body></html>

```bash
praveshkumar@fedora:~/newone$ minikube service nginx-app -n uat
|-----------|-----------|-------------|-----------------------------|
| NAMESPACE |   NAME    | TARGET PORT |             URL             |
|-----------|-----------|-------------|-----------------------------|
| uat       | nginx-app |          80 | http://192.168.39.169:32142 |
|-----------|-----------|-------------|-----------------------------|
🎉  Opening service uat/nginx-app in default browser...
```



# IMPLEMENTATION JOURNAL - ESB-OCP

## Submitted By
**Pravesh Kumar**

## Submitted To
**Pawan, Pankaj sir**

## Version
**1.0**

## Reviewer Name
**Pawan, Pankaj sir**

---

## Goal
This document provides a step-by-step implementation guide for migrating microservices from the UAT (User Acceptance Testing) environment to the Production environment in OpenShift.

---

## Table of Contents

1. [Pre-requisites](#pre-requisites)
   - [Hardware Requirements](#hardware-requirements)
   - [Software Requirements](#software-requirements)
   - [Networking Requirements](#networking-requirements)
2. [Steps](#steps)
   - [Step 1: Access UAT Server](#step-1-access-uat-server)
   - [Step 2: Access Production Server](#step-2-access-production-server)
   - [Step 3: Verify Running Pod in UAT](#step-3-verify-running-pod-in-uat)
   - [Step 4: Retrieve ConfigMap and Deployment](#step-4-retrieve-configmap-and-deployment)
   - [Step 5: Copy YAML Files to Production](#step-5-copy-yaml-files-to-production)
   - [Step 6: Update YAML Files](#step-6-update-yaml-files)
   - [Step 7: Apply Updated YAML Files in Production](#step-7-apply-updated-yaml-files-in-production)
   - [Step 8: Verify Running Pod in Production](#step-8-verify-running-pod-in-production)
   - [Step 9: Expose Service in Production](#step-9-expose-service-in-production)

---

## Pre-requisites

### Hardware Requirements
- 12 CPUs  
- 16 GB RAM  
- 200 GB disk space  

### Software Requirements
- OpenShift CLI (`oc`)  

### Networking Requirements
- Internal network access between UAT and Production clusters  

---

## Steps

### Step 1: Access UAT Server
```sh
praveshkumar@fedora:~$ ssh esbocp@10.181.50.69
esbocp@10.181.50.69's password: 
Last login: Tue Mar 4 11:39:13 2025 from 10.161.7.33
[esbocp@dhcp ~]$
```

---

### Step 2: Access Production Server
```sh
praveshkumar@fedora:~$ ssh esbadmin@10.71.86.228
esbadmin@10.71.86.228's password: 
Last login: Tue Mar 4 11:48:27 2025 from 10.161.8.200
[esbadmin@bastion ~]$
```

---

### Step 3: Verify Running Pod in UAT
```sh
[esbocp@dhcp ~]$ oc get po -A | grep -iw cibilcheck
esb-uat  cibilcheck-6ff9f7b7dd-tlcnp  2/2  Running  0  35h
```

---

### Step 4: Retrieve ConfigMap and Deployment
```sh
[esbocp@dhcp ~]$ oc get cm cibilcheck -n esb-uat -o yaml > cibilcheck_CM.yaml

[esbocp@dhcp ~]$ oc get deployment cibilcheck -n esb-uat -o yaml > cibilcheck_deployment.yaml
```

---

### Step 5: Copy YAML Files to Production
```sh
[esbocp@dhcp ~]$ scp cibilcheck_CM.yaml esbadmin@10.71.86.228:/home/esbadmin/migration/
[esbocp@dhcp ~]$ scp cibilcheck_deployment.yaml esbadmin@10.71.86.228:/home/esbadmin/migration/
```
Switch to the production server and verify:
```sh
[esbadmin@bastion migration]$ ls -lrth
total 8.0K
-rw-r--r--. 1 esbadmin esbadmin 2.0K Feb 22 14:58 cibilcheck_CM.yaml
-rw-r--r--. 1 esbadmin esbadmin 1.5K Feb 22 15:06 cibilcheck_deployment.yaml
```

---

### Step 6: Update YAML Files
```sh
[esbadmin@bastion migration]$ vi cibilcheck_CM.yaml 
[esbadmin@bastion migration]$ vi cibilcheck_deployment.yaml 
```

---

### Step 7: Apply Updated YAML Files in Production
```sh
[esbadmin@bastion migration]$ oc apply -f cibilcheck_CM.yaml 
configmap/cibilcheck created

[esbadmin@bastion migration]$ oc apply -f cibilcheck_deployment.yaml 
deployment/cibilcheck created
```

---

### Step 8: Verify Running Pod in Production
```sh
[esbadmin@bastion migration]$ oc get pods -A | grep -i cibilcheck
esb-user-service  cibilcheck-66f7fb5cfc-6grpr  2/2  Running  0  30s
```
The pod should be in the **Running** state.

---

### Step 9: Expose Service in Production
```sh
[esbadmin@bastion migration]$ oc expose deployment cibilcheck -n esb-user-service
Error from server (AlreadyExists): services "cibilcheck" already exists
```
OpenShift successfully exposes the deployment. A service is created for **cibilcheck** in the `esb-user-service` namespace.

---

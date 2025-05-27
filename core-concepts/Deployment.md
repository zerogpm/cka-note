# Kubernetes Deployments Guide

This repository contains examples and solutions for working with Kubernetes Deployments, including troubleshooting and creating deployments using both imperative commands and declarative YAML files.

## Original Problem Statement

**Question**: How many Deployments exist on the system now? We just created a Deployment! Check again! What is the image used to create the pods in the new deployment?

**Additional Tasks**:

1. Create a new Deployment using the deployment-definition-1.yaml file located at /root/ (fix issues if any)
2. Create a new Deployment with attributes: Name: httpd-frontend; Replicas: 3; Image: httpd:2.4-alpine

## Current System Status

### Checking Existing Deployments

```bash
controlplane ~ ➜  k get deploy
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
frontend-deployment   0/4     4            0           15s
```

**Analysis**: There is currently **1 deployment** named `frontend-deployment` with 4 desired replicas, but 0 are currently available (pods are still starting up).

### Deployment Details

```bash
controlplane ~ ➜  k describe deployment frontend-deployment
Name:                   frontend-deployment
Namespace:              default
CreationTimestamp:      Tue, 27 May 2025 21:11:13 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=busybox-pod
Replicas:               4 desired | 4 updated | 4 total | 0 available | 4 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=busybox-pod
  Containers:
   busybox-container:
    Image:      busybox888
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      echo Hello Kubernetes! && sleep 3600
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   frontend-deployment-cd6b557c (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  84s   deployment-controller  Scaled up replica set frontend-deployment-cd6b557c from 0 to 4
```

**Answer**: The image used in the frontend-deployment is **`busybox888`**.

## Task 1: Fix and Deploy using deployment-definition-1.yaml

### Original YAML File (with issues)

```yaml
---
apiVersion: apps/v1
kind: deployment # Issue: should be "Deployment" (capital D)
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
        - name: busybox-container
          image: busybox888
          command:
            - sh
            - "-c"
            - echo Hello Kubernetes! && sleep 3600
```

### Fixed YAML File

```yaml
---
apiVersion: apps/v1
kind: Deployment # Fixed: Capital "D"
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
        - name: busybox-container
          image: busybox888
          command:
            - sh
            - "-c"
            - echo Hello Kubernetes! && sleep 3600
```

### Commands to Apply the Fix

```bash
# Edit the file to fix the issue
vi /root/deployment-definition-1.yaml

# Apply the corrected deployment
kubectl apply -f /root/deployment-definition-1.yaml

# Verify the deployment was created
kubectl get deployments
```

## Task 2: Create httpd-frontend Deployment

### Method 1: Imperative Command (Recommended)

```bash
k create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
```

### Method 2: Declarative YAML File

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      name: httpd-frontend
  template:
    metadata:
      labels:
        name: httpd-frontend
    spec:
      containers:
        - name: httpd-frontend
          image: httpd:2.4-alpine
```

## Step-by-Step Command Explanations

### 1. Checking Deployments

```bash
k get deploy
```

- **Purpose**: Lists all deployments in the current namespace
- **Output**: Shows deployment name, ready replicas, up-to-date replicas, available replicas, and age
- **Key Information**: Reveals the current state of all deployments

### 2. Describing a Deployment

```bash
k describe deployment frontend-deployment
```

- **Purpose**: Provides detailed information about a specific deployment
- **Output**: Shows configuration, current status, events, and pod template details
- **Key Information**: Container image, replica counts, rolling update strategy, and recent

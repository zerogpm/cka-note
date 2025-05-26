# Kubernetes Flask Application Deployment Guide

This repository contains a complete guide for deploying a Flask web application on Kubernetes, including scaling operations and monitoring setup.

## Problem Statement

The original task involves:

- Creating a Kubernetes deployment for a Flask application using a provided manifest file
- Discovering and observing deployment status using kubectl commands
- Understanding and implementing manual scaling operations
- Setting up monitoring with Skooner dashboard

## YAML Configuration

### deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
        - name: flask
          image: rakshithraka/flask-web-app
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: flask-web-app-service
spec:
  type: ClusterIP
  selector:
    app: flask-app
  ports:
    - port: 80
      targetPort: 80
```

### Configuration Breakdown

**Deployment Configuration:**

- **apiVersion**: `apps/v1` - Uses the stable API version for deployments
- **kind**: `Deployment` - Defines this as a Deployment resource
- **replicas**: `2` - Initially creates 2 pod instances
- **selector**: Matches pods with label `app: flask-app`
- **image**: `rakshithraka/flask-web-app` - Docker image for the Flask application
- **containerPort**: `80` - Exposes port 80 within the container

**Service Configuration:**

- **type**: `ClusterIP` - Creates an internal service (cluster-only access)
- **selector**: Routes traffic to pods with `app: flask-app` label
- **port/targetPort**: Both set to 80 for HTTP traffic

## Step-by-Step Deployment Process

### 1. Create the Deployment

```bash
kubectl apply -f /root/deployment.yml
```

**Purpose**: This command creates both the Deployment and Service resources defined in the YAML file.

### 2. Observe Deployment Status

```bash
kubectl get deployments
```

**Expected Output**:

```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
flask-web-app   2/2     2            2           30s
```

**Explanation**: Shows that 2 out of 2 replicas are ready and available.

### 3. Check Running Pods

```bash
kubectl get pods
```

**Expected Output**:

```
NAME                             READY   STATUS    RESTARTS   AGE
flask-web-app-7d4b8c9f8d-abc12   1/1     Running   0          45s
flask-web-app-7d4b8c9f8d-def34   1/1     Running   0          45s
```

**Explanation**: Displays the individual pod instances created by the deployment.

### 4. Manual Scaling Operation

```bash
kubectl scale deployment flask-web-app --replicas=3
```

**Purpose**: Increases the number of running replicas from 2 to 3.

### 5. Verify Scaling Results

```bash
kubectl get deployments
```

**Expected Output After Scaling**:

```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
flask-web-app   3/3     3            3           2m
```

```bash
kubectl get pods
```

**Expected Output After Scaling**:

```
NAME                             READY   STATUS    RESTARTS   AGE
flask-web-app-7d4b8c9f8d-abc12   1/1     Running   0          2m15s
flask-web-app-7d4b8c9f8d-def34   1/1     Running   0          2m15s
flask-web-app-7d4b8c9f8d-ghi56   1/1     Running   0          30s
```

## Monitoring Setup

### Skooner Dashboard Access

**Token Location**: `/root/skooner-sa-token.txt`

**Token Value**:

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IjNnbThpSmFZZER3ZERYS2RKRkkzX0JtTjJieEx4dDRZeFBZTFhtdTR3bUUifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQ4Mjk1NTE4LCJpYXQiOjE3NDgyOTE5MTgsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiZjdhNWI5N2UtZDM2OS00NjUwLTlkZGUtNWJjZWNmZmVhOWJlIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJza29vbmVyLXNhIiwidWlkIjoiNjRmMDJmZDgtMjc2ZS00MDc4LWFhZjQtOTY2YzgzMGFjNGE0In19LCJuYmYiOjE3NDgyOTE5MTgsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpza29vbmVyLXNhIn0.VDq3ADRJXepv1lB-QIFqDu6e2i86PiZs9EWsGs-GVTDTfKl2Srs1JucAX0ctm7mp5AMDjR-nF5E3YfCTLgOQO801LjDi5-hVvF9yeklRNG9ZKPOtubL2q4POfjqmnhrl7Nlp3RABn-6x81VqXIcbHmHZmma6545-lRh_yu6bykGyN8NzqXWPvkXyTqnfMirLLyHiF0Z7eHQyGC5UPQnTQMSFuGRrAzdi3ZsLigad4w9MNX1ispxrG4ss8CyfpWjpfZoso2FVvt7ekQUrF1sWtTPveNWOnkK1oo8-3GkLw0dTViCDgEv_IV6yaUzXrUwKcOIphu7QrOKEPnaT_RQE8Q
```

### Accessing the Application

1. **Via Ingress Button**: Click the Ingress button at the top of the terminal
2. **Via Skooner Dashboard**: Use the provided token to access the monitoring interface

## Understanding kubectl scale Command

### Primary Purpose

The `kubectl scale` command is used to change the number of running replicas of a deployment, replicaset, or other scalable resources. This is useful for:

- Adjusting capacity based on demand
- Managing workloads in a Kubernetes cluster
- Horizontal scaling operations

### Scaling StatefulSets

Yes, the `kubectl scale` command can be used to scale both deployments and StatefulSets. Key differences:

**Deployments**: Pods can be created and destroyed in any order
**StatefulSets**: Kubernetes ensures that the state and order of the pods are maintained

### Resource Constraint Behavior

When scaling a deployment to a higher number of replicas than the cluster can support due to resource constraints:

1. **Kubernetes creates as many replicas as possible** within available resources
2. **Remaining replicas enter a pending state** until sufficient resources are freed up or added
3. **Dynamic resource management** allows Kubernetes to maintain the desired state as closely as possible

## Why This Approach Solves the Problem

This deployment strategy addresses several key requirements:

### 1. **High Availability**

- Multiple replicas (2-3) ensure service continuity if one pod fails
- Load distribution across multiple instances

### 2. **Scalability**

- Easy horizontal scaling using `kubectl scale` command
- Dynamic adjustment based on demand

### 3. **Service Discovery**

- ClusterIP service provides stable internal endpoint
- Label-based pod selection ensures traffic routing

### 4. **Monitoring and Observability**

- Skooner dashboard provides visual cluster monitoring
- kubectl commands offer command-line observability

### 5. **Resource Management**

- Kubernetes handles pod placement and resource allocation
- Graceful handling of resource constraints

## Quick Reference Commands

```bash
# Deploy the application
kubectl apply -f /root/deployment.yml

# Check deployment status
kubectl get deployments

# View running pods
kubectl get pods

# Scale the deployment
kubectl scale deployment flask-web-app --replicas=3

# Get detailed deployment info
kubectl describe deployment flask-web-app

# View service details
kubectl get services

# Access Skooner token
cat /root/skooner-sa-token.txt
```

## Troubleshooting

### Common Issues

1. **Pods stuck in Pending state**: Check cluster resources with `kubectl describe nodes`
2. **Image pull errors**: Verify the Docker image `rakshithraka/flask-web-app` is accessible
3. **Service connectivity**: Ensure labels match between Deployment and Service configurations

### Useful Debug Commands

```bash
# Get detailed pod information
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# Check cluster events
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

**Note**: This guide assumes you have a working Kubernetes cluster and kubectl configured properly.

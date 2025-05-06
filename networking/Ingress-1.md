# Kubernetes Ingress Controller Setup Guide

## Overview

This document explains the setup and configuration of an Nginx Ingress Controller in a Kubernetes cluster to route traffic to multiple applications in different namespaces. The guide focuses on configuring path-based routing to several microservices.

## Problem Statement

We have deployed an Ingress Controller, resources, and applications in different namespaces. We need to:

1. Identify the namespaces and deployments
2. Configure path-based routing for applications
3. Modify existing routes
4. Add new routes for additional services
5. Configure cross-namespace routing for a critical service

## Environment Configuration

### Namespaces Structure

- `ingress-nginx`: Contains the Ingress Controller deployment
- `app-space`: Contains regular applications (wear, video, food)
- `critical-space`: Contains the payment application that requires special handling

### Initial Cluster Resources

```bash
NAMESPACE       NAME                                            READY   STATUS      RESTARTS   AGE
app-space       pod/default-backend-569f95b877-tp26f            1/1     Running     0          37s
app-space       pod/webapp-video-7d6646445c-zszk7               1/1     Running     0          37s
app-space       pod/webapp-wear-7cf6df9954-4j8fx                1/1     Running     0          37s
ingress-nginx   pod/ingress-nginx-admission-create-sd4c4        0/1     Completed   0          35s
ingress-nginx   pod/ingress-nginx-admission-patch-vptzb         0/1     Completed   0          35s
ingress-nginx   pod/ingress-nginx-controller-6c4c749b95-7zvfp   1/1     Running     0          36s
kube-flannel    pod/kube-flannel-ds-bw8kk                       1/1     Running     0          3m15s
kube-system     pod/coredns-7484cd47db-b85rv                    1/1     Running     0          3m15s
kube-system     pod/coredns-7484cd47db-pdgpd                    1/1     Running     0          3m15s
kube-system     pod/etcd-controlplane                           1/1     Running     0          3m20s
kube-system     pod/kube-apiserver-controlplane                 1/1     Running     0          3m20s
kube-system     pod/kube-controller-manager-controlplane        1/1     Running     0          3m20s
kube-system     pod/kube-proxy-kbf5j                            1/1     Running     0          3m15s
kube-system     pod/kube-scheduler-controlplane                 1/1     Running     0          3m20s

NAMESPACE       NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
app-space       service/default-backend-service              ClusterIP   172.20.33.173   <none>        80/TCP                       37s
app-space       service/video-service                        ClusterIP   172.20.5.197    <none>        8080/TCP                     37s
app-space       service/wear-service                         ClusterIP   172.20.193.22   <none>        8080/TCP                     37s
default         service/kubernetes                           ClusterIP   172.20.0.1      <none>        443/TCP                      3m22s
ingress-nginx   service/ingress-nginx-controller             NodePort    172.20.38.185   <none>        80:30080/TCP,443:32103/TCP   36s
ingress-nginx   service/ingress-nginx-controller-admission   ClusterIP   172.20.59.5     <none>        443/TCP                      36s
kube-system     service/kube-dns                             ClusterIP   172.20.0.10     <none>        53/UDP,53/TCP,9153/TCP       3m21s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   3m20s
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   3m21s

NAMESPACE       NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
app-space       deployment.apps/default-backend            1/1     1            1           37s
```

## Step 1: Exploring the environment

### Identifying Ingress Controller Namespace

```bash
k get pods -A
```

Output:

```bash
NAMESPACE       NAME                                        READY   STATUS      RESTARTS   AGE
app-space       default-backend-569f95b877-tp26f            1/1     Running     0          6m33s
app-space       webapp-video-7d6646445c-zszk7               1/1     Running     0          6m33s
app-space       webapp-wear-7cf6df9954-4j8fx                1/1     Running     0          6m33s
ingress-nginx   ingress-nginx-admission-create-sd4c4        0/1     Completed   0          6m31s
ingress-nginx   ingress-nginx-admission-patch-vptzb         0/1     Completed   0          6m31s
ingress-nginx   ingress-nginx-controller-6c4c749b95-7zvfp   1/1     Running     0          6m32s
```

From the output, we can identify that the Ingress Controller is deployed in the `ingress-nginx` namespace.

### Determining Ingress Controller Deployment Name

```bash
k get deploy -n ingress-nginx
```

Output:

```bash
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx-controller   1/1     1            1           10m
```

The Ingress Controller deployment is named `ingress-nginx-controller`.

### Identifying Application Namespace and Deployments

```bash
k get deploy -n app-space
```

Output:

```bash
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
default-backend   1/1     1            1           12m
webapp-video      1/1     1            1           12m
webapp-wear       1/1     1            1           12m
```

There are 3 deployments in the `app-space` namespace.

### Finding Ingress Resource Information

```bash
k get ingress -A
```

Output:

```bash
NAMESPACE   NAME                 CLASS    HOSTS   ADDRESS         PORTS   AGE
app-space   ingress-wear-watch   <none>   *       172.20.38.185   80      19m
```

The Ingress Resource is deployed in the `app-space` namespace with the name `ingress-wear-watch`.

### Examining Ingress Configuration

```bash
k describe ingress ingress-wear-watch -n app-space
```

Output:

```
Name:             ingress-wear-watch
Labels:           <none>
Namespace:        app-space
Address:          172.20.38.185
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /wear    wear-service:8080 (172.17.0.4:8080)
              /watch   video-service:8080 (172.17.0.5:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
              nginx.ingress.kubernetes.io/ssl-redirect: false
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    30m (x2 over 30m)  nginx-ingress-controller  Scheduled for sync
```

## Step 2: Modifying Routes

### Changing Video Application Path from /watch to /stream

We need to update the Ingress configuration to change the path for the video application from `/watch` to `/stream`.

Existing Ingress YAML:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: ingress-wear-watch
  namespace: app-space
spec:
  rules:
    - http:
        paths:
          - backend:
              service:
                name: wear-service
                port:
                  number: 8080
            path: /wear
            pathType: Prefix
          - backend:
              service:
                name: video-service
                port:
                  number: 8080
            path: /watch
            pathType: Prefix
```

Modified Ingress YAML (updating /watch to /stream):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: ingress-wear-watch
  namespace: app-space
spec:
  rules:
    - http:
        paths:
          - backend:
              service:
                name: wear-service
                port:
                  number: 8080
            path: /wear
            pathType: Prefix
          - backend:
              service:
                name: video-service
                port:
                  number: 8080
            path: /stream
            pathType: Prefix
```

### Adding Food Application Route at /eat

After creating a new food application, we need to add a new path to the Ingress configuration for the food service at `/eat`.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: ingress-wear-watch
  namespace: app-space
spec:
  rules:
    - http:
        paths:
          - backend:
              service:
                name: wear-service
                port:
                  number: 8080
            path: /wear
            pathType: Prefix
          - backend:
              service:
                name: video-service
                port:
                  number: 8080
            path: /stream
            pathType: Prefix
          - backend:
              service:
                name: food-service
                port:
                  number: 8080
            path: /eat
            pathType: Prefix
```

## Step 3: Handling Critical Payment Service

### Identifying the New Critical Service

```bash
k get all -A | grep critical
```

Output:

```bash
critical-space   pod/webapp-pay-7df499586f-w7rp4                 1/1     Running     0          89s
critical-space   service/pay-service                          ClusterIP   172.20.254.248   <none>        8282/TCP                     89s
critical-space   deployment.apps/webapp-pay                 1/1     1            1           89s
critical-space   replicaset.apps/webapp-pay-7df499586f                 1         1         1       89s
```

### Getting Information About the Payment Service

```bash
k get svc -n critical-space
```

Output:

```bash
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
pay-service   ClusterIP   172.20.254.248   <none>        8282/TCP   40m
```

### Creating Ingress for Payment Service

Since the payment service is in a different namespace, we need to create a separate Ingress resource for it.

```bash
k create ingress ingress-pay -n critical-space --rule="/pay=pay-service:8282"
```

However, this basic command doesn't include the necessary annotations. We need to create a proper YAML file:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-pay
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /pay
            pathType: Prefix
            backend:
              service:
                name: pay-service
                port:
                  number: 8282
```

Apply the configuration:

```bash
kubectl apply -f ingress-pay.yaml
```

### Verifying the Configuration

```bash
k get ingress -n critical-space
```

Output:

```bash
NAME          CLASS    HOSTS   ADDRESS        PORTS   AGE
ingress-pay   <none>   *       172.20.121.18  80      23s
```

```bash
k describe ingress ingress-pay -n critical-space
```

Output:

```
Name:             ingress-pay
Labels:           <none>
Namespace:        critical-space
Address:          172.20.121.18
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /pay   pay-service:8282 (172.17.0.11:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason  Age                 From                      Message
  ----    ------  ----                ----                      -------
  Normal  Sync    48s (x2 over 106s)  nginx-ingress-controller  Scheduled for sync
```

## Understanding the Rewrite Target Annotation

The `nginx.ingress.kubernetes.io/rewrite-target: /` annotation is crucial for proper routing:

When a request comes in to the Kubernetes cluster at path `/pay`, the Nginx Ingress Controller forwards this request to the backend service. Without the rewrite-target annotation:

1. User accesses: `http://your-ingress-ip/pay`
2. Request arrives at backend service as: `/pay`
3. But the application is expecting requests at: `/`

With the rewrite-target annotation:

1. User accesses: `http://your-ingress-ip/pay`
2. Ingress controller rewrites path to: `/`
3. Request arrives at backend service with path: `/`
4. The application handles it correctly since it's served at root

This way, applications don't need to be aware that they're being accessed through specific path prefixes in the ingress controller. They continue to work as if they're being accessed directly at the root path.

## Summary

This configuration establishes an Nginx Ingress Controller that routes traffic based on URL paths to different services:

1. `/wear` → `wear-service` in `app-space` namespace
2. `/stream` → `video-service` in `app-space` namespace
3. `/eat` → `food-service` in `app-space` namespace
4. `/pay` → `pay-service` in `critical-space` namespace

Requests that don't match any configured paths are sent to the default backend service.

The cross-namespace routing for the payment service demonstrates how to configure Ingress resources in different namespaces that work with the same Ingress Controller, allowing for better organization of services by their criticality or function.

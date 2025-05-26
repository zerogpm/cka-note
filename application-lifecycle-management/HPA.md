# Kubernetes Horizontal Pod Autoscaler (HPA) Exercise

## Problem Statement

**Original Question:** Create a Kubernetes deployment for the nginx application using the provided manifest file, then create an autoscaler with specific requirements and troubleshoot HPA issues.

**Multiple Choice Question:** We have a manifest file to create autoscaling for the Nginx deployment located at `/root/autoscale.yml`. Review the manifest file and identify the current replicas and desired replicas?

- A. Current replicas= 7, Desired replicas= 3
- B. Current replicas= 3, Desired replicas= 7
- C. Current replicas= 7, Desired replicas= 1
- **D. Current replicas= 0, Desired replicas= 0** ✅

## YAML Configuration Files

### 1. Initial Deployment Manifest (`/root/deployment.yml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 7
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
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

### 2. HorizontalPodAutoscaler Manifest (`/root/autoscale.yml`)

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: nginx-deployment
spec:
  maxReplicas: 3
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  targetCPUUtilizationPercentage: 80
status:
  currentReplicas: 0
  desiredReplicas: 0
```

### 3. Updated Deployment with Resource Requests

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-deployment","namespace":"default"},"spec":{"replicas":7,"selector":{"matchLabels":{"app":"nginx"}},"template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx","ports":[{"containerPort":80}]}]}}}}
  creationTimestamp: "2025-05-26T20:47:05Z"
  generation: 2
  labels:
    app: nginx
  name: nginx-deployment
  namespace: default
  resourceVersion: "5707"
  uid: 349ad853-7453-4c59-9b67-59392f6849bf
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx:1.14.2
          imagePullPolicy: IfNotPresent
          name: nginx
          ports:
            - containerPort: 80
              protocol: TCP
          resources: {} # This was updated to include CPU requests
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

## Step-by-Step Commands and Explanations

### Step 1: Create the Initial Deployment

```bash
kubectl apply -f /root/deployment.yml
```

**Purpose:** Creates a Kubernetes deployment with 7 nginx pod replicas using the provided manifest file.

### Step 2: Create the HorizontalPodAutoscaler

```bash
kubectl autoscale deployment nginx-deployment --max=3 --cpu-percent=80
```

**Purpose:** Creates an HPA that will automatically scale the nginx-deployment between 1 and 3 replicas based on CPU utilization target of 80%.

**Alternative:** You could also use `kubectl apply -f /root/autoscale.yml`

### Step 3: Check HPA Status

```bash
kubectl get hpa
```

**Output:**

```bash
NAME               REFERENCE                     TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   cpu: <unknown>/80%   1         3         3          4m20s
```

**Explanation:** The HPA shows `<unknown>/80%` for CPU targets, indicating it cannot retrieve current metrics.

### Step 4: Describe HPA for Detailed Information

```bash
kubectl describe hpa nginx-deployment
```

**Output:**

```bash
Name:                                                  nginx-deployment
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Mon, 26 May 2025 20:49:44 +0000
Reference:                                             Deployment/nginx-deployment
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  <unknown> / 80%
Min replicas:                                          1
Max replicas:                                          3
Deployment pods:                                       3 current / 3 desired
Conditions:
  Type           Status  Reason                   Message
  ----           ------  ------                   -------
  AbleToScale    True    SucceededGetScale        the HPA controller was able to get the target's current scale
  ScalingActive  False   FailedGetResourceMetric  the HPA was unable to compute the replica count: failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-647677fc66-svffr
Events:
  Type     Reason                        Age                    From                       Message
  ----     ------                        ----                   ----                       -------
  Normal   SuccessfulRescale             5m29s                  horizontal-pod-autoscaler  New size: 3; reason: Current number of replicas above Spec.MaxReplicas
  Warning  FailedGetResourceMetric       3m58s (x3 over 5m13s)  horizontal-pod-autoscaler  failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-647677fc66-sb76d
  Warning  FailedComputeMetricsReplicas  3m58s (x3 over 5m13s)  horizontal-pod-autoscaler  invalid metrics (1 invalid out of 1), first error is: failed to get cpu resource metric value: failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-647677fc66-sb76d
```

**Key Issue Identified:** Missing CPU resource requests in the container specification.

### Step 5: Update Deployment with Resource Requests

```bash
kubectl apply -f /root/deployment.yml
```

**Purpose:** Apply the updated deployment manifest that includes CPU resource requests, enabling HPA to function properly.

### Step 6: Monitor HPA Changes

```bash
kubectl get hpa --watch
```

**Output:**

```bash
NAME               REFERENCE                     TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   cpu: <unknown>/80%   1         3         3          7m39s
nginx-deployment   Deployment/nginx-deployment   cpu: <unknown>/80%   1         3         7          7m46s
nginx-deployment   Deployment/nginx-deployment   cpu: 0%/80%          1         3         3          8m1s
nginx-deployment   Deployment/nginx-deployment   cpu: 0%/80%          1         3         1          8m16s
```

**Explanation:** Shows the HPA transitioning from `<unknown>` to `0%` CPU utilization and automatically scaling down from 3 to 1 replica due to low CPU usage.

### Step 7: Check Scaling Events

```bash
kubectl events hpa nginx-deployment | grep -i "ScalingReplicaSet"
```

**Output:**

```bash
11m                      Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-647677fc66 from 0 to 7
9m2s                     Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-647677fc66 from 7 to 3
104s                     Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-7998fdcbb8 from 2 to 3
104s                     Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-647677fc66 from 7 to 6
104s                     Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-7998fdcbb8 from 0 to 2
104s                     Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-647677fc66 from 3 to 7
99s                      Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-7998fdcbb8 from 3 to 4
99s                      Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-647677fc66 from 6 to 5
99s                      Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-647677fc66 from 5 to 4
76s (x9 over 99s)        Normal    ScalingReplicaSet              Deployment/nginx-deployment                (combined from similar events): Scaled down replica set nginx-deployment-7998fdcbb8 from 3 to 1
```

### Step 8: Check Failed Resource Metric Events

```bash
kubectl events hpa nginx-deployment | grep -i "FailedGetResourceMetric"
```

**Output:**

```bash
8m35s (x3 over 9m50s)    Warning   FailedGetResourceMetric        HorizontalPodAutoscaler/nginx-deployment   failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-647677fc66-sb76d
7m20s (x2 over 9m5s)     Warning   FailedGetResourceMetric        HorizontalPodAutoscaler/nginx-deployment   failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-647677fc66-9ghmw
4m50s (x12 over 9m35s)   Warning   FailedGetResourceMetric        HorizontalPodAutoscaler/nginx-deployment   failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-647677fc66-svffr
```

## Authentication

### Skooner Access Token

```bash
cat /root/skooner-sa-token.txt
```

**Token:**

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IjNscW1zNElqdDZXa0xFZlZXR0REZVVoR21JY194WnFrbWU3YkFEcXlmc00ifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQ4Mjk1OTI1LCJpYXQiOjE3NDgyOTIzMjUsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiMzhhMjEwNWQtOThlOS00N2MzLTgxNTMtZTE4MDQzMjg0NGViIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJza29vbmVyLXNhIiwidWlkIjoiZmI2MTU4NGMtMGY4Ny00MWZkLTk2NGUtNDZiODI2ODRlN2RhIn19LCJuYmYiOjE3NDgyOTIzMjUsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpza29vbmVyLXNhIn0.f0yrGkbHa5_N_ttKAP2xQMaEMvy76yfY8km2L5zHDliG7gMcYftCYJbvHFBDZjbYI7Q2R9dYhcZW59T1agtH5ymCWxHWyK_ODBk_bRPciRTYVsYqaJkT55mJmOFC0sDavpotMtMO94Cqb1Bn6TRAbVyu1Xv2Dy20M8y8-npISAi4NgclRraba6e_zgm3atpQnElsP8g5nIfLsSKerR7XjNoQrUSBVGHxKHyV94meJHAxOPiGqa7antBmbi4-Pyd6B4eJGnmqlZ2NVHbMOE4AH7RfhHYGAE4SGh9L3-SQFeopQdoy8VXvDQRSt7wKmbXew2YpCoP2k3bYtPV56KmL4g
```

**Purpose:** This JWT token provides access to the Skooner monitoring tool for viewing Kubernetes cluster resources.

## Key Concepts and Solutions

### What is HPA?

**Primary Purpose:** The Horizontal Pod Autoscaler (HPA) automatically adjusts the number of pod replicas in a deployment, replication controller, or replica set based on observed metrics such as CPU utilization or custom metrics provided by the metrics server.

### Metrics Server

**Component Responsible:** The metrics-server is a cluster-wide aggregator of resource usage data, and it provides the metrics required by the HPA to make scaling decisions.

### Problem Analysis

#### Root Cause of `<unknown>/80%` Status

The HPA status shows `<unknown>/80%` for the CPU target because:

1. **Missing Resource Requests:** The original deployment didn't specify CPU resource requests in the container specification
2. **HPA Dependency:** HPA requires resource requests to calculate CPU utilization percentages
3. **Metrics Server Issue:** Without resource requests, the metrics server cannot determine the baseline for CPU utilization calculations

#### Solution Approach

1. **Resource Requests:** Updated the deployment manifest to include CPU resource requests
2. **HPA Functionality:** This enables HPA to calculate CPU utilization as a percentage of requested resources
3. **Automatic Scaling:** Once metrics are available, HPA can make informed scaling decisions

### Event Analysis

#### ScalingReplicaSet Events

**Meaning:** These events indicate that the deployment controller is scaling replica sets up or down based on:

- Initial deployment creation (0 → 7 replicas)
- HPA constraints (7 → 3 replicas due to maxReplicas limit)
- Rolling updates (creating new replica sets with updated resource specifications)
- CPU-based scaling decisions (scaling down to 1 replica due to low CPU usage)

#### FailedGetResourceMetric Events

**Cause:** The HPA cannot retrieve CPU utilization metrics because the nginx container doesn't have CPU resource requests defined, which are required for calculating CPU utilization percentages.

## Summary

This exercise demonstrates the complete lifecycle of deploying and troubleshooting a Kubernetes HPA:

1. **Initial Setup:** Deploy nginx with 7 replicas
2. **HPA Creation:** Set up autoscaling with max 3 replicas and 80% CPU target
3. **Problem Identification:** HPA cannot get metrics due to missing resource requests
4. **Root Cause Analysis:** Use `kubectl describe` to identify the specific issue
5. **Solution Implementation:** Update deployment with proper resource specifications
6. **Validation:** Monitor HPA behavior and scaling events

The key learning point is that HPA requires resource requests to function properly, as it calculates utilization as a percentage of requested resources rather than absolute values.

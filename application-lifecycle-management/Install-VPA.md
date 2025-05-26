# Kubernetes Vertical Pod Autoscaler (VPA) Setup and Troubleshooting

## Overview

This guide demonstrates how to set up the Kubernetes Vertical Pod Autoscaler (VPA) and resolve common issues related to pod replica constraints that prevent VPA from functioning properly.

## Original Problem Statement

**Question**: Clone the VPA Repository and Set Up the Vertical Pod Autoscaler. You are required to clone the Kubernetes Autoscaler repository into the /root directory and set up the Vertical Pod Autoscaler (VPA) by running the provided script.

**Additional Issue**: After deploying a Flask application with VPA, the vpa-updater pod shows an error: "too few replicas for ReplicaSet" which prevents VPA from optimizing resources.

## Prerequisites

- Kubernetes cluster with admin access
- kubectl configured and working
- Git installed

## Step 1: Clone and Set Up VPA

### Commands and Execution

```bash
# Navigate to root directory and clone the repository
git clone https://github.com/kubernetes/autoscaler.git
```

**Console Output:**

```bash
controlplane ~ ➜    git clone https://github.com/kubernetes/autoscaler.git
Cloning into 'autoscaler'...
remote: Enumerating objects: 223608, done.
remote: Counting objects: 100% (1465/1465), done.
remote: Compressing objects: 100% (1045/1045), done.
remote: Total 223608 (delta 917), reused 422 (delta 420), pack-reused 222143 (from 2)
Receiving objects: 100% (223608/223608), 247.62 MiB | 30.58 MiB/s, done.
Resolving deltas: 100% (144947/144947), done.
Updating files: 100% (8021/8021), done.
```

```bash
# Navigate to the VPA directory
cd autoscaler/vertical-pod-autoscaler
```

```bash
# Run the VPA setup script
./hack/vpa-up.sh
```

**Console Output:**

```bash
controlplane autoscaler/vertical-pod-autoscaler on  master via 🐹 ➜  ./hack/vpa-up.sh
HEAD is now at d0b04562c Merge pull request #8154 from raywainman/vpa-release-1.4
customresourcedefinition.apiextensions.k8s.io/verticalpodautoscalercheckpoints.autoscaling.k8s.io configured
customresourcedefinition.apiextensions.k8s.io/verticalpodautoscalers.autoscaling.k8s.io configured
clusterrole.rbac.authorization.k8s.io/system:metrics-reader unchanged
clusterrole.rbac.authorization.k8s.io/system:vpa-actor configured
clusterrole.rbac.authorization.k8s.io/system:vpa-status-actor unchanged
clusterrole.rbac.authorization.k8s.io/system:vpa-checkpoint-actor unchanged
clusterrole.rbac.authorization.k8s.io/system:evictioner unchanged
clusterrole.rbac.authorization.k8s.io/system:vpa-updater-in-place created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-updater-in-place-binding created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-reader unchanged
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-actor unchanged
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-status-actor unchanged
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-checkpoint-actor unchanged
clusterrole.rbac.authorization.k8s.io/system:vpa-target-reader unchanged
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-target-reader-binding unchanged
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-evictioner-binding unchanged
serviceaccount/vpa-admission-controller unchanged
serviceaccount/vpa-recommender unchanged
serviceaccount/vpa-updater unchanged
clusterrole.rbac.authorization.k8s.io/system:vpa-admission-controller configured
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-admission-controller unchanged
clusterrole.rbac.authorization.k8s.io/system:vpa-status-reader unchanged
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-status-reader-binding unchanged
role.rbac.authorization.k8s.io/system:leader-locking-vpa-updater created
rolebinding.rbac.authorization.k8s.io/system:leader-locking-vpa-updater created
role.rbac.authorization.k8s.io/system:leader-locking-vpa-recommender created
rolebinding.rbac.authorization.k8s.io/system:leader-locking-vpa-recommender created
deployment.apps/vpa-updater created
deployment.apps/vpa-recommender created
Generating certs for the VPA Admission Controller in /tmp/vpa-certs.
Certificate request self-signature ok
subject=CN = vpa-webhook.kube-system.svc
Uploading certs to the cluster.
error: failed to create secret secrets "vpa-tls-certs" already exists
service/vpa-webhook created
deployment.apps/vpa-admission-controller created
```

### Step-by-Step Explanation

1. **Repository Cloning**: Downloads the complete Kubernetes autoscaler project containing VPA components
2. **Directory Navigation**: Moves into the specific VPA subdirectory
3. **Script Execution**: Runs the automated setup script that:
   - Creates Custom Resource Definitions (CRDs)
   - Sets up RBAC permissions
   - Deploys VPA components
   - Generates TLS certificates for the admission controller

## Step 2: Verify VPA Installation

### Check VPA Pods

```bash
kubectl get pods --all-namespaces
```

**Console Output:**

```bash
controlplane autoscaler/vertical-pod-autoscaler on  HEAD (d0b0456) via 🐹 ➜  k get pods --all-namespaces
NAMESPACE      NAME                                        READY   STATUS    RESTARTS      AGE
kube-flannel   kube-flannel-ds-7nbqd                       1/1     Running   0             35m
kube-flannel   kube-flannel-ds-k82zd                       1/1     Running   0             34m
kube-system    coredns-768b85b76f-6frx2                    1/1     Running   0             35m
kube-system    coredns-768b85b76f-gx4bq                    1/1     Running   0             35m
kube-system    etcd-controlplane                           1/1     Running   0             35m
kube-system    kube-apiserver-controlplane                 1/1     Running   0             35m
kube-system    kube-controller-manager-controlplane        1/1     Running   0             35m
kube-system    kube-proxy-jmkgw                            1/1     Running   0             35m
kube-system    kube-proxy-zt4cz                            1/1     Running   0             34m
kube-system    kube-scheduler-controlplane                 1/1     Running   0             35m
kube-system    vpa-admission-controller-64d9db488b-l77hh   0/1     Error     2 (18s ago)   25s
kube-system    vpa-recommender-76d8f55f9c-nt4sn            1/1     Running   0             26s
kube-system    vpa-updater-5d94c78548-g9mj6                1/1     Running   0             26s
```

### Check VPA Custom Resource Definitions

```bash
kubectl get crds | grep verticalpodautoscaler
```

**Console Output:**

```bash
controlplane autoscaler/vertical-pod-autoscaler on  HEAD (d0b0456) via 🐹 ➜  kubectl get crds | grep verticalpodautoscaler
verticalpodautoscalercheckpoints.autoscaling.k8s.io   2025-05-26T21:09:53Z
verticalpodautoscalers.autoscaling.k8s.io             2025-05-26T21:09:53Z
```

### Check VPA Deployments

```bash
kubectl get deployments -n kube-system | grep vpa
```

**Console Output:**

```bash
controlplane autoscaler/vertical-pod-autoscaler on  HEAD (d0b0456) via 🐹 ➜  kubectl get deployments -n kube-system | grep vpa
vpa-admission-controller   0/1     1            0           112s
vpa-recommender            1/1     1            1           113s
vpa-updater                1/1     1            1           113s
```

## VPA Components Overview

The Vertical Pod Autoscaler consists of **3 main components**:

### 1. Recommender

- **Role**: Monitors current and past resource consumption (CPU and memory) of containers
- **Functionality**: Provides recommended values for containers' CPU and memory requests based on observed usage
- **Purpose**: Ensures optimal resource allocation to avoid over-provisioning and under-provisioning

### 2. Updater

- **Role**: Ensures running pods have correct resource requests per Recommender's suggestions
- **Functionality**: Evicts pods with outdated resource settings so they can be recreated with updated requests
- **Purpose**: Maintains optimal resource allocation for running pods

### 3. Admission Plugin (Admission Controller)

- **Role**: Sets correct resource requests on new pods during creation or recreation
- **Functionality**: Modifies pod resource requests to reflect recommended values during pod creation
- **Purpose**: Ensures new pods start with optimal resource requests

## Step 3: Deploy Flask Application with VPA

### Deploy Application

```bash
kubectl apply -f /root/flask-app.yml
```

**Console Output:**

```bash
controlplane autoscaler/vertical-pod-autoscaler on  HEAD (d0b0456) via 🐹 ➜  kubectl apply -f /root/flask-app.yml
deployment.apps/flask-app created
service/flask-app-service created
verticalpodautoscaler.autoscaling.k8s.io/flask-app created
```

### Verify Deployment

```bash
kubectl get pods
```

**Console Output:**

```bash
controlplane autoscaler/vertical-pod-autoscaler on  HEAD (d0b0456) via 🐹 ➜  k get pods
NAME                         READY   STATUS    RESTARTS   AGE
flask-app-67b666c5fc-4qqjr   1/1     Running   0          16s
```

## Step 4: Identify and Resolve VPA Issue

### Problem Discovery

Check VPA updater logs to identify issues:

```bash
kubectl logs $(kubectl get pods -n kube-system --no-headers -o custom-columns=":metadata.name" | grep vpa-updater) -n kube-system
```

**Expected Error Output:**

```bash
pods_eviction_restriction.go:226] too few replicas for ReplicaSet default/flask-app-b6c9c4f78. Found 1 live pods, needs 2 (global 2)
```

### Problem Analysis

**Root Cause**:

- Flask application runs with only 1 replica pod
- VPA needs to evict the existing pod to create one with updated resource settings
- Kubernetes safety feature prevents removing the last pod to avoid service downtime
- When only 1 replica exists, VPA cannot evict it, blocking resource optimization

### Solution Implementation

#### 1. Increase Replica Count

```bash
kubectl scale deployment flask-app --replicas=2
```

**Explanation**: This command increases the number of pod replicas from 1 to 2, allowing VPA to safely evict one pod while keeping the service running.

#### 2. Verify Deployment Update

```bash
kubectl get deployment flask-app -o wide
```

**Expected Result**: DESIRED column should show 2, and CURRENT column should match.

#### 3. Check Pod Status

```bash
kubectl get pods -l app=flask-app
```

**Expected Result**: Two pods in Running status.

#### 4. Verify VPA Operation

```bash
kubectl describe vpa flask-app
```

**Expected Result**: Should show resource recommendations for CPU and memory.

## Why This Approach Solves the Problem

### Safety Mechanism

Kubernetes implements a **pod disruption budget** safety mechanism that prevents the removal of the last remaining pod in a deployment to ensure service availability.

### VPA Operation Requirements

- **Eviction Process**: VPA updater needs to evict pods to apply new resource recommendations
- **High Availability**: With multiple replicas, VPA can safely evict one pod while others continue serving traffic
- **Zero Downtime**: The approach maintains service availability during resource optimization

### Resource Optimization Benefits

1. **Automatic Scaling**: VPA continuously monitors and adjusts resource requests
2. **Cost Efficiency**: Prevents over-provisioning of resources
3. **Performance Optimization**: Ensures adequate resources for application needs
4. **Operational Simplicity**: Reduces manual resource tuning requirements

## Key Takeaways

- **VPA requires at least 2 replicas** to function properly in most scenarios
- **Safety first**: Kubernetes prioritizes service availability over resource optimization
- **Monitoring is crucial**: Always check VPA logs to ensure proper operation
- **Resource recommendations**: VPA provides ongoing optimization based on actual usage patterns

## Troubleshooting Commands

```bash
# Check VPA status
kubectl get vpa

# View VPA recommendations
kubectl describe vpa <vpa-name>

# Monitor VPA logs
kubectl logs -n kube-system -l app=vpa-updater

# Check pod resource usage
kubectl top pods
```

This setup ensures that your Kubernetes applications benefit from automatic resource optimization while maintaining high availability and service reliability.

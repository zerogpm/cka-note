# Kubernetes Metrics Server Deployment and Monitoring

## Overview

This guide demonstrates how to deploy the Kubernetes Metrics Server and use it to monitor resource consumption across nodes and pods in a Kubernetes cluster. The Metrics Server provides container resource metrics for use by autoscaling pipelines and monitoring tools.

## Problem Statement

The original task was to:

- Deploy the Metrics Server in a Kubernetes cluster
- Monitor CPU and memory usage of nodes and pods
- Identify the highest resource-consuming components

## Prerequisites

- Access to a Kubernetes cluster with `kubectl` configured
- Appropriate RBAC permissions to deploy cluster-wide resources

## Step-by-Step Implementation

### Step 1: Deploy the Metrics Server

Deploy the latest Metrics Server components using the official manifest:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

**Expected Output:**

```bash
controlplane ~ ➜  kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```

#### What This Command Does:

- Creates a dedicated service account for the Metrics Server
- Sets up necessary RBAC permissions (ClusterRoles and bindings)
- Deploys the Metrics Server as a Deployment
- Creates a Service to expose the Metrics Server
- Registers the Metrics API with the Kubernetes API server

### Step 2: Wait for Metrics Collection

The Metrics Server needs time to start collecting data from the cluster nodes.

```bash
kubectl top node
```

**Initial Output (when metrics not ready):**

```bash
controlplane ~ ➜  k top node
error: Metrics API not available
```

**After waiting (successful output):**

```bash
controlplane ~ ✖ k top node
NAME           CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
controlplane   331m         2%       892Mi           1%
node01         27m          0%       178Mi           0%
```

### Step 3: Identify Highest CPU Consuming Node

Sort nodes by CPU usage and display the top consumer:

```bash
kubectl top node --sort-by='cpu' --no-headers | head -1
```

**Output:**

```bash
controlplane ~ ➜  kubectl top node --sort-by='cpu' --no-headers | head -1
controlplane   304m   1%    894Mi   1%
```

#### Command Breakdown:

- `--sort-by='cpu'`: Sorts output by CPU usage (descending)
- `--no-headers`: Removes column headers from output
- `head -1`: Shows only the first line (highest CPU consumer)

### Step 4: Identify Highest Memory Consuming Node

Sort nodes by memory usage and display the top consumer:

```bash
kubectl top node --sort-by='memory' --no-headers | head -1
```

**Output:**

```bash
controlplane   311m   1%    900Mi   1%
```

#### Command Breakdown:

- `--sort-by='memory'`: Sorts output by memory usage (descending)
- `--no-headers`: Removes column headers from output
- `head -1`: Shows only the first line (highest memory consumer)

### Step 5: Identify Highest Memory Consuming Pod

Find the pod with highest memory usage in the default namespace:

```bash
kubectl top pod --sort-by='memory' --no-headers | head -1
```

**Alternative command shown (CPU sorting):**

```bash
kubectl top pod --sort-by='cpu' --no-headers | tail -1
```

**Output:**

```bash
controlplane ~ ➜  kubectl top pod --sort-by='cpu' --no-headers | tail -1
lion       1m     16Mi
```

## Key YAML Configuration

The Metrics Server deployment uses the official `components.yaml` manifest which includes:

### Service Account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: kube-system
```

### RBAC Configuration

The deployment creates several ClusterRoles and bindings:

- `system:aggregated-metrics-reader`
- `system:metrics-server`
- `metrics-server:system:auth-delegator`

### Deployment

The Metrics Server runs as a Deployment in the `kube-system` namespace with:

- Resource requests and limits
- Security contexts
- Liveness and readiness probes

### API Service Registration

```yaml
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  name: v1beta1.metrics.k8s.io
spec:
  service:
    name: metrics-server
    namespace: kube-system
```

## Results Summary

From the monitoring commands, we identified:

| Metric                  | Resource     | Value           |
| ----------------------- | ------------ | --------------- |
| **Highest CPU Node**    | controlplane | 304m cores (1%) |
| **Highest Memory Node** | controlplane | 900Mi (1%)      |
| **Highest CPU Pod**     | lion         | 1m cores        |
| **Pod Memory Usage**    | lion         | 16Mi            |

## Why This Solution Works

### 1. **Official Source**

Using the official GitHub release ensures compatibility and includes all necessary components.

### 2. **Complete RBAC Setup**

The manifest includes all required permissions for the Metrics Server to:

- Read node and pod metrics
- Register with the Kubernetes API
- Serve metrics to clients

### 3. **Automated Resource Discovery**

The Metrics Server automatically discovers and monitors all nodes and pods in the cluster.

### 4. **Standard Kubernetes API**

Once deployed, metrics are available through standard `kubectl top` commands, making it easy to integrate with monitoring and autoscaling systems.

### 5. **Efficient Querying**

The sorting and filtering commands allow quick identification of resource hotspots without manual analysis.

## Troubleshooting

### Common Issues:

1. **"Metrics API not available"** - Wait 2-3 minutes for the server to start collecting data
2. **No output from `kubectl top`** - Check if Metrics Server pods are running: `kubectl get pods -n kube-system | grep metrics-server`
3. **Permission errors** - Ensure you have cluster-admin privileges to deploy RBAC resources

## Additional Commands

Monitor Metrics Server status:

```bash
kubectl get deployment metrics-server -n kube-system
kubectl get pods -n kube-system -l k8s-app=metrics-server
```

View Metrics Server logs:

```bash
kubectl logs -n kube-system deployment/metrics-server
```

# Kubernetes DaemonSet Analysis and FluentD Deployment

## Original Problem Statement

**How many DaemonSets are created in the cluster in all namespaces? Check all namespaces**

**Which of the below is a DaemonSet?**

- kube-flannel-ds

**Deploy a DaemonSet for FluentD Logging using the given specifications:**

- Name: elasticsearch
- Namespace: kube-system
- Image: registry.k8s.io/fluentd-elasticsearch:1.20

## Solution Overview

This document covers the analysis of existing DaemonSets in a Kubernetes cluster and the deployment of a new FluentD logging DaemonSet.

## Step 1: Analyzing Existing DaemonSets

### Command Used

```bash
kubectl get daemonsets --all-namespaces
```

### Console Output

```bash
controlplane ~ ➜  kubectl get daemonsets --all-namespaces
NAMESPACE      NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   kube-flannel-ds   1         1         1       1            1           <none>                   12m
kube-system    kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   12m
```

### Analysis Results

**Answer to "How many DaemonSets are created in the cluster?"**

- **Total DaemonSets: 2**
  1. `kube-flannel-ds` in the `kube-flannel` namespace
  2. `kube-proxy` in the `kube-system` namespace

**Answer to "Which of the below is a DaemonSet?"**

- **kube-flannel-ds** ✅ (Confirmed as a DaemonSet from the output above)

### Command Explanation

The `kubectl get daemonsets --all-namespaces` command:

- **`kubectl get daemonsets`**: Retrieves DaemonSet resources
- **`--all-namespaces`**: Searches across all namespaces in the cluster (not just the current/default namespace)
- **Output columns**:
  - `NAMESPACE`: The namespace where the DaemonSet is deployed
  - `NAME`: The name of the DaemonSet
  - `DESIRED`: Number of nodes that should be running the pod
  - `CURRENT`: Number of nodes currently running the pod
  - `READY`: Number of pods that are ready
  - `UP-TO-DATE`: Number of pods that are up to date
  - `AVAILABLE`: Number of pods available
  - `NODE SELECTOR`: Node selection criteria (if any)
  - `AGE`: How long the DaemonSet has been running

## Step 2: Deploying FluentD DaemonSet

### YAML Configuration

```yaml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
        - image: registry.k8s.io/fluentd-elasticsearch:1.20
          name: fluentd-elasticsearch
```

### Deployment Commands

1. **Save the YAML configuration to a file:**

   ```bash
   cat > fluentd-daemonset.yaml << EOF
   ---
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     labels:
       app: elasticsearch
     name: elasticsearch
     namespace: kube-system
   spec:
     selector:
       matchLabels:
         app: elasticsearch
     template:
       metadata:
         labels:
           app: elasticsearch
       spec:
         containers:
           - image: registry.k8s.io/fluentd-elasticsearch:1.20
             name: fluentd-elasticsearch
   EOF
   ```

2. **Apply the DaemonSet configuration:**

   ```bash
   kubectl apply -f fluentd-daemonset.yaml
   ```

3. **Verify the deployment:**

   ```bash
   kubectl get daemonsets -n kube-system
   ```

4. **Check the pods created by the DaemonSet:**
   ```bash
   kubectl get pods -n kube-system -l app=elasticsearch
   ```

## YAML Configuration Breakdown

### Metadata Section

- **`name: elasticsearch`**: The name of the DaemonSet resource
- **`namespace: kube-system`**: Deploys in the system namespace alongside other cluster services
- **`labels: app: elasticsearch`**: Labels for resource identification and organization

### Spec Section

- **`selector.matchLabels`**: Defines how the DaemonSet identifies which pods it manages
- **`template`**: Defines the pod template that will be created on each node

### Pod Template

- **`metadata.labels`**: Labels applied to each pod (must match the selector)
- **`spec.containers`**: Defines the container specification
  - **`image`**: Uses the FluentD Elasticsearch image version 1.20
  - **`name`**: Container name within the pod

## Why This Approach Solves the Problem

### DaemonSet Benefits for Logging

1. **Node Coverage**: DaemonSets ensure that exactly one pod runs on every node in the cluster, providing comprehensive log collection coverage
2. **Automatic Scaling**: When new nodes are added to the cluster, the DaemonSet automatically deploys FluentD pods to them
3. **System Integration**: Deploying in the `kube-system` namespace groups it with other system-level services
4. **Resource Efficiency**: One logging agent per node is more efficient than multiple logging pods per node

### FluentD for Elasticsearch Integration

- **Log Aggregation**: FluentD collects logs from all containers on each node
- **Elasticsearch Integration**: The `fluentd-elasticsearch` image is specifically configured to forward logs to Elasticsearch
- **Centralized Logging**: Enables centralized log storage and analysis through the Elasticsearch stack

## Verification Steps

After deployment, verify the solution:

1. **Check DaemonSet status:**

   ```bash
   kubectl get daemonsets --all-namespaces
   ```

2. **Verify pods are running on all nodes:**

   ```bash
   kubectl get pods -n kube-system -l app=elasticsearch -o wide
   ```

3. **Check DaemonSet events:**
   ```bash
   kubectl describe daemonset elasticsearch -n kube-system
   ```

This implementation provides a robust logging solution that automatically scales with your cluster and ensures comprehensive log collection from all nodes.

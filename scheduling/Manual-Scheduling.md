# Kubernetes Manual Pod Scheduling

This README explains how to manually schedule a Kubernetes pod when the scheduler component is missing or non-functional in a cluster.

## Problem Statement

The pod named "nginx" is stuck in a "Pending" state and cannot be scheduled automatically due to a missing scheduler component in the Kubernetes cluster.

Original console output showing the problem:

```
controlplane ~ ➜ k get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Pending   0          17s

controlplane ~ ➜
```

## Investigation

### Inspecting Control Plane Components

First, we check the status of the Kubernetes control plane components to identify any issues:

```bash
controlplane ~ ➜ k get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-7484cd47db-2xnq2               1/1     Running   0          3m22s
coredns-7484cd47db-g7ws7               1/1     Running   0          3m22s
etcd-controlplane                      1/1     Running   0          3m26s
kube-apiserver-controlplane            1/1     Running   0          3m26s
kube-controller-manager-controlplane   1/1     Running   0          3m26s
kube-proxy-gnbdv                       1/1     Running   0          2m52s
kube-proxy-w4pxh                       1/1     Running   0          3m22s
```

Upon inspection, we notice the `kube-scheduler` pod is missing from the kube-system namespace. The Kubernetes scheduler is responsible for assigning nodes to newly created pods. Without it, pods will remain in "Pending" state indefinitely.

### Checking Available Nodes

We need to verify what nodes are available in our cluster:

```bash
kubectl get nodes
```

This shows us the available nodes where we can manually schedule our pod.

## Solution: Manual Pod Scheduling

When the Kubernetes scheduler is missing or non-functional, we can manually schedule pods by specifying the `nodeName` field in the pod specification.

### Step 1: Delete the existing pending pod (if necessary)

```bash
kubectl delete pod nginx
```

### Step 2: Create or modify the pod YAML configuration with nodeName

Create a file named `nginx.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: node01 # Manually specifying the node
  containers:
    - image: nginx
      name: nginx
```

### Step 3: Apply the configuration to create the pod

```bash
kubectl apply -f nginx.yaml
```

### Step 4: Verify the pod is running

```bash
kubectl get pods
```

This should show the nginx pod as "Running" rather than "Pending".

## Why This Solution Works

1. **Root Cause**: The Kubernetes scheduler component is missing from the cluster, which prevents automatic pod scheduling.

2. **Solution Mechanism**: By explicitly specifying the `nodeName` field in the pod specification, we bypass the need for the scheduler. This tells Kubernetes exactly which node should run the pod.

3. **Key Points**:

   - The `nodeName` field directly assigns a pod to a specific node
   - This approach is useful in emergency situations when the scheduler is unavailable
   - It's a temporary solution; fixing or reinstalling the scheduler is recommended for normal operation

4. **Limitations**:
   - Manual scheduling doesn't take into account node resources, affinity rules, taints, or other scheduling constraints
   - You need to manually ensure the target node has sufficient resources
   - Each new pod must be manually assigned, which doesn't scale well

## Additional Information

In a production environment, you should:

1. Investigate why the scheduler is missing
2. Reinstall or fix the scheduler component
3. Use manual scheduling only as a temporary workaround

Manual scheduling is helpful for emergencies but is not recommended for regular operations in a Kubernetes cluster.

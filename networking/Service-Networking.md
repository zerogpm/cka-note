# Kubernetes Networking Analysis

This document explains the commands used to investigate the networking configuration of a Kubernetes cluster. We'll explore node networking, pod networking, service ranges, and various components like kube-proxy.

## Table of Contents

- [Node Network Range](#node-network-range)
- [Pod IP Range](#pod-ip-range)
- [Service IP Range](#service-ip-range)
- [Kube-Proxy Configuration](#kube-proxy-configuration)
- [Common Command Patterns](#common-command-patterns)

## Node Network Range

To determine the network range for nodes in the cluster, we examine both the node information and interface configuration.

```bash
# Get detailed node information including IP addresses
kubectl get nodes -o wide
```

This command shows the INTERNAL-IP and EXTERNAL-IP of each node. In our case, we see:

- controlplane: 192.6.173.11
- node01: 192.6.173.3

Next, we verify this on the host system:

```bash
# Show network interfaces
ip add
```

To filter for specific interfaces, we can use:

```bash
# Filter network interfaces with grep
# -A 3: show 3 lines after match
# -E: use extended regex
ip add | grep -A 3 -E "eth[0-9]+@[0-9a-z]+"
```

From the output, we can see the network interface has IP 192.6.173.11/24, confirming the nodes are on the 192.6.173.0/24 network.

## Pod IP Range

To determine the pod IP range, we need to check the CNI (Container Network Interface) configuration. In this cluster, we're using Weave Net.

First, let's list all resources to see the networking components:

```bash
# List all resources across all namespaces
kubectl get all -A
```

This shows the running weave-net pods (among other components). We can check the weave logs for the IP allocation range:

```bash
# Check weave logs for ipalloc-range setting
kubectl logs weave-net-jxcmw -n kube-system | grep -i ipalloc-range
```

The output confirms the pod IP range is 10.244.0.0/16, which is the CIDR block allocated for pods in this cluster.

## Service IP Range

The service IP range is defined in the kube-apiserver configuration. We can examine this from the apiserver manifest:

```bash
# View the kube-apiserver configuration
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Looking at the command arguments, we find the `--service-cluster-ip-range=10.96.0.0/12` parameter, which defines the range of IP addresses assigned to services.

## Kube-Proxy Configuration

### Counting kube-proxy Pods

To count the number of kube-proxy pods:

```bash
# Count kube-proxy pods
kubectl get all -A | grep -i pod/kube-proxy | wc -l
```

This command:

1. Lists all resources across all namespaces
2. Filters for lines containing "pod/kube-proxy"
3. Counts the resulting lines

From our output, we have 2 kube-proxy pods running.

### Proxy Mode

To determine the proxy mode used by kube-proxy:

```bash
# Check kube-proxy logs
kubectl logs kube-proxy-ndxkk -n kube-system
```

The logs would show the proxy mode (iptables, ipvs, etc.) configured for kube-proxy.

### Deployment Method

To see how kube-proxy is deployed in the cluster:

```bash
# Look at the resources
kubectl get all -A
```

From the output, we can see kube-proxy is deployed as a DaemonSet:

```
NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   36m
```

Using a DaemonSet ensures that one kube-proxy pod runs on each node in the cluster (that matches the node selector). The DaemonSet controller automatically deploys a kube-proxy pod to any new node added to the cluster, ensuring network policies and service proxying work on all nodes.

## Common Command Patterns

### Filtering and Counting

To count specific resources:

```bash
# Pattern: Get resources | Filter | Count
kubectl get all -A | grep PATTERN | wc -l
```

This is useful for:

- Counting specific pod types
- Finding resources matching certain criteria
- Quick verification of expected object counts

### Examining Logs

To investigate configurations:

```bash
# Check logs for specific configurations
kubectl logs POD_NAME -n NAMESPACE | grep SEARCH_TERM
```

This helps:

- Find specific configuration settings
- Troubleshoot issues
- Verify expected behaviors

### Getting Detailed Resource Information

```bash
# Get more information with -o wide flag
kubectl get RESOURCE -o wide
```

The `-o wide` flag provides additional details without the verbosity of `-o yaml`.

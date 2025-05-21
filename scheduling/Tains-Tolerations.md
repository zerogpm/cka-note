# Kubernetes Taints and Tolerations Demo

This repository demonstrates how taints and tolerations work in Kubernetes. Taints are properties added to nodes that allow them to repel certain pods, while tolerations are applied to pods to allow them to schedule on nodes with matching taints.

## Environment

- Kubernetes version: v1.32.0
- 2 nodes:
  - controlplane (control-plane node)
  - node01 (worker node)

## Problem Statement

The original task was to:

1. Check how many nodes exist in the system
2. Check for any taints on node01
3. Create a taint on node01
4. Create a pod without tolerations and observe its behavior
5. Create a pod with tolerations and verify it can be scheduled
6. Identify taints on the controlplane node
7. Remove taints from the controlplane node
8. Verify that pods can now schedule on the controlplane node

## Step-by-Step Walkthrough

### 1. Checking the Nodes in the System

```bash
controlplane ~ ➜ k get node
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   12m   v1.32.0
node01         Ready    <none>          11m   v1.32.0
```

**Explanation:** The command `k get node` (shorthand for `kubectl get node`) lists all nodes in the cluster. There are 2 nodes: the controlplane node and node01.

### 2. Checking for Taints on node01

```bash
controlplane ~ ➜ k describe node node01
```

**Output:**

```
Name:               node01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
...
Taints:             <none>
...
```

**Explanation:** The command `k describe node node01` shows detailed information about node01, including any taints. Initially, there are no taints on node01.

### 3. Creating a Taint on node01

```bash
kubectl taint nodes node01 spray=mortein:NoSchedule
```

**Explanation:** This command adds a taint to node01 with:

- Key: spray
- Value: mortein
- Effect: NoSchedule

The NoSchedule effect means that no pod will be scheduled on this node unless it has a matching toleration.

### 4. Creating a Pod without Tolerations

```bash
k run mosquito --image=nginx
```

**Explanation:** Creates a pod named "mosquito" using the nginx image without any tolerations.

**Checking the Pod Status:**

```bash
controlplane ~ ➜ k get pods
NAME       READY   STATUS    RESTARTS   AGE
mosquito   0/1     Pending   0          16s
```

**Investigating why the Pod is Pending:**

```bash
controlplane ~ ✖ k describe pod mosquito
```

**Output:**

```
Name:             mosquito
Namespace:        default
...
Status:           Pending
...
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  65s   default-scheduler  0/2 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 1 node(s) had untolerated taint {spray: mortein}. preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.
```

**Explanation:** The pod is in a Pending state because it can't be scheduled on any node:

- node01 has our custom taint `spray=mortein:NoSchedule`
- controlplane has the taint `node-role.kubernetes.io/control-plane:NoSchedule`
- The pod doesn't have tolerations for either taint

### 5. Creating a Pod with Tolerations

We create a YAML manifest for a pod that has the necessary toleration:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
    - image: nginx
      name: bee
  tolerations:
    - key: spray
      value: mortein
      effect: NoSchedule
      operator: Equal
```

**Explanation:** This YAML defines a pod named "bee" with a toleration that matches the taint on node01. The toleration allows the pod to be scheduled on a node with the taint `spray=mortein:NoSchedule`.

### 6. Checking for Taints on the Controlplane Node

```bash
controlplane ~ ✖ k describe node controlplane
```

**Output:**

```
Name:               controlplane
Roles:              control-plane
...
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
...
```

**Explanation:** The controlplane node has a taint `node-role.kubernetes.io/control-plane:NoSchedule`, which prevents regular pods from being scheduled on it. This is a default taint for control-plane nodes to ensure they're dedicated to control-plane components.

### 7. Removing the Taint from the Controlplane Node

```bash
k taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```

**Explanation:** This command removes the taint from the controlplane node. The `-` at the end of the taint specification indicates removal of the taint.

### 8. Checking the Status of the mosquito Pod

```bash
controlplane ~ ➜ k get pods
NAME       READY   STATUS    RESTARTS   AGE
bee        1/1     Running   0          4m17s
mosquito   1/1     Running   0          11m
```

**Explanation:** After removing the taint from the controlplane node, the mosquito pod is now in a Running state. This is because:

- The pod mosquito can now be scheduled on the controlplane node since the taint preventing scheduling has been removed
- The bee pod is running on node01 because it has a toleration for the taint on that node

## Summary

This demonstration shows how Kubernetes taints and tolerations work:

1. Taints are applied to nodes and prevent pods from being scheduled on them
2. Tolerations are applied to pods and allow them to be scheduled on nodes with matching taints
3. A pod without tolerations cannot be scheduled on a tainted node
4. A pod with the appropriate toleration can be scheduled on a tainted node
5. Removing a taint from a node allows pods without tolerations to be scheduled on it

Taints and tolerations are useful for:

- Dedicating nodes to specific workloads
- Preventing workloads from running on inappropriate nodes
- Setting up node affinity and anti-affinity

## Commands Reference

| Command                                                   | Description                                     |
| --------------------------------------------------------- | ----------------------------------------------- |
| `kubectl get node` or `k get node`                        | List all nodes in the cluster                   |
| `kubectl describe node <node-name>`                       | Show detailed information about a specific node |
| `kubectl taint nodes <node-name> <key>=<value>:<effect>`  | Add a taint to a node                           |
| `kubectl taint nodes <node-name> <key>=<value>:<effect>-` | Remove a taint from a node                      |
| `kubectl run <pod-name> --image=<image>`                  | Create a pod with the specified image           |
| `kubectl get pods`                                        | List all pods in the current namespace          |
| `kubectl describe pod <pod-name>`                         | Show detailed information about a specific pod  |

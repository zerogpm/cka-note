# Kubernetes Node Affinity Lab

This repository contains a hands-on exercise demonstrating Kubernetes node affinity concepts, including node labeling, deployment creation, and pod placement control using node affinity rules.

## Overview

This lab walks through the process of:

- Examining node labels in a Kubernetes cluster
- Adding custom labels to nodes
- Creating deployments with node affinity rules
- Controlling pod placement using node affinity

## Cluster Setup

The lab uses a 2-node Kubernetes cluster:

- `controlplane` - Control plane node with master components
- `node01` - Worker node

## Lab Exercises

### 1. Initial Node Inspection

**Question**: How many Labels exist on node node01?

**Command**:

```bash
kubectl describe node node01
```

**Output** (Labels section):

```
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
```

**Answer**: 5 labels exist on node01

### 2. Examining Specific Label Values

**Question**: What is the value set to the label key beta.kubernetes.io/arch on node01?

**Answer**: `amd64`

### 3. Adding Custom Labels

**Command**:

```bash
kubectl label node node01 color=blue
```

**Output**:

```
node/node01 labeled
```

**Verification**:

```bash
kubectl describe node node01
```

The Labels section now shows:

```
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    color=blue
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
```

### 4. Creating a Deployment

**Command**:

```bash
kubectl create deploy blue --image=nginx --replicas=3
```

### 5. Analyzing Pod Placement Options

**Question**: Which nodes can the pods for the blue deployment be placed on?

**Commands to check node taints**:

```bash
kubectl get nodes
kubectl describe node node01
kubectl describe node controlplane
```

**Node Status**:

```
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   18m   v1.32.0
node01         Ready    <none>          17m   v1.32.0
```

**Taint Analysis**:

- **node01**: `Taints: <none>` - No taints, accepts any pods
- **controlplane**: `Taints: <none>` - No taints, accepts any pods

**Answer**: Pods can be placed on both nodes since neither has taints that would prevent scheduling.

### 6. Implementing Node Affinity

**Objective**: Set Node Affinity to place pods on node01 only using the `color=blue` label.

**YAML Configuration**:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
        - image: nginx
          imagePullPolicy: Always
          name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: color
                    operator: In
                    values:
                      - blue
```

**Verification Command**:

```bash
kubectl get pods -o wide
```

**Output**:

```
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
blue-679c44c97c-5fdr7   1/1     Running   0          67s   172.17.1.6   node01   <none>           <none>
blue-679c44c97c-9rlpt   1/1     Running   0          67s   172.17.1.4   node01   <none>           <none>
blue-679c44c97c-kxfgx   1/1     Running   0          67s   172.17.1.5   node01   <none>           <none>
```

**Result**: All 3 pods are successfully scheduled on node01.

### 7. Creating a Control Plane Specific Deployment

**Objective**: Create a deployment named "red" that runs only on the controlplane node.

**YAML Configuration**:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
        - image: nginx
          imagePullPolicy: Always
          name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: node-role.kubernetes.io/control-plane
                    operator: Exists
```

## Key Concepts Explained

### Node Affinity Types

1. **requiredDuringSchedulingIgnoredDuringExecution**: Hard requirement - pods will not be scheduled if the rule isn't satisfied
2. **preferredDuringSchedulingIgnoredDuringExecution**: Soft requirement - scheduler tries to satisfy but will schedule elsewhere if needed

### Match Expressions Operators

- **In**: Label value must be in the specified list
- **NotIn**: Label value must not be in the specified list
- **Exists**: Label key must exist (value ignored)
- **DoesNotExist**: Label key must not exist

### Why This Approach Works

1. **Custom Labels**: By adding `color=blue` to node01, we created a unique identifier for targeting
2. **Built-in Labels**: Using `node-role.kubernetes.io/control-plane` leverages existing Kubernetes labels
3. **Node Affinity**: Provides more flexibility than nodeSelector with complex matching expressions
4. **Scheduling Control**: Ensures pods run only on intended nodes, useful for:
   - Hardware-specific workloads
   - Compliance requirements
   - Resource isolation
   - Performance optimization

## Best Practices

1. **Use Meaningful Labels**: Choose descriptive label keys and values
2. **Leverage Built-in Labels**: Utilize existing Kubernetes labels when possible
3. **Combine with Taints/Tolerations**: For stronger isolation guarantees
4. **Test Thoroughly**: Verify pod placement after applying affinity rules
5. **Documentation**: Keep track of custom labels and their purposes

## Troubleshooting

- **Pods Pending**: Check if node affinity rules can be satisfied
- **Uneven Distribution**: Consider using `podAntiAffinity` for spreading
- **Label Mismatches**: Verify label keys and values are correct
- **Operator Usage**: Ensure correct operators (`In`, `Exists`, etc.) are used

## Commands Reference

```bash
# View node labels
kubectl describe node <node-name>

# Add label to node
kubectl label node <node-name> <key>=<value>

# Remove label from node
kubectl label node <node-name> <key>-

# Check pod placement
kubectl get pods -o wide

# View deployment YAML
kubectl get deployment <deployment-name> -o yaml
```

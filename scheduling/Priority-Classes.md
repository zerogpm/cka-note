# Kubernetes PriorityClasses Lab

This lab demonstrates how to work with PriorityClasses in Kubernetes, including creating custom priority classes and understanding how pod scheduling works with different priorities.

## Table of Contents

- [Overview](#overview)
- [Default PriorityClasses](#default-priorityclasses)
- [Creating Custom PriorityClasses](#creating-custom-priorityclasses)
- [Creating Pods with PriorityClasses](#creating-pods-with-priorityclasses)
- [Pod Preemption Scenario](#pod-preemption-scenario)
- [Key Concepts](#key-concepts)

## Overview

PriorityClasses in Kubernetes allow you to define the relative importance of pods. When cluster resources are constrained, higher priority pods can preempt lower priority ones to ensure critical workloads get scheduled.

## Default PriorityClasses

### Question: Which of the following PriorityClasses are part of a default Kubernetes setup?

```bash
root@controlplane ~ ➜  k get priorityclass
NAME                      VALUE        GLOBAL-DEFAULT   AGE   PREEMPTIONPOLICY
system-cluster-critical   2000000000   false            10m   PreemptLowerPriority
system-node-critical      2000001000   false            10m   PreemptLowerPriority
```

**Answer:** The default Kubernetes setup includes:

- `system-cluster-critical` (value: 2000000000)
- `system-node-critical` (value: 2000001000)

### Question: What is the priority value assigned to system-node-critical?

**Answer:** 2000001000

### Question: What is the value of preemptionPolicy in system-node-critical?

**Answer:** PreemptLowerPriority

### Detailed View of Default PriorityClasses

```bash
root@controlplane ~ ➜  k describe priorityclass
system-node-critical
Name:              system-cluster-critical
Value:             2000000000
GlobalDefault:     false
PreemptionPolicy:  PreemptLowerPriority
Description:       Used for system critical pods that must run in the cluster, but can be moved to another node if necessary.
Annotations:       <none>
Events:            <none>


Name:              system-node-critical
Value:             2000001000
GlobalDefault:     false
PreemptionPolicy:  PreemptLowerPriority
Description:       Used for system critical pods that must not be moved from their current node.
Annotations:       <none>
Events:            <none>
-bash: system-node-critical: command not found
```

### YAML Output of system-node-critical

```yaml
root@controlplane ~ ✖ kubectl get priorityclass system-node-critical -o yaml
apiVersion: scheduling.k8s.io/v1
description: Used for system critical pods that must not be moved from their current
  node.
kind: PriorityClass
metadata:
  creationTimestamp: "2025-05-22T17:01:17Z"
  generation: 1
  name: system-node-critical
  resourceVersion: "69"
  uid: 3033d54f-85e2-4b05-b71e-7e7e246ea70e
preemptionPolicy: PreemptLowerPriority
value: 2000001000
```

## Creating Custom PriorityClasses

### Task 1: Create a high-priority PriorityClass

**Requirement:** Create a PriorityClass named high-priority, a value of 100000, and a preemption policy of PreemptLowerPriority. Do not set this class as a global default.

**Solution:**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
globalDefault: false
description: "This priority class is used for high-priority pods."
preemptionPolicy: PreemptLowerPriority
```

**Command to apply:**

```bash
kubectl apply -f high-priority.yaml
```

### Task 2: Create a low-priority PriorityClass

**Requirement:** Create another PriorityClass named low-priority with a value of 1000. Do not set this class as a global default.

**Solution:**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 1000
globalDefault: false
description: "This priority class is used for low-priority pods."
```

**Command to apply:**

```bash
kubectl apply -f low-priority.yaml
```

## Creating Pods with PriorityClasses

### Task 3: Create a low-priority pod

**Requirement:** In the default namespace, create a pod named low-prio-pod that runs an nginx image and uses the low-priority PriorityClass.

**Solution:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: low-prio-pod
spec:
  containers:
    - name: nginx
      image: nginx
  priorityClassName: low-priority
```

**Command to apply:**

```bash
kubectl apply -f low-prio-pod.yaml
```

### Task 4: Create a high-priority pod

**Requirement:** Now, create another pod in the default namespace named high-prio-pod that runs an nginx image and uses the high-priority PriorityClass.

**Solution:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-prio-pod
spec:
  containers:
    - name: nginx
      image: nginx
  priorityClassName: high-priority
```

**Command to apply:**

```bash
kubectl apply -f high-prio-pod.yaml
```

### Verifying Pod Priorities

You can compare the priority classes on both pods using the following command:

```bash
kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priorityClassName"
```

## Pod Preemption Scenario

### Initial State

The following assets have been provisioned on the environment:

- A pod named low-app
- A pod named critical-app

View the pod status using the following command:

```bash
root@controlplane ~ ➜  k get pods
NAME            READY   STATUS    RESTARTS   AGE
critical-app    0/1     Pending   0          22s
high-prio-pod   1/1     Running   0          57s
low-app         1/1     Running   0          22s
low-prio-pod    1/1     Running   0          2m49s
```

**Observation:** The critical-app is stuck in Pending state.

### Problem Analysis

The pod critical-app is stuck in Pending state due to the low-app pod getting scheduled and requesting high resources.

Check the status of the critical-app pod using the following command:

```bash
kubectl describe pod critical-app
```

You should see a similar message in the Events section:

```
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  21s   default-scheduler  0/1 nodes are available ...
```

### Solution

To resolve this situation and get the critical-app to a running state, you should:

1. Assign the high-priority class to the critical-app
2. Delete and recreate the pod with the new priority
3. Make sure the pod is in running state after applying the actions

**Updated critical-app.yaml:**

```yaml
# critical-app.yaml
apiVersion: v1
kind: Pod
metadata:
  ...
  name: critical-app
  ...
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: critical-container
    ...
  dnsPolicy: ClusterFirst
  priorityClassName: high-priority   # Add the high-priority class
  enableServiceLinks: true
  preemptionPolicy: PreemptLowerPriority
  # priority: 0  # Remove this line as this is the old default priority
  ...
```

**Commands to fix the issue:**

```bash
# Delete the existing pod
kubectl delete pod critical-app

# Apply the updated configuration
kubectl apply -f critical-app.yaml

# Verify the pod is now running
kubectl get pods
```

## Key Concepts

### PriorityClass Fields

- **value**: An integer that determines the priority. Higher numbers mean higher priority.
- **globalDefault**: If true, this priority class will be used for pods that don't specify a priorityClassName.
- **preemptionPolicy**: Determines if this pod can preempt other pods. Options:
  - `PreemptLowerPriority`: Can preempt pods with lower priority
  - `Never`: Will not preempt other pods
- **description**: Human-readable description of the priority class

### How Preemption Works

1. When a high-priority pod cannot be scheduled due to resource constraints
2. The scheduler looks for lower-priority pods that can be evicted
3. If evicting lower-priority pods would allow the high-priority pod to be scheduled, those pods are evicted
4. The high-priority pod is then scheduled on the freed resources

### Best Practices

1. **System Priority Classes**: Never use values above 1 billion as they're reserved for system components
2. **Priority Values**: Choose values that clearly differentiate between priority levels (e.g., 1000, 10000, 100000)
3. **Global Default**: Be cautious when setting a global default as it affects all pods without explicit priority
4. **Preemption Policy**: Use `PreemptLowerPriority` for critical workloads that must run
5. **Testing**: Always test priority configurations in non-production environments first

### Common Use Cases

- **Critical Applications**: Database pods, API gateways, monitoring systems
- **Batch Jobs**: Data processing, backups, reports (low priority)
- **Development Workloads**: Testing pods, development environments (low priority)
- **Production Services**: Customer-facing applications (high priority)

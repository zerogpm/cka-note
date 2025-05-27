# Kubernetes ReplicaSet Management

This repository demonstrates how to manage ReplicaSets in Kubernetes, including how to create, inspect, troubleshoot, and fix common issues. The exercises show how to handle image pull errors, apiVersion inconsistencies, and label mismatches.

## Table of Contents

- [Problem Statement](#problem-statement)
- [Examining ReplicaSets](#examining-replicasets)
- [Troubleshooting Pod Issues](#troubleshooting-pod-issues)
- [Understanding ReplicaSet Behavior](#understanding-replicaset-behavior)
- [Fixing YAML Configuration Files](#fixing-yaml-configuration-files)
- [Scaling ReplicaSets](#scaling-replicasets)

## Problem Statement

The original problem involves a series of exercises to check understanding of Kubernetes ReplicaSets:

1. Count the number of ReplicaSets in the default namespace
2. Examine the desired, current, and ready states of Pods in a ReplicaSet
3. Identify the image used by the Pods
4. Troubleshoot why Pods are not running correctly
5. Test ReplicaSet behavior when Pods are deleted
6. Fix and create ReplicaSets from YAML definition files
7. Scale a ReplicaSet to the desired number of Pods

## Examining ReplicaSets

### Check existing ReplicaSets

**Command:**

```bash
k get rs
```

**Output:**

```
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       2m58s
```

**Explanation:**
This command lists all ReplicaSets in the current namespace. We can see there is one ReplicaSet named `new-replica-set` that desires 4 Pods, has 4 Pods created (current), but none are ready.

### Examine the Pods

**Command:**

```bash
k get pods
```

**Output:**

```
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-b49st   0/1     ImagePullBackOff   0          2m51s
new-replica-set-kz2g6   0/1     ImagePullBackOff   0          2m51s
new-replica-set-rfm6t   0/1     ImagePullBackOff   0          2m51s
new-replica-set-svntr   0/1     ImagePullBackOff   0          2m51s
```

**Explanation:**
This shows the 4 Pods managed by the ReplicaSet, all in `ImagePullBackOff` status, meaning they are unable to pull their container images.

## Troubleshooting Pod Issues

### Inspect a Pod for more details

**Command:**

```bash
k describe pod new-replica-set-kz2g6
```

**Output (partial):**

```
Name:             new-replica-set-kz2g6
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.183.254
Start Time:       Tue, 20 May 2025 14:14:31 +0000
Labels:           name=busybox-pod
Annotations:      <none>
Status:           Pending
IP:               10.22.0.9
IPs:
  IP:           10.22.0.9
Controlled By:  ReplicaSet/new-replica-set
Containers:
  busybox-container:
    Container ID:
    Image:         busybox777
    Image ID:
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo Hello Kubernetes! && sleep 3600
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
```

**Events section from describe:**

```
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  6m7s                  default-scheduler  Successfully assigned default/new-replica-set-kz2g6 to controlplane
  Normal   Pulling    3m16s (x5 over 6m7s)  kubelet            Pulling image "busybox777"
  Warning  Failed     3m15s (x5 over 6m7s)  kubelet            Failed to pull image "busybox777": failed to pull and unpack image "docker.io/library/busybox777:latest": failed to resolve reference "docker.io/library/busybox777:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     3m15s (x5 over 6m7s)  kubelet            Error: ErrImagePull
  Normal   BackOff    55s (x21 over 6m6s)   kubelet            Back-off pulling image "busybox777"
  Warning  Failed     55s (x21 over 6m6s)   kubelet            Error: ImagePullBackOff
```

**Explanation:**
The `describe` command gives detailed information about the Pod. From this, we can see:

1. The Pod is using an image called `busybox777`
2. It's in a state of `ImagePullBackOff`
3. The Events section shows the image does not exist in the registry (`repository does not exist`)

This explains why all Pods are stuck - they're trying to use an image that doesn't exist.

## Understanding ReplicaSet Behavior

### Delete a Pod and observe behavior

**Command used (instead of deleting a single Pod):**

```bash
k delete rs new-replica-set
```

**Output:**

```
replicaset.apps "new-replica-set" deleted
```

**Check Pods after deletion:**

```bash
k get pods
```

**Output:**

```
No resources found in default namespace.
```

**Explanation:**
Instead of deleting a single Pod as requested in the original exercise, the entire ReplicaSet was deleted, which removed all Pods. In a proper scenario:

- If we had deleted a single Pod, the ReplicaSet controller would immediately create a new one to maintain the desired number of replicas (4)
- This is the self-healing mechanism of ReplicaSets
- Their purpose is to maintain a stable set of replica Pods running at any given time

## Fixing YAML Configuration Files

**Original content:**

```yaml
apiVersion: v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
```

**Fixed content:**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx
```

**Explanation:**
Two issues were fixed:

1. Changed `apiVersion: v1` to `apiVersion: apps/v1` (ReplicaSets belong to the apps API group)
2. Changed Pod template label from `tier: nginx` to `tier: frontend` to match the selector

**Apply the fixed configuration:**

```bash
k apply -f replicaset-definition-2.yaml
```

### Delete the ReplicaSets

**Commands:**

```bash
k delete rs replicaset-1
k delete rs replicaset-2
```

## Fixing the Original ReplicaSet

To fix the `new-replica-set` with the incorrect image, you would:

1. Edit the ReplicaSet to use the correct image

```bash
kubectl edit rs new-replica-set
```

Change `image: busybox777` to `image: busybox` in the editor

2. Delete the existing Pods so that new ones are created with the correct image

```bash
kubectl delete pods -l name=busybox-pod
```

## Scaling ReplicaSets

**Command:**

```bash
kubectl scale rs new-replica-set --replicas=5
```

**Explanation:**
This command increases the number of desired Pods in the ReplicaSet from 4 to 5, causing the controller to create one additional Pod.

## Why This Approach Solves the Original Problem

The exercises demonstrate several key concepts of Kubernetes ReplicaSets:

1. **Observability**: Using `kubectl get` and `kubectl describe` to observe the state of resources
2. **Troubleshooting**: Identifying why Pods were failing to start (incorrect image name)
3. **Self-healing**: Understanding how ReplicaSets maintain the desired number of Pods
4. **Configuration fixing**: Correcting common YAML errors like apiVersion and label mismatches
5. **Scaling**: Adjusting the number of replicas to meet demand

By going through these exercises, you gain practical experience with ReplicaSet management, which is a fundamental part of deploying resilient applications in Kubernetes. The approach shows that understanding the desired state versus actual state is crucial for debugging Kubernetes applications.

# Kubernetes Pod Management Exercises

This repository contains a series of Kubernetes exercises focused on basic pod management operations. The exercises cover pod creation, inspection, troubleshooting, modification, and deletion.

## Exercise Overview

These exercises guide you through:

1. Inspecting existing pods
2. Creating pods imperatively with `kubectl run`
3. Examining pod details with `kubectl describe`
4. Troubleshooting pod issues
5. Deleting pods
6. Editing pod configurations

## Prerequisites

- A running Kubernetes cluster
- kubectl CLI installed and configured

## Exercise Walkthrough

### 1. Checking Existing Pods

**Question:** How many pods exist on the system?

**Command:**

```bash
kubectl get pod
```

or using the short form:

```bash
k get pod
```

**Output:** (Output not provided in the original exercise)

This command lists all pods in the current namespace, showing their names, status, and age.

### 2. Creating a New Pod

**Question:** Create a new pod with the nginx image.

**Command:**

```bash
kubectl run nginx --image=nginx
```

This command creates a new pod named "nginx" using the nginx Docker image.

### 3. Verifying Pod Creation

**Question:** How many pods are created now?

**Command:**

```bash
kubectl get pod
```

**Answer:** 4

The output shows that there are now 4 pods in the default namespace.

### 4. Finding Pod Image Details

**Question:** What is the image used to create the new pods?

**Command:**

```bash
kubectl describe pod somepod
```

This command shows detailed information about a pod named "somepod", including the image used to create it.

### 5. Finding Node Placement

**Question:** Which nodes are these pods placed on?

**Command:**

```bash
kubectl describe pod somepod
```

This command shows detailed information about a pod, including the node it's running on. To find the node placement for all pods, you would need to run this command for each pod.

### 6. Examining Multi-Container Pods

**Question:** How many containers are part of the pod webapp?

**Command:**

```bash
kubectl describe pod webapp
```

**Output:**

```
Name:             webapp
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.20.165
Start Time:       Tue, 20 May 2025 00:06:42 +0000
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               10.22.0.13
IPs:
  IP:  10.22.0.13
Containers:
  nginx:
    Container ID:   containerd://54d469bda6e2047c282f2e5d2cca1355a272edad91a1c7ecb31929d276ac7f02
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:c15da6c91de8d2f436196f3a768483ad32c258ed4e1beb3d367a27ed67253e66
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 20 May 2025 00:06:43 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-s7k8t (ro)
  agentx:
    Container ID:
    Image:          agentx
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-s7k8t (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-s7k8t:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  70s                default-scheduler  Successfully assigned default/webapp to controlplane
  Normal   Pulling    69s                kubelet            Pulling image "nginx"
  Normal   Pulled     69s                kubelet            Successfully pulled image "nginx" in 154ms (154ms including waiting). Image size: 72404038 bytes.
  Normal   Created    69s                kubelet            Created container: nginx
  Normal   Started    69s                kubelet            Started container nginx
  Normal   Pulling    27s (x3 over 69s)  kubelet            Pulling image "agentx"
  Warning  Failed     27s (x3 over 69s)  kubelet            Failed to pull image "agentx": failed to pull and unpack image "docker.io/library/agentx:latest": failed to resolve reference "docker.io/library/agentx:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     27s (x3 over 69s)  kubelet            Error: ErrImagePull
  Normal   BackOff    0s (x5 over 68s)   kubelet            Back-off pulling image "agentx"
  Warning  Failed     0s (x5 over 68s)   kubelet            Error: ImagePullBackOff
```

**Answer:** 2

The webapp pod has two containers: "nginx" and "agentx".

### 7. Identifying Container Images

**Question:** What images are used in the new webapp pod?

**Answer:** nginx and agentx

As seen in the output above, the webapp pod uses the "nginx" and "agentx" images.

### 8. Checking Container State

**Question:** What is the state of the container agentx in the pod webapp?

**Answer:** error and waiting

The agentx container is in a "Waiting" state with a reason of "ImagePullBackOff".

### 9. Troubleshooting Container Issues

**Question:** Why do you think the container agentx in pod webapp is in error?

**Answer:** the image doesn't exist on dockerhub

From the events section, we can see:

```
Warning  Failed     27s (x3 over 69s)  kubelet            Failed to pull image "agentx": failed to pull and unpack image "docker.io/library/agentx:latest": failed to resolve reference "docker.io/library/agentx:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
```

### 10. Understanding Pod Status

**Question:** What does the READY column in the output of the kubectl get pods command indicate?

**Answer:** running containers in pod/total containers in pod

The READY column shows the number of ready containers compared to the total number of containers in the pod (e.g., "1/2").

### 11. Deleting a Pod

**Question:** Delete the webapp Pod. Once deleted, wait for the pod to fully terminate.

**Command:**

```bash
kubectl delete pod webapp
```

This command deletes the pod named "webapp".

### 12. Creating a Pod with a Specific Image

**Question:** Create a new pod with the name redis and the image redis123. Use a pod-definition YAML file. And yes the image name is wrong!

**Command:**

```bash
kubectl run redis --image=redis123
```

This creates a pod named "redis" using the image "redis123" (which is intentionally incorrect).

### 13. Editing a Pod's Image

**Question:** Now change the image on this pod to redis. Once done, the pod should be in a running state.

**Command:**

```bash
kubectl edit pod redis
```

This opens the pod configuration in an editor, where you can change the image from "redis123" to "redis".

## Pod YAML Configuration Example

For the Redis pod, the YAML configuration would look similar to this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
    - name: redis
      image: redis123 # This is intentionally wrong and needs to be changed to 'redis'
```

After editing with `kubectl edit pod redis`, the image should be changed to:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
    - name: redis
      image: redis # Changed from 'redis123' to 'redis'
```

## Key Kubernetes Concepts Demonstrated

1. **Pod Management**: Creating, inspecting, and deleting pods
2. **Multi-Container Pods**: Working with pods that have multiple containers
3. **Troubleshooting**: Identifying and resolving pod issues, especially related to container images
4. **Pod Configuration**: Editing pod configurations after creation
5. **Kubernetes CLI**: Using various kubectl commands to manage resources

## Common kubectl Commands Used

- `kubectl get pod` - List all pods
- `kubectl run <name> --image=<image>` - Create a new pod
- `kubectl describe pod <name>` - Show detailed information about a pod
- `kubectl delete pod <name>` - Delete a pod
- `kubectl edit pod <name>` - Edit a pod's configuration

## Why This Approach Works

The approach demonstrated in these exercises works effectively because:

1. It uses the imperative command line approach for quick pod creation and management
2. It leverages kubectl's powerful inspection capabilities to troubleshoot issues
3. It shows how to correct pod configurations after creation
4. It covers the entire lifecycle of a pod from creation to deletion
5. It demonstrates real-world troubleshooting for common pod issues like image problems

These commands and techniques are fundamental to day-to-day Kubernetes operations and form the foundation of more complex Kubernetes resource management.

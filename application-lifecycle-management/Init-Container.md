# Kubernetes InitContainer Troubleshooting Guide

This repository contains examples and solutions for working with Kubernetes initContainers, including identification, configuration, and troubleshooting common issues.

## Table of Contents

- [Overview](#overview)
- [Scenario 1: Identifying InitContainers](#scenario-1-identifying-initcontainers)
- [Scenario 2: Multiple InitContainers](#scenario-2-multiple-initcontainers)
- [Scenario 3: Configuring InitContainers](#scenario-3-configuring-initcontainers)
- [Scenario 4: Troubleshooting Failed InitContainers](#scenario-4-troubleshooting-failed-initcontainers)
- [Key Concepts](#key-concepts)
- [Best Practices](#best-practices)

## Overview

InitContainers are specialized containers that run before app containers in a Pod. They must complete successfully before the main containers start. This guide demonstrates common scenarios and troubleshooting techniques.

## Scenario 1: Identifying InitContainers

### Problem Statement

**Identify the pod that has an initContainer configured.**

### Solution

Use the `kubectl describe` command to examine pod details:

```bash
kubectl describe pod blue
```

### Console Output

```bash
controlplane ~ ➜  k describe pod blue
Name:             blue
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.129.238
Start Time:       Mon, 26 May 2025 18:11:59 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.22.0.11
IPs:
  IP:  10.22.0.11
Init Containers:
  init-myservice:
    Container ID:  containerd://6968a5fb0346109ef52f00748e37522f9d49d27ce27100a0bfefce8e6c4f831a
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:f64ff79725d0070955b368a4ef8dc729bd8f3d8667823904adcb299fe58fc3da
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 5
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Mon, 26 May 2025 18:12:01 +0000
      Finished:     Mon, 26 May 2025 18:12:06 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-p4j6t (ro)
Containers:
  green-container-1:
    Container ID:  containerd://d2fbd72b30e698be8bb88a0470abc4cdc5717b5ce2fc04b9448d4ccc39711076
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Running
      Started:      Mon, 26 May 2025 18:12:07 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-p4j6t (ro)
```

### Analysis Questions and Answers

**Q: What is the image used by the initContainer on the blue pod?**
**A:** `busybox`

**Q: What is the state of the initContainer on pod blue?**
**A:** `Terminated` with reason `Completed`

**Q: Why is the initContainer terminated? What is the reason?**
**A:** The initContainer completed successfully (Exit Code: 0) after running the `sleep 5` command. It terminated with reason "Completed", which is the expected behavior for a successful initContainer.

### Why This Approach Works

- The `kubectl describe` command provides comprehensive information about pod components
- InitContainers are clearly listed in a separate section from regular containers
- The state and reason fields indicate whether the initContainer completed successfully

## Scenario 2: Multiple InitContainers

### Problem Statement

**We just created a new app named purple. How many initContainers does it have?**

### Solution

```bash
kubectl describe pod purple
```

### Console Output

```bash
controlplane ~ ➜  k describe pod purple
Name:             purple
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.129.238
Start Time:       Mon, 26 May 2025 18:14:44 +0000
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               10.22.0.12
IPs:
  IP:  10.22.0.12
Init Containers:
  warm-up-1:
    Container ID:  containerd://0c6d74bbdcf49ff27dde76a97425c08f47d934e55171207e7e17a74c635124df
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 600
    State:          Running
      Started:      Mon, 26 May 2025 18:14:46 +0000
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gvh5x (ro)
  warm-up-2:
    Container ID:
    Image:         busybox:1.28
    Image ID:
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 1200
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gvh5x (ro)
```

### Analysis Questions and Answers

**Q: How many initContainers does the purple pod have?**
**A:** 2 initContainers (`warm-up-1` and `warm-up-2`)

**Q: What is the state of the POD?**
**A:** `Pending` - because the initContainers haven't completed yet

**Q: How long after the creation of the POD will the application come up and be available to users?**
**A:** The application will be available after both initContainers complete:

- `warm-up-1`: sleeps for 600 seconds (10 minutes)
- `warm-up-2`: sleeps for 1200 seconds (20 minutes)
- **Total time: 20 minutes** (since initContainers run sequentially)

### Why This Approach Works

- InitContainers run sequentially, not in parallel
- Each must complete successfully before the next one starts
- The main application container only starts after all initContainers complete

## Scenario 3: Configuring InitContainers

### Problem Statement

**Update the pod red to use an initContainer that uses the busybox image and sleeps for 20 seconds. Delete and re-create the pod if necessary. But make sure no other configurations change.**

### Solution

Create or update the pod configuration with an initContainer:

### YAML Configuration

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: red
  namespace: default
spec:
  containers:
    - command:
        - sh
        - -c
        - echo The app is running! && sleep 3600
      image: busybox:1.28
      name: red-container
  initContainers:
    - image: busybox
      name: red-initcontainer
      command:
        - "sleep"
        - "20"
```

### Commands to Apply

```bash
# Delete the existing pod
kubectl delete pod red

# Apply the new configuration
kubectl apply -f red-pod.yaml

# Verify the pod status
kubectl describe pod red
```

### Why This Approach Works

- The `initContainers` section is added to the pod spec
- The initContainer uses the specified `busybox` image
- The `sleep 20` command ensures the initContainer runs for exactly 20 seconds
- The main container configuration remains unchanged

## Scenario 4: Troubleshooting Failed InitContainers

### Problem Statement

**A new application orange is deployed. There is something wrong with it. Identify and fix the issue.**

### Step 1: Diagnose the Problem

```bash
kubectl describe pod orange
```

### Console Output

```bash
controlplane ~ ➜  k describe pod orange
Name:             orange
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.129.238
Start Time:       Mon, 26 May 2025 18:19:48 +0000
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               10.22.0.14
IPs:
  IP:  10.22.0.14
Init Containers:
  init-myservice:
    Container ID:  containerd://f2b97eb2a9aa26625efa189cfc846bb01fa2dcadfa2ea589d6a98ed8b42f782b
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:f64ff79725d0070955b368a4ef8dc729bd8f3d8667823904adcb299fe58fc3da
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleeeep 2;
    State:          Terminated
      Reason:       Error
      Exit Code:    127
      Started:      Mon, 26 May 2025 18:20:09 +0000
      Finished:     Mon, 26 May 2025 18:20:09 +0000
    Last State:     Terminated
      Reason:       Error
      Exit Code:    127
      Started:      Mon, 26 May 2025 18:19:51 +0000
      Finished:     Mon, 26 May 2025 18:19:51 +0000
    Ready:          False
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2jrxw (ro)
```

### Step 2: Identify the Issue

**Problem Identified:** The command contains a typo: `sleeeep 2` instead of `sleep 2`

- Exit Code 127 indicates "command not found"
- The initContainer keeps failing and restarting

### Step 3: Fix the Issue

```bash
# Get the current pod configuration
kubectl get pod orange -o yaml > orange-pod.yaml

# Edit the file to fix the typo
# Change "sleeeep 2" to "sleep 2" in the initContainer command

# Delete the existing pod
kubectl delete pod orange

# Apply the corrected configuration
kubectl apply -f orange-pod.yaml
```

### Corrected YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: orange
  namespace: default
spec:
  containers:
    - command:
        - sh
        - -c
        - echo The app is running! && sleep 3600
      image: busybox:1.28
      name: orange-container
  initContainers:
    - image: busybox
      name: init-myservice
      command:
        - sh
        - -c
        - sleep 2 # Fixed: was "sleeeep 2"
```

### Step 4: Verify the Fix

```bash
# Check pod status
kubectl get pods

# Verify the pod is running
kubectl describe pod orange
```

### Why This Approach Works

- Exit code 127 is a clear indicator of "command not found" errors
- The typo in the command (`sleeeep` instead of `sleep`) caused the failure
- Fixing the command spelling resolves the issue
- The pod can now complete initialization and start the main container

## Key Concepts

### InitContainer Behavior

1. **Sequential Execution**: InitContainers run one at a time, in the order specified
2. **Must Complete Successfully**: Each initContainer must exit with code 0 before the next starts
3. **Restart Policy**: Failed initContainers are restarted according to the pod's restart policy
4. **Resource Sharing**: InitContainers share volumes and network with the main containers

### Common Use Cases

- **Database Setup**: Initialize databases or run migrations
- **Configuration**: Download configuration files or certificates
- **Dependencies**: Wait for dependent services to become available
- **Security**: Set up security contexts or permissions

### Troubleshooting Tips

1. **Check Exit Codes**: Non-zero exit codes indicate failures
2. **Review Commands**: Syntax errors are common causes of failures
3. **Monitor Logs**: Use `kubectl logs <pod-name> -c <init-container-name>` for detailed logs
4. **Verify Images**: Ensure initContainer images are available and correct

## Best Practices

### Configuration

- Use specific image tags rather than `latest`
- Keep initContainer tasks simple and focused
- Set appropriate resource limits
- Use meaningful names for initContainers

### Debugging

- Always check the describe output for error details
- Pay attention to exit codes and restart counts
- Use logs to understand what's happening inside containers
- Test initContainer logic independently when possible

### Performance

- Minimize initContainer execution time
- Use efficient base images
- Consider parallelization where appropriate (though initContainers themselves run sequentially)

---

## Commands Reference

| Command                             | Purpose                                               |
| ----------------------------------- | ----------------------------------------------------- |
| `kubectl describe pod <name>`       | Get detailed pod information including initContainers |
| `kubectl logs <pod> -c <container>` | View logs for a specific initContainer                |
| `kubectl get pod <name> -o yaml`    | Export pod configuration to YAML                      |
| `kubectl delete pod <name>`         | Delete a pod                                          |
| `kubectl apply -f <file.yaml>`      | Apply configuration from file                         |

This guide demonstrates common scenarios when working with Kubernetes initContainers and provides practical solutions for identification, configuration, and troubleshooting.

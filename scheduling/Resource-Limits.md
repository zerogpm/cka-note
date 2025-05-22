# Kubernetes Pod Resource Management and Troubleshooting

This repository demonstrates how to identify CPU requirements and troubleshoot memory-related issues in Kubernetes pods, specifically dealing with Out of Memory (OOM) killed containers.

## Problem Statement

### Part 1: CPU Requirements Analysis

A pod called `rabbit` is deployed. Identify the CPU requirements set on the Pod in the current (default) namespace.

### Part 2: Memory Issue Troubleshooting

Another pod called `elephant` has been deployed in the default namespace. It fails to get to a running state. Inspect this pod and identify the reason why it is not running.

### Part 3: Resource Limit Adjustment

The elephant pod runs a process that consumes 15Mi of memory. Increase the limit of the elephant pod to 20Mi. Delete and recreate the pod if required. Do not modify anything other than the required fields.

## Solution

### Step 1: Analyzing CPU Requirements for the Rabbit Pod

**Command:**

```bash
kubectl describe pod rabbit
```

**Output:**

```bash
controlplane ~ ➜  k describe pod rabbit
Name:             rabbit
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.126.191
Start Time:       Thu, 22 May 2025 03:24:39 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.22.0.9
IPs:
  IP:  10.22.0.9
Containers:
  cpu-stress:
    Container ID:  containerd://d08e399eb4e0c9a4ed48e105dd9ccbe77cab87a471749f32de3bce5a77b223bf
    Image:         ubuntu
    Image ID:      docker.io/library/ubuntu@sha256:6015f66923d7afbc53558d7ccffd325d43b4e249f41a6e93eef074c9505d2233
    Port:          <none>
    Host Port:     <none>
    Args:
      sleep
      1000
    State:          Running
      Started:      Thu, 22 May 2025 03:24:41 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:  1
    Requests:
      cpu:        500m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-v6vgw (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-v6vgw:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  11m   default-scheduler  Successfully assigned default/rabbit to controlplane
  Normal  Pulling    11m   kubelet            Pulling image "ubuntu"
  Normal  Pulled     11m   kubelet            Successfully pulled image "ubuntu" in 1.532s (1.532s including waiting). Image size: 29726937 bytes.
  Normal  Created    11m   kubelet            Created container: cpu-stress
  Normal  Started    11m   kubelet            Started container cpu-stress
```

**Analysis:** The rabbit pod has the following CPU requirements:

- **CPU Requests:** 500m (0.5 CPU cores)
- **CPU Limits:** 1 (1 CPU core)

### Step 2: Diagnosing the Elephant Pod Issue

**Command:**

```bash
kubectl describe pod elephant
```

**Output:**

```bash
controlplane ~ ✖ k describe pod elephant
Name:             elephant
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.96.172
Start Time:       Thu, 22 May 2025 04:14:53 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.22.0.10
IPs:
  IP:  10.22.0.10
Containers:
  mem-stress:
    Container ID:  containerd://ad2ef06373d8e32c029069315a861befd871c6aefb194a6cd3f80a0603130b57
    Image:         polinux/stress
    Image ID:      docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa
    Port:          <none>
    Host Port:     <none>
    Command:
      stress
    Args:
      --vm
      1
      --vm-bytes
      15M
      --vm-hang
      1
    State:          Terminated
      Reason:       OOMKilled
      Exit Code:    1
      Started:      Thu, 22 May 2025 04:15:11 +0000
      Finished:     Thu, 22 May 2025 04:15:11 +0000
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    1
      Started:      Thu, 22 May 2025 04:14:55 +0000
      Finished:     Thu, 22 May 2025 04:14:55 +0000
    Ready:          False
    Restart Count:  2
    Limits:
      memory:  10Mi
    Requests:
      memory:     5Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-brnzc (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-brnzc:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  23s               default-scheduler  Successfully assigned default/elephant to controlplane
  Normal   Pulled     22s               kubelet            Successfully pulled image "polinux/stress" in 603ms (603ms including waiting). Image size: 4041495 bytes.
  Normal   Pulled     21s               kubelet            Successfully pulled image "polinux/stress" in 136ms (137ms including waiting). Image size: 4041495 bytes.
  Normal   Pulling    6s (x3 over 22s)  kubelet            Pulling image "polinux/stress"
  Normal   Created    6s (x3 over 22s)  kubelet            Created container: mem-stress
  Normal   Pulled     6s                kubelet            Successfully pulled image "polinux/stress" in 147ms (147ms including waiting). Image size: 4041495 bytes.
  Normal   Started    5s (x3 over 22s)  kubelet            Started container mem-stress
  Warning  BackOff    5s (x3 over 20s)  kubelet            Back-off restarting failed container mem-stress in pod elephant_default(11b9e5d3-be7f-42e1-9f13-eef1fdc941e0)
```

**Problem Identified:** The elephant pod is failing because of **OOMKilled** (Out of Memory). The container is trying to allocate 15Mi of memory (`--vm-bytes 15M`) but the memory limit is set to only 10Mi, causing the process to be killed by the kernel.

### Step 3: Fixing the Memory Limit Issue

#### Step 3.1: Export Current Pod Configuration

**Command:**

```bash
kubectl get po elephant -o yaml > elephant.yaml
```

**Explanation:** This command exports the current pod configuration to a YAML file for editing.

#### Step 3.2: Edit the YAML Configuration

Create or modify the `elephant.yaml` file with the corrected memory limits:

**File: elephant.yaml**

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: elephant
  namespace: default
spec:
  containers:
    - args:
        - --vm
        - "1"
        - --vm-bytes
        - 15M
        - --vm-hang
        - "1"
      command:
        - stress
      image: polinux/stress
      name: mem-stress
      resources:
        limits:
          memory: 20Mi
        requests:
          memory: 5Mi
```

**Key Changes:**

- **Memory Limit:** Increased from `10Mi` to `20Mi`
- **Memory Requests:** Kept at `5Mi` (unchanged)

#### Step 3.3: Replace the Pod

**Command:**

```bash
kubectl replace -f elephant.yaml --force
```

**Explanation:** This command forcefully deletes the existing pod and recreates it with the new configuration. The `--force` flag is necessary because you cannot directly update the resource limits of a running pod.

## Why This Approach Solves the Problem

### Root Cause Analysis

The original elephant pod was experiencing OOMKilled errors because:

1. **Memory Consumption vs. Limits Mismatch:** The stress test was trying to allocate 15Mi of memory (`--vm-bytes 15M`)
2. **Insufficient Memory Limit:** The pod had a memory limit of only 10Mi
3. **Kernel Intervention:** When the process exceeded the 10Mi limit, the Linux kernel's OOM killer terminated the container

### Solution Effectiveness

1. **Adequate Resource Allocation:** By increasing the memory limit to 20Mi, we provide sufficient headroom for the 15Mi memory allocation
2. **Safety Margin:** The 20Mi limit provides a 33% buffer above the required 15Mi, accounting for any additional memory overhead
3. **Maintained Requests:** Keeping the memory request at 5Mi ensures efficient scheduling while allowing burst usage up to the limit

### Resource Management Best Practices

- **Requests vs. Limits:**
  - Requests guarantee minimum resources for scheduling
  - Limits prevent runaway resource consumption
- **Monitoring:** Always monitor actual resource usage to set appropriate limits
- **Testing:** Test applications under load to determine realistic resource requirements

## Commands Summary

```bash
# Analyze CPU requirements
kubectl describe pod rabbit

# Diagnose pod issues
kubectl describe pod elephant

# Export pod configuration
kubectl get po elephant -o yaml > elephant.yaml

# Apply fixed configuration
kubectl replace -f elephant.yaml --force
```

## Files Involved

- `elephant.yaml` - Pod configuration with corrected memory limits

## Expected Results

After applying the fix, the elephant pod should:

- Successfully start and remain in Running state
- Complete the stress test without being OOMKilled
- Show `Ready: True` status for all containers

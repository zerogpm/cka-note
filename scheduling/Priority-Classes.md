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

# Understanding globalDefault in Kubernetes PriorityClasses

## Table of Contents

- [Original Question/Problem](#original-questionproblem)
- [What is globalDefault?](#what-is-globaldefault)
- [Hands-on Examples](#hands-on-examples)
  - [Example 1: Without Global Default](#example-1-without-global-default)
  - [Example 2: With Global Default](#example-2-with-global-default)
- [Real-world Scenario](#real-world-scenario)
- [Important Rules and Limitations](#important-rules-and-limitations)
- [Troubleshooting Common Issues](#troubleshooting-common-issues)

## Original Question/Problem

**Question:** "I still don't get **globalDefault**: what does it mean?"

This README explains the `globalDefault` field in Kubernetes PriorityClasses through practical examples and demonstrations.

## What is globalDefault?

The `globalDefault` field in a PriorityClass is a boolean that determines whether this PriorityClass should be automatically assigned to ALL pods that don't explicitly specify a `priorityClassName`.

### Key Points:

- **When `globalDefault: false`** (default): Pods must explicitly reference the PriorityClass to use it
- **When `globalDefault: true`**: This PriorityClass becomes the DEFAULT for all pods without a specified priorityClassName
- **Only ONE** PriorityClass in the entire cluster can have `globalDefault: true`

## Hands-on Examples

### Example 1: Without Global Default

Let's create a PriorityClass without global default and see how pods behave.

#### Step 1: Create a PriorityClass with globalDefault: false

**File: `medium-priority.yaml`**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: medium-priority
value: 5000
globalDefault: false # NOT a global default
description: "Medium priority class for regular workloads"
preemptionPolicy: PreemptLowerPriority
```

**Apply the PriorityClass:**

```bash
kubectl apply -f medium-priority.yaml
```

**Console Output:**

```
priorityclass.scheduling.k8s.io/medium-priority created
```

#### Step 2: Create a pod WITHOUT specifying priorityClassName

**File: `pod-without-priority.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-without-priority
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

**Apply the pod:**

```bash
kubectl apply -f pod-without-priority.yaml
```

**Console Output:**

```
pod/pod-without-priority created
```

#### Step 3: Check the pod's priority

```bash
kubectl get pod pod-without-priority -o jsonpath='{.spec.priority}'
```

**Console Output:**

```
0
```

**Explanation:** The pod has priority 0 because no global default is set, and we didn't specify a priorityClassName.

#### Step 4: Create a pod WITH priorityClassName

**File: `pod-with-priority.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-priority
spec:
  containers:
    - name: nginx
      image: nginx:latest
  priorityClassName: medium-priority # Explicitly using our PriorityClass
```

**Apply the pod:**

```bash
kubectl apply -f pod-with-priority.yaml
```

**Console Output:**

```
pod/pod-with-priority created
```

#### Step 5: Check this pod's priority

```bash
kubectl get pod pod-with-priority -o jsonpath='{.spec.priority}'
```

**Console Output:**

```
5000
```

**Explanation:** This pod has priority 5000 because we explicitly specified the medium-priority PriorityClass.

#### Step 6: Compare both pods

```bash
kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priority,PRIORITY_CLASS:.spec.priorityClassName"
```

**Console Output:**

```
NAME                   PRIORITY   PRIORITY_CLASS
pod-without-priority   0          <none>
pod-with-priority      5000       medium-priority
```

### Example 2: With Global Default

Now let's see what happens when we set a global default.

#### Step 1: Create a PriorityClass with globalDefault: true

**File: `baseline-priority.yaml`**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: baseline-priority
value: 2000
globalDefault: true # This IS a global default
description: "Default priority for all pods in the cluster"
preemptionPolicy: PreemptLowerPriority
```

**Apply the PriorityClass:**

```bash
kubectl apply -f baseline-priority.yaml
```

**Console Output:**

```
priorityclass.scheduling.k8s.io/baseline-priority created
```

#### Step 2: Create a new pod without specifying priorityClassName

**File: `pod-with-auto-priority.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-auto-priority
spec:
  containers:
    - name: nginx
      image: nginx:latest
  # Note: No priorityClassName specified
```

**Apply the pod:**

```bash
kubectl apply -f pod-with-auto-priority.yaml
```

**Console Output:**

```
pod/pod-with-auto-priority created
```

#### Step 3: Check the pod's priority and priorityClassName

```bash
kubectl get pod pod-with-auto-priority -o jsonpath='{.spec.priority}{"\n"}{.spec.priorityClassName}'
```

**Console Output:**

```
2000
baseline-priority
```

**Explanation:** Even though we didn't specify a priorityClassName, the pod automatically got:

- Priority value: 2000
- PriorityClassName: baseline-priority

#### Step 4: Verify all pods now

```bash
kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priority,PRIORITY_CLASS:.spec.priorityClassName"
```

**Console Output:**

```
NAME                     PRIORITY   PRIORITY_CLASS
pod-without-priority     0          <none>
pod-with-priority        5000       medium-priority
pod-with-auto-priority   2000       baseline-priority
```

**Note:** The first pod still has priority 0 because it was created before the global default was set. Only NEW pods get the global default.

## Real-world Scenario

### Setting Up a Production Cluster with Default Priorities

Let's implement a real-world scenario where we want:

- All pods to have at least priority 1000 by default
- Critical system pods to have priority 10000
- Batch jobs to have priority 100

#### Step 1: Create the global default for regular workloads

**File: `default-workload-priority.yaml`**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: default-workload-priority
value: 1000
globalDefault: true
description: "Default priority for all regular workloads"
preemptionPolicy: PreemptLowerPriority
```

```bash
kubectl apply -f default-workload-priority.yaml
```

#### Step 2: Create high priority for critical services

**File: `critical-priority.yaml`**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: critical-priority
value: 10000
globalDefault: false
description: "For critical system services"
preemptionPolicy: PreemptLowerPriority
```

```bash
kubectl apply -f critical-priority.yaml
```

#### Step 3: Create low priority for batch jobs

**File: `batch-priority.yaml`**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: batch-priority
value: 100
globalDefault: false
description: "For batch processing jobs"
preemptionPolicy: Never # Batch jobs should not preempt others
```

```bash
kubectl apply -f batch-priority.yaml
```

#### Step 4: Deploy different types of workloads

**Regular app (uses global default automatically):**

```yaml
# regular-app.yaml
apiVersion: v1
kind: Pod
metadata:
  name: regular-app
spec:
  containers:
    - name: app
      image: nginx:latest
  # No priorityClassName - will get default-workload-priority (1000)
```

**Critical database (explicitly high priority):**

```yaml
# critical-database.yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-database
spec:
  containers:
    - name: postgres
      image: postgres:latest
  priorityClassName: critical-priority # Explicitly set to 10000
```

**Batch job (explicitly low priority):**

```yaml
# batch-job.yaml
apiVersion: v1
kind: Pod
metadata:
  name: batch-processor
spec:
  containers:
    - name: processor
      image: busybox
      command: ["sh", "-c", "echo Processing batch job && sleep 3600"]
  priorityClassName: batch-priority # Explicitly set to 100
```

#### Step 5: Apply all workloads and verify

```bash
kubectl apply -f regular-app.yaml
kubectl apply -f critical-database.yaml
kubectl apply -f batch-job.yaml
```

```bash
kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priority,PRIORITY_CLASS:.spec.priorityClassName,STATUS:.status.phase"
```

**Console Output:**

```
NAME                PRIORITY   PRIORITY_CLASS              STATUS
regular-app         1000       default-workload-priority   Running
critical-database   10000      critical-priority          Running
batch-processor     100        batch-priority             Running
```

## Important Rules and Limitations

### 1. Only One Global Default

If you try to create a second PriorityClass with `globalDefault: true`:

**File: `another-default.yaml`**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: another-default
value: 3000
globalDefault: true
```

```bash
kubectl apply -f another-default.yaml
```

**Console Output (Error):**

```
The PriorityClass "another-default" is invalid: metadata.annotations: Forbidden: only one PriorityClass can be marked as globalDefault
```

### 2. Changing the Global Default

To change which PriorityClass is the global default:

```bash
# First, remove global default from current one
kubectl patch priorityclass default-workload-priority -p '{"globalDefault": false}'

# Then, set the new one as global default
kubectl patch priorityclass another-priority -p '{"globalDefault": true}'
```

### 3. Viewing Current Global Default

```bash
kubectl get priorityclass -o custom-columns="NAME:.metadata.name,VALUE:.value,GLOBAL-DEFAULT:.globalDefault"
```

**Console Output:**

```
NAME                        VALUE        GLOBAL-DEFAULT
system-cluster-critical     2000000000   false
system-node-critical        2000001000   false
default-workload-priority   1000         true
critical-priority          10000        false
batch-priority             100          false
```

## Troubleshooting Common Issues

### Issue 1: Pod not getting expected priority

**Check if a global default exists:**

```bash
kubectl get priorityclass -o json | jq '.items[] | select(.globalDefault==true) | .metadata.name'
```

### Issue 2: Cannot create new global default

**Find and update the existing global default:**

```bash
# Find current global default
kubectl get priorityclass -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.globalDefault}{"\n"}{end}' | grep true

# Update it to false
kubectl patch priorityclass <name> -p '{"globalDefault": false}'
```

### Issue 3: Existing pods not updated after setting global default

**Remember:** Global default only affects NEW pods. Existing pods keep their original priority.

To update existing pods, you must recreate them:

```bash
kubectl delete pod <pod-name>
kubectl apply -f <pod-definition.yaml>
```

## Summary

The `globalDefault` field in PriorityClasses:

- Automatically assigns a priority to pods that don't specify one
- Ensures no pod is left with the default priority of 0
- Helps establish a baseline priority for all workloads
- Only one can exist in the cluster at a time
- Only affects newly created pods, not existing ones

This approach solves the problem of:

- Developers forgetting to set priorities on their pods
- Ensuring all workloads have a reasonable baseline priority
- Preventing low-priority system pods from being starved by user workloads
- Maintaining consistent scheduling behavior across the cluster

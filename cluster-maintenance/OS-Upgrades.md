# Kubernetes Node Maintenance Guide

## Overview

This guide demonstrates how to perform maintenance operations on Kubernetes nodes, including draining nodes for maintenance, handling different types of pods, and managing node scheduling. The scenario covers both automated pod management (ReplicaSets) and standalone pods that require special handling.

## Original Problem Statement

**Question:** Which nodes are the applications hosted on?

**Follow-up Tasks:**

1. Take node01 out for maintenance by emptying it of all applications and marking it unschedulable
2. After maintenance, make node01 schedulable again
3. Handle a scenario where a standalone pod (not managed by a controller) prevents normal draining
4. Protect critical applications while preventing new pod scheduling

## Initial State Assessment

### Command: Check Pod Distribution

```bash
k get pods -o wide
```

**Output:**

```
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-69968556cc-27p7g   1/1     Running   0          2m35s   172.17.1.2   node01         <none>           <none>
blue-69968556cc-bm46m   1/1     Running   0          2m35s   172.17.0.4   controlplane   <none>           <none>
blue-69968556cc-t66p2   1/1     Running   0          2m35s   172.17.1.3   node01         <none>           <none>
```

**Analysis:** Applications are distributed across both `controlplane` and `node01` nodes, with 2 pods on node01 and 1 pod on controlplane.

## Step-by-Step Node Maintenance Process

### Step 1: Drain Node for Maintenance

**Command:**

```bash
k drain node01 --ignore-daemonsets
```

**Output:**

```
node/node01 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-ngc2l, kube-system/kube-proxy-g4w5p
evicting pod default/blue-69968556cc-t66p2
evicting pod default/blue-69968556cc-27p7g
pod/blue-69968556cc-t66p2 evicted
pod/blue-69968556cc-27p7g evicted
node/node01 drained
```

**Explanation:**

- `kubectl drain` performs two actions: cordons the node (marks it unschedulable) and evicts all pods
- `--ignore-daemonsets` flag allows the command to proceed despite DaemonSet pods (which run on every node)
- The ReplicaSet automatically recreates the evicted pods on available nodes

**Verification:**

```bash
k get pods -o wide
```

**Output:**

```
NAME                    READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
blue-69968556cc-bm46m   1/1     Running   0          4m3s   172.17.0.4   controlplane   <none>           <none>
blue-69968556cc-csvqm   1/1     Running   0          31s    172.17.0.6   controlplane   <none>           <none>
blue-69968556cc-ld9zh   1/1     Running   0          31s    172.17.0.5   controlplane   <none>           <none>
```

**Result:** All application pods are now running on the controlplane node.

### Step 2: Complete Maintenance and Restore Scheduling

**Command:**

```bash
k uncordon node01
```

**Output:**

```
node/node01 uncordoned
```

**Explanation:**

- `kubectl uncordon` removes the scheduling restriction from the node
- Existing pods remain where they are; only new pods can be scheduled on node01

**Verification:**

```bash
k get pods -o wide
```

**Output:**

```
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           READINESS GATES
blue-69968556cc-bm46m   1/1     Running   0          5m19s   172.17.0.4   controlplane   <none>           <none>
blue-69968556cc-csvqm   1/1     Running   0          107s    172.17.0.6   controlplane   <none>           <none>
blue-69968556cc-ld9zh   1/1     Running   0          107s    172.17.0.5   controlplane   <none>           <none>
```

**Key Insight:** No pods are scheduled on node01 because `uncordon` only makes the node available for new pods, not existing ones.

### Step 3: Understanding Pod Placement

**Why are pods on controlplane?**

**Command:**

```bash
k describe node controlplane | grep -i taint
```

**Output:**

```
Taints:             <none>
```

**Explanation:** The controlplane node has no taints, making it available for regular pod scheduling (which is not typical in production environments).

## Handling Standalone Pods

### Step 4: Attempting to Drain with Standalone Pod

**Command:**

```bash
k drain node01 --ignore-daemonsets
```

**Output:**

```
node/node01 cordoned
error: unable to drain node "node01" due to error: cannot delete cannot delete Pods that declare no controller (use --force to override): default/hr-app, continuing command...
There are pending nodes to be drained:
 node01
cannot delete cannot delete Pods that declare no controller (use --force to override): default/hr-app
```

**Problem Analysis:**

```bash
k get pods -o wide
```

**Output:**

```
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-69968556cc-bm46m   1/1     Running   0          8m31s   172.17.0.4   controlplane   <none>           <none>
blue-69968556cc-csvqm   1/1     Running   0          4m59s   172.17.0.6   controlplane   <none>           <none>
blue-69968556cc-ld9zh   1/1     Running   0          4m59s   172.17.0.5   controlplane   <none>           <none>
hr-app                  1/1     Running   0          83s     172.17.1.4   node01         <none>           <none>
```

**Issue Identified:** The `hr-app` pod is not managed by a controller (ReplicaSet, Deployment, etc.), so draining fails to protect against data loss.

### Step 5: Alternative Solution - Cordoning

**Command:**

```bash
kubectl cordon node01
```

**Explanation:**

- `kubectl cordon` only marks the node as unschedulable without evicting existing pods
- This protects critical standalone pods like `hr-app` while preventing new pod scheduling
- Existing pods continue running normally

## Key Concepts and Solutions

### Drain vs Cordon Comparison

| Operation  | Schedulable | Existing Pods | Use Case                                     |
| ---------- | ----------- | ------------- | -------------------------------------------- |
| `drain`    | No          | Evicted       | Full maintenance, pods have controllers      |
| `cordon`   | No          | Remain        | Partial maintenance, protect standalone pods |
| `uncordon` | Yes         | Remain        | Restore normal scheduling                    |

### Pod Controller Types

**Managed Pods (safe to drain):**

- Pods created by Deployments, ReplicaSets, DaemonSets
- Automatically recreated when evicted
- Examples: `blue-69968556cc-*` pods

**Standalone Pods (require caution):**

- Pods created directly without controllers
- Lost permanently when evicted
- Examples: `hr-app`

### Best Practices

1. **Always check pod types before draining:**

   ```bash
   kubectl get pods -o wide
   kubectl describe pod <pod-name> | grep "Controlled By"
   ```

2. **Use appropriate flags:**

   - `--ignore-daemonsets`: Skip DaemonSet pods
   - `--force`: Forcefully evict standalone pods (use with caution)
   - `--delete-emptydir-data`: Allow deletion of pods with emptyDir volumes

3. **For critical standalone pods:**
   - Use `cordon` instead of `drain`
   - Plan for pod migration before maintenance
   - Consider converting to managed workloads

## Command Reference

```bash
# Check current pod distribution
kubectl get pods -o wide

# Drain node (evict all pods, mark unschedulable)
kubectl drain <node-name> --ignore-daemonsets

# Forcefully drain node (including standalone pods)
kubectl drain <node-name> --ignore-daemonsets --force

# Mark node as unschedulable only (preserve existing pods)
kubectl cordon <node-name>

# Mark node as schedulable
kubectl uncordon <node-name>

# Check node taints
kubectl describe node <node-name> | grep -i taint
```

## Conclusion

This guide demonstrates the importance of understanding pod management in Kubernetes when performing node maintenance. The key takeaway is choosing the right approach based on your workload types:

- **Use `drain`** for nodes with only controller-managed pods
- **Use `cordon`** when standalone critical pods must be preserved
- **Always verify** pod types and controllers before maintenance operations

The `hr-app` scenario highlights why production workloads should typically be managed by controllers rather than deployed as standalone pods.

# Kubernetes Cluster Upgrade Lab

## Problem Statement

This lab tests your skills on upgrading a Kubernetes cluster. We have a production cluster with applications running on it. You are tasked to upgrade the cluster from v1.31.0 to v1.32.0. Users accessing the applications must not be impacted, and you cannot provision new VMs.

## Initial Cluster Assessment

### Current Cluster Version

```bash
controlplane ~ ➜  k get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   66m   v1.31.0
node01         Ready    <none>          65m   v1.31.0
```

**Answer**: The current cluster version is **v1.31.0**

### Cluster Composition

- **Total nodes**: 2 (including controlplane and worker nodes)
- **Workload-capable nodes**: 2 (both controlplane and node01 can host workloads)

### Node Taints Inspection

```bash
root@controlplane:~# kubectl describe nodes controlplane | grep -i taint
Taints:             <none>
root@controlplane:~# kubectl describe nodes node01 | grep -i taint
Taints:             <none>
```

Both nodes have no taints, meaning they can schedule pods.

### Application Analysis

```bash
root@controlplane:~# kubectl get deployments.apps
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
blue   5/5     5            5           119s
```

**Applications hosted**: 1 deployment named "blue" with 5 replicas

### Pod Distribution

```bash
controlplane ~ ➜  k get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-56dd475db5-2m4nr   1/1     Running   0          3m23s   172.17.0.4   controlplane   <none>           <none>
blue-56dd475db5-c2jg7   1/1     Running   0          3m23s   172.17.1.2   node01         <none>           <none>
blue-56dd475db5-ddtsn   1/1     Running   0          3m23s   172.17.1.3   node01         <none>           <none>
blue-56dd475db5-frhfx   1/1     Running   0          3m23s   172.17.1.4   node01         <none>           <none>
blue-56dd475db5-qpb5z   1/1     Running   0          3m23s   172.17.0.5   controlplane   <none>           <none>
```

Pods are distributed across both nodes:

- **controlplane**: 2 pods
- **node01**: 3 pods

## Upgrade Strategy

**Rolling Upgrade Strategy**: Since we cannot provision new VMs and must maintain application availability, we'll use a rolling upgrade approach:

1. Drain and upgrade the controlplane node first
2. Drain and upgrade the worker node
3. Ensure pods are rescheduled to maintain availability

## Available Upgrade Version

```bash
controlplane ~ ➜  kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.31.0
[upgrade/versions] kubeadm version: v1.31.0
I0526 22:57:26.759359   29968 version.go:261] remote version is much newer: v1.33.1; falling back to: stable-1.31
[upgrade/versions] Target version: v1.31.9
[upgrade/versions] Latest version in the v1.31 series: v1.31.9

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE           CURRENT   TARGET
kubelet     controlplane   v1.31.0   v1.31.9
kubelet     node01         v1.31.0   v1.31.9

Upgrade to the latest version in the v1.31 series:

COMPONENT                 NODE           CURRENT    TARGET
kube-apiserver            controlplane   v1.31.0    v1.31.9
kube-controller-manager   controlplane   v1.31.0    v1.31.9
kube-scheduler            controlplane   v1.31.0    v1.31.9
kube-proxy                               1.31.0     v1.31.9
CoreDNS                                  v1.10.1    v1.11.1
etcd                      controlplane   3.5.15-0   3.5.15-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.31.9

Note: Before you can perform this upgrade, you have to update kubeadm to v1.31.9.

_____________________________________________________________________

The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1.beta1          v1beta1             no
_____________________________________________________________________
```

## Step-by-Step Upgrade Process

### Phase 1: Upgrade Controlplane Node

#### Step 1: Drain Controlplane Node

```bash
kubectl drain controlplane --ignore-daemonsets
```

**Explanation**: This command safely evicts all pods from the controlplane node and marks it as unschedulable. The `--ignore-daemonsets` flag allows the operation to proceed despite DaemonSets running on the node.

#### Step 2: Update Package Repository

```bash
vim /etc/apt/sources.list.d/kubernetes.list
```

**Configuration Update**: Change the repository URL to target v1.32:

```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /
```

**Explanation**: This updates the apt repository to access Kubernetes v1.32 packages.

#### Step 3: Update Package Cache

```bash
apt update
```

**Explanation**: Refreshes the package cache with the new repository information.

#### Step 4: Check Available kubeadm Versions

```bash
apt-cache madison kubeadm
```

**Explanation**: Displays available kubeadm package versions. For v1.32.0, the package version is `1.32.0-1.1`.

#### Step 5: Install kubeadm v1.32.0

```bash
apt-get install kubeadm=1.32.0-1.1
```

**Explanation**: Installs the specific version of kubeadm needed for the upgrade.

#### Step 6: Plan the Upgrade

```bash
kubeadm upgrade plan v1.32.0
```

**Explanation**: Validates the upgrade path and shows what components will be upgraded.

#### Step 7: Apply the Upgrade

```bash
kubeadm upgrade apply v1.32.0
```

**Explanation**: Upgrades the control plane components (API server, controller manager, scheduler, etc.) to v1.32.0.

#### Step 8: Upgrade kubelet

```bash
apt-get install kubelet=1.32.0-1.1
```

**Explanation**: Updates the kubelet to match the control plane version.

#### Step 9: Restart kubelet Service

```bash
systemctl daemon-reload
systemctl restart kubelet
```

**Explanation**:

- `daemon-reload`: Reloads systemd configuration files
- `restart kubelet`: Restarts the kubelet service with the new version

#### Step 10: Uncordon Controlplane Node

```bash
kubectl uncordon controlplane
```

**Explanation**: Marks the controlplane node as schedulable again, allowing pods to be scheduled on it.

### Phase 2: Upgrade Worker Node

#### Step 1: Drain Worker Node

```bash
kubectl drain node01 --ignore-daemonsets
```

**Explanation**: Safely evicts all pods from node01 and marks it unschedulable.

#### Step 2: Access Worker Node

```bash
ssh node01
```

**Explanation**: Log into the worker node to perform the upgrade.

#### Step 3: Update Package Repository on Worker Node

```bash
vim /etc/apt/sources.list.d/kubernetes.list
```

**Configuration Update**: Same as controlplane:

```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /
```

#### Step 4: Update Package Cache

```bash
apt update
```

#### Step 5: Check Available Versions

```bash
apt-cache madison kubeadm
```

#### Step 6: Install kubeadm

```bash
apt-get install kubeadm=1.32.0-1.1
```

#### Step 7: Upgrade Worker Node

```bash
kubeadm upgrade node
```

**Explanation**: Upgrades the worker node configuration and components.

#### Step 8: Upgrade kubelet on Worker Node

```bash
apt-get install kubelet=1.32.0-1.1
```

#### Step 9: Restart kubelet Service

```bash
systemctl daemon-reload
systemctl restart kubelet
```

#### Step 10: Return to Controlplane

```bash
exit
```

**Explanation**: Exit the SSH session to return to the controlplane node.

#### Step 11: Uncordon Worker Node

```bash
kubectl uncordon node01
```

**Explanation**: Makes the worker node schedulable again.

## Why This Approach Solves the Problem

### Zero-Downtime Strategy

1. **Rolling Upgrade**: By upgrading nodes one at a time, the application remains available
2. **Pod Redistribution**: When a node is drained, pods are automatically rescheduled to available nodes
3. **Gradual Transition**: The upgrade happens incrementally, maintaining cluster stability

### Resource Constraints Addressed

- **No New VMs Required**: Uses existing infrastructure
- **Maintains Capacity**: Always keeps at least one node available for workloads
- **Preserves Application State**: Pods are gracefully moved, not terminated abruptly

### Safety Measures

- **Drain Before Upgrade**: Ensures no workloads are running during node upgrade
- **Version Consistency**: All components are upgraded to the same target version
- **Service Restart**: Ensures new versions are properly loaded and running

### Application Continuity

With 5 replicas of the "blue" application distributed across 2 nodes, the rolling upgrade ensures:

- When controlplane is drained, its 2 pods move to node01
- When node01 is drained, its pods move to the upgraded controlplane
- At no point are all replicas unavailable

This approach successfully upgrades the cluster from v1.31.0 to v1.32.0 while maintaining application availability and working within the given constraints.

## Verification Commands

After completion, verify the upgrade:

```bash
# Check node versions
kubectl get nodes

# Check pod distribution
kubectl get pods -o wide

# Verify application availability
kubectl get deployments
```

The cluster should now be running v1.32.0 with all applications functioning normally.

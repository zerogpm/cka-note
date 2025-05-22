# Kubernetes Static Pods Lab

This repository contains a comprehensive lab exercise for understanding and working with static pods in Kubernetes. Static pods are pods that are directly managed by the kubelet on a specific node, rather than by the API server.

## Table of Contents

- [Overview](#overview)
- [Lab Exercises](#lab-exercises)
- [Key Concepts](#key-concepts)
- [Prerequisites](#prerequisites)

## Overview

Static pods are Kubernetes pods that are managed directly by the kubelet daemon on a specific node, without the API server observing them. They are typically used for running critical system components like the API server, etcd, and scheduler on control plane nodes.

## Lab Exercises

### Exercise 1: Count Static Pods in the Cluster

**Question:** How many static pods exist in this cluster in all namespaces?

**Command:**

```bash
kubectl get pods --all-namespaces
```

**Solution:** Look for pods with `-controlplane` appended in the name.

**Expected Output:**

```bash
NAMESPACE      NAME                                   READY   STATUS    RESTARTS   AGE     IP                NODE           NOMINATED NODE   READINESS GATES
kube-flannel   kube-flannel-ds-66tr7                  1/1     Running   0          9m45s   192.168.227.153   controlplane   <none>           <none>
kube-flannel   kube-flannel-ds-b47lz                  1/1     Running   0          9m22s   192.168.59.129    node01         <none>           <none>
kube-system    coredns-7484cd47db-28cxt               1/1     Running   0          9m45s   172.17.0.3        controlplane   <none>           <none>
kube-system    coredns-7484cd47db-crttx               1/1     Running   0          9m45s   172.17.0.2        controlplane   <none>           <none>
kube-system    etcd-controlplane                      1/1     Running   0          9m49s   192.168.227.153   controlplane   <none>           <none>
kube-system    kube-apiserver-controlplane            1/1     Running   0          9m49s   192.168.227.153   controlplane   <none>           <none>
kube-system    kube-controller-manager-controlplane   1/1     Running   0          9m49s   192.168.227.153   controlplane   <none>           <none>
kube-system    kube-proxy-4zzvv                       1/1     Running   0          9m45s   192.168.227.153   controlplane   <none>           <none>
kube-system    kube-proxy-m2cvj                       1/1     Running   0          9m22s   192.168.59.129    node01         <none>           <none>
kube-system    kube-scheduler-controlplane            1/1     Running   0          9m49s   192.168.227.153   controlplane   <none>           <none>
```

**Answer:** 4 static pods exist (etcd, kube-apiserver, kube-controller-manager, kube-scheduler - all with `-controlplane` suffix)

---

### Exercise 2: Identify Non-Static Pod Components

**Question 1:** Which of the below components is NOT deployed as a static pod?

**Answer:** CoreDNS - The coredns pods are created as part of the coredns deployment and hence, it is not a static pod.

**Question 2:** Which of the below components is NOT deployed as a static POD?

**Answer:** kube-proxy - kube-proxy is deployed as a DaemonSet and hence, it is not a static pod.

---

### Exercise 3: Locate Static Pod Nodes

**Question:** On which nodes are the static pods created currently?

**Command:**

```bash
kubectl get pods --all-namespaces -o wide
```

**Answer:** Static pods are created on the `controlplane` node (identifiable by the `-controlplane` suffix in pod names).

---

### Exercise 4: Find Static Pod Definition Directory

**Question:** What is the path of the directory holding the static pod definition files?

**Step-by-step Solution:**

1. **Find the kubelet process and its configuration:**

```bash
ps -aux | grep /usr/bin/kubelet
```

**Output:**

```bash
root      3668  0.0  1.5 1933476 63076 ?       Ssl  Mar13  16:18 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
root      4879  0.0  0.0  11468  1040 pts/0    S+   00:06   0:00 grep --color=auto /usr/bin/kubelet
```

2. **Look up the staticPodPath in the kubelet configuration:**

```bash
grep -i staticpod /var/lib/kubelet/config.yaml
```

**Output:**

```bash
staticPodPath: /etc/kubernetes/manifests
```

**Answer:** `/etc/kubernetes/manifests`

---

### Exercise 5: Count Static Pod Definition Files

**Question:** How many pod definition files are present in the manifests directory?

**Command:**

```bash
ls -la /etc/kubernetes/manifests/*.yaml | wc -l
```

**Explanation:** This command lists all YAML files in the manifests directory and counts them.

---

### Exercise 6: Identify Docker Image for kube-apiserver

**Question:** What is the docker image used to deploy the kube-api server as a static pod?

**Command:**

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

**Key Section from Output:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.227.153:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
    - command:
        - kube-apiserver
        - --advertise-address=192.168.227.153
        - --allow-privileged=true
        - --authorization-mode=Node,RBAC
      # ... many more arguments ...
      image: registry.k8s.io/kube-apiserver:v1.32.0
      imagePullPolicy: IfNotPresent
      # ... rest of configuration ...
```

**Answer:** `registry.k8s.io/kube-apiserver:v1.32.0`

---

### Exercise 7: Create a Static Pod

**Question:** Create a static pod named static-busybox that uses the busybox image and the command sleep 1000

**Command:**

```bash
kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```

**Explanation:**

- `kubectl run`: Creates a pod
- `--restart=Never`: Ensures it's a pod, not a deployment
- `--image=busybox`: Specifies the Docker image
- `static-busybox`: Pod name
- `--dry-run=client -o yaml`: Generates YAML without creating the resource
- `--command -- sleep 1000`: Sets the command to run
- `> /etc/kubernetes/manifests/static-busybox.yaml`: Redirects output to the static pod directory

---

### Exercise 8: Update Static Pod Image

**Question:** Edit the image on the static pod to use busybox:1.28.4

**Command:**

```bash
kubectl run --restart=Never --image=busybox:1.28.4 static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```

**Explanation:** This overwrites the previous definition with the updated image version.

---

### Exercise 9: Delete a Static Pod on Another Node

**Question:** We just created a new static pod named static-greenbox. Find it and delete it.

**Step-by-step Solution:**

1. **Identify which node the static pod is running on:**

```bash
kubectl get pods --all-namespaces -o wide | grep static-greenbox
```

**Output:**

```bash
default       static-greenbox-node01                 1/1     Running   0          19s     10.244.1.2   node01       <none>           <none>
```

2. **SSH to node01 and find the kubelet configuration:**

```bash
# On node01
ps -ef | grep /usr/bin/kubelet
```

**Output:**

```bash
root       12114       1  0 14:36 ?        00:00:03 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.10
root       15063   14776  0 14:41 pts/0    00:00:00 grep /usr/bin/kubelet
```

3. **Check the static pod path configuration:**

```bash
grep -i staticpod /var/lib/kubelet/config.yaml
```

**Output:**

```bash
staticPodPath: /etc/just-to-mess-with-you
```

4. **Delete the static pod definition file:**

```bash
rm -rf /etc/just-to-mess-with-you/greenbox.yaml
```

**Important Note:** The static pod path on node01 is different from the control plane (`/etc/just-to-mess-with-you` instead of `/etc/kubernetes/manifests`). This demonstrates that each node can have its own static pod configuration.

---

## Key Concepts

### What are Static Pods?

- Pods managed directly by kubelet on a specific node
- Not managed by the Kubernetes API server
- Typically used for critical system components
- Defined by YAML files in a directory watched by kubelet

### Static Pod Characteristics

- Pod names include the node name as a suffix (e.g., `pod-name-nodename`)
- Cannot be deleted using `kubectl delete` (only by removing the definition file)
- Automatically recreated if the definition file exists
- Visible via `kubectl get pods` but managed locally

### Common Static Pod Components

- **etcd**: Key-value store for cluster data
- **kube-apiserver**: API server for Kubernetes
- **kube-controller-manager**: Controller manager
- **kube-scheduler**: Pod scheduler

### Non-Static Pod Components

- **CoreDNS**: Deployed as a Deployment
- **kube-proxy**: Deployed as a DaemonSet
- **CNI pods** (like Flannel): Deployed as DaemonSets

## Prerequisites

- Kubernetes cluster with at least one control plane and one worker node
- `kubectl` configured to access the cluster
- SSH access to cluster nodes
- Basic understanding of Kubernetes concepts

## Why This Approach Works

This lab demonstrates static pod management because:

1. **Direct kubelet management**: Static pods bypass the API server, making them perfect for bootstrapping critical components
2. **Node-specific configuration**: Each node can have different static pod directories and configurations
3. **High availability**: Critical components run as static pods to ensure they start with the kubelet
4. **Troubleshooting skills**: Understanding static pods is crucial for debugging cluster issues

The exercises progress from basic identification to advanced operations, building practical skills for managing Kubernetes infrastructure components.

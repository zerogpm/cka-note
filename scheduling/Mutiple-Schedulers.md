# Kubernetes Custom Scheduler Setup

This repository demonstrates how to deploy a custom scheduler in a Kubernetes cluster and configure pods to use it instead of the default scheduler.

## Problem Statement

The original questions being addressed are:

1. **What is the name of the POD that deploys the default kubernetes scheduler in this environment?**
2. **What is the image used to deploy the kubernetes scheduler?**
3. **How to deploy an additional custom scheduler to the cluster?**
4. **How to configure a pod to use the custom scheduler?**

## Prerequisites

- A running Kubernetes cluster
- `kubectl` command-line tool configured
- Admin access to the cluster

## Step-by-Step Implementation

### Step 1: Identify the Default Scheduler Pod

First, we need to identify the default Kubernetes scheduler pod running in the cluster.

```bash
kubectl get pod --all-namespaces
```

**Console Output:**

```
NAMESPACE      NAME                                   READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-v8qdb                  1/1     Running   0          6m14s
kube-system    coredns-7484cd47db-6tnkl               1/1     Running   0          6m14s
kube-system    coredns-7484cd47db-bc57f               1/1     Running   0          6m14s
kube-system    etcd-controlplane                      1/1     Running   0          6m20s
kube-system    kube-apiserver-controlplane            1/1     Running   0          6m20s
kube-system    kube-controller-manager-controlplane   1/1     Running   0          6m20s
kube-system    kube-proxy-bzp5z                       1/1     Running   0          6m14s
kube-scheduler-controlplane            1/1     Running   0          6m20s
```

**Answer:** The default Kubernetes scheduler pod is named `kube-scheduler-controlplane`.

### Step 2: Inspect the Default Scheduler Image

To identify the image used by the default scheduler, we inspect the pod:

```bash
kubectl describe pod kube-scheduler-controlplane -n kube-system
```

**Key Information from Console Output:**

```yaml
Containers:
  kube-scheduler:
    Container ID: containerd://8a5b834e95bd02fcea12c10ea2cd03d2e0ee3960a4a266a5bbb03a9c6a961c64
    Image: registry.k8s.io/kube-scheduler:v1.32.0
    Image ID: registry.k8s.io/kube-scheduler@sha256:84c998f7610b356a5eed24f801c01b273cf3e83f081f25c9b16aa8136c2cafb1
```

**Answer:** The default scheduler uses the image `registry.k8s.io/kube-scheduler:v1.32.0`.

### Step 3: Verify Pre-created Resources

Check that the necessary ServiceAccount and ClusterRoleBindings are already created:

```bash
kubectl get serviceaccount -n kube-system
kubectl get clusterrolebinding
```

**ServiceAccount Output (relevant line):**

```
my-scheduler                                  0         47s
```

**ClusterRoleBinding Output (relevant lines):**

```
my-scheduler-as-kube-scheduler                  ClusterRole/system:kube-scheduler                70s
my-scheduler-as-volume-scheduler                ClusterRole/system:volume-scheduler               70s
```

### Step 4: Create the Scheduler ConfigMap

Create a ConfigMap that contains the configuration for our custom scheduler:

**File: `my-scheduler-configmap.yaml`**

```yaml
apiVersion: v1
data:
  my-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    profiles:
      - schedulerName: my-scheduler
    leaderElection:
      leaderElect: false
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: my-scheduler-config
  namespace: kube-system
```

Apply the ConfigMap:

```bash
kubectl create -f /root/my-scheduler-configmap.yaml
```

### Step 5: Deploy the Custom Scheduler

**Original File: `my-scheduler.yaml`** (with placeholder image)

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-scheduler
  name: my-scheduler
  namespace: kube-system
spec:
  serviceAccountName: my-scheduler
  containers:
    - command:
        - /usr/local/bin/kube-scheduler
        - --config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
      image: <use-correct-image>
      livenessProbe:
        httpGet:
          path: /healthz
          port: 10259
          scheme: HTTPS
        initialDelaySeconds: 15
      name: kube-second-scheduler
      readinessProbe:
        httpGet:
          path: /healthz
          port: 10259
          scheme: HTTPS
      resources:
        requests:
          cpu: "0.1"
      securityContext:
        privileged: false
      volumeMounts:
        - name: config-volume
          mountPath: /etc/kubernetes/my-scheduler
  hostNetwork: false
  hostPID: false
  volumes:
    - name: config-volume
      configMap:
        name: my-scheduler-config
```

**Updated File: `my-scheduler.yaml`** (with correct image)

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-scheduler
  name: my-scheduler
  namespace: kube-system
spec:
  serviceAccountName: my-scheduler
  containers:
    - command:
        - /usr/local/bin/kube-scheduler
        - --config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
      image: registry.k8s.io/kube-scheduler:v1.32.0 # Updated with correct image
      livenessProbe:
        httpGet:
          path: /healthz
          port: 10259
          scheme: HTTPS
        initialDelaySeconds: 15
      name: kube-second-scheduler
      readinessProbe:
        httpGet:
          path: /healthz
          port: 10259
          scheme: HTTPS
      resources:
        requests:
          cpu: "0.1"
      securityContext:
        privileged: false
      volumeMounts:
        - name: config-volume
          mountPath: /etc/kubernetes/my-scheduler
  hostNetwork: false
  hostPID: false
  volumes:
    - name: config-volume
      configMap:
        name: my-scheduler-config
```

Deploy the custom scheduler:

```bash
kubectl apply -f my-scheduler.yaml
```

### Step 6: Create a Pod Using the Custom Scheduler

**Original File: `nginx-pod.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
```

**Updated File: `nginx-pod.yaml`** (with custom scheduler)

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  schedulerName: my-scheduler # Added custom scheduler
  containers:
    - image: nginx
      name: nginx
```

Deploy the pod:

```bash
kubectl apply -f nginx-pod.yaml
```

## Configuration Details

### Key Components

1. **ServiceAccount**: `my-scheduler` - Provides identity for the custom scheduler pod
2. **ClusterRoleBindings**:
   - `my-scheduler-as-kube-scheduler` - Grants scheduler permissions
   - `my-scheduler-as-volume-scheduler` - Grants volume scheduling permissions
3. **ConfigMap**: `my-scheduler-config` - Contains scheduler configuration
4. **Pod**: `my-scheduler` - Runs the custom scheduler instance

### ConfigMap Configuration

The ConfigMap defines:

- **schedulerName**: `my-scheduler` - The name pods will reference
- **leaderElection**: `false` - Disables leader election for this scheduler

### Scheduler Pod Configuration

Key aspects of the scheduler pod:

- Uses the same image as the default scheduler
- Mounts the ConfigMap as a volume at `/etc/kubernetes/my-scheduler`
- Configured with health checks on port 10259
- Runs with minimal CPU requirements (0.1 CPU)

## Why This Approach Works

### 1. **Reuses Existing Infrastructure**

- Uses the same proven scheduler image (`registry.k8s.io/kube-scheduler:v1.32.0`)
- Leverages existing RBAC with pre-created ServiceAccount and ClusterRoleBindings

### 2. **Configuration-Driven Approach**

- The ConfigMap allows easy customization of scheduler behavior
- Separates configuration from the pod definition
- Enables updates without rebuilding images

### 3. **Pod-Level Scheduler Selection**

- The `schedulerName` field in pod specs allows selective use of custom schedulers
- Pods without this field continue using the default scheduler
- Provides flexibility for different workload requirements

### 4. **Isolation and Safety**

- Custom scheduler runs as a separate pod
- Doesn't interfere with the default scheduler
- Can be easily removed or updated without affecting existing workloads

## Verification

To verify the setup is working:

```bash
# Check if custom scheduler pod is running
kubectl get pods -n kube-system | grep my-scheduler

# Check if nginx pod was scheduled by custom scheduler
kubectl describe pod nginx | grep "Scheduled"

# View scheduler logs
kubectl logs my-scheduler -n kube-system
```

## Troubleshooting

Common issues and solutions:

1. **Scheduler Pod Not Starting**: Check image name and RBAC permissions
2. **Pods Not Being Scheduled**: Verify the `schedulerName` matches the ConfigMap
3. **Health Check Failures**: Ensure port 10259 is accessible and HTTPS is configured

## Next Steps

- Customize the scheduler configuration for specific requirements
- Implement custom scheduling algorithms
- Add monitoring and alerting for the custom scheduler
- Consider high availability with multiple scheduler replicas

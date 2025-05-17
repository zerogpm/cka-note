# Kubernetes Cluster Troubleshooting Guide

## Problem Description

The cluster is broken. We tried deploying an application but it's not working. Troubleshoot and fix the issue.

## Troubleshooting Process

### Step 1: Examine the Deployments

First, we need to check the status of all resources in the cluster to identify any issues:

```bash
controlplane ~ ➜  k get all -A
NAMESPACE      NAME                                       READY   STATUS             RESTARTS      AGE
default        pod/app-7f9667c9d9-s4j46                   0/1     Pending            0             18s
kube-flannel   pod/kube-flannel-ds-vj5rb                  1/1     Running            0             14m
kube-system    pod/coredns-7484cd47db-48qrp               1/1     Running            0             14m
kube-system    pod/coredns-7484cd47db-vfw8x               1/1     Running            0             14m
kube-system    pod/etcd-controlplane                      1/1     Running            0             14m
kube-system    pod/kube-apiserver-controlplane            1/1     Running            0             14m
kube-system    pod/kube-controller-manager-controlplane   1/1     Running            0             14m
kube-system    pod/kube-proxy-4z68x                       1/1     Running            0             14m
kube-system    pod/kube-scheduler-controlplane            0/1     CrashLoopBackOff   1 (18s ago)   19s

NAMESPACE     NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   172.20.0.1    <none>        443/TCP                  14m
kube-system   service/kube-dns     ClusterIP   172.20.0.10   <none>        53/UDP,53/TCP,9153/TCP   14m

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   14m
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   14m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/app       0/1     1            0           18s
kube-system   deployment.apps/coredns   2/2     2            2           14m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
default       replicaset.apps/app-7f9667c9d9       1         1         0       18s
kube-system   replicaset.apps/coredns-7484cd47db   2         2         2       14m
```

We observe that:

- The `kube-scheduler-controlplane` pod in the `kube-system` namespace is in a `CrashLoopBackOff` state
- The `app` deployment has 0/1 pods ready and its pod is in `Pending` state

### Step 2: Investigate the Scheduler Issue

Since the scheduler is responsible for assigning pods to nodes, let's first examine the scheduler pod:

```bash
controlplane ~ ➜  k describe pod kube-scheduler-controlplane -n kube-system
Name:                 kube-scheduler-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 controlplane/192.168.28.36
Start Time:           Fri, 16 May 2025 22:09:59 +0000
Labels:               component=kube-scheduler
                      tier=control-plane
Annotations:          kubernetes.io/config.hash: e62b9c3ab48bb2ebb392179898b5495e
                      kubernetes.io/config.mirror: e62b9c3ab48bb2ebb392179898b5495e
                      kubernetes.io/config.seen: 2025-05-16T22:23:53.921842802Z
                      kubernetes.io/config.source: file
Status:               Running
SeccompProfile:       RuntimeDefault
IP:                   192.168.28.36
IPs:
  IP:           192.168.28.36
Controlled By:  Node/controlplane
Containers:
  kube-scheduler:
    Container ID:  containerd://2493b6d740662be24e969f171a13ce9f5296f15de53fdd0dcc88b526f0e4099b
    Image:         registry.k8s.io/kube-scheduler:v1.32.0
    Image ID:      registry.k8s.io/kube-scheduler@sha256:84c998f7610b356a5eed24f801c01b273cf3e83f081f25c9b16aa8136c2cafb1
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-schedulerrrr
      --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
      --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
      --bind-address=127.0.0.1
      --kubeconfig=/etc/kubernetes/scheduler.conf
      --leader-elect=true
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       StartError
      Message:      failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "kube-schedulerrrr": executable file not found in $PATH: unknown
      Exit Code:    128
      Started:      Thu, 01 Jan 1970 00:00:00 +0000
      Finished:     Fri, 16 May 2025 22:25:48 +0000
    Ready:          False
    Restart Count:  4
    Requests:
      cpu:        100m
    Liveness:     http-get https://127.0.0.1:10259/livez delay=10s timeout=15s period=10s #success=1 #failure=8
    Readiness:    http-get https://127.0.0.1:10259/readyz delay=0s timeout=15s period=1s #success=1 #failure=3
    Startup:      http-get https://127.0.0.1:10259/livez delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/kubernetes/scheduler.conf from kubeconfig (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kubeconfig:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/scheduler.conf
    HostPathType:  FileOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:
  Type     Reason   Age                   From     Message
  ----     ------   ----                  ----     -------
  Normal   Pulled   54s (x5 over 2m33s)   kubelet  Container image "registry.k8s.io/kube-scheduler:v1.32.0" already present on machine
  Normal   Created  54s (x5 over 2m33s)   kubelet  Created container: kube-scheduler
  Warning  Failed   53s (x5 over 2m33s)   kubelet  Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "kube-schedulerrrr": executable file not found in $PATH: unknown
  Warning  BackOff  31s (x20 over 2m32s)  kubelet  Back-off restarting failed container kube-scheduler in pod kube-scheduler-controlplane_kube-system(e62b9c3ab48bb2ebb392179898b5495e)
```

The error message indicates there's a typo in the command name: `kube-schedulerrrr` (with extra 'r's). The correct binary name should be `kube-scheduler`.

### Step 3: Locate and Fix the Scheduler Configuration

Since the scheduler is a static pod (managed by the kubelet directly), we need to check its manifest file:

```bash
controlplane ~ ➜  ls /etc/kubernetes/manifests/
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

Edit the kube-scheduler.yaml file:

```bash
controlplane ~ ➜ vim /etc/kubernetes/manifests/kube-scheduler.yaml
```

Here's the problematic YAML with the typo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
    - command:
        - kube-schedulerrrr # here should be kube-scheduler
        - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
        - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
        - --bind-address=127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --leader-elect=true
      image: registry.k8s.io/kube-scheduler:v1.32.0
      imagePullPolicy: IfNotPresent
      livenessProbe:
        failureThreshold: 8
        httpGet:
          host: 127.0.0.1
          path: /livez
          port: 10259
          scheme: HTTPS
        initialDelaySeconds: 10
```

Fix: Change `kube-schedulerrrr` to `kube-scheduler` and save the file. The kubelet will automatically detect the change and recreate the pod with the correct configuration.

### Step 4: Wait for the Pod to Be Scheduled

After fixing the scheduler, wait for the application pod to be scheduled:

```bash
controlplane ~ ➜  k get pods --watch
NAME                   READY   STATUS              RESTARTS   AGE
app-7f9667c9d9-s4j46   0/1     ContainerCreating   0          7m54s
app-7f9667c9d9-s4j46   1/1     Running             0          7m54s
```

The pod is now successfully running.

### Step 5: Scale the Deployment

Next task is to scale the deployment to 2 pods:

```bash
controlplane ~ ➜  k edit deploy app
deployment.apps/app edited
```

In the editor, change the `replicas` value from 1 to 2 and save.

### Step 6: Investigate Why Scaling Didn't Work

After scaling the deployment, we notice that the number of pods doesn't increase:

```bash
controlplane ~ ➜  k get all -A
NAMESPACE      NAME                                       READY   STATUS             RESTARTS        AGE
default        pod/app-7f9667c9d9-s4j46                   1/1     Running            0               27m
kube-flannel   pod/kube-flannel-ds-vj5rb                  1/1     Running            0               41m
kube-system    pod/coredns-7484cd47db-48qrp               1/1     Running            0               41m
kube-system    pod/coredns-7484cd47db-vfw8x               1/1     Running            0               41m
kube-system    pod/etcd-controlplane                      1/1     Running            0               42m
kube-system    pod/kube-apiserver-controlplane            1/1     Running            0               42m
kube-system    pod/kube-controller-manager-controlplane   0/1     CrashLoopBackOff   8 (2m48s ago)   18m
kube-system    pod/kube-proxy-4z68x                       1/1     Running            0               41m
kube-system    pod/kube-scheduler-controlplane            1/1     Running            0               19m

NAMESPACE     NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   172.20.0.1    <none>        443/TCP                  42m
kube-system   service/kube-dns     ClusterIP   172.20.0.10   <none>        53/UDP,53/TCP,9153/TCP   41m

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   41m
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   41m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/app       1/2     1            1           27m
kube-system   deployment.apps/coredns   2/2     2            2           41m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
default       replicaset.apps/app-7f9667c9d9       1         1         1       27m
kube-system   replicaset.apps/coredns-7484cd47db   2         2         2       41m
```

We notice that:

- Deployment shows `1/2` pods ready (should be 2)
- The ReplicaSet still shows `DESIRED` as 1 (should be 2)
- The `kube-controller-manager-controlplane` pod is in `CrashLoopBackOff` state

### Step 7: Investigate the Controller Manager Issue

The Controller Manager is responsible for maintaining the correct number of replicas in a deployment. Let's examine why it's failing:

```bash
controlplane ~ ➜  k describe pod kube-controller-manager-controlplane -n kube-system
Name:                 kube-controller-manager-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 controlplane/192.168.28.36
Start Time:           Fri, 16 May 2025 22:09:59 +0000
Labels:               component=kube-controller-manager
                      tier=control-plane
Annotations:          kubernetes.io/config.hash: f0fc25996d5c40e4de93ac6c104994e2
                      kubernetes.io/config.mirror: f0fc25996d5c40e4de93ac6c104994e2
                      kubernetes.io/config.seen: 2025-05-16T22:32:49.449521675Z
                      kubernetes.io/config.source: file
Status:               Running
SeccompProfile:       RuntimeDefault
IP:                   192.168.28.36
IPs:
  IP:           192.168.28.36
Controlled By:  Node/controlplane
Containers:
  kube-controller-manager:
    Container ID:  containerd://f13edd74fffcbbf6830058801d84c447d9fc696d4b29d57bf8c6fb9520b2cab2
    Image:         registry.k8s.io/kube-controller-manager:v1.32.0
    Image ID:      registry.k8s.io/kube-controller-manager@sha256:c8faedf1a5f3981ffade770c696b676d30613681a95be3287c1f7eec50e49b6d
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-controller-manager
      --allocate-node-cidrs=true
      --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
      --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
      --bind-address=127.0.0.1
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --cluster-cidr=172.17.0.0/16
      --cluster-name=kubernetes
      --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
      --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
      --controllers=*,bootstrapsigner,tokencleaner
      --kubeconfig=/etc/kubernetes/controller-manager-XXXX.conf
      --leader-elect=true
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
      --root-ca-file=/etc/kubernetes/pki/ca.crt
      --service-account-private-key-file=/etc/kubernetes/pki/sa.key
      --service-cluster-ip-range=172.20.0.0/16
      --use-service-account-credentials=true
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Fri, 16 May 2025 22:49:09 +0000
      Finished:     Fri, 16 May 2025 22:49:09 +0000
    Ready:          False
    Restart Count:  8
    Requests:
      cpu:        200m
    Liveness:     http-get https://127.0.0.1:10257/healthz delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get https://127.0.0.1:10257/healthz delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/ca-certificates from etc-ca-certificates (ro)
      /etc/kubernetes/controller-manager.conf from kubeconfig (ro)
      /etc/kubernetes/pki from k8s-certs (ro)
      /etc/ssl/certs from ca-certs (ro)
      /usr/libexec/kubernetes/kubelet-plugins/volume/exec from flexvolume-dir (rw)
      /usr/local/share/ca-certificates from usr-local-share-ca-certificates (ro)
      /usr/share/ca-certificates from usr-share-ca-certificates (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  etc-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ca-certificates
    HostPathType:  DirectoryOrCreate
  flexvolume-dir:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/libexec/kubernetes/kubelet-plugins/volume/exec
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
  kubeconfig:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/controller-manager.conf
    HostPathType:  FileOrCreate
  usr-local-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/local/share/ca-certificates
    HostPathType:  DirectoryOrCreate
  usr-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/share/ca-certificates
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:
  Type     Reason   Age                  From     Message
  ----     ------   ----                 ----     -------
  Normal   Pulled   4m35s (x9 over 20m)  kubelet  Container image "registry.k8s.io/kube-controller-manager:v1.32.0" already present on machine
  Normal   Created  4m35s (x9 over 20m)  kubelet  Created container: kube-controller-manager
  Normal   Started  4m34s (x9 over 20m)  kubelet  Started container kube-controller-manager
  Warning  BackOff  40s (x99 over 20m)   kubelet  Back-off restarting failed container kube-controller-manager in pod kube-controller-manager-controlplane_kube-system(f0fc25996d5c40e4de93ac6c104994e2)
```

Now let's check the controller manager logs:

```bash
controlplane ~ ➜  k logs kube-controller-manager-controlplane -n kube-system
I0516 22:54:12.872807       1 serving.go:386] Generated self-signed cert in-memory
E0516 22:54:12.872959       1 run.go:72] "command failed" err="stat /etc/kubernetes/controller-manager-XXXX.conf: no such file or directory"
```

We identified the issue: The controller manager is trying to use a non-existent configuration file: `/etc/kubernetes/controller-manager-XXXX.conf`.

The correct file should be `/etc/kubernetes/controller-manager.conf`.

### Step 8: Fix the Controller Manager Configuration

We need to edit the controller manager's manifest file:

```bash
controlplane ~ ➜ vim /etc/kubernetes/manifests/kube-controller-manager.yaml
```

Here's the problematic section in the YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
    - command:
        - kube-controller-manager
        - --allocate-node-cidrs=true
        - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
        - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
        - --bind-address=127.0.0.1
        - --client-ca-file=/etc/kubernetes/pki/ca.crt
        - --cluster-cidr=172.17.0.0/16
        - --cluster-name=kubernetes
        - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
        - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
        - --controllers=*,bootstrapsigner,tokencleaner
        - --kubeconfig=/etc/kubernetes/controller-manager-XXXX.conf
        - --leader-elect=true
        - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
        - --root-ca-file=/etc/kubernetes/pki/ca.crt
        - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
        - --service-cluster-ip-range=172.20.0.0/16
        - --use-service-account-credentials=true
      image: registry.k8s.io/kube-controller-manager:v1.32.0
      imagePullPolicy: IfNotPresent
```

Fix: Change `--kubeconfig=/etc/kubernetes/controller-manager-XXXX.conf` to `--kubeconfig=/etc/kubernetes/controller-manager.conf` and save the file. The kubelet will automatically detect the change and recreate the pod with the correct configuration.

After fixing both the kube-scheduler and kube-controller-manager, the cluster should function properly, and the application deployment should scale to 2 pods as requested.

## Solution Summary

We identified and fixed two issues in the Kubernetes control plane:

1. **kube-scheduler issue**:

   - Problem: Typo in the command name (`kube-schedulerrrr` instead of `kube-scheduler`)
   - Solution: Corrected the command name in `/etc/kubernetes/manifests/kube-scheduler.yaml`
   - Result: Scheduler pod started correctly and was able to schedule the application pod

2. **kube-controller-manager issue**:
   - Problem: Incorrect path to kubeconfig file (`controller-manager-XXXX.conf` instead of `controller-manager.conf`)
   - Solution: Corrected the path in `/etc/kubernetes/manifests/kube-controller-manager.yaml`
   - Result: Controller manager pod started correctly and was able to scale the deployment to the desired number of replicas

These fixes restored the functionality of the Kubernetes control plane, allowing the cluster to work as expected.

## Understanding the Solution

1. **Static Pods**: Both the kube-scheduler and kube-controller-manager are static pods managed directly by the kubelet. Their configurations are stored as YAML files in the `/etc/kubernetes/manifests/` directory.

2. **Scheduler Role**: The kube-scheduler is responsible for selecting nodes for pods to run on. When it was not functioning, new pods remained in the `Pending` state.

3. **Controller Manager Role**: The kube-controller-manager is responsible for maintaining the desired state of the system, including ensuring that deployments have the correct number of replicas. When it was not functioning, scaling the deployment didn't work.

4. **Automatic Healing**: When you modify the YAML files in the manifests directory, the kubelet automatically detects the changes and recreates the pods with the updated configuration. This is a self-healing mechanism of Kubernetes.

By understanding the role of each component and how they interact, we were able to identify and fix the issues in the control plane, restoring the cluster to a fully functional state.

## Important Kubernetes Commands Used

- `kubectl get all -A`: Display all resources across all namespaces
- `kubectl describe pod <pod-name> -n <namespace>`: Show detailed information about a specific pod
- `kubectl logs <pod-name> -n <namespace>`: View the logs of a specific pod
- `kubectl edit deploy <deployment-name>`: Edit a deployment configuration
- `kubectl get pods --watch`: Watch for changes to pods in real-time

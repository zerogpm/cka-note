How many Labels exist on node node01?

```bash
controlplane ~ ➜  k describe node node01
Name:               node01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"6e:24:b2:f2:86:4f"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 192.168.129.238
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 21 May 2025 20:37:47 +0000
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  node01
  AcquireTime:     <unset>
  RenewTime:       Wed, 21 May 2025 20:51:03 +0000
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Wed, 21 May 2025 20:37:53 +0000   Wed, 21 May 2025 20:37:53 +0000   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Wed, 21 May 2025 20:47:49 +0000   Wed, 21 May 2025 20:37:47 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Wed, 21 May 2025 20:47:49 +0000   Wed, 21 May 2025 20:37:47 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Wed, 21 May 2025 20:47:49 +0000   Wed, 21 May 2025 20:37:47 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Wed, 21 May 2025 20:47:49 +0000   Wed, 21 May 2025 20:37:51 +0000   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  192.168.129.238
  Hostname:    node01
Capacity:
  cpu:                16
  ephemeral-storage:  772706776Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             65838272Ki
  pods:               110
Allocatable:
  cpu:                16
  ephemeral-storage:  712126563583
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             65735872Ki
  pods:               110
System Info:
  Machine ID:                 132e3d2451f947fe9214456160254717
  System UUID:                b142c4a6-6155-37b9-1183-1a3b031a9dde
  Boot ID:                    49b8e54a-4c75-4a41-97ae-1d42bda304ca
  Kernel Version:             5.15.0-1083-gcp
  OS Image:                   Ubuntu 22.04.4 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.26
  Kubelet Version:            v1.32.0
  Kube-Proxy Version:         v1.32.0
PodCIDR:                      172.17.1.0/24
PodCIDRs:                     172.17.1.0/24
Non-terminated Pods:          (2 in total)
  Namespace                   Name                     CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                     ------------  ----------  ---------------  -------------  ---
  kube-flannel                kube-flannel-ds-2nsqf    100m (0%)     0 (0%)      50Mi (0%)        0 (0%)         13m
  kube-system                 kube-proxy-m49qp         0 (0%)        0 (0%)      0 (0%)           0 (0%)         13m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                100m (0%)  0 (0%)
  memory             50Mi (0%)  0 (0%)
  ephemeral-storage  0 (0%)     0 (0%)
  hugepages-1Gi      0 (0%)     0 (0%)
  hugepages-2Mi      0 (0%)     0 (0%)
Events:
  Type     Reason                   Age                From             Message
  ----     ------                   ----               ----             -------
  Normal   Starting                 13m                kube-proxy
  Normal   Starting                 13m                kubelet          Starting kubelet.
  Warning  InvalidDiskCapacity      13m                kubelet          invalid capacity 0 on image filesystem
  Warning  CgroupV1                 13m                kubelet          cgroup v1 support is in maintenance mode, please migrate to cgroup v2
  Normal   RegisteredNode           13m                node-controller  Node node01 event: Registered Node node01 in Controller
  Normal   NodeHasSufficientMemory  13m (x2 over 13m)  kubelet          Node node01 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    13m (x2 over 13m)  kubelet          Node node01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     13m (x2 over 13m)  kubelet          Node node01 status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  13m                kubelet          Updated Node Allocatable limit across pods
  Normal   NodeReady                13m                kubelet          Node node01 status is now: NodeReady
```

What is the value set to the label key beta.kubernetes.io/arch on node01?

Apply a label color=blue to node node01

controlplane ~ ➜ k label node node01 color=blue
node/node01 labeled

Create a new deployment named blue with the nginx image and 3 replicas.

k create deploy blue --image=nginx --replicas=3

Which nodes can the pods for the blue deployment be placed on?

Make sure to check taints on both nodes!

```bash
controlplane ~ ➜  k get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   18m   v1.32.0
node01         Ready    <none>          17m   v1.32.0

controlplane ~ ➜  k describe node node01
Name:               node01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    color=blue
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"6e:24:b2:f2:86:4f"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 192.168.129.238
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 21 May 2025 20:37:47 +0000
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  node01
  AcquireTime:     <unset>
  RenewTime:       Wed, 21 May 2025 20:55:48 +0000
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Wed, 21 May 2025 20:37:53 +0000   Wed, 21 May 2025 20:37:53 +0000   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Wed, 21 May 2025 20:55:19 +0000   Wed, 21 May 2025 20:37:47 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Wed, 21 May 2025 20:55:19 +0000   Wed, 21 May 2025 20:37:47 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Wed, 21 May 2025 20:55:19 +0000   Wed, 21 May 2025 20:37:47 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Wed, 21 May 2025 20:55:19 +0000   Wed, 21 May 2025 20:37:51 +0000   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  192.168.129.238
  Hostname:    node01
Capacity:
  cpu:                16
  ephemeral-storage:  772706776Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             65838272Ki
  pods:               110
Allocatable:
  cpu:                16
  ephemeral-storage:  712126563583
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             65735872Ki
  pods:               110
System Info:
  Machine ID:                 132e3d2451f947fe9214456160254717
  System UUID:                b142c4a6-6155-37b9-1183-1a3b031a9dde
  Boot ID:                    49b8e54a-4c75-4a41-97ae-1d42bda304ca
  Kernel Version:             5.15.0-1083-gcp
  OS Image:                   Ubuntu 22.04.4 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.26
  Kubelet Version:            v1.32.0
  Kube-Proxy Version:         v1.32.0
PodCIDR:                      172.17.1.0/24
PodCIDRs:                     172.17.1.0/24
Non-terminated Pods:          (4 in total)
  Namespace                   Name                     CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                     ------------  ----------  ---------------  -------------  ---
  default                     blue-d7967dc55-rxtvg     0 (0%)        0 (0%)      0 (0%)           0 (0%)         45s
  default                     blue-d7967dc55-wb66k     0 (0%)        0 (0%)      0 (0%)           0 (0%)         45s
  kube-flannel                kube-flannel-ds-2nsqf    100m (0%)     0 (0%)      50Mi (0%)        0 (0%)         18m
  kube-system                 kube-proxy-m49qp         0 (0%)        0 (0%)      0 (0%)           0 (0%)         18m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                100m (0%)  0 (0%)
  memory             50Mi (0%)  0 (0%)
  ephemeral-storage  0 (0%)     0 (0%)
  hugepages-1Gi      0 (0%)     0 (0%)
  hugepages-2Mi      0 (0%)     0 (0%)
Events:
  Type     Reason                   Age                From             Message
  ----     ------                   ----               ----             -------
  Normal   Starting                 18m                kube-proxy
  Normal   Starting                 18m                kubelet          Starting kubelet.
  Warning  InvalidDiskCapacity      18m                kubelet          invalid capacity 0 on image filesystem
  Warning  CgroupV1                 18m                kubelet          cgroup v1 support is in maintenance mode, please migrate to cgroup v2
  Normal   RegisteredNode           18m                node-controller  Node node01 event: Registered Node node01 in Controller
  Normal   NodeHasSufficientMemory  18m (x2 over 18m)  kubelet          Node node01 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    18m (x2 over 18m)  kubelet          Node node01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     18m (x2 over 18m)  kubelet          Node node01 status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  18m                kubelet          Updated Node Allocatable limit across pods
  Normal   NodeReady                18m                kubelet          Node node01 status is now: NodeReady

controlplane ~ ➜  k describe node controlplane
Name:               controlplane
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=controlplane
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"c2:af:61:f0:1d:91"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 192.168.242.168
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 21 May 2025 20:37:07 +0000
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  controlplane
  AcquireTime:     <unset>
  RenewTime:       Wed, 21 May 2025 20:55:51 +0000
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Wed, 21 May 2025 20:37:17 +0000   Wed, 21 May 2025 20:37:17 +0000   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Wed, 21 May 2025 20:55:41 +0000   Wed, 21 May 2025 20:37:07 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Wed, 21 May 2025 20:55:41 +0000   Wed, 21 May 2025 20:37:07 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Wed, 21 May 2025 20:55:41 +0000   Wed, 21 May 2025 20:37:07 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Wed, 21 May 2025 20:55:41 +0000   Wed, 21 May 2025 20:37:15 +0000   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  192.168.242.168
  Hostname:    controlplane
Capacity:
  cpu:                16
  ephemeral-storage:  772706776Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             65838272Ki
  pods:               110
Allocatable:
  cpu:                16
  ephemeral-storage:  712126563583
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             65735872Ki
  pods:               110
System Info:
  Machine ID:                 46cb0b5c00f54031b637ce02a7343826
  System UUID:                1f3975ab-7e62-2502-2d4e-a0643ffd158c
  Boot ID:                    874c9d4a-65a7-4429-ab24-4ff6ace26421
  Kernel Version:             5.15.0-1081-gcp
  OS Image:                   Ubuntu 22.04.5 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.26
  Kubelet Version:            v1.32.0
  Kube-Proxy Version:         v1.32.0
PodCIDR:                      172.17.0.0/24
PodCIDRs:                     172.17.0.0/24
Non-terminated Pods:          (9 in total)
  Namespace                   Name                                    CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                    ------------  ----------  ---------------  -------------  ---
  default                     blue-d7967dc55-x9lrc                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         53s
  kube-flannel                kube-flannel-ds-9qfbg                   100m (0%)     0 (0%)      50Mi (0%)        0 (0%)         18m
  kube-system                 coredns-7484cd47db-d6n4z                100m (0%)     0 (0%)      70Mi (0%)        170Mi (0%)     18m
  kube-system                 coredns-7484cd47db-jvdwj                100m (0%)     0 (0%)      70Mi (0%)        170Mi (0%)     18m
  kube-system                 etcd-controlplane                       100m (0%)     0 (0%)      100Mi (0%)       0 (0%)         18m
  kube-system                 kube-apiserver-controlplane             250m (1%)     0 (0%)      0 (0%)           0 (0%)         18m
  kube-system                 kube-controller-manager-controlplane    200m (1%)     0 (0%)      0 (0%)           0 (0%)         18m
  kube-system                 kube-proxy-mhnvs                        0 (0%)        0 (0%)      0 (0%)           0 (0%)         18m
  kube-system                 kube-scheduler-controlplane             100m (0%)     0 (0%)      0 (0%)           0 (0%)         18m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                950m (5%)   0 (0%)
  memory             290Mi (0%)  340Mi (0%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-1Gi      0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:
  Type     Reason                   Age   From             Message
  ----     ------                   ----  ----             -------
  Normal   Starting                 18m   kube-proxy
  Normal   Starting                 18m   kubelet          Starting kubelet.
  Warning  CgroupV1                 18m   kubelet          cgroup v1 support is in maintenance mode, please migrate to cgroup v2
  Warning  InvalidDiskCapacity      18m   kubelet          invalid capacity 0 on image filesystem
  Normal   NodeAllocatableEnforced  18m   kubelet          Updated Node Allocatable limit across pods
  Normal   NodeHasSufficientMemory  18m   kubelet          Node controlplane status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    18m   kubelet          Node controlplane status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     18m   kubelet          Node controlplane status is now: NodeHasSufficientPID
  Normal   RegisteredNode           18m   node-controller  Node controlplane event: Registered Node controlplane in Controller
  Normal   NodeReady                18m   kubelet          Node controlplane status is now: NodeReady
```

Set Node Affinity to the deployment to place the pods on node01 only.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
        - image: nginx
          imagePullPolicy: Always
          name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: color
                    operator: In
                    values:
                      - blue
```

Which nodes are the pods placed on now?

```bash
controlplane ~ ✖ k get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
blue-679c44c97c-5fdr7   1/1     Running   0          67s   172.17.1.6   node01   <none>           <none>
blue-679c44c97c-9rlpt   1/1     Running   0          67s   172.17.1.4   node01   <none>           <none>
blue-679c44c97c-kxfgx   1/1     Running   0          67s   172.17.1.5   node01   <none>           <none>
```

Create a new deployment named red with the nginx image and 2 replicas, and ensure it gets placed on the controlplane node only.

Use the label key - node-role.kubernetes.io/control-plane - which is already set on the controlplane node.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
        - image: nginx
          imagePullPolicy: Always
          name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: node-role.kubernetes.io/control-plane
                    operator: Exists
```

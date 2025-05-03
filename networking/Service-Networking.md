What network range are the nodes in the cluster part of?

k get nodes -o wide

```bash
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
controlplane   Ready    control-plane   26m   v1.32.0   192.6.173.11   <none>        Ubuntu 22.04.5 LTS   5.4.0-1106-gcp   containerd://1.6.26
node01         Ready    <none>          25m   v1.32.0   192.6.173.3    <none>        Ubuntu 22.04.4 LTS   5.4.0-1106-gcp   containerd://1.6.26
```

check for the node ip with the range
ip add

you can filter information like this
-A 3 show 3 lines -E is regular expression

ip add | grep -A 3 -E "eth[0-9]+@[0-9a-z]+"

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 32:0d:55:79:a1:c3 brd ff:ff:ff:ff:ff:ff
4: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP group default qlen 1000
    link/ether 76:cb:be:0f:a2:ae brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.1/16 brd 10.244.255.255 scope global weave
       valid_lft forever preferred_lft forever
6: vethwe-datapath@vethwe-bridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master datapath state UP group default
    link/ether 4e:8b:ee:63:76:51 brd ff:ff:ff:ff:ff:ff
7: vethwe-bridge@vethwe-datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP group default
    link/ether 4a:7b:4f:2b:59:1f brd ff:ff:ff:ff:ff:ff
8: vxlan-6784: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65535 qdisc noqueue master datapath state UNKNOWN group default qlen 1000
    link/ether a2:fa:3f:f6:63:66 brd ff:ff:ff:ff:ff:ff
10: vethweple404b6c@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP group default
    link/ether 72:0e:15:04:b7:78 brd ff:ff:ff:ff:ff:ff link-netns cni-d484a071-56a9-86ea-d31f-47f763c376f6
12: vethwepl346f68e@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP group default
    link/ether aa:ae:7b:e0:ab:5c brd ff:ff:ff:ff:ff:ff link-netns cni-1501776b-407a-18cb-ebbc-026d3c948f5a
5134: eth0@if5135: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default
    link/ether 02:42:c0:06:ad:0b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.6.173.11/24 brd 192.6.173.255 scope global eth0
       valid_lft forever preferred_lft forever
5138: eth1@if5139: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:19:00:0e brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 172.25.0.14/24 brd 172.25.0.255 scope global eth1
       valid_lft forever preferred_lft forever
```

The answer is that the Kubernetes nodes in this cluster are part of the 192.6.173.0/24 network range. This can be verified through the following commands and their outputs:

kubectl get nodes -o wide shows:

The controlplane node has INTERNAL-IP 192.6.173.11
The node01 worker has INTERNAL-IP 192.6.173.3
Both nodes are using IPs from the same 192.6.173.0/24 subnet

ip add confirms:

The eth0 interface has IP 192.6.173.11/24
This matches the controlplane node's internal IP
The subnet mask (/24) defines the network range as 192.6.173.0/24

Additional Network Information

Pod Network: 10.244.0.0/16 (implemented via Weave CNI)
Node Network: 192.6.173.0/24 (for node-to-node communication)
Additional Network: 172.25.0.0/24 (on eth1 interface)

The commands demonstrate that both Kubernetes nodes have been assigned IP addresses from the 192.6.173.0/24 network range, making this the correct answer for the cluster's node network.

What is the range of IP addresses configured for PODs on this cluster?

```bash

k get all -A #<this list all pods in a namespace>

NAMESPACE     NAME                                       READY   STATUS    RESTARTS      AGE
kube-system   pod/coredns-7484cd47db-ndgsn               1/1     Running   0             87m
kube-system   pod/coredns-7484cd47db-rvkv2               1/1     Running   0             87m
kube-system   pod/etcd-controlplane                      1/1     Running   0             87m
kube-system   pod/kube-apiserver-controlplane            1/1     Running   0             87m
kube-system   pod/kube-controller-manager-controlplane   1/1     Running   0             87m
kube-system   pod/kube-proxy-ckm84                       1/1     Running   0             86m
kube-system   pod/kube-proxy-vlzvh                       1/1     Running   0             87m
kube-system   pod/kube-scheduler-controlplane            1/1     Running   0             87m
kube-system   pod/weave-net-g9bh2                        2/2     Running   1 (87m ago)   87m
kube-system   pod/weave-net-mvlrb                        2/2     Running   0             86m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  87m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   87m

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   87m
kube-system   daemonset.apps/weave-net    2         2         2       2            2           <none>                   87m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           87m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-7484cd47db   2         2         2       87m
```

check ipalloc-range

```bash
k logs weave-net-jxcmw -n kube-system | grep -i ipalloc-range:

Defaulted container "weave" out of: weave, weave-npc, weave-init (init)
INFO: 2025/05/03 22:34:27.076106 Command line options: map[conn-limit:200 datapath:datapath db-prefix:/weavedb/weave-net docker-api: expect-npc:true http-addr:127.0.0.1:6784 ipalloc-init:consensus=1 ipalloc-range:10.244.0.0/16 metrics-addr:0.0.0.0:6782 name:f6:cd:9d:99:24:5e nickname:node01 no-dns:true no-masq-local:true port:6783]
```

What is the IP Range configured for the services within the cluster?

we might need take a look at kube-apiserver.yaml

cat

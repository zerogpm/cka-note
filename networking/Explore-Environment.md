# Kubernetes Cluster Documentation

This document provides information about the Kubernetes cluster configuration and network setup.

## Cluster Overview

The cluster consists of 2 nodes:

- 1 control plane node (controlplane)
- 1 worker node (node01)

```bash
# View nodes in the cluster
kubectl get node

# View detailed node information
kubectl get node -o wide
```

**Output:**

```
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
controlplane   Ready    control-plane   24m   v1.32.0   192.168.65.209    <none>        Ubuntu 22.04.5 LTS   5.15.0-1077-gcp   containerd://1.6.26
node01         Ready    <none>          24m   v1.32.0   192.168.129.217   <none>        Ubuntu 22.04.4 LTS   5.15.0-1075-gcp   containerd://1.6.26
```

## Network Configuration

### Control Plane Node

**IP Address:** 192.168.65.209  
**MAC Address:** 7e:1f:64:dd:2b:80

```bash
# View network interfaces on controlplane
ip address
```

**Key Interfaces:**

- **eth0:** Primary network interface (192.168.65.209/32)
- **flannel.1:** Overlay network for pod-to-pod communication (172.17.0.0/32)
- **cni0:** Container Network Interface bridge (172.17.0.1/24)

#### CNI Details

The cluster uses Flannel and the Container Network Interface (CNI) for networking:

```bash
# Check the CNI bridge status
ip address show cni0
```

**Output:**

```
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UP group default qlen 1000
link/ether 16:08:a7:d4:15:5d brd ff:ff:ff:ff:ff:ff
inet 172.17.0.1/24 brd 172.17.0.255 scope global cni0
valid_lft forever preferred_lft forever
inet6 fe80::1408:a7ff:fed4:155d/64 scope link
valid_lft forever preferred_lft forever
```

### Worker Node

**IP Address:** 192.168.129.217

To check the worker node's MAC address:

```bash
ssh node01
ip address
```

### Routing Configuration

Default gateway for external network communication:

```bash
# View routing table
ip route
```

**Output:**

```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         169.254.1.1     0.0.0.0         UG    0      0        0 eth0
169.254.1.1     0.0.0.0         255.255.255.255 UH    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.255.0   U     0      0        0 cni0
172.17.1.0      172.17.1.0      255.255.255.0   UG    0      0        0 flannel.1
```

**Default Gateway:** 169.254.1.1

## Kubernetes Components

### Control Plane Services

```bash
# Check listening ports
netstat -plnt
```

**Key Service Ports:**

- **kube-scheduler:** 10259
- **kube-controller:** 10257
- **etcd:** 2379 (client), 2380 (peer)
- **kube-apiserver:** 6443
- **kubelet:** 10250
- **kube-proxy:** 10256

### Container Runtime

The cluster uses Containerd (version 1.6.26) as the container runtime.

## ETCD Information

ETCD listens on two ports:

- **2379:** Client communication port (used by control plane components)
- **2380:** Peer-to-peer communication port (used in multi-control plane setups)

```bash
# Check ETCD connections
netstat -npa | grep -i etcd
```

Note: Port 2379 has more client connections as it's used by all control plane components, while 2380 is only for etcd peer-to-peer communication.

# Kubernetes Networking Guide: Weave-Net Configuration

## Cluster Networking Solution

This cluster uses Weave Net as its networking solution. This can be verified by checking the CNI configuration directory:

```bash
ls /etc/cni/net.d/
10-weave.conflist
```

## Weave Deployment Details

### Weave Agents

The cluster has two weave agents/peers deployed:

```bash
kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS      AGE
coredns-7484cd47db-k546v               1/1     Running   0             59m
coredns-7484cd47db-vsg9v               1/1     Running   0             59m
etcd-controlplane                      1/1     Running   0             59m
kube-apiserver-controlplane            1/1     Running   0             59m
kube-controller-manager-controlplane   1/1     Running   0             59m
kube-proxy-5wv25                       1/1     Running   0             59m
kube-proxy-f4zr4                       1/1     Running   0             59m
kube-scheduler-controlplane            1/1     Running   0             59m
weave-net-p8lqs                        2/2     Running   0             59m
weave-net-rnw6l                        2/2     Running   1 (59m ago)   59m
```

### Node Distribution

Weave peers are distributed with one on each node in the cluster:

```bash
kubectl get pods -n kube-system -o wide
```

## Weave Network Configuration

### Network Interface

The bridge network/interface created by Weave on each node is named `weave`.

```bash
ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid_lft forever preferred_lft forever
2: datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UNKNOWN group default qlen 1000
link/ether 32:0d:55:79:a1:c3 brd ff:ff:ff:ff:ff:ff
4: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP group default qlen 1000
link/ether c2:e7:fb:1a:91:d9 brd ff:ff:ff:ff:ff:ff
inet 10.244.0.1/16 brd 10.244.255.255 scope global weave
valid_lft forever preferred_lft forever
# ... other interfaces omitted for brevity
```

### IP Address Range

The Pod IP address range configured by Weave can be found by examining the logs:

```bash
kubectl logs -n kube-system weave-net-jwbc9 | grep -i ipalloc-range
```

This will show the IP allocation range (typically 10.244.0.0/16).

## Pod Network Configuration

### Default Gateway

To check the default gateway configured for pods on node01, deploy a test pod:

1. Create a pod specification file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  nodeName: node01
  containers:
    - args:
        - sleep
        - "1000"
      image: busybox
      name: busybox
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

2. Apply the configuration:

```bash
kubectl apply -f busybox.yaml
```

3. Check the pod's routing table:

```bash
kubectl exec busybox -- ip route
default via 10.244.192.0 dev eth0
10.244.0.0/16 dev eth0 scope link src 10.244.192.1
```

The default gateway for pods on node01 is `10.244.192.0`.

## Troubleshooting Pod Updates

If you encounter an error like:

```
The Pod "busybox" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`,`spec.initContainers[*].image`,`spec.activeDeadlineSeconds`,`spec.tolerations` (only additions to existing tolerations),`spec.terminationGracePeriodSeconds`
```

Remember that Pods are largely immutable. To update a pod configuration, you can:

1. Delete and recreate the pod:

```bash
kubectl delete pod busybox
kubectl apply -f busybox.yaml
```

2. Or use the force replace option:

```bash
kubectl replace --force -f busybox.yaml
```

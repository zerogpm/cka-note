# Kubernetes ETCD Backup and Restore Guide

## 1. Checking ETCD Cluster Configuration

First, check how many nodes are part of the external ETCD cluster:

You need to ssh into extrnal EC server

```bash
ssh etcd-server
```

Then look at the process

```bash
ps -ef | grep etcd
```

Example output:

```
etcd 779 1 0 17:54 ? 00:00:38 /usr/local/bin/etcd --name etcd-server --data-dir=/var/lib/etcd-data --cert-file=/etc/etcd/pki/etcd.pem --key-file=/etc/etcd/pki/etcd-key.pem --peer-cert-file=/etc/etcd/pki/etcd.pem --peer-key-file=/etc/etcd/pki/etcd-key.pem --trusted-ca-file=/etc/etcd/pki/ca.pem --peer-trusted-ca-file=/etc/etcd/pki/ca.pem --peer-client-cert-auth --client-cert-auth --initial-advertise-peer-urls https://192.168.28.29:2380 --listen-peer-urls https://192.168.28.29:2380 --advertise-client-urls https://192.168.28.29:2379 --listen-client-urls https://192.168.28.29:2379,https://127.0.0.1:2379 --initial-cluster-token etcd-cluster-1 --initial-cluster etcd-server=https://192.168.28.29:2380 --initial-cluster-state new
root 996 914 0 18:34 pts/0 00:00:00 grep etcd
```

## 2. Setting ETCD API Version

```bash
export ETCDCTL_API=3
```

## 3. Listing ETCD Cluster Members

Use the client URLs from the ETCD configuration:

```bash
etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/etcd/pki/ca.pem \
--cert=/etc/etcd/pki/etcd.pem \
--key=/etc/etcd/pki/etcd-key.pem \
member list
```

Example output:

```
a857094cb473a041, started, etcd-server, https://192.168.28.29:2380, https://192.168.28.29:2379, false
```

## 4. Creating a Snapshot for Cluster1

### 4.1 Get Cluster1 Information

```bash
kubectl describe pod etcd-cluster1-controlplane -n kube-system
```

### 4.2 Find Cluster1 Nodes

```bash
kubectl get node
```

Example output:

```
cluster1-controlplane   Ready    control-plane   68m   v1.29.0
cluster1-node01         Ready    <none>          67m   v1.29.0
```

### 4.3 Create the Snapshot

SSH into the control plane node:

```bash
ssh cluster1-controlplane
```

Create the snapshot:

```bash
export ETCDCTL_API=3

etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/cluster1.db
```

Verify the snapshot was created:

```bash
ls /opt/
```

Exit the control plane node:

```bash
exit
```

### 4.4 Copy the Snapshot to Your Local Machine

```bash
scp cluster1-controlplane:/opt/cluster1.db /opt/
```

## 5. Restoring External ETCD Data

### 5.1 Copy the Snapshot to the ETCD Server

```bash
scp /opt/cluster2.db etcd-server:/root/
```

### 5.2 SSH into the ETCD Server

```bash
ssh etcd-server
```

### 5.3 Verify the Snapshot

```bash
ls -la
```

### 5.4 Set ETCD API Version

```bash
export ETCDCTL_API=3
```

### 5.5 Restore the Snapshot

```bash
etcdctl snapshot restore /root/cluster2.db --data-dir /var/lib/etcd-data-new
```

### 5.6 Set Correct Ownership

```bash
chown -R etcd:etcd /var/lib/etcd-data-new/
```

### 5.7 Update ETCD Configuration

Edit the ETCD service file:

```bash
vim /etc/systemd/system/etcd.service
```

Update the data directory in the configuration:

```
[Unit]
Description=etcd key-value store
Documentation=https://github.com/etcd-io/etcd
After=network.target

[Service]
User=etcd
Type=notify
ExecStart=/usr/local/bin/etcd \
 --name etcd-server \
 --data-dir=/var/lib/etcd-data-new \ # <-- Changed this line
 --cert-file=/etc/etcd/pki/etcd.pem \
 --key-file=/etc/etcd/pki/etcd-key.pem \
 --peer-cert-file=/etc/etcd/pki/etcd.pem \
 --peer-key-file=/etc/etcd/pki/etcd-key.pem \
 --trusted-ca-file=/etc/etcd/pki/ca.pem \
 --peer-trusted-ca-file=/etc/etcd/pki/ca.pem \
 --peer-client-cert-auth \
 --client-cert-auth \
 --initial-advertise-peer-urls https://192.168.48.42:2380 \
 --listen-peer-urls https://192.168.48.42:2380 \
 --advertise-client-urls https://192.168.48.42:2379 \
 --listen-client-urls https://192.168.48.42:2379,https://127.0.0.1:2379 \
 --initial-cluster-token etcd-cluster-1 \
 --initial-cluster etcd-server=https://192.168.48.42:2380 \
 --initial-cluster-state new
Restart=on-failure
RestartSec=5
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```

### 5.8 Reload and Restart ETCD

```bash
systemctl daemon-reload
systemctl restart etcd
```

### 5.9 Apply Changes to the Cluster

Return to the student node and delete pods to refresh the configuration:

```bash
kubectl delete pods kube-controller-manager-cluster2-controlplane kube-scheduler-cluster2-controlplane -n kube-system
```

### 5.10 Restart Kubelet on Cluster2

```bash
ssh cluster2-controlplane
systemctl restart kubelet
```

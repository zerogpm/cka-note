# ETCD Backup and Restore Guide

This guide outlines the process for creating and restoring backups of etcd in a Kubernetes cluster.

## Prerequisites

- Access to the Kubernetes master node
- Sufficient permissions to access etcd configuration files and execute system commands

## Steps for Backing Up ETCD

### 1. Gather Information from etcd Configuration

First, examine the etcd configuration file to get necessary certificate paths and endpoints:

```bash
cat /etc/kubernetes/manifests/etcd.yaml
```

Take note of the following information:
- Endpoint address (typically https://127.0.0.1:2379)
- Certificate paths:
  - CA certificate: `/etc/kubernetes/pki/etcd/ca.crt`
  - Server certificate: `/etc/kubernetes/pki/etcd/server.crt`
  - Server key: `/etc/kubernetes/pki/etcd/server.key`

### 2. Set Up etcdctl Environment

Open a new terminal window (keeping the first terminal with the etcd.yaml information open), and set the etcdctl API version:

```bash
export ETCDCTL_API=3
```

### 3. Verify etcdctl Setup

Confirm that etcdctl is properly configured:

```bash
etcdctl version
```

Review the available options:

```bash
etcdctl -h
```

Pay special attention to these required options:
- `--endpoints`
- `--cacert`
- `--cert`
- `--key`

### 4. Create an ETCD Snapshot

Using the information from step 1, create a snapshot:

```bash
etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/snapshot-pre-boot.db
```

## Steps for Restoring ETCD

### 1. Restore the Snapshot

Use the same connection parameters as for the backup, but add the `--data-dir` option and change the command from `save` to `restore`:

```bash
etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --data-dir=/var/lib/etcd-from-backup \
  snapshot restore /opt/snapshot-pre-boot.db
```

### 2. Update the ETCD Configuration

Edit the etcd static pod configuration to use the restored data directory:

```bash
vim /etc/kubernetes/manifests/etcd.yaml
```

Search for the `hostPath` section (/hostPath:) and update the path from:

```yaml
- hostPath:
    path: /var/lib/etcd
    type: DirectoryOrCreate
  name: etcd-data
```

to:

```yaml
- hostPath:
    path: /var/lib/etcd-from-backup
    type: DirectoryOrCreate
  name: etcd-data
```

### 3. Restart Kubelet to Apply Changes

Restart the kubelet service to load the updated configuration:

```bash
systemctl restart kubelet
sudo systemctl daemon-reload
```

### 4. Troubleshooting

If the changes don't take effect, try:

1. Moving the manifest files to a temporary location and then back:

```bash
mkdir /temp
mv /etc/kubernetes/manifests/*.yaml /temp/
# Wait for kubelet to detect the missing manifests
mv /temp/*.yaml /etc/kubernetes/manifests/
```

2. Restart the kubelet again:

```bash
systemctl restart kubelet
sudo systemctl daemon-reload
```

## Verification

After completing the restoration, verify that:
1. The etcd pods have restarted successfully
2. All Kubernetes components are functioning properly
3. Your previously backed-up data is accessible

## Best Practices

- Regularly schedule etcd backups as part of your disaster recovery plan
- Test the restoration process periodically to ensure backups are valid
- Store backups in multiple secure locations
- Document the specific certificate paths and endpoints for your environment

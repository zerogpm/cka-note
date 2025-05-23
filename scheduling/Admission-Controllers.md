# Kubernetes Admission Controllers Configuration

This guide demonstrates how to configure Kubernetes admission controllers, specifically enabling `NamespaceAutoProvision` and disabling `DefaultStorageClass` admission controllers.

## Table of Contents

- [Problem Statement](#problem-statement)
- [Prerequisites](#prerequisites)
- [Configuration Steps](#configuration-steps)
- [YAML Configuration](#yaml-configuration)
- [Verification Commands](#verification-commands)
- [Troubleshooting](#troubleshooting)

## Problem Statement

### Original Questions/Problems:

1. **What is not a function of admission controller?**

   - Answer: `authenticate user`

2. **Which admission controller is not enabled by default?**

```bash
k exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins'
```

- Answer: `NamespaceAutoProvision`

3. **Which admission controller is enabled in this cluster which is normally disabled?**
   - Solution: Use `grep enable-admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml`

### Core Issue:

The Kubernetes cluster has the `NamespaceExists` admission controller enabled, which rejects requests to namespaces that do not exist. To automatically create namespaces that don't exist, we need to enable the `NamespaceAutoProvision` admission controller.

## Prerequisites

- Access to the Kubernetes control plane node
- Sudo/root privileges to modify kube-apiserver configuration
- Understanding that kube-apiserver will automatically restart after configuration changes

## Configuration Steps

### Step 1: Check Current Admission Plugins

```bash
grep enable-admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml
```

This command searches for the current admission plugins configuration in the kube-apiserver manifest file.

### Step 2: Enable NamespaceAutoProvision Admission Controller

Edit the kube-apiserver configuration file:

```bash
sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

**Important:** After updating the kube-apiserver YAML file, wait a few minutes for the kube-apiserver to restart completely.

### Step 3: Disable DefaultStorageClass Admission Controller (Optional)

The `DefaultStorageClass` admission controller automatically adds a default storage class to PersistentVolumeClaim objects that don't request a specific storage class. To disable it, add the disable flag to the configuration.

## YAML Configuration

### Updated kube-apiserver.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.104.19:6443
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
        - --advertise-address=192.168.104.19
        - --allow-privileged=true
        - --authorization-mode=Node,RBAC
        - --client-ca-file=/etc/kubernetes/pki/ca.crt
        - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
        - --disable-admission-plugins=DefaultStorageClass
        - --enable-bootstrap-token-auth=true
        - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
        - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
        - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
        - --etcd-servers=https://127.0.0.1:2379
        - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
        - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
        - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
        - --requestheader-allowed-names=front-proxy-client
        - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
        - --requestheader-extra-headers-prefix=X-Remote-Extra-
        - --requestheader-group-headers=X-Remote-Group
        - --requestheader-username-headers=X-Remote-User
        - --secure-port=6443
        - --service-account-issuer=https://kubernetes.default.svc.cluster.local
        - --service-account-key-file=/etc/kubernetes/pki/sa.pub
        - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
        - --service-cluster-ip-range=172.20.0.0/16
        - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
        - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
      image: registry.k8s.io/kube-apiserver:v1.32.0
      imagePullPolicy: IfNotPresent
      livenessProbe:
        failureThreshold: 8
        httpGet:
          host: 192.168.104.19
          path: /livez
          port: 6443
          scheme: HTTPS
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 15
      name: kube-apiserver
      readinessProbe:
        failureThreshold: 3
        httpGet:
          host: 192.168.104.19
          path: /readyz
          port: 6443
          scheme: HTTPS
        periodSeconds: 1
        timeoutSeconds: 15
      resources:
        requests:
          cpu: 250m
      startupProbe:
        failureThreshold: 24
        httpGet:
          host: 192.168.104.19
          path: /livez
          port: 6443
          scheme: HTTPS
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 15
      volumeMounts:
        - mountPath: /etc/ssl/certs
          name: ca-certs
          readOnly: true
        - mountPath: /etc/ca-certificates
          name: etc-ca-certificates
          readOnly: true
        - mountPath: /etc/kubernetes/pki
          name: k8s-certs
          readOnly: true
        - mountPath: /usr/local/share/ca-certificates
          name: usr-local-share-ca-certificates
          readOnly: true
        - mountPath: /usr/share/ca-certificates
          name: usr-share-ca-certificates
          readOnly: true
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
    - hostPath:
        path: /etc/ssl/certs
        type: DirectoryOrCreate
      name: ca-certs
    - hostPath:
        path: /etc/ca-certificates
        type: DirectoryOrCreate
      name: etc-ca-certificates
    - hostPath:
        path: /etc/kubernetes/pki
        type: DirectoryOrCreate
      name: k8s-certs
    - hostPath:
        path: /usr/local/share/ca-certificates
        type: DirectoryOrCreate
      name: usr-local-share-ca-certificates
    - hostPath:
        path: /usr/share/ca-certificates
        type: DirectoryOrCreate
      name: usr-share-ca-certificates
status: {}
```

### Key Configuration Changes

#### Enable NamespaceAutoProvision

```yaml
- --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

#### Disable DefaultStorageClass (Optional)

```yaml
- --disable-admission-plugins=DefaultStorageClass
```

## Verification Commands

### Check Running kube-apiserver Process

```bash
ps -ef | grep kube-apiserver | grep admission-plugins
```

This command will show the currently running kube-apiserver process and display the enabled/disabled admission plugins.

### Verify Configuration File

```bash
grep -E "(enable|disable)-admission-plugins" /etc/kubernetes/manifests/kube-apiserver.yaml
```

## Step-by-Step Command Explanation

1. **`grep enable-admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml`**

   - Searches for the current admission plugins configuration
   - Helps identify which plugins are currently enabled
   - Useful for troubleshooting and verification

2. **`ps -ef | grep kube-apiserver | grep admission-plugins`**
   - Lists all running processes (`ps -ef`)
   - Filters for kube-apiserver process (`grep kube-apiserver`)
   - Further filters for admission-plugins configuration (`grep admission-plugins`)
   - Shows the actual runtime configuration of the API server

## Why This Approach Solves the Problem

### Problem Analysis:

- **NamespaceExists Controller**: Rejects requests to non-existent namespaces
- **User Need**: Automatically create namespaces when they don't exist
- **Solution**: Enable `NamespaceAutoProvision` admission controller

### How NamespaceAutoProvision Works:

1. **Intercepts Requests**: Catches requests to non-existent namespaces
2. **Auto-Creation**: Automatically creates the namespace before processing the request
3. **Seamless Experience**: Users don't need to pre-create namespaces manually

### Benefits:

- **Improved User Experience**: No need to manually create namespaces
- **Reduced Errors**: Eliminates "namespace not found" errors
- **Automation**: Supports GitOps and automated deployment workflows

### DefaultStorageClass Disable Rationale:

- **Custom Control**: Prevents automatic assignment of default storage classes
- **Explicit Configuration**: Forces users to specify storage requirements explicitly
- **Resource Management**: Better control over storage provisioning

## Troubleshooting

### Common Issues:

1. **API Server Not Restarting**

   - Wait 2-3 minutes after configuration changes
   - Check kubelet logs: `journalctl -u kubelet`

2. **Configuration Not Applied**

   - Verify YAML syntax is correct
   - Ensure file permissions are appropriate
   - Check for typos in admission controller names

3. **Verification Commands**

   ```bash
   # Check API server pod status
   kubectl get pods -n kube-system | grep kube-apiserver

   # Check API server logs
   kubectl logs -n kube-system kube-apiserver-<node-name>
   ```

## Important Notes

- The kube-apiserver runs as a static pod managed by kubelet
- Configuration changes trigger automatic restart of the API server
- Always wait for the restart to complete before testing changes
- Keep backups of configuration files before making changes
- Test changes in a development environment first

## References

- [Kubernetes Admission Controllers Documentation](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
- [kube-apiserver Configuration Reference](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)

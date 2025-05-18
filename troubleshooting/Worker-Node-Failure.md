# Kubernetes Node Troubleshooting

This guide documents the process of troubleshooting and resolving issues with Kubernetes nodes that appear in a `NotReady` state. The document walks through diagnosing and fixing common kubelet configuration issues that can cause node connectivity problems.

## Problem Statement

The original problem presented was:

```
Fix the broken nodes

controlplane ~ ➜  k get nodes
NAME           STATUS     ROLES           AGE     VERSION
controlplane   Ready      control-plane   5m10s   v1.32.0
node01         NotReady   <none>          4m25s   v1.32.0

controlplane ~ ➜
```

The Kubernetes cluster has a node (`node01`) in a `NotReady` state.

## Troubleshooting Process

### 1. Initial Investigation

First, we check the current node status to identify the issue:

```bash
controlplane ~ ➜  k get nodes
NAME           STATUS     ROLES           AGE     VERSION
controlplane   Ready      control-plane   5m10s   v1.32.0
node01         NotReady   <none>          4m25s   v1.32.0
```

### 2. SSH into the Problematic Node

We need to connect to the problematic node to investigate:

```bash
controlplane ~ ✖ ssh node01
```

### 3. Check Kubelet Service Status

Since the node is `NotReady`, we first check the status of the kubelet service, which is the primary node agent that communicates with the Kubernetes API server:

```bash
node01 ~ ➜  service kubelet status
○ kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead) since Sat 2025-05-17 17:53:53 UTC; 3min 27s ago
       Docs: https://kubernetes.io/docs/
    Process: 2587 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=0/SUCCESS)
   Main PID: 2587 (code=exited, status=0/SUCCESS)
```

The output shows that the kubelet service is inactive, which explains why the node is `NotReady`.

### 4. Start the Kubelet Service

We attempt to start the kubelet service:

```bash
node01 ~ ✖ service kubelet start
```

### 5. Verify Kubelet Service Status Again

After starting the service, we check its status:

```bash
node01 ~ ➜  service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Sat 2025-05-17 17:58:36 UTC; 2s ago
       Docs: https://kubernetes.io/docs/
   Main PID: 6609 (kubelet)
      Tasks: 20 (limit: 77143)
     Memory: 22.5M
     CGroup: /system.slice/kubelet.service
             └─6609 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml
```

Now the service is active and running, but we encounter additional issues.

### 6. Investigating Kubelet Crash

The kubelet service starts but then fails:

```bash
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Sat 2025-05-17 18:00:52 UTC; 2s ago
       Docs: https://kubernetes.io/docs/
    Process: 7934 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAILURE)
   Main PID: 7934 (code=exited, status=1/FAILURE)
```

### 7. Checking Kubelet Service Logs

We examine the kubelet logs to identify the issue:

```bash
node01 ~ ➜  journalctl -u kubelet
```

In the logs, we find the key error message:

```
"command failed" err="failed to load kubelet config file, path: /var/lib/kubelet/config.yaml, error:"
```

### 8. Examining Kubelet Configuration

We check the kubelet configuration file:

```bash
node01 ~ ➜  cd /var/lib/kubelet/
node01 /var/lib/kubelet ➜  ls -la
total 56
drwxr-xr-x 9 root root 4096 May 17 17:59 .
drwxr-xr-x 1 root root 4096 Dec 17 12:19 ..
drwx------ 2 root root 4096 May 17 17:51 checkpoints
-rw-r--r-- 1 root root 1136 May 17 17:59 config.yaml
-rw------- 1 root root   62 May 17 17:51 cpu_manager_state
drwxr-xr-x 2 root root 4096 May 17 17:58 device-plugins
-rw-r--r-- 1 root root  150 May 17 17:51 kubeadm-flags.env
-rw-r--r-- 1 root root    0 Dec 11 18:39 .kubelet-keep
-rw------- 1 root root   61 May 17 17:51 memory_manager_state
drwxr-xr-x 2 root root 4096 May 17 17:51 pki
drwxr-x--- 2 root root 4096 May 17 17:51 plugins
drwxr-x--- 2 root root 4096 May 17 17:51 plugins_registry
drwxr-x--- 2 root root 4096 May 17 17:58 pod-resources
drwxr-x--- 4 root root 4096 May 17 17:51 pods
```

And examine its contents:

```bash
node01 /var/lib/kubelet ➜  cat config.yaml
```

### 9. First Configuration Issue: Incorrect CA File Path

We identify the first issue - an incorrect CA file path:

```yaml
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/WRONG-CA-FILE.crt
```

The file path `/etc/kubernetes/pki/WRONG-CA-FILE.crt` is clearly incorrect. It should be `/etc/kubernetes/pki/ca.crt`.

### 10. Fix the CA File Path and Restart Kubelet

We modify the configuration file and restart kubelet:

```bash
# Modified the file to use the correct CA path: /etc/kubernetes/pki/ca.crt
# Then restart kubelet
node01 /var/lib/kubelet ➜  service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Sat 2025-05-17 18:21:43 UTC; 15s ago
```

### 11. Second Investigation - API Server Connection Issues

Despite fixing the CA file path, we encounter more issues:

```bash
node01 ~ ➜  service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Sun 2025-05-18 03:08:01 UTC; 8min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 14594 (kubelet)
```

The service is running but logs show connection errors:

```
E0518 03:16:24.078928   14594 controller.go:145] "Failed to ensure lease exists, will retry" err="Get \"https://controlplane:6553/apis/coord>"
```

### 12. Examining Kubelet Configuration File

We check the kubelet configuration file to identify the API server connection issue:

```bash
node01 /etc/kubernetes ➜  cat kubelet.conf
```

### 13. Second Configuration Issue: Incorrect API Server Port

We find the second issue - an incorrect API server port in the `kubelet.conf` file:

```yaml
clusters:
  - cluster:
      certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ...
      server: https://controlplane:6553
    name: default-cluster
```

The port is incorrectly set to `6553` instead of the standard Kubernetes API server port `6443`.

### 14. Fix the API Server Port

The solution is to modify the `kubelet.conf` file to use the correct port:

- Change `https://controlplane:6553` to `https://controlplane:6443`

After making this change and restarting kubelet, the node should return to a `Ready` state.

## Summary of Issues and Fixes

### Issue 1: Incorrect CA File Path

- **Problem**: The kubelet configuration specified a non-existent CA file: `/etc/kubernetes/pki/WRONG-CA-FILE.crt`
- **Solution**: Changed the path to the correct one: `/etc/kubernetes/pki/ca.crt`

### Issue 2: Incorrect API Server Port

- **Problem**: The kubelet was trying to connect to the API server on port `6553` instead of `6443`
- **Solution**: Updated the server URL in the kubelet configuration from `https://controlplane:6553` to `https://controlplane:6443`

## Commands Used for Troubleshooting

1. Check node status: `k get nodes`
2. SSH into problematic node: `ssh node01`
3. Check kubelet service status: `service kubelet status`
4. Start kubelet service: `service kubelet start`
5. View kubelet logs: `journalctl -u kubelet`
6. Examine configuration files:
   - `cat /var/lib/kubelet/config.yaml`
   - `cat /etc/kubernetes/kubelet.conf`

## Why This Approach Works

This troubleshooting approach works because it follows a systematic process:

1. **Identify the symptom**: Node in `NotReady` state
2. **Examine the primary node agent**: Check kubelet service status
3. **Analyze logs**: Look for specific error messages
4. **Inspect configurations**: Find misconfigured paths and connections
5. **Make targeted fixes**: Correct specific configuration issues
6. **Verify solutions**: Ensure the node returns to `Ready` state

By addressing the root causes (incorrect CA file path and API server port), we restore proper communication between the node and the control plane, which is essential for a functional Kubernetes cluster.

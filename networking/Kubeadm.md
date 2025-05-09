# Kubernetes Cluster Setup Guide

## Original Problem

> Install the kubeadm, kubelet and kubectl packages at exact version 1.24.0 on the controlplane and node01.

## Solution Overview

This guide provides detailed instructions for setting up a Kubernetes cluster with one control plane node and one worker node using kubeadm. The setup includes configuring kernel parameters, installing the required Kubernetes components, initializing the control plane, and setting up a network plugin (Flannel).

## Prerequisites

- Two nodes: controlplane and node01
- Root or sudo access on both nodes
- Container runtime (containerD) pre-installed on both nodes

## Step-by-Step Implementation

### 1. Configure Kernel Parameters on Both Nodes

First, we need to enable IPv4 packet forwarding on both the controlplane and worker nodes.

```bash
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

Verify that the parameter is correctly set:

```bash
sysctl net.ipv4.ip_forward
```

Expected output:

```
net.ipv4.ip_forward = 1
```

### 2. Set Up Kubernetes Package Repository on Both Nodes

Create the directory for apt keyrings if it doesn't exist:

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
cd /etc/apt/keyrings
```

Update package lists and install necessary dependencies:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

Add Kubernetes repository key and create the repository file:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### 3. Install Kubernetes Components on Both Nodes

Update package lists to include the Kubernetes repository:

```bash
sudo apt-get update
```

Check available versions (optional):

```bash
apt-cache madison kubeadm
```

Install the specified versions of kubeadm, kubelet, and kubectl:

```bash
sudo apt-get install -y kubelet=1.32.0-1.1 kubeadm=1.32.0-1.1 kubectl=1.32.0-1.1
```

Prevent automatic updates of Kubernetes components:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

### 4. Initialize the Control Plane Node

On the controlplane node only, first check the IP address:

```bash
ifconfig
```

Sample output:

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1410
        inet 192.168.75.150  netmask 255.255.255.255  broadcast 0.0.0.0
        inet6 fe80::9834:f9ff:feee:73ea  prefixlen 64  scopeid 0x20<link>
        ether 9a:34:f9:ee:73:ea  txqueuelen 0  (Ethernet)
        RX packets 14631  bytes 140550021 (140.5 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12482  bytes 2244460 (2.2 MB)
        TX errors 0  dropped 1 overruns 0  carrier 0  collisions 0
```

Initialize the Kubernetes control plane with specific network configurations:

```bash
kubeadm init \
 --apiserver-advertise-address=192.168.75.150 \
 --apiserver-cert-extra-sans=controlplane \
 --pod-network-cidr=172.17.0.0/16 \
 --service-cidr=172.20.0.0/16
```

Set up kubeconfig for the current user:

```bash
# Create the .kube directory in your home directory if it doesn't exist
mkdir -p $HOME/.kube

# Copy the Kubernetes admin configuration to your user's .kube directory
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

# Change the ownership of the configuration file to your user
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 5. Join the Worker Node to the Cluster

After initializing the control plane, a join command will be displayed. Run this command on the worker node:

```bash
kubeadm join 192.168.231.159:6443 --token vn8rym.t694sfw6lux18bxa \
        --discovery-token-ca-cert-hash sha256:a54674bee7a79fa4029c3c6c1366852649064026acab97110629494f7c9556c0
```

Note: The actual values for the token and discovery-token-ca-cert-hash will be different in your environment.

### 6. Install and Configure the Network Plugin (Flannel)

On the control plane node, download the Flannel manifest:

```bash
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Edit the kube-flannel.yml file to match our pod CIDR configuration:

```bash
vim kube-flannel.yml
```

Change the Network value in the ConfigMap from `10.244.0.0/16` to `172.17.0.0/16`:

```yaml
net-conf.json: |
  {
    "Network": "172.17.0.0/16",
    "Backend": {
      "Type": "vxlan"
    }
  }
```

Also, add the `--iface=eth0` argument to the Flannel container to use the correct network interface:

```yaml
args:
  - --ip-masq
  - --kube-subnet-mgr
  - --iface=eth0
```

Apply the modified manifest:

```bash
kubectl apply -f kube-flannel.yml
```

### 7. Verify the Cluster Setup

Check the status of the nodes:

```bash
kubectl get nodes
```

Expected output:

```
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   15m   v1.32.0
node01         Ready    <none>          15m   v1.32.0
```

## Troubleshooting Network Configuration

If you forget to edit the Flannel configuration before applying it, you can fix it with:

```bash
kubectl edit cm -n kube-flannel kube-flannel-cfg
```

Change `"Network": "10.244.0.0/16"` to `"Network": "172.17.0.0/16"`.

To explicitly set Flannel to use eth0, you can add this to the ConfigMap:

- In the net-conf.json section, add: `"Interface": "eth0"`

Or use annotations:

```bash
kubectl annotate node <node-name> flannel.alpha.coreos.com/public-ip=<node-ip>
```

After making these changes, restart the Flannel pods:

```bash
kubectl rollout restart ds -n kube-flannel kube-flannel-ds
```

## Why This Approach Works

1. **Kernel Parameter Configuration**: Enabling IP forwarding is essential for container networking in Kubernetes.

2. **Specific Version Installation**: Using pinned versions (`1.32.0-1.1`) ensures compatibility between components and prevents automatic updates that might break the cluster.

3. **Custom Network Configuration**:

   - The Pod CIDR (`172.17.0.0/16`) and Service CIDR (`172.20.0.0/16`) are configured to be non-overlapping.
   - Using `--apiserver-advertise-address` ensures the API server binds to the correct interface.
   - The `--apiserver-cert-extra-sans` option adds the hostname to the API server certificate for easier access.

4. **Network Plugin Configuration**:
   - Flannel is configured to use the same Pod CIDR as specified during cluster initialization.
   - Using the `--iface=eth0` argument ensures Flannel uses the correct network interface for inter-host communication.

This setup creates a functional Kubernetes cluster with proper networking between pods across nodes, which is essential for a working Kubernetes environment.

# Kubernetes Cluster Setup - Version 1.33.0

## Original Problem Statement

**Prepare both nodes (controlplane and node01) for Kubernetes by completing the following steps:**

1. Apply the necessary sysctl parameters for networking.
2. Install the kubeadm and kubelet packages at the exact version 1.33.0-1.1 on both nodes.
3. Install the kubectl package at the exact version 1.33.0-1.1 exclusively on the controlplane.
4. Initialize Control Plane Node (Master Node) with specific network configurations.

## Solution Overview

This solution provides a complete step-by-step process to set up a Kubernetes cluster using kubeadm with version 1.33.0. The approach ensures proper networking configuration, installs exact package versions to maintain consistency across nodes, and initializes the control plane with custom network CIDRs.

## Prerequisites

- Two Ubuntu/Debian nodes (controlplane and node01)
- Root or sudo access on both nodes
- Container runtime already installed
- Network connectivity between nodes

## Step 1: Configure Network Settings (Both Nodes)

### Apply Kernel Modules and Sysctl Parameters

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
```

### Why These Network Settings Are Required

The sysctl parameters are essential for container networking across the entire Kubernetes cluster:

- **`br_netfilter` module**: Enables bridge netfilter functionality, allowing iptables rules to work with bridged network traffic (container communication)
- **`net.bridge.bridge-nf-call-iptables = 1`**: Ensures bridged network packets are processed by iptables rules, crucial for:
  - Kubernetes service networking and load balancing
  - Container-to-container communication across nodes
  - Network policies and traffic routing
- **`net.bridge.bridge-nf-call-ip6tables = 1`**: Same functionality for IPv6 traffic

**Why both nodes need this**: Every node runs container runtime, kubelet, and kube-proxy, all requiring proper network bridge functionality.

## Step 2: Install Kubernetes Components (Both Nodes)

### Update Package Repository

```bash
# Refresh package lists from all configured repositories
sudo apt-get update
```

### Install Required Dependencies

```bash
# Install HTTPS transport, certificates, and curl for secure package downloads
sudo apt-get install -y apt-transport-https ca-certificates curl
```

**Package purposes:**

- `apt-transport-https`: Enables apt to download over HTTPS
- `ca-certificates`: Contains trusted certificate authorities
- `curl`: Downloads files from the internet

### Add Kubernetes GPG Key

```bash
# Download and install Kubernetes GPG key for package verification
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

**Command breakdown:**

- `curl -fsSL`: Downloads the GPG key
  - `-f`: Fail silently on server errors
  - `-s`: Silent mode (no progress bar)
  - `-S`: Show errors even in silent mode
  - `-L`: Follow redirects
- `sudo gpg --dearmor`: Converts key to binary format
- GPG keys verify package authenticity and prevent tampering

### Add Kubernetes Repository

```bash
# Add Kubernetes repository to apt sources
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

**Configuration breakdown:**

- `deb`: Debian package repository
- `[signed-by=...]`: Specifies GPG key for verification
- Repository URL points to Kubernetes v1.33 packages

### Update Package Index with New Repository

```bash
# Refresh package lists including new Kubernetes repository
sudo apt-get update
```

### Verify Available Versions

```bash
# Display all available kubeadm versions
sudo apt-cache madison kubeadm
```

### Install Kubernetes Components

```bash
# Install exact versions of Kubernetes components
sudo apt-get install -y kubelet=1.33.0-1.1 kubeadm=1.33.0-1.1 kubectl=1.33.0-1.1
```

**Component functions:**

- **`kubelet`**: Node agent that manages pods and containers
- **`kubeadm`**: Cluster bootstrapping tool (init/join commands)
- **`kubectl`**: Command-line interface for Kubernetes clusters

### Prevent Automatic Updates

```bash
# Lock package versions to prevent automatic updates
sudo apt-mark hold kubelet kubeadm kubectl
```

**Why lock versions**: Kubernetes clusters require all nodes to run identical versions to prevent compatibility issues.

## Step 3: Initialize Control Plane (Master Node Only)

### Get Network Interface IP Address

```bash
# Extract IP address from eth0 interface
IP_ADDR=$(ip addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
```

### Initialize Kubernetes Cluster

```bash
# Initialize control plane with custom network configuration
kubeadm init --apiserver-cert-extra-sans=controlplane --apiserver-advertise-address $IP_ADDR --pod-network-cidr=172.17.0.0/16 --service-cidr=172.20.0.0/16
```

**Configuration parameters:**

- `--apiserver-advertise-address`: IP address for API server communication
- `--apiserver-cert-extra-sans=controlplane`: Additional certificate subject alternative names
- `--pod-network-cidr=172.17.0.0/16`: IP range for pod networking
- `--service-cidr=172.20.0.0/16`: IP range for services

### Expected Console Output

```bash
[init] Using Kubernetes version: v1.33.1
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: cgroups v1 support is in maintenance mode, please migrate to cgroups v2
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
W0520 03:05:34.890872   11137 checks.go:846] detected that the sandbox image "registry.k8s.io/pause:3.6" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.10" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [controlplane kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [172.20.0.1 192.168.9.95]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [controlplane localhost] and IPs [192.168.9.95 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [controlplane localhost] and IPs [192.168.9.95 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 1.001862161s
[control-plane-check] Waiting for healthy control plane components. This can take up to 4m0s
[control-plane-check] Checking kube-apiserver at https://192.168.9.95:6443/livez
[control-plane-check] Checking kube-controller-manager at https://127.0.0.1:10257/healthz
[control-plane-check] Checking kube-scheduler at https://127.0.0.1:10259/livez
[control-plane-check] kube-scheduler is healthy after 12.743187359s
[control-plane-check] kube-controller-manager is healthy after 17.259346714s
[control-plane-check] kube-apiserver is healthy after 35.501756821s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node controlplane as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node controlplane as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: 50bd8b.x003t0tq1fzmultg
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.9.95:6443 --token 50bd8b.x003t0tq1fzmultg \
        --discovery-token-ca-cert-hash sha256:70a59cdbfc5f5a3f0d49ca90198525870da1fe6f02def53d80a2c00fcc4bde72
```

## Step 4: Configure kubectl Access (Control Plane Only)

### For Regular User

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### For Root User

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## Configuration Files

### Network Module Configuration (`/etc/modules-load.d/k8s.conf`)

```yaml
br_netfilter
```

### Sysctl Network Parameters (`/etc/sysctl.d/k8s.conf`)

```yaml
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

### Kubernetes Repository Configuration (`/etc/apt/sources.list.d/kubernetes.list`)

```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /
```

## Why This Approach Solves the Problem

This solution addresses the original requirements through a systematic approach:

### 1. **Network Configuration**

- Configures essential kernel modules and sysctl parameters on both nodes
- Ensures iptables integration with container networking
- Enables proper communication between pods and services

### 2. **Version Consistency**

- Installs exact package versions (1.33.0-1.1) on all nodes
- Prevents automatic updates that could cause version drift
- Ensures cluster stability and component compatibility

### 3. **Secure Package Management**

- Uses official Kubernetes repository with GPG verification
- Prevents installation of unauthorized or malicious packages
- Maintains package authenticity throughout the process

### 4. **Custom Network Architecture**

- Implements specific CIDR ranges for pods (172.17.0.0/16) and services (172.20.0.0/16)
- Configures API server with proper advertise address
- Adds certificate SANs for hostname resolution

### 5. **Automated Configuration**

- Uses dynamic IP detection to avoid hardcoding addresses
- Generates all necessary certificates and configurations
- Provides join command for worker nodes

## Next Steps

1. **Deploy Pod Network**: Install a CNI plugin (e.g., Flannel, Calico, Weave)
2. **Join Worker Nodes**: Use the provided `kubeadm join` command on node01
3. **Verify Cluster**: Check node status with `kubectl get nodes`
4. **Deploy Applications**: Start deploying workloads to your cluster

## Troubleshooting

- **Network Issues**: Verify firewall rules allow traffic on ports 6443, 2379-2380, 10250-10252
- **DNS Resolution**: Ensure nodes can resolve each other's hostnames
- **Container Runtime**: Verify container runtime is running and properly configured
- **Certificates**: Check certificate validity if API server connection fails

## Security Considerations

- Change default tokens in production environments
- Implement RBAC policies for user access control
- Enable audit logging for security monitoring
- Use network policies to restrict pod-to-pod communication
- Regularly update and patch all cluster components

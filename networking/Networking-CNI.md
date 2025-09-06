# Flannel to Calico CNI Migration

## Original Problem

**Question:** Which CNI plugin is currently installed on the cluster?

**Issue:** Two applications (frontend and backend) have been provisioned in the default namespace. A network policy `deny-backend` has been provisioned to block traffic to the backend app, but the curl command from the frontend app to the backend app succeeded and returned the NGINX welcome page. This should not be the expected behavior given that we have a deny-backend network policy in place.

**Root Cause:** The Flannel CNI does not support NetworkPolicies, which is why the network policy was not being enforced.

## Current CNI Configuration

### Identifying the Current CNI

```bash
cat /etc/cni/net.d/10-flannel.conflist
```

**Output:**

```json
{
  "name": "cbr0",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```

This confirms that **Flannel** is currently installed as the CNI plugin.

## Testing Network Policy Enforcement

### Get Pod Information

```bash
kubectl get pods -o wide
```

**Output:**

```
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
backend    1/1     Running   0          6m31s   172.17.0.5   controlplane   <none>           <none>
frontend   1/1     Running   0          6m31s   172.17.0.4   controlplane   <none>           <none>
```

### Test Network Connectivity

```bash
kubectl exec -it frontend -- curl -m 5 172.17.0.5
```

**Result:** The curl command succeeded and returned the NGINX welcome page, proving that the network policy is not being enforced.

## Solution: Migrate from Flannel to Calico

### Why This Approach Solves the Problem

Flannel CNI **does not support NetworkPolicies**. NetworkPolicy is a Kubernetes feature that requires CNI plugin support for enforcement. Calico CNI **does support NetworkPolicies**, making it the appropriate choice when network segmentation and security policies are required.

### Step 1: Remove Flannel CNI

Remove the Flannel DaemonSet and ConfigMap:

```bash
kubectl delete daemonset -n kube-flannel kube-flannel-ds
kubectl delete cm kube-flannel-cfg -n kube-flannel
```

Remove the Flannel CNI configuration file:

```bash
rm /etc/cni/net.d/10-flannel.conflist
```

**Explanation:**

- `kubectl delete daemonset` removes the Flannel pods running on all nodes
- `kubectl delete cm` removes the Flannel configuration stored in the cluster
- `rm /etc/cni/net.d/10-flannel.conflist` removes the local CNI configuration file

### Step 2: Install Calico CNI

#### Install the Tigera Operator

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/tigera-operator.yaml
```

**Explanation:** The Tigera operator manages the lifecycle of Calico components in the Kubernetes cluster.

#### Download Custom Resource Definition

```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/custom-resources.yaml -O
```

**Explanation:** This downloads the custom resource definition that configures how Calico will be installed and configured.

### Step 3: Configure Calico for the Cluster

#### Edit the Custom Resource Definition

Create/edit `custom-resources.yaml`:

```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    ipPools:
      - name: default-ipv4-ippool
        blockSize: 26
        cidr: 172.17.0.0/16 # Updated to match existing pod network
        encapsulation: VXLANCrossSubnet
        natOutgoing: Enabled
        nodeSelector: all()
```

**Key Configuration Changes:**

- `cidr: 172.17.0.0/16`: Updated from the default to match the existing pod IP range (172.17.0.4, 172.17.0.5 as seen in the pod listing)
- `encapsulation: VXLANCrossSubnet`: Enables VXLAN encapsulation for cross-subnet communication
- `natOutgoing: Enabled`: Enables NAT for traffic leaving the cluster

#### Apply the Configuration

```bash
kubectl apply -f custom-resources.yaml
```

**Explanation:** This creates the Calico installation with the custom configuration, ensuring pod networking continues to work with the existing IP addresses.

## Verification

After the migration is complete, the same network policy test should now properly enforce the `deny-backend` policy:

```bash
kubectl exec -it frontend -- curl -m 5 172.17.0.5
```

This command should now timeout or be denied, proving that NetworkPolicies are being enforced.

## Key Differences: Flannel vs Calico

| Feature               | Flannel           | Calico              |
| --------------------- | ----------------- | ------------------- |
| NetworkPolicy Support | ❌ No             | ✅ Yes              |
| Network Segmentation  | Basic             | Advanced            |
| Security Policies     | None              | Comprehensive       |
| Performance           | Lightweight       | Feature-rich        |
| Use Case              | Simple networking | Production security |

## References

- [Calico Installation Guide](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)
- [Calico v3.29.3 Manifests](https://github.com/projectcalico/calico/tree/v3.29.3/manifests)

## Summary

This migration solves the original problem by replacing Flannel (which doesn't support NetworkPolicies) with Calico (which does support NetworkPolicies). The key steps are:

1. **Remove Flannel** completely from the cluster
2. **Install Calico** with proper CIDR configuration
3. **Verify** that NetworkPolicies now work as expected

The CIDR configuration (`172.17.0.0/16`) ensures compatibility with existing pod IP addresses, making this a seamless migration that enables NetworkPolicy enforcement without breaking existing connectivity.

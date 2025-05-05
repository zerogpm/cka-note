# Kubernetes DNS Troubleshooting Guide

This guide documents the steps taken to troubleshoot and solve DNS-related issues in a Kubernetes cluster.

## CoreDNS Investigation

### Identifying the DNS Solution

```bash
# Check for DNS pods in kube-system namespace
kubectl get pods -n kube-system
```

This command showed that CoreDNS is the DNS solution implemented in the cluster. The output displays multiple pods including CoreDNS pods.

### Counting CoreDNS Pods

```bash
# Count the number of CoreDNS pods
kubectl get all -A | grep -E "pod/coredns-*" | wc -l
```

This command counts the number of CoreDNS pods deployed in the cluster by:

1. Getting all resources across all namespaces
2. Filtering for CoreDNS pods using grep
3. Counting the lines with wc -l

### Finding the CoreDNS Service

```bash
# Check for DNS service in kube-system namespace
kubectl get svc -n kube-system
```

This revealed the service `kube-dns` with ClusterIP `172.20.0.10` which is used for accessing CoreDNS. This IP should be configured on pods to resolve services.

### Locating the CoreDNS Configuration

```bash
# Describe CoreDNS pod to find configuration details
kubectl describe pod coredns-7484cd47db-pgq4m -n kube-system
```

From the output, we found:

- The configuration file is located at `/etc/coredns/Corefile`
- The file is mounted as a volume named `config-volume`

### Understanding How Corefile is Mounted

```bash
# Export pod details to YAML for inspection
kubectl get pod coredns-7484cd47db-pgq4m -n kube-system -o yaml > view.yaml
```

The YAML output revealed that the Corefile is mounted from a ConfigMap named `coredns`.

### Finding and Examining the CoreDNS ConfigMap

```bash
# List available ConfigMaps
kubectl get configmap -n kube-system

# Describe the CoreDNS ConfigMap
kubectl describe configmap coredns -n kube-system
```

From the ConfigMap, we learned:

- The root domain/zone configured is `cluster.local`
- The CoreDNS configuration details are stored in this ConfigMap

## DNS Service Resolution Testing

### Service Access Testing

Testing which service names could be used to access the HR service from the test pod.

Valid formats:

- `web-service.default.svc`
- `web-service.default`
- `web-service`

Invalid format:

- `web-service.default.pod` (doesn't follow Kubernetes DNS naming convention)

Testing payroll service access:

Valid formats:

- `web-service.payroll.svc.cluster.local`
- `web-service.payroll.svc`
- `web-service.payroll`

Invalid format:

- `web-service.payroll.svc.cluster` (must use full `cluster.local` suffix)

## Troubleshooting Cross-Namespace DNS Resolution

### Problem Identification

Web application fails to connect to MySQL database with error "Can't connect to MySQL server on 'mysql:3306' (-2 Name does not resolve)".

### Investigation Steps

```bash
# Find all pods across namespaces
kubectl get pods -A

# Check services in payroll namespace
kubectl get svc -n payroll

# Examine the deployment configuration
kubectl describe deploy webapp
```

The issue was discovered: the web application was trying to connect to `mysql` without specifying the namespace, but the MySQL service is in the `payroll` namespace.

### Solution

```bash
# Edit the deployment to add namespace to DB_Host
kubectl edit deploy webapp
```

Modified the environment variable:

```yaml
- name: DB_Host
  value: mysql.payroll # Changed from just "mysql"
```

This change allows the web application to properly resolve the MySQL service in the payroll namespace.

### Verification

```bash
# Test DNS resolution from a pod
kubectl exec hr -- nslookup mysql.payroll > /root/CKA/nslookup.out
```

The nslookup output confirmed the DNS resolution works correctly, showing:

- DNS server: `172.20.0.10` (kube-dns service IP)
- Resolved address: `mysql.payroll.svc.cluster.local` to `172.20.128.131`

## Key Kubernetes DNS Concepts Demonstrated

1. Services are accessible within Kubernetes via DNS names
2. Format: `<service-name>.<namespace>.svc.cluster.local`
3. Services in the same namespace can be accessed by just `<service-name>`
4. Cross-namespace access requires at least `<service-name>.<namespace>`
5. CoreDNS serves as the cluster DNS provider
6. DNS configuration is stored in a ConfigMap in the kube-system namespace
7. The DNS service IP (`172.20.0.10`) is automatically configured for pods

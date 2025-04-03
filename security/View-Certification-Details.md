# Kubernetes Certificate Management Guide

## Identifying Certificate Files in Kubernetes

### Kube-API Server Certificate

To check the certificate file used for the kube-api server, examine the YAML configuration file:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

In the command section, look for this entry:

```yaml
- --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
```

### Kube-API Server as Client to ETCD Server

The certificate file used to authenticate kube-apiserver as a client to ETCD Server:

```yaml
--etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
```

### Kube-API Server to Kubelet Server Authentication Key

The key used to authenticate kubeapi-server to the kubelet server:

```yaml
--kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
```

### ETCD Server Certificate

To identify the ETCD Server Certificate, check the ETCD configuration:

```bash
cat /etc/kubernetes/manifests/etcd.yaml
```

Output shows:

```yaml
- command:
    - etcd
    - --advertise-client-urls=https://192.168.100.160:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
  # more config...
```

The ETCD Server Certificate is:

```yaml
--cert-file=/etc/kubernetes/pki/etcd/server.crt
```

### ETCD Server CA Root Certificate

ETCD can have its own CA, which may be different from the one used by kube-api server:

```yaml
--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

## Certificate Details

### Kube-API Server Certificate Common Name (CN)

To find the Common Name configured on the Kube API Server Certificate:

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

Output shows:

```
Subject: CN = kube-apiserver
```

Note: Don't confuse this with the Issuer CN (which is "kubernetes").

### Certificate Issuer

The certificate issuer is:

```
Issuer: CN = kubernetes
```

### Subject Alternative Names (SANs)

Alternative names configured on the Kube API Server Certificate:

```
X509v3 Subject Alternative Name:
DNS:controlplane, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP Address:172.20.0.1, IP Address:192.168.100.160
```

Note: "kube-master" is not configured.

### ETCD Server Certificate Common Name

```bash
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text -noout
```

Output shows:

```
Subject: CN = controlplane
```

### Certificate Validity Periods

#### Kube-API Server Certificate Validity

```
Validity
Not Before: Apr 3 16:11:37 2025 GMT
Not After : Apr 3 16:16:37 2026 GMT
```

Valid for: 1 year

#### Root CA Certificate Validity

```
Validity
Not Before: Apr 3 16:11:37 2025 GMT
Not After : Apr 1 16:16:37 2035 GMT
```

Valid for: 10 years

## Troubleshooting

### Scenario: Kubectl stops responding

If kubectl suddenly stops responding with an error like:

```
The connection to the server controlplane:6443 was refused - did you specify the right host or port?
```

This indicates the API server is down or cannot access ETCD. Investigate using container runtime tools:

```bash
crictl ps -a | grep kube-apiserver
crictl ps -a | grep etcd
```

> Note: In modern Kubernetes deployments (especially after v1.24), `crictl` is preferred over `docker ps` as Docker is no longer the default container runtime. Most clusters use containerd or CRI-O.

#### Check container logs:

```bash
crictl logs <container-id>
```

#### Potential Issues and Solutions:

1. **Incorrect certificate file path**:

   ```
   "open /etc/kubernetes/pki/etcd/server-certificate.crt: no such file or directory"
   ```

   Solution: Check if the file exists and correct the path in the configuration file.

2. **TLS handshake failure**:
   ```
   "transport: authentication handshake failed: tls: failed to verify certificate: x509: certificate signed by unknown authority"
   ```
   Solution: Ensure the correct CA certificate is being used. For example, if API server is using:
   ```yaml
   --etcd-cafile=/etc/kubernetes/pki/ca.crt
   ```
   It should likely be using the ETCD-specific CA:
   ```yaml
   --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
   ```

Remember that ETCD typically uses its own separate CA, so ensure that the API server is configured to use the correct CA certificate when communicating with ETCD.

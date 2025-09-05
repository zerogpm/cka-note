Check controplane services

```bash
service kube-api-server status
```

```bash
service kube-controller-manager status
```

```bash
service kube-scheduler status
```

```bash
service kube-apiserver status
```

```bash
service kubelet status
```

```bash
service kube-proxy status
```

```bash
service logs kube-apiserver-master -n kube-system
```

```bash
sudo journalctl -u kube-apiserver
```

# troubleshoot files that might useful to look at

```bash
controlplane /etc/kubernetes ➜  ls
admin.conf  controller-manager.conf  kubelet.conf  manifests  pki  scheduler.conf  super-admin.conf
```

```bash
controlplane /etc/kubernetes/manifests ➜  ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

To seamlessly transition from Kubernetes v1.32 to v1.33 and gain access to the packages specific to the desired Kubernetes minor version, follow these essential steps during the upgrade process. This ensures that your environment is appropriately configured and aligned with the features and improvements introduced in Kubernetes v1.33.

vim /etc/apt/sources.list.d/kubernetes.list

deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /

```bash
apt update

apt-cache madison kubeadm
```

create CA

generate keys

```bash
openssl genrsa -out ca.key 2048
```

certificate signing request

```bash
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```

sign certificate

```bash
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

creat a client

make have a group system:masters as admin

```bash

openssl genrsa -out admin.key 2048

openssl req -new -key admin.key -subj \"/CN=kube-admin/O=system:masters" -out  admin.csr

openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt

```

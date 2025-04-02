# Kubernetes Basic Authentication Setup

## Warning

**This is not recommended in a production environment.** This setup is only for learning purposes. Also note that this approach is deprecated in Kubernetes version 1.19 and is no longer available in later releases.

## Overview

This guide walks through configuring basic authentication in a kubeadm setup. Basic authentication uses a simple CSV file containing usernames, passwords, and user IDs to authenticate users to the Kubernetes API server.

## Step 1: Create the User Details File

Create the directory and file for user credentials:

```bash
mkdir -p /tmp/users
vim /tmp/users/user-details.csv
```

Add the following content to the file:

```
password123,user1,u0001
password123,user2,u0002
password123,user3,u0003
password123,user4,u0004
password123,user5,u0005
```

## Step 2: Configure the API Server

Edit the kube-apiserver manifest:

```bash
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add the volume mount to the container spec:

```yaml
volumeMounts:
  - mountPath: /tmp/users
    name: usr-details
    readOnly: true
```

Add the volume definition:

```yaml
volumes:
  - hostPath:
      path: /tmp/users
      type: DirectoryOrCreate
    name: usr-details
```

Add the basic auth file option to the command arguments:

```yaml
- --basic-auth-file=/tmp/users/user-details.csv
```

Your kube-apiserver.yaml should look something like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
    - command:
        - kube-apiserver
        - --authorization-mode=Node,RBAC
          <content-hidden>
        - --basic-auth-file=/tmp/users/user-details.csv
      image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
      name: kube-apiserver
      volumeMounts:
        - mountPath: /tmp/users
          name: usr-details
          readOnly: true
  volumes:
    - hostPath:
        path: /tmp/users
        type: DirectoryOrCreate
      name: usr-details
```

## Step 3: Set Up RBAC

Create a file for the RBAC configuration:

```bash
vim basic-auth-rbac.yaml
```

Add the following content:

```yaml
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
---
# This role binding allows "user1" to read pods in the "default" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
  - kind: User
    name: user1 # Name is case sensitive
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

Apply the RBAC configuration:

```bash
kubectl apply -f basic-auth-rbac.yaml
```

## Step 4: Test Authentication

After the API server restarts (which happens automatically when the manifest is changed), test the authentication:

```bash
curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"
```

If successful, you should see a list of pods in the default namespace that user1 has permission to view.

## Important Notes

1. The API server pod will automatically restart when the manifest file is modified.
2. This method is deprecated since Kubernetes 1.19 and removed in later versions.
3. For production environments, consider using more secure authentication methods like:
   - X.509 client certificates
   - OpenID Connect tokens with an identity provider
   - Service account tokens for applications within the cluster

This setup is intended for learning purposes only to understand how authentication works in Kubernetes.

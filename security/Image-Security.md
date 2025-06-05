# Kubernetes Private Registry Authentication

## Problem Statement

**Original Question**: "What secret type must we choose for docker registry?"

This tutorial demonstrates how to configure a Kubernetes deployment to pull images from a private Docker registry using proper authentication credentials.

## Overview

When working with private container registries, Kubernetes pods need proper authentication to pull images. This is accomplished by:

1. Creating a `docker-registry` type secret with registry credentials
2. Configuring the deployment to use `imagePullSecrets` to authenticate with the private registry

## Secret Types Available

Kubernetes provides several secret types for different use cases:

```bash
k create secret
```

**Available Commands:**

- `docker-registry` - Create a secret for use with a Docker registry
- `generic` - Create a secret from a local file, directory, or literal value (Opaque secret type)
- `tls` - Create a secret that holds TLS certificate and its associated key

**Answer**: For Docker registry authentication, we must use the `docker-registry` secret type.

## Step-by-Step Implementation

### Step 1: Explore the Current Application

First, let's examine the existing deployment to understand what image it's currently using:

```bash
k describe deploy web
```

This command shows the current deployment configuration, including the container image being used.

### Step 2: Update Deployment to Use Private Registry Image

Update the deployment to use an image from the private registry:

```bash
k edit deploy web
```

**YAML Configuration:**

```yaml
spec:
  containers:
    - image: myprivateregistry.com:5000/nginx:alpine
      imagePullPolicy: IfNotPresent
      name: nginx
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
  dnsPolicy: ClusterFirst
```

### Step 3: Verify Pod Status

After updating the image, check if the new pods are running successfully:

```bash
k get pods
```

**Expected Result**: The pods will fail to start because they cannot authenticate with the private registry.

### Step 4: Create Docker Registry Secret

Create a secret with the credentials required to access the private registry:

**Registry Details:**

- Name: `private-reg-cred`
- Username: `dock_user`
- Password: `dock_password`
- Server: `myprivateregistry.com:5000`
- Email: `dock_user@myprivateregistry.com`

**Command:**

```bash
kubectl create secret docker-registry private-reg-cred \
  --docker-server=myprivateregistry.com:5000 \
  --docker-username=dock_user \
  --docker-password=dock_password \
  --docker-email=dock_user@myprivateregistry.com
```

### Step 5: Configure Deployment to Use the Secret

Update the deployment to use the credentials from the secret:

```bash
k edit deploy web
```

**Updated YAML Configuration:**

```yaml
spec:
  imagePullSecrets:
    - name: private-reg-cred
  containers:
    - image: myprivateregistry.com:5000/nginx:alpine
      imagePullPolicy: IfNotPresent
      name: nginx
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
  dnsPolicy: ClusterFirst
```

## Key Configuration Elements

### Docker Registry Secret Structure

The `docker-registry` secret type automatically creates a secret in the format expected by Kubernetes for Docker authentication:

```bash
kubectl create secret docker-registry [SECRET_NAME] \
  --docker-server=[REGISTRY_SERVER] \
  --docker-username=[USERNAME] \
  --docker-password=[PASSWORD] \
  --docker-email=[EMAIL]
```

### ImagePullSecrets Configuration

The `imagePullSecrets` field in the deployment spec tells Kubernetes which secret to use for authentication:

```yaml
spec:
  imagePullSecrets:
    - name: private-reg-cred # References the secret we created
```

## Why This Approach Solves the Problem

1. **Authentication**: The `docker-registry` secret stores the necessary credentials in a format that Kubernetes understands
2. **Security**: Credentials are stored securely in Kubernetes secrets rather than in plain text
3. **Automation**: Once configured, Kubernetes automatically uses these credentials when pulling images
4. **Scope**: The secret can be referenced by multiple deployments if needed

## Verification Commands

To verify the configuration is working:

```bash
# Check if the secret was created
kubectl get secret private-reg-cred

# Verify the deployment configuration
kubectl describe deploy web

# Check pod status
kubectl get pods

# View detailed pod information
kubectl describe pod [POD_NAME]
```

## Troubleshooting

If pods still fail to start after configuration:

1. Verify the secret exists: `kubectl get secrets`
2. Check the deployment configuration: `kubectl describe deploy web`
3. Examine pod events: `kubectl describe pod [POD_NAME]`
4. Verify registry credentials are correct
5. Ensure the registry URL is accessible from the cluster

## Common Issues

- **Flag Syntax Error**: Remember to use double dashes (`--`) for all docker flags, not single dashes (`-`)
- **Secret Name Mismatch**: Ensure the secret name in `imagePullSecrets` matches the actual secret name
- **Registry URL**: Verify the registry URL format and port number are correct
- **Network Access**: Ensure the Kubernetes cluster can reach the private registry

## Security Best Practices

1. Use strong passwords for registry authentication
2. Limit secret access using RBAC
3. Regularly rotate registry credentials
4. Monitor secret usage and access logs
5. Use service accounts with specific permissions when possible

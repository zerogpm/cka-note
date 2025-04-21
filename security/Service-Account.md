# Kubernetes Dashboard Setup

This guide walks through setting up and troubleshooting a Kubernetes dashboard application.

## Checking Service Accounts

To view existing Service Accounts in the default namespace:

```bash
kubectl get sa
```

Output:

```
NAME     SECRETS   AGE
default  0         5m16s
dev      0         35s
```

## Examining the Default Service Account

To inspect the default service account:

```bash
kubectl describe sa default
```

Output:

```
Name:                default
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>
```

## Inspecting Dashboard Deployment

View all deployments:

```bash
kubectl get deploy
```

Then inspect the dashboard deployment:

```bash
kubectl describe deploy web-dashboard
```

Key information:

```
web-dashboard:
  Image:        gcr.io/kodekloud/customimage/my-kubernetes-dashboard
  Port:         8080/TCP
  Host Port:    0/TCP
  Environment:  PYTHONUNBUFFERED: 1
```

## Dashboard Status

The current state of the dashboard is: **Failed**

The dashboard application uses a **Service Account** to query the Kubernetes API.

## Examining Dashboard Pod Details

Inspect a dashboard pod to see which service account it's using:

```bash
kubectl describe pod web-dashboard-5f88cdc488-7rprr
```

Service account credentials are mounted at:

```
Mounts:
  /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qhzc9 (ro)
```

## Creating a Service Account with Proper Permissions

The default ServiceAccount has limited access. Create a new one:

```bash
kubectl create sa dashboard-sa
```

## Creating and Using an Access Token

Generate a token for the new service account:

```bash
kubectl create token dashboard-sa
```

Copy the generated token and paste it into the token field of the dashboard UI, then click "Load Dashboard" button.

## Updating Deployment to Use New Service Account

For persistent configuration, update the deployment to use the new service account:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-dashboard
spec:
  # ... other configurations
  template:
    metadata:
      # ... other configurations
    spec:
      serviceAccountName: dashboard-sa # Update this line
      # ... other configurations
```

To apply this change, either:

1. Edit the deployment directly:

   ```bash
   kubectl edit deploy web-dashboard
   ```

   Find and change `serviceAccountName: default` to `serviceAccountName: dashboard-sa`

2. Or apply a patch:
   ```bash
   kubectl patch deployment web-dashboard -p '{"spec":{"template":{"spec":{"serviceAccountName":"dashboard-sa"}}}}'
   ```

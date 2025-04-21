# Working with Private Docker Registries in Kubernetes

This guide demonstrates how to configure Kubernetes to work with private Docker registries.

## Secret Types in Kubernetes

When creating secrets in Kubernetes, we can choose from several types:

```bash
k create secret  # Create a secret with specified type
```

Available secret types:

- `docker-registry`: For accessing a container registry
- `generic`: Creates an Opaque secret type
- `tls`: Holds TLS certificate and its associated key

Available Commands:

```bash
docker-registry  # Create a secret for use with a Docker registry
generic          # Create a secret from a local file, directory, or literal value
tls              # Create a TLS secret
```

Command syntax for docker-registry secret:

```bash
k create secret docker-registry
```

## Exploring the Current Deployment

First, let's examine what image our application is currently using:

```bash
k describe deploy web
```

## Updating the Deployment Image

We want to use a modified version of the application from an internal private registry located at `myprivateregistry.com:5000`:

```bash
k edit deploy web
```

Update the container image in the deployment:

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

## Checking Pod Status

After updating the image, we check if the new pods are running successfully:

```bash
# Command to check pod status (implicit)
# Result: no
```

The pods are not running successfully because we need to configure credentials for the private registry.

## Creating Registry Credentials Secret

Create a secret with the credentials required to access the private registry:

```bash
kubectl create secret docker-registry private-reg-cred \
  --docker-server=myprivateregistry.com:5000 \
  --docker-username=dock_user \
  --docker-password=dock_password \
  --docker-email=dock_user@myprivateregistry.com
```

Secret details:

- Name: `private-reg-cred`
- Username: `dock_user`
- Password: `dock_password`
- Server: `myprivateregistry.com:5000`
- Email: `dock_user@myprivateregistry.com`

## Configuring the Deployment to Use the Secret

Update the deployment to use the credentials from the new secret:

```bash
k edit deploy web
```

Add the `imagePullSecrets` field to the spec:

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
```

After this configuration, the deployment should be able to successfully pull images from the private registry.

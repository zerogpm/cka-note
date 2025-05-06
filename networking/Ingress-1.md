# Kubernetes Ingress Controller Setup and Configuration

This README documents a step-by-step guide to exploring and configuring a Kubernetes Ingress Controller and resources in a cluster.

## Environment Overview

We have a Kubernetes cluster with the following components deployed:

- Ingress Controller
- Various resources and applications in different namespaces

## Exploring the Setup

### Check All Resources in All Namespaces

```bash
kubectl get all -A
```

This command lists all resources across all namespaces, giving us a complete overview of the cluster. It shows pods, services, deployments, and other resources along with their namespaces, status, and age.

### Question: Which namespace is the Ingress Controller deployed in?

To find the namespace of the Ingress Controller, we examine the output from the previous command or specifically look for pods:

```bash
kubectl get pods -A
```

**Answer: ingress-nginx**

We can identify this by looking at the pods list and seeing the ingress-nginx-controller pod running in the ingress-nginx namespace.

### Question: What is the name of the Ingress Controller Deployment?

To specifically check deployments in the ingress-nginx namespace:

```bash
kubectl get deploy -n ingress-nginx
```

**Answer: ingress-nginx-controller**

This command reveals the deployment name in the ingress-nginx namespace.

### Question: Which namespace are the applications deployed in?

Looking at the output from our initial `kubectl get all -A` command.

**Answer: app-space**

We can see multiple application pods running in the app-space namespace.

### Question: How many applications are deployed in the app-space namespace?

To count the number of deployments:

```bash
kubectl get deploy -n app-space
```

Alternatively, a more precise count:

```bash
kubectl get deploy -n app-space --no-headers | wc -l
```

**Answer: 3**

These commands show three deployments in the app-space namespace: default-backend, webapp-video, and webapp-wear.

### Question: Which namespace is the Ingress Resource deployed in?

```bash
kubectl get ingress -A
```

**Answer: app-space**

This command lists all ingress resources across namespaces, showing that the ingress is in the app-space namespace.

### Question: What is the name of the Ingress Resource?

From the previous command's output:

**Answer: ingress-wear-watch**

### Question: What is the Host configured on the Ingress Resource?

```bash
kubectl describe ingress ingress-wear-watch -n app-space
```

**Answer: \***

The wildcard (\*) host means this ingress will respond to any hostname.

### Question: What backend is the /wear path on the Ingress configured with?

From the describe command output:

**Answer: wear-service:8080**

This shows the /wear path routes traffic to the wear-service on port 8080.

### Question: At what path is the video streaming application made available on the Ingress?

From the describe command output:

**Answer: /watch**

The video streaming application is accessible at the /watch path.

### Question: If the requirement does not match any of the configured paths in the Ingress, to which service are the requests forwarded?

**Answer: default-backend-service**

This is the default service that handles any requests that don't match configured paths.

## Modifying the Ingress

### Task: Make the video application available at /stream

We need to change the path for the video service from /watch to /stream in the ingress configuration:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: ingress-wear-watch
  namespace: app-space
spec:
  rules:
    - http:
        paths:
          - backend:
              service:
                name: wear-service
                port:
                  number: 8080
            path: /wear
            pathType: Prefix
          - backend:
              service:
                name: video-service
                port:
                  number: 8080
            path: /stream
            pathType: Prefix
```

This YAML configuration maintains the /wear path and changes the /watch path to /stream, still routing to the video-service on port 8080.

### A user is trying to view the /eat URL on the Ingress Service. Which page would he see?

**Answer: 404 page**

Since there's no path configured for /eat, the request will be handled by the default backend, which returns a 404 page.

### Task: Add a new path to make the food delivery application available at /eat

We would need to add a new path to the ingress:

```yaml
- backend:
    service:
      name: food-service
      port:
        number: 8080
  path: /eat
  pathType: Prefix
```

This configuration adds a new path /eat that routes to the food-service.

## Handling Applications in Different Namespaces

### Question: Identify the namespace in which the new payment application is deployed

```bash
kubectl get all -A
```

**Answer: critical-space**

### Question: What is the name of the deployment of the new application?

```bash
kubectl get deploy -n critical-space
```

**Answer: webapp-pay**

### Task: Make the new application available at /pay

For applications in different namespaces, we need to create a separate Ingress in that namespace:

```bash
kubectl get svc -n critical-space
```

This shows us the service name (pay-service) and port (8282) for the payment application.

To create the ingress:

```bash
kubectl create ingress ingress-pay -n critical-space --rule="/pay=pay-service:8282"
```

However, this doesn't work correctly without the proper rewrite annotation. We need to create a proper YAML file:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-pay
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /pay
            pathType: Prefix
            backend:
              service:
                name: pay-service
                port:
                  number: 8282
```

Then apply it:

```bash
kubectl apply -f ingress-pay.yaml
```

The rewrite-target annotation is crucial here because it modifies the path from /pay to / before forwarding the request to the backend service. This ensures that the application, which expects requests at the root path (/), can properly handle them.

Without this annotation:

- User request: http://ingress-ip/pay
- Backend receives: /pay (which it doesn't recognize)

With the annotation:

- User request: http://ingress-ip/pay
- Ingress rewrites to: /
- Backend receives: / (which it can handle)

This ensures the application doesn't need to be aware of the specific path it's being accessed through in the ingress controller.

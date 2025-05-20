# Kubernetes Imperative Commands README

This repository demonstrates how to deploy various Kubernetes resources using imperative commands. Each task is explained with the command used, the output, and a detailed explanation.

## Table of Contents

- [Task 1: Deploy nginx-pod](#task-1-deploy-nginx-pod)
- [Task 2: Deploy redis pod with labels](#task-2-deploy-redis-pod-with-labels)
- [Task 3: Create redis-service](#task-3-create-redis-service)
- [Task 4: Create webapp deployment](#task-4-create-webapp-deployment)
- [Task 5: Create custom-nginx pod with container port](#task-5-create-custom-nginx-pod-with-container-port)
- [Task 6: Create dev-ns namespace](#task-6-create-dev-ns-namespace)
- [Task 7: Create redis-deploy in dev-ns namespace](#task-7-create-redis-deploy-in-dev-ns-namespace)
- [Task 8: Create httpd pod and service](#task-8-create-httpd-pod-and-service)

## Original Problem Statement

```
Deploy a pod named nginx-pod using the nginx:alpine image. Use imperative commands only.
k run nginx-pod --image=nginx:alpine

Deploy a redis pod using the redis:alpine image with the labels set to tier=db.
Either use imperative commands to create the pod with the labels. Or else use imperative commands to generate the pod definition file, then add the labels before creating the pod using the file.
k run redis --image=redis:alpine --labels="tier=db"

Create a service redis-service to expose the redis application within the cluster on port 6379.
Use imperative commands.
k expose pod redis --port=6379 --name redis-service

Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas.
Try to use imperative commands only. Do not create definition files.
Name: webapp
Image: kodekloud/webapp-color
Replicas: 3
k create deploy webapp --image=kodekloud/webapp-color --replicas=3

Create a new pod called custom-nginx using the nginx image and run it on container port 8080.
k run custom-nginx --image=nginx --port=8080

Create a new namespace called dev-ns.
Use imperative commands.
controlplane ~ ➜ k create ns dev-ns
namespace/dev-ns created

Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas.
Use imperative commands.
controlplane ~ ✖ k create deploy redis-deploy --image=redis --replicas=2 -n dev-ns
deployment.apps/redis-deploy created

Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80.
Try to do this with as few steps as possible.
controlplane ~ ➜ kubectl run httpd --image=httpd:alpine --port=80 --expose
service/httpd created
pod/httpd created
```

## Task 1: Deploy nginx-pod

### Command

```bash
kubectl run nginx-pod --image=nginx:alpine
```

### Output

```
pod/nginx-pod created
```

### Explanation

This command creates a pod with the following characteristics:

- Pod name: `nginx-pod`
- Image: `nginx:alpine` (lightweight version of the Nginx web server)

### YAML Equivalent

The command generates a pod configuration similar to this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-pod
      image: nginx:alpine
```

## Task 2: Deploy redis pod with labels

### Command

```bash
kubectl run redis --image=redis:alpine --labels="tier=db"
```

### Output

```
pod/redis created
```

### Explanation

This command creates a pod with the following characteristics:

- Pod name: `redis`
- Image: `redis:alpine` (lightweight version of Redis database)
- Label: `tier=db` (assigns a label to categorize this pod as part of the database tier)

### YAML Equivalent

The command generates a pod configuration similar to this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
  labels:
    tier: db
spec:
  containers:
    - name: redis
      image: redis:alpine
```

## Task 3: Create redis-service

### Command

```bash
kubectl expose pod redis --port=6379 --name redis-service
```

### Output

```
service/redis-service created
```

### Explanation

This command creates a service that exposes the Redis pod:

- Service name: `redis-service`
- Target pod: `redis` (the pod created in the previous step)
- Port: `6379` (the standard Redis port)
- Service type: `ClusterIP` (default service type when not specified)

### YAML Equivalent

The command generates a service configuration similar to this:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    tier: db # This matches the label we set on the redis pod
  ports:
    - port: 6379
      targetPort: 6379
  type: ClusterIP
```

## Task 4: Create webapp deployment

### Command

```bash
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
```

### Output

```
deployment.apps/webapp created
```

### Explanation

This command creates a deployment with the following characteristics:

- Deployment name: `webapp`
- Image: `kodekloud/webapp-color` (a web application with color feature)
- Replicas: `3` (creates 3 identical pods for high availability)

### YAML Equivalent

The command generates a deployment configuration similar to this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp
          image: kodekloud/webapp-color
```

## Task 5: Create custom-nginx pod with container port

### Command

```bash
kubectl run custom-nginx --image=nginx --port=8080
```

### Output

```
pod/custom-nginx created
```

### Explanation

This command creates a pod with the following characteristics:

- Pod name: `custom-nginx`
- Image: `nginx` (latest Nginx web server)
- Container port: `8080` (sets the container port for the application)

### YAML Equivalent

The command generates a pod configuration similar to this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-nginx
spec:
  containers:
    - name: custom-nginx
      image: nginx
      ports:
        - containerPort: 8080
```

## Task 6: Create dev-ns namespace

### Command

```bash
kubectl create namespace dev-ns
```

### Output

```
namespace/dev-ns created
```

### Explanation

This command creates a new namespace:

- Namespace name: `dev-ns` (typically used for development environments)

### YAML Equivalent

The command generates a namespace configuration similar to this:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-ns
```

## Task 7: Create redis-deploy in dev-ns namespace

### Command

```bash
kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns
```

### Output

```
deployment.apps/redis-deploy created
```

### Explanation

This command creates a deployment in a specified namespace:

- Deployment name: `redis-deploy`
- Image: `redis` (latest Redis database)
- Replicas: `2` (creates 2 identical pods for redundancy)
- Namespace: `dev-ns` (the namespace created in the previous step)

### YAML Equivalent

The command generates a deployment configuration similar to this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deploy
  namespace: dev-ns
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis-deploy
  template:
    metadata:
      labels:
        app: redis-deploy
    spec:
      containers:
        - name: redis-deploy
          image: redis
```

## Task 8: Create httpd pod and service

### Command

```bash
kubectl run httpd --image=httpd:alpine --port=80 --expose
```

### Output

```
service/httpd created
pod/httpd created
```

### Explanation

This command creates both a pod and a service in a single step:

- Pod name: `httpd`
- Image: `httpd:alpine` (lightweight Apache HTTP Server)
- Container port: `80` (standard HTTP port)
- Service: Creates a ClusterIP service with the same name as the pod

### YAML Equivalent

The command generates configurations similar to these:

Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd
  labels:
    run: httpd
spec:
  containers:
    - name: httpd
      image: httpd:alpine
      ports:
        - containerPort: 80
```

Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd
spec:
  selector:
    run: httpd
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

## Why This Approach Works

Using imperative commands in Kubernetes offers several advantages:

1. **Speed and Efficiency**: Imperative commands allow you to quickly create resources without writing YAML files
2. **Reduced Error Risk**: The commands handle the proper syntax and structure automatically
3. **Learning Path**: They provide a simpler way to understand Kubernetes operations before moving to declarative approaches
4. **Debugging and Testing**: Perfect for quick testing scenarios and troubleshooting
5. **Automation Friendly**: These commands can be easily included in scripts or CI/CD pipelines

The approach demonstrated here shows how to create various Kubernetes objects (pods, services, deployments, namespaces) using only imperative commands. This method is particularly useful for:

- Quick deployments in development environments
- Learning Kubernetes fundamentals
- Creating resources when you don't need complex configurations
- Creating test environments rapidly

Note: While imperative commands are convenient for quick operations, declarative YAML files are generally preferred for production environments as they provide better versioning, documentation, and reproducibility.

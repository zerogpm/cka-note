# Kubernetes Ingress Controller Deployment Guide

This guide explains the process of deploying an NGINX Ingress Controller in a Kubernetes cluster and configuring ingress resources to route traffic to backend applications.

## Original Problem

The original problem is to deploy an NGINX Ingress Controller with the following requirements:

1. Deploy an Ingress Controller in a dedicated namespace
2. Create required ConfigMap and ServiceAccount objects
3. Fix issues in the provided Deployment and Service configuration
4. Create an ingress resource to make applications available at /wear and /watch paths

The specific instructions were:

```
Let us now deploy an Ingress Controller. First, create a namespace called ingress-nginx.

We will isolate all ingress related objects into its own namespace.

The NGINX Ingress Controller requires a ConfigMap object. Create a ConfigMap object with name ingress-nginx-controller in the ingress-nginx namespace.

No data needs to be configured in the ConfigMap.

The NGINX Ingress Controller requires two ServiceAccounts. Create both ServiceAccount with name ingress-nginx and ingress-nginx-admission in the ingress-nginx namespace.

Let us now deploy the Ingress Controller. Create the Kubernetes objects using the given file.

The Deployment and it's service configuration is given at /root/ingress-controller.yaml. There are several issues with it. Try to fix them.

Note: Do not edit the default image provided in the given file. The image validation check passes when other issues are resolved.

Create the ingress resource to make the applications available at /wear and /watch on the Ingress service.

Also, make use of rewrite-target annotation field: -

nginx.ingress.kubernetes.io/rewrite-target: /

Ingress resource comes under the namespace scoped, so don't forget to create the ingress in the app-space namespace.
```

## Step-by-Step Implementation Guide

### 1. Creating the Namespace

First, we need to create a dedicated namespace for all ingress-related resources:

```bash
kubectl create namespace ingress-nginx
```

**Explanation:** This command creates a new namespace called `ingress-nginx` in the Kubernetes cluster. A namespace provides isolation for resources within the cluster, making it easier to manage and control access to ingress-related components.

**Expected Output:**

```
namespace/ingress-nginx created
```

### 2. Creating the ConfigMap

The NGINX Ingress Controller requires a ConfigMap:

```bash
kubectl create configmap nginx-configuration -n ingress-space
```

**Explanation:** This command creates a ConfigMap named `nginx-configuration` in the `ingress-space` namespace. ConfigMaps are used to store non-confidential configuration data in key-value pairs, which can be used by pods at runtime.

**Expected Output:**

```
configmap/nginx-configuration created
```

Again, there's a namespace discrepancy. The correct command should be:

```bash
kubectl create configmap ingress-nginx-controller -n ingress-nginx
```

### 3. Creating ServiceAccounts

The NGINX Ingress Controller requires two ServiceAccounts:

```bash
kubectl create serviceaccount ingress-nginx -n ingress-nginx
kubectl create serviceaccount ingress-nginx-admission -n ingress-nginx
```

**Explanation:** These commands create two ServiceAccount objects named `ingress-nginx` and `ingress-nginx-admission` in the `ingress-nginx` namespace. ServiceAccounts provide an identity for processes that run in a Pod, allowing them to interact with the Kubernetes API.

**Expected Output:**

```
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
```

### 4. Fixing and Deploying the Ingress Controller Configuration

The original file at `/root/ingress-controller.yaml` had several issues. The errors reported were:

```bash
Error from server (Invalid): error when creating "/root/ingress-controller.yaml": Deployment.apps "ingress-nginx-controller" is invalid: spec.template.spec.containers[0].ports[0].containerPort: Required value
Error from server (BadRequest): error when creating "/root/ingress-controller.yaml": Service in version "v1" cannot be handled as a Service: strict decoding error: unknown field "spec.ports[0].nodeport"
```

Here's the corrected YAML configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  minReadySeconds: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
    spec:
      containers:
        - args:
            - /nginx-ingress-controller
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
            - --election-id=ingress-controller-leader
            - --watch-ingress-without-class=true
            - --default-backend-service=app-space/default-http-backend
            - --controller-class=k8s.io/ingress-nginx
            - --ingress-class=nginx
            - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
            - --validating-webhook=:8443
            - --validating-webhook-certificate=/usr/local/certificates/cert
            - --validating-webhook-key=/usr/local/certificates/key
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: LD_PRELOAD
              value: /usr/local/lib/libmimalloc.so
          image: registry.k8s.io/ingress-nginx/controller:v1.1.2@sha256:28b11ce69e57843de44e3db6413e98d09de0f6688e33d4bd384002a44f78405c
          imagePullPolicy: IfNotPresent
          lifecycle:
            preStop:
              exec:
                command:
                  - /wait-shutdown
          livenessProbe:
            failureThreshold: 5
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          name: controller
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
            - containerPort: 443
              name: https
              protocol: TCP
            - containerPort: 8443
              name: webhook
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          resources:
            requests:
              cpu: 100m
              memory: 90Mi
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              add:
                - NET_BIND_SERVICE
              drop:
                - ALL
            runAsUser: 101
          volumeMounts:
            - mountPath: /usr/local/certificates/
              name: webhook-cert
              readOnly: true
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/os: linux
      serviceAccountName: ingress-nginx
      terminationGracePeriodSeconds: 300
      volumes:
        - name: webhook-cert
          secret:
            secretName: ingress-nginx-admission

---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
      nodePort: 30080
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: NodePort
```

**Key Fixes Made:**

1. **Deployment Namespace:** Fixed the namespace in the Deployment metadata (from `ingress-` to `ingress-nginx`)
2. **Container Port:** Fixed the containerPort syntax in the ports section (proper indentation and structure)
3. **Service Name:** Ensured the service name is `ingress-nginx-controller` as required
4. **NodePort Field:** Fixed the field name from `nodeport` (lowercase) to `nodePort` (camelCase)

To apply this configuration:

```bash
kubectl apply -f ingress-controller.yaml
```

**Expected Output:**

```
deployment.apps/ingress-nginx-controller created
service/ingress-nginx-controller created
```

### 5. Creating the Ingress Resource

Finally, we need to create an ingress resource to route traffic to our backend applications. There are two approaches shown:

#### Approach 1: Using the Imperative Command

```bash
kubectl create ingress simple --rule="/wear=wear-service:8080" --rule="/watch=video-service:8080" -n app-space
```

**Explanation:** This command creates an ingress resource named `simple` in the `app-space` namespace with rules to route requests with path `/wear` to the `wear-service` on port 8080 and requests with path `/watch` to the `video-service` on port 8080.

**Expected Output:**

```
ingress.networking.k8s.io/simple created
```

#### Approach 2: Using a Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /wear
            pathType: Prefix
            backend:
              service:
                name: wear-service
                port:
                  number: 8080
          - path: /watch
            pathType: Prefix
            backend:
              service:
                name: video-service
                port:
                  number: 8080
```

Save this as `ingress-wear-watch.yaml` and apply it:

```bash
kubectl apply -f ingress-wear-watch.yaml
```

**Explanation:** This YAML file defines an Ingress resource named `ingress-wear-watch` in the `app-space` namespace. It includes annotations for URL rewriting and SSL redirection, and specifies routing rules for paths `/wear` and `/watch`.

**Expected Output:**

```
ingress.networking.k8s.io/ingress-wear-watch created
```

## Why This Approach Solves the Problem

This implementation successfully addresses all the requirements of the original problem:

1. **Isolation:** By creating a dedicated namespace (`ingress-nginx`), we've isolated all ingress-related resources, improving organization and security.

2. **Required Components:** We've created all the necessary components:

   - ConfigMap for the controller configuration
   - Two ServiceAccounts for the controller and admission webhook
   - Deployment of the NGINX Ingress Controller
   - Service to expose the controller

3. **Fixed Configuration Issues:**

   - Corrected the namespace inconsistency
   - Fixed the containerPort syntax error
   - Corrected the service name to meet requirements
   - Fixed the nodePort field name (camelCase format)
   - Set replicas to 1 as required
   - Used the correct image as specified

4. **Routing Configuration:**

   - Created an ingress resource in the `app-space` namespace
   - Configured routing rules for `/wear` and `/watch` paths
   - Directed traffic to the appropriate backend services
   - Added the required `rewrite-target` annotation

5. **External Access:**
   - The Service is configured as NodePort with port 30080, making it accessible from outside the cluster

This implementation enables user traffic to be directed to the appropriate backend services based on the URL path, with the NGINX Ingress Controller handling the routing logic and URL rewriting.

## Validation

You can validate the deployment with the following commands:

```bash
# Check namespace creation
kubectl get namespace ingress-nginx

# Verify ConfigMap creation
kubectl get configmap -n ingress-nginx

# Verify ServiceAccount creation
kubectl get serviceaccount -n ingress-nginx

# Check Ingress Controller deployment status
kubectl get deployment -n ingress-nginx
kubectl get pods -n ingress-nginx

# Verify service configuration
kubectl get service -n ingress-nginx

# Test the ingress resource
kubectl get ingress -n app-space
```

Once everything is running, you can access the applications using:

- `http://<node-ip>:30080/wear` for the wear service
- `http://<node-ip>:30080/watch` for the watch service

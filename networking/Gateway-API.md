# Kubernetes Gateway API Implementation

## Problem Statement

The task required implementing a Kubernetes Gateway API configuration with specific requirements:

1. Create a Gateway resource with:

   - Name: nginx-gateway
   - Namespace: nginx-gateway
   - Gateway Class Name: nginx
   - Listeners: HTTP protocol on port 80 with name "http"
   - Allowed Routes: All namespaces

2. Expose a service called "frontend-svc" in the default namespace on the root path (/) using an HTTPRoute named "frontend-route"

## Implementation Details

### Gateway Resource

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
    - name: http
      port: 80
      protocol: HTTP
      allowedRoutes:
        namespaces:
          from: All
```

### HTTPRoute Resource

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: frontend-route
  namespace: default
spec:
  parentRefs:
    - name: nginx-gateway
      namespace: nginx-gateway
      sectionName: http
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: frontend-svc
          port: 80
```

## Commands and Their Outputs

### Creating the Gateway Resource

```bash
kubectl apply -f gateway.yaml
```

### Verifying the Gateway Status

```bash
kubectl get gateways -n nginx-gateway
```

Output:

```
NAME            CLASS   ADDRESS   PROGRAMMED   AGE
nginx-gateway   nginx             True         9s
```

### Checking Service and Pod Status

```bash
kubectl get pod,svc
```

Output:

```
NAME               READY   STATUS    RESTARTS   AGE
pod/frontend-app   1/1     Running   0          40s

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/frontend-svc   ClusterIP   172.20.222.197   <none>        80/TCP    40s
service/kubernetes     ClusterIP   172.20.0.1       <none>        443/TCP   35m
```

## Step-by-Step Explanation

1. **Gateway Resource Creation**:

   - We defined a Gateway resource in the nginx-gateway namespace
   - The Gateway uses the "nginx" GatewayClass, which determines the implementation controller
   - We configured a single HTTP listener on port 80 named "http"
   - The allowedRoutes field is set to accept routes from all namespaces

2. **Gateway Verification**:

   - After applying the Gateway resource, we checked its status
   - The PROGRAMMED field shows "True", indicating the Gateway was successfully configured
   - No ADDRESS is shown yet as the Gateway is still initializing

3. **Service and Pod Check**:

   - We confirmed that the frontend-app pod is running in the default namespace
   - The frontend-svc service is available on port 80 with a ClusterIP of 172.20.222.197

4. **HTTPRoute Creation**:
   - We defined an HTTPRoute in the default namespace to connect to our Gateway
   - The parentRefs field links the HTTPRoute to our Gateway in the nginx-gateway namespace
   - The sectionName field specifies which listener to use ("http")
   - We configured a path match for "/" (PathPrefix) to route all traffic to frontend-svc
   - The backendRefs field directs traffic to the frontend-svc service on port 80

## Why This Approach Works

This implementation properly leverages the Kubernetes Gateway API, which offers several advantages over the older Ingress API:

1. **Separation of Concerns**:

   - The Gateway resource defines infrastructure concerns (listening ports, protocols)
   - The HTTPRoute resource defines application routing concerns
   - This separation allows infrastructure teams to manage Gateways while application teams manage their Routes

2. **Cross-Namespace Routing**:

   - The Gateway's allowedRoutes field explicitly permits routes from all namespaces
   - Our HTTPRoute in the default namespace can reference the Gateway in nginx-gateway namespace

3. **Modern API Design**:

   - The Gateway API is more expressive and extensible than Ingress
   - It supports multiple protocols (not just HTTP)
   - It provides clearer status information (the PROGRAMMED field)

4. **Advanced Routing Capabilities**:
   - The PathPrefix type in our HTTPRoute allows for hierarchical routing
   - If needed, we could add headers, query parameters, and method-based matching

This approach successfully meets all requirements by creating a Gateway that accepts HTTP traffic on port 80 and routes requests for the root path to the frontend-svc service, making the application accessible to external traffic.

## Key Gateway API Concepts Demonstrated

1. **GatewayClass**: Defines the implementation of the Gateway (nginx in this case)
2. **Gateway**: Defines the listener configuration (port 80, HTTP protocol)
3. **HTTPRoute**: Defines how traffic is routed to backends (frontend-svc)
4. **Cross-namespace references**: Connecting resources across different namespaces
5. **PathPrefix matching**: Directing traffic based on URL paths

# Kubernetes Service Configuration

## Original Problem

The original service configuration contained an error in the selector field:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  selector: simple-webapp # ERROR: This is incorrectly formatted as a string
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080
```

The error was that the selector was specified as a string (`simple-webapp`) rather than as a key-value pair. In Kubernetes, selectors must be specified as key-value pairs to match the labels on pods.

## Environment Analysis

Before solving the problem, we analyzed the current Kubernetes environment:

### Checking Services in Default Namespace

```bash
controlplane ~ ➜ k get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP    16m
```

- Only one service exists: the default `kubernetes` service
- Type: ClusterIP
- No custom services

### Examining the Default Kubernetes Service

```bash
controlplane ~ ➜ k describe svc kubernetes
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.0.1
IPs:               10.43.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         192.168.75.146:6443
Session Affinity:  None
Internal Traffic Policy: Cluster
Events:            <none>
```

- The `kubernetes` service has **2 labels**: `component=apiserver` and `provider=kubernetes`
- TargetPort: 6443
- Endpoints: 1 endpoint (192.168.75.146:6443)

### Checking Deployments

```bash
controlplane ~ ➜ k get deploy
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
simple-webapp-deployment  4/4     4            4           17s
```

- There is **1 deployment**: `simple-webapp-deployment`
- It has 4 pods, all ready and available

### Examining the Deployment

```bash
controlplane ~ ✖ k describe deploy simple-webapp-deployment
Name:                   simple-webapp-deployment
Namespace:              default
CreationTimestamp:      Tue, 20 May 2025 23:34:33 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=simple-webapp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=simple-webapp
  Containers:
   simple-webapp:
    Image:        kodekloud/simple-webapp:red
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
  Node-Selectors: <none>
  Tolerations:    <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   simple-webapp-deployment-8555484b96 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  73s   deployment-controller  Scaled up replica set simple-webapp-deployment-8555484b96 from 0 to 4
```

Key information:

- Image: `kodekloud/simple-webapp:red`
- The pods have the label `name=simple-webapp`
- The port exposed is 8080/TCP

## Solution

The solution was to create a service with the correct selector format that matches the pod labels. The correct YAML configuration is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  selector:
    name: simple-webapp # FIXED: This is now correctly formatted as a key-value pair
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080
```

### Why This Fixes the Problem

1. The selector is now correctly formatted as a key-value pair (`name: simple-webapp`) instead of a simple string
2. This selector exactly matches the label used in the deployment's pod template (`name=simple-webapp`)
3. The service will now properly select the 4 pods created by the deployment
4. The specified ports (8080) match the container port exposed by the application (8080)
5. The NodePort (30080) is properly specified for external access

## Implementation Steps

1. Create the service definition file:

```bash
# Create the service definition file
cat > service-definition-1.yaml << EOF
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  selector:
    name: simple-webapp
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080
EOF
```

2. Apply the service definition:

```bash
kubectl apply -f service-definition-1.yaml
```

3. Verify the service was created correctly:

```bash
kubectl get svc webapp-service
```

Expected output:

```
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
webapp-service    NodePort   10.43.x.x       <none>        8080:30080/TCP   xx
```

4. Test the service:

```bash
# Get the Node IP
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')

# Test the service
curl $NODE_IP:30080
```

## Key Learning Points

1. In Kubernetes, selectors must be specified as key-value pairs, not as strings
2. Service selectors must match pod labels
3. The labels are defined in the pod template section of a deployment
4. For a NodePort service, three ports are relevant:
   - `port`: The port exposed internally on the ClusterIP
   - `targetPort`: The port on the pod where the application is running
   - `nodePort`: The port exposed externally on the node (must be between 30000-32767)

## Additional Commands for Debugging

If you need to troubleshoot service connectivity issues:

```bash
# Check if endpoints are created
kubectl get endpoints webapp-service

# Verify pod labels match the service selector
kubectl get pods --show-labels

# Check if pods are running and ready
kubectl get pods -l name=simple-webapp

# Debug service connection
kubectl describe svc webapp-service
```

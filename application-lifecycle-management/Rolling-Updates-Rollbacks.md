# Kubernetes Deployment Strategies Tutorial

This tutorial demonstrates different Kubernetes deployment strategies, focusing on **Rolling Updates** and **Recreate** strategies for application upgrades with zero downtime.

## Problem Statement

The original exercise involves:

- Testing a web application with multiple requests
- Understanding deployment strategies (Rolling Update vs Recreate)
- Upgrading applications using different strategies
- Observing the behavior during upgrades

## Prerequisites

- Kubernetes cluster with kubectl configured
- Access to the `kube-public` namespace
- `curl` pod available for testing

## Initial Setup

### 1. Test Script Setup

First, we execute a test script to send multiple requests to our web application:

```bash
# Execute the script at /root/curl-test.sh
for i in {1..35}; do
   kubectl exec --namespace=kube-public curl -- sh -c 'test=`wget -qO- -T 2  http://webapp-service.default.svc.cluster.local:8080/info 2>&1` && echo "$test OK" || echo "Failed"';
   echo ""
done
```

### 2. Inspect Current Deployment

Check the current deployment status:

```bash
kubectl get deployments.apps
```

**Output:**

```
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   4/4     4            4           2m10s
```

### 3. Detailed Deployment Information

Examine the deployment configuration:

```bash
kubectl describe deploy frontend
```

**Output:**

```
Name:                   frontend
Namespace:              default
CreationTimestamp:      Sun, 25 May 2025 18:35:58 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        20
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=webapp
  Containers:
   simple-webapp:
    Image:         kodekloud/webapp-color:v1
    Port:          8080/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   frontend-6765b99794 (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m11s  deployment-controller  Scaled up replica set frontend-6765b99794 from 0 to 4
```

## Key Configuration Details

### Initial Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2025-05-25T18:35:58Z"
  generation: 1
  name: frontend
  namespace: default
  resourceVersion: "907"
  uid: 0389c86a-f6a6-45cd-9972-354cd29996f1
spec:
  minReadySeconds: 20
  progressDeadlineSeconds: 600
  replicas: 4
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: webapp
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: webapp
    spec:
      containers:
        - image: kodekloud/webapp-color:v1
          imagePullPolicy: IfNotPresent
          name: simple-webapp
          ports:
            - containerPort: 8080
              protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 4
  conditions:
    - lastTransitionTime: "2025-05-25T18:36:23Z"
      lastUpdateTime: "2025-05-25T18:36:23Z"
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: "True"
      type: Available
    - lastTransitionTime: "2025-05-25T18:35:58Z"
      lastUpdateTime: "2025-05-25T18:36:23Z"
      message: ReplicaSet "frontend-6765b99794" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: "True"
      type: Progressing
  observedGeneration: 1
  readyReplicas: 4
  replicas: 4
  updatedReplicas: 4
```

## Part 1: Rolling Update Strategy

### Step 1: Identify Current Strategy

The deployment uses **RollingUpdate** strategy with:

- **maxSurge: 25%** - Can create 1 additional pod (25% of 4 = 1)
- **maxUnavailable: 25%** - Can terminate 1 pod at a time (25% of 4 = 1)

### Step 2: Upgrade to v2 with Rolling Update

Update the image version:

```bash
kubectl edit deployment frontend
```

Change the image from `kodekloud/webapp-color:v1` to `kodekloud/webapp-color:v2`

### Step 3: Test During Rolling Update

Run the test script again during the upgrade:

```bash
# Execute the script at /root/curl-test.sh
```

**Output (showing both versions running simultaneously):**

```
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v1 ; Color: blue OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v1 ; Color: blue OK
Hello, Application Version: v1 ; Color: blue OK
Hello, Application Version: v1 ; Color: blue OK
Hello, Application Version: v1 ; Color: blue OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v1 ; Color: blue OK
Hello, Application Version: v1 ; Color: blue OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v1 ; Color: blue OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v1 ; Color: blue OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v1 ; Color: blue OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
Hello, Application Version: v2 ; Color: green OK
```

### Analysis of Rolling Update

- **Maximum pods down**: 1 pod (25% of 4 replicas)
- **Zero downtime**: No requests failed
- **Gradual transition**: Both v1 and v2 served requests simultaneously
- **Blue-green traffic**: v1 (blue) and v2 (green) responses mixed

## Part 2: Recreate Strategy

### Step 1: Change Strategy to Recreate

Update the deployment strategy:

```bash
kubectl edit deployment frontend
```

**Updated YAML configuration:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      name: webapp
  strategy:
    type: Recreate # Changed from RollingUpdate
  template:
    metadata:
      labels:
        name: webapp
    spec:
      containers:
        - image: kodekloud/webapp-color:v2
          name: simple-webapp
          ports:
            - containerPort: 8080
              protocol: TCP
```

### Step 2: Upgrade to v3 with Recreate Strategy

Update the image version:

```bash
kubectl edit deployment frontend
```

Change the image from `kodekloud/webapp-color:v2` to `kodekloud/webapp-color:v3`

## Commands Summary

| Command                            | Purpose                             |
| ---------------------------------- | ----------------------------------- |
| `kubectl get deployments.apps`     | List all deployments                |
| `kubectl describe deploy frontend` | Get detailed deployment information |
| `kubectl edit deployment frontend` | Edit deployment configuration       |
| `./curl-test.sh`                   | Test application availability       |

## Key Differences Between Strategies

### Rolling Update Strategy

- **Pros:**

  - Zero downtime deployments
  - Gradual rollout reduces risk
  - Can handle traffic during updates
  - Configurable surge and unavailability limits

- **Cons:**
  - More complex
  - Temporary mixed version traffic
  - Requires more resources during update

### Recreate Strategy

- **Pros:**

  - Simple and straightforward
  - No mixed version traffic
  - Lower resource usage during update

- **Cons:**
  - **Downtime during deployment**
  - All pods terminated simultaneously
  - Higher risk for production environments

## Why This Approach Solves the Problem

This tutorial demonstrates:

1. **Zero-downtime deployments** using Rolling Update strategy
2. **Resource optimization** through controlled pod replacement
3. **Risk mitigation** by gradually introducing new versions
4. **Testing strategies** to validate deployment behavior
5. **Strategy comparison** to understand trade-offs

The Rolling Update strategy solves the original problem of maintaining application availability during upgrades by:

- Ensuring minimum replica availability
- Controlling the pace of updates
- Providing immediate rollback capability
- Maintaining service continuity

## Best Practices

1. **Use Rolling Updates for production** to maintain availability
2. **Configure appropriate `minReadySeconds`** for stability
3. **Set reasonable `maxUnavailable` and `maxSurge`** values
4. **Test thoroughly** before production deployments
5. **Monitor during deployments** to catch issues early
6. **Have rollback plans** ready for quick recovery

## Conclusion

This exercise demonstrates that Kubernetes deployment strategies provide flexible options for application updates, with Rolling Updates being ideal for production environments requiring high availability, while Recreate strategy offers simplicity for scenarios where brief downtime is acceptable.

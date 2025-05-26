# Kubernetes Multi-Container Pods Lab

This repository contains exercises for working with multi-container pods in Kubernetes, including sidecar patterns for logging with Elastic Stack.

## Table of Contents

- [Exercise 1: Identify Container Names in Blue Pod](#exercise-1-identify-container-names-in-blue-pod)
- [Exercise 2: Create Multi-Container Yellow Pod](#exercise-2-create-multi-container-yellow-pod)
- [Exercise 3: Elastic Stack Application Analysis](#exercise-3-elastic-stack-application-analysis)
- [Exercise 4: Add Sidecar Container for Log Shipping](#exercise-4-add-sidecar-container-for-log-shipping)

---

## Exercise 1: Identify Container Names in Blue Pod

### Problem Statement

**Identify the name of the containers running in the blue pod.**

### Solution

Use the `kubectl describe` command to inspect the pod:

```bash
kubectl describe pod blue
```

### Output Analysis

```bash
controlplane ~ ✖ k describe pod blue
Name:             blue
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.96.174
Start Time:       Mon, 26 May 2025 17:49:15 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               172.17.0.10
IPs:
  IP:  172.17.0.10
Containers:
  teal:
    Container ID:  containerd://d12366d861773846ee35a3e2fd8e7a2fb58d6dbfced5bf739812497ec402c2cb
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:f64ff79725d0070955b368a4ef8dc729bd8f3d8667823904adcb299fe58fc3da
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      4500
    State:          Running
      Started:      Mon, 26 May 2025 17:49:38 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-w2n7s (ro)
  navy:
    Container ID:  containerd://628cea18607fc97f5d4607c12887333839fe0005a3d6c836bd57a4a2fcaf6e2d
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:f64ff79725d0070955b368a4ef8dc729bd8f3d8667823904adcb299fe58fc3da
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      4500
    State:          Running
      Started:      Mon, 26 May 2025 17:49:39 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-w2n7s (ro)
```

### Answer

The containers running in the blue pod are:

1. **teal** (busybox image)
2. **navy** (busybox image)

---

## Exercise 2: Create Multi-Container Yellow Pod

### Problem Statement

**Create a multi-container pod with 2 containers.**

Use the spec given below:

- If the pod goes into crashloopbackoff then add the command sleep 1000 in the lemon container
- Name: yellow
- Container 1 Name: lemon
- Container 1 Image: busybox
- Container 2 Name: gold
- Container 2 Image: redis

### YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
    - name: lemon
      image: busybox
      command:
        - sleep
        - "1000"

    - name: gold
      image: redis
```

### Commands to Execute

```bash
# Apply the pod configuration
kubectl apply -f yellow-pod.yaml

# Verify pod creation
kubectl get pods yellow

# Describe the pod to verify containers
kubectl describe pod yellow
```

### Why This Solution Works

- **Multi-container Design**: The pod contains two containers that share the same network namespace and storage volumes
- **Crash Prevention**: The `sleep 1000` command keeps the busybox container running, preventing crashloopbackoff
- **Container Isolation**: Each container runs its own process but can communicate via localhost

---

## Exercise 3: Elastic Stack Application Analysis

### Problem Statement

**Inspect the app pod and identify the number of containers in it. It is deployed in the elastic-stack namespace.**

### Commands Used

```bash
# Describe the app pod in default namespace (initial inspection)
kubectl describe pods app

# View application logs
kubectl -n elastic-stack exec -it app -- cat /log/app.log
```

### Pod Description Output

```bash
controlplane ~ ➜  k describe pods app
Name:             app
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.96.174
Start Time:       Mon, 26 May 2025 17:48:40 +0000
Labels:           name=app
Annotations:      <none>
Status:           Running
IP:               172.17.0.8
IPs:
  IP:  172.17.0.8
Containers:
  app:
    Container ID:   containerd://1b17bb21f96c76eede50eb9d3123df25a4a68a43542d82b80015f032894e6f06
    Image:          kodekloud/event-simulator
    Image ID:       docker.io/kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 26 May 2025 17:49:37 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /log from log-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pvpc9 (ro)
```

### Log Analysis Output

```bash
kubectl -n elastic-stack exec -it app -- cat /log/app.log

[2025-05-26 17:49:44,825] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2025-05-26 17:49:44,825] INFO in event-simulator: USER2 is viewing page3
[2025-05-26 17:49:45,021] INFO in event-simulator: USER1 logged out
[2025-05-26 17:49:45,827] INFO in event-simulator: USER3 is viewing page3
[2025-05-26 17:49:46,022] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2025-05-26 17:49:46,022] INFO in event-simulator: USER1 logged out
[2025-05-26 17:49:46,828] INFO in event-simulator: USER3 is viewing page1
[2025-05-26 17:49:47,023] INFO in event-simulator: USER4 is viewing page3
[2025-05-26 17:49:47,828] INFO in event-simulator: USER1 is viewing page2
[2025-05-26 17:49:48,025] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-26 17:49:48,025] INFO in event-simulator: USER4 is viewing page3
[2025-05-26 17:49:48,830] INFO in event-simulator: USER4 logged out
[2025-05-26 17:49:49,026] INFO in event-simulator: USER3 is viewing page1
[2025-05-26 17:49:49,830] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-26 17:49:49,830] INFO in event-simulator: USER4 is viewing page1
[2025-05-26 17:49:50,028] INFO in event-simulator: USER4 is viewing page1
[2025-05-26 17:49:50,832] INFO in event-simulator: USER4 is viewing page2
[2025-05-26 17:49:51,028] INFO in event-simulator: USER3 is viewing page1
[2025-05-26 17:49:51,833] INFO in event-simulator: USER4 is viewing page2
[2025-05-26 17:49:52,029] INFO in event-simulator: USER4 logged out
[2025-05-26 17:49:52,834] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2025-05-26 17:49:52,835] INFO in event-simulator: USER4 logged out
[2025-05-26 17:49:53,030] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-26 17:49:53,030] INFO in event-simulator: USER4 is viewing page1
[2025-05-26 17:49:53,836] INFO in event-simulator: USER1 logged in
[2025-05-26 17:49:54,032] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2025-05-26 17:49:54,032] INFO in event-simulator: USER4 is viewing page1
[2025-05-26 17:49:54,837] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-26 17:49:54,837] INFO in event-simulator: USER3 logged out
[2025-05-26 17:49:55,033] INFO in event-simulator: USER1 is viewing page1
[2025-05-26 17:49:55,838] INFO in event-simulator: USER4 logged out
[2025-05-26 17:49:56,033] INFO in event-simulator: USER1 logged out
[2025-05-26 17:49:56,839] INFO in event-simulator: USER1 logged in
```

### Analysis Results

- **Number of containers**: 1 (the `app` container)
- **User with login issues**: **USER5** - account locked due to many failed login attempts
- **Log location**: `/log/app.log` (mounted from `/var/log/webapp` host path)

---

## Exercise 4: Add Sidecar Container for Log Shipping

### Problem Statement

**Edit the pod in the elastic-stack namespace to add a sidecar container to send logs to Elastic Search.**

Requirements:

- Name: app
- Container Name: sidecar
- Container Image: kodekloud/filebeat-configured
- Volume Mount: log-volume
- Mount Path: /var/log/event-simulator/
- Existing Container Name: app
- Existing Container Image: kodekloud/event-simulator

### YAML Configuration

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: elastic-stack
  labels:
    name: app
spec:
  containers:
    - name: app
      image: kodekloud/event-simulator
      volumeMounts:
        - mountPath: /log
          name: log-volume

    - name: sidecar
      image: kodekloud/filebeat-configured
      volumeMounts:
        - mountPath: /var/log/event-simulator/
          name: log-volume

  volumes:
    - name: log-volume
      hostPath:
        # directory location on host
        path: /var/log/webapp
        # this field is optional
        type: DirectoryOrCreate
```

### Commands to Execute

```bash
# Apply the updated pod configuration with force replacement
kubectl replace -f app-with-sidecar.yaml --force

# Verify the pod has been updated
kubectl -n elastic-stack get pods app

# Check that both containers are running
kubectl -n elastic-stack describe pod app

# Monitor the sidecar container logs
kubectl -n elastic-stack logs app -c sidecar
```

### Step-by-Step Explanation

1. **Pod Replacement**: Using `kubectl replace --force` recreates the pod with the new configuration
2. **Shared Volume**: Both containers mount the same `log-volume` but at different paths
3. **Log Flow**:
   - App container writes logs to `/log/app.log`
   - Sidecar container reads from `/var/log/event-simulator/`
   - Filebeat ships logs to Elasticsearch
4. **Sidecar Pattern**: The sidecar container handles log shipping without modifying the main application

### Why This Solution Works

**Sidecar Pattern Benefits**:

- **Separation of Concerns**: Main app focuses on business logic, sidecar handles log shipping
- **Shared Resources**: Both containers share the same network and storage
- **Independent Scaling**: Each container can be updated independently
- **Reusability**: The sidecar can be reused across different applications

**Volume Mounting Strategy**:

- **Single Volume**: `log-volume` is shared between containers
- **Different Mount Points**: App writes to `/log`, sidecar reads from `/var/log/event-simulator/`
- **Host Path Persistence**: Logs persist on the host at `/var/log/webapp`

**Elastic Stack Integration**:

- **Filebeat**: Lightweight log shipper that monitors log files
- **Elasticsearch**: Stores and indexes the log data
- **Kibana**: Provides visualization and analysis interface
- **Complete Pipeline**: app → logs → filebeat → elasticsearch → kibana → users

## Key Concepts Demonstrated

1. **Multi-Container Pods**: Running multiple containers in the same pod
2. **Volume Sharing**: Containers sharing storage volumes
3. **Sidecar Pattern**: Using helper containers for cross-cutting concerns
4. **Logging Architecture**: Implementing centralized logging with Elastic Stack
5. **Pod Lifecycle Management**: Updating running pods with new configurations

## References

- [Kubernetes Documentation: Communicate Between Containers in Same Pod](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)
- [Sidecar Pattern Documentation](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-logging-agent)

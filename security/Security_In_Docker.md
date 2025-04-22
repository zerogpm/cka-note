# Kubernetes Pod Security Contexts

This document demonstrates working with security contexts in Kubernetes pods, including user IDs and capabilities.

## Checking Current User in a Pod

To check the user executing processes in a pod:

```bash
k exec ubuntu-sleeper -- whoami
```

## Modifying Pod to Run with Specific User ID

Edit the pod `ubuntu-sleeper` to run with user ID 1010:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-04-22T21:30:35Z"
  name: ubuntu-sleeper
  namespace: default
  resourceVersion: "805"
  uid: e6c725b6-5e4d-4032-accb-169daab2423f
spec:
  containers:
    - command:
        - sleep
        - "4800"
      image: ubuntu
      imagePullPolicy: Always
      name: ubuntu
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-7w99s
          readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext:
    runAsUser: 1010
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
```

After making changes, reapply the configuration:

```bash
# Delete the existing pod
k delete pod ubuntu-sleeper --force

# Apply the updated configuration
k apply -f 1.yaml
```

## Understanding Security Context Hierarchy

### Multiple Containers with Different Security Contexts

Examining `multi-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
    - image: ubuntu
      name: web
      command: ["sleep", "5000"]
      securityContext:
        runAsUser: 1002
    - image: ubuntu
      name: sidecar
      command: ["sleep", "5000"]
```

#### User ID in Containers:

1. **web container**: Runs with user ID **1002**

   - Container-level security context overrides the pod-level setting

2. **sidecar container**: Runs with user ID **1001**
   - Uses pod-level security context as no container-level override is specified

## Adding Root User and Capabilities

Update `ubuntu-sleeper` pod to run as root and with the `SYS_TIME` capability:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-04-22T17:51:08Z"
  name: ubuntu-sleeper
  namespace: default
  resourceVersion: "802"
  uid: 64ed5caf-2d5b-4e15-98ae-7dbf4ec43e64
spec:
  containers:
    - command:
        - sleep
        - "4800"
      image: ubuntu
      imagePullPolicy: Always
      name: ubuntu
      securityContext:
        capabilities:
          add: ["SYS_TIME"]
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-5l2t8
          readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
```

Note: We only add the SYS_TIME capability without changing the user.

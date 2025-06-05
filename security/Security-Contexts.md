# Kubernetes Pod Security Context Configuration Guide

This guide demonstrates how to configure security contexts for Kubernetes pods, including user permissions and capabilities management.

## Original Problem Statement

**What is the user used to execute the sleep process within the ubuntu-sleeper pod? In the current(default) namespace.**

## Commands and Solutions

### 1. Checking Current User in Pod

```bash
controlplane ~ ➜  k exec ubuntu-sleeper -- whoami
root
```

**Explanation:** This command executes the `whoami` command inside the `ubuntu-sleeper` pod to determine which user is running the processes. The output shows that the processes are running as `root` user.

### 2. Modifying Pod to Run with Specific User ID

**Problem:** Edit the pod ubuntu-sleeper to run the sleep process with user ID 1010.

**Requirements:**

- Pod Name: ubuntu-sleeper
- Image Name: ubuntu
- SecurityContext: User 1010
- Note: Only make the necessary changes. Do not modify the name or image of the pod.

**Solution:**

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
    - command:
        - sleep
        - "4800"
      image: ubuntu
      name: ubuntu-sleeper
```

**Key Changes:**

- Added `securityContext` at the pod level
- Set `runAsUser: 1010` to run all containers in the pod with user ID 1010

### 3. Multi-Container Pod Security Context Analysis

**Given Configuration:**

```yaml
controlplane ~ ➜  cat multi-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002
   -  image: ubuntu
      name: sidecar
      command: ["sleep", "5000"]
```

**Analysis Questions and Answers:**

**Q: With what user are the processes in the web container started?**
**A: 1002**

**Explanation:** The `web` container has its own `securityContext` with `runAsUser: 1002`, which overrides the pod-level security context.

**Q: With what user are the processes in the sidecar container started?**
**A: 1001**

**Explanation:** The `sidecar` container doesn't have a container-level security context, so it inherits the pod-level `runAsUser: 1001`.

### 4. Adding Capabilities to Pod

**Problem:** Update pod ubuntu-sleeper to run as Root user and with the SYS_TIME capability.

**Solution:**

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
    - command:
        - sleep
        - "4800"
      image: ubuntu
      name: ubuntu-sleeper
      securityContext:
        capabilities:
          add: ["SYS_TIME"]
```

**Key Changes:**

- Removed pod-level `securityContext` (allowing default root user)
- Added container-level `securityContext` with `capabilities.add: ["SYS_TIME"]`

### 5. Adding Multiple Capabilities

**Problem:** Now update the pod to also make use of the NET_ADMIN capability.

**Final Solution:**

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
    - command:
        - sleep
        - "4800"
      image: ubuntu
      name: ubuntu-sleeper
      securityContext:
        capabilities:
          add: ["SYS_TIME", "NET_ADMIN"]
```

**Key Changes:**

- Added `NET_ADMIN` to the capabilities list alongside `SYS_TIME`

## Security Context Hierarchy

Understanding how Kubernetes applies security contexts:

1. **Pod-level Security Context**: Applies to all containers in the pod
2. **Container-level Security Context**: Overrides pod-level settings for that specific container

### Priority Order:

```
Container SecurityContext > Pod SecurityContext > Default Values
```

## Key Concepts Explained

### `runAsUser`

- Specifies the user ID to run the container processes
- Can be set at pod level (applies to all containers) or container level (specific to that container)

### `capabilities`

- Linux capabilities that can be added or dropped from containers
- Common capabilities:
  - `SYS_TIME`: Allows setting system clock
  - `NET_ADMIN`: Allows network administration tasks
  - Must be specified at container level, not pod level

## Best Practices

1. **Principle of Least Privilege**: Only grant the minimum required permissions
2. **Avoid Running as Root**: Use specific user IDs when possible
3. **Container-level Override**: Use container-level security contexts when different containers need different permissions
4. **Capability Management**: Only add necessary capabilities, consider dropping dangerous ones

## Troubleshooting

### Common Issues:

1. **Permission Denied**: Check if the user has appropriate file system permissions
2. **Capability Not Working**: Ensure capabilities are added at container level, not pod level
3. **User Override Not Working**: Verify container-level settings aren't overriding pod-level settings

### Verification Commands:

```bash
# Check current user in pod
kubectl exec <pod-name> -- whoami

# Check current user ID
kubectl exec <pod-name> -- id

# Check capabilities (if available in container)
kubectl exec <pod-name> -- capsh --print
```

# Kubernetes Pod Configuration and Docker Command Guide

This repository contains examples and solutions for creating Kubernetes pods with specific configurations, focusing on command execution, argument passing, and troubleshooting common YAML configuration issues.

## Table of Contents

- [Pod Configuration Examples](#pod-configuration-examples)
- [Docker Command Analysis](#docker-command-analysis)
- [Troubleshooting Guide](#troubleshooting-guide)
- [Step-by-Step Solutions](#step-by-step-solutions)

## Original Problems and Solutions

### Problem 1: Ubuntu Sleeper Pod Configuration

**Original Question:**

> Create a pod with the ubuntu image to run a container to sleep for 5000 seconds. Modify the file ubuntu-sleeper-2.yaml. Note: Only make the necessary changes. Do not modify the name.

#### Solution: ubuntu-sleeper-2.yaml

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-2
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command:
        - "sleep"
        - "5000"
```

**Explanation:**

- Uses the `ubuntu` base image
- Overrides the default command with `sleep`
- Passes `5000` seconds as an argument (both as strings)
- The pod will run for approximately 83 minutes before terminating

### Problem 2: Fixing YAML String Format Issues

**Original Question:**

> Create a pod using the file named ubuntu-sleeper-3.yaml. There is something wrong with it. Try to fix it! Note: Only make the necessary changes. Do not modify the name. Both sleep and 1200 should be defined as a string.

#### Original (Broken) Configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-3
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command:
        - "sleep"
        - 1200 # ❌ This should be a string
```

#### Fixed Configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-3
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command:
        - "sleep"
        - "1200" # ✅ Now properly quoted as a string
```

**Why This Fix Works:**

- YAML requires consistent data types in arrays
- Kubernetes expects command arguments to be strings
- The numeric value `1200` was causing type inconsistency
- Wrapping in quotes ensures proper string formatting

## Docker Command Analysis

### Dockerfile Analysis

#### Dockerfile 1: Basic Python Flask Application

```dockerfile
FROM python:3.6-alpine
RUN pip install flask
COPY . /opt/
EXPOSE 8080
WORKDIR /opt
ENTRYPOINT ["python", "app.py"]
```

**Command at startup:** `python app.py`

#### Dockerfile2: Python Flask with Default Arguments

```dockerfile
FROM python:3.6-alpine
RUN pip install flask
COPY . /opt/
EXPOSE 8080
WORKDIR /opt
ENTRYPOINT ["python", "app.py"]
CMD ["--color", "red"]
```

**Command at startup:** `python app.py --color red`

**Key Difference:** The second Dockerfile includes a `CMD` instruction that provides default arguments to the `ENTRYPOINT`.

### Container Command Override Examples

#### Example 1: webapp-color-2 Directory Analysis

**Dockerfile:**

```dockerfile
FROM python:3.6-alpine
RUN pip install flask
COPY . /opt/
EXPOSE 8080
WORKDIR /opt
ENTRYPOINT ["python", "app.py"]
CMD ["--color", "red"]
```

**Pod Configuration (webapp-color-pod.yaml):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
    name: webapp-green
spec:
  containers:
    - name: simple-webapp
      image: kodekloud/webapp-color
      command: ["--color", "green"] # ❌ Incorrect: overwrites ENTRYPOINT
```

**Actual command executed:** `--color green` (This will likely fail)

**Problem:** The `command` field in Kubernetes overwrites the Docker `ENTRYPOINT`, so `python app.py` is lost.

#### Example 2: webapp-color-3 Directory Analysis (Correct Approach)

**Dockerfile:**

```dockerfile
FROM python:3.6-alpine
RUN pip install flask
COPY . /opt/
EXPOSE 8080
WORKDIR /opt
ENTRYPOINT ["python", "app.py"]
CMD ["--color", "red"]
```

**Pod Configuration (webapp-color-pod-2.yaml):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
    name: webapp-green
spec:
  containers:
    - name: simple-webapp
      image: kodekloud/webapp-color
      command: ["python", "app.py"] # ✅ Preserves the application command
      args: ["--color", "pink"] # ✅ Properly overrides CMD arguments
```

**Actual command executed:** `python app.py --color pink`

## Step-by-Step Solutions

### Creating the Green Background Pod

**Original Question:**

> Create a pod with the given specifications. By default it displays a blue background. Set the given command line arguments to change it to green.

#### Final Solution:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
    name: webapp-green
spec:
  containers:
    - name: simple-webapp
      image: kodekloud/webapp-color
      args: ["--color", "green"]
```

### Command Execution Process:

1. **Deploy the pod:**

   ```bash
   kubectl apply -f webapp-green.yaml
   ```

2. **Verify pod creation:**

   ```bash
   kubectl get pods
   ```

3. **Check pod logs:**

   ```bash
   kubectl logs webapp-green
   ```

4. **Access the application (if service is configured):**
   ```bash
   kubectl port-forward pod/webapp-green 8080:8080
   ```

## Key Concepts Explained

### Docker vs Kubernetes Command Handling

| Docker       | Kubernetes | Result                      |
| ------------ | ---------- | --------------------------- |
| `ENTRYPOINT` | `command`  | Overrides Docker ENTRYPOINT |
| `CMD`        | `args`     | Overrides Docker CMD        |

### Best Practices

1. **Use `args` instead of `command`** when you want to preserve the Docker ENTRYPOINT
2. **Always quote string values** in YAML to avoid type conversion issues
3. **Test configurations** in development before applying to production
4. **Use meaningful labels** for pod identification and selection

### Common Troubleshooting Steps

1. **Check pod status:**

   ```bash
   kubectl get pods -o wide
   ```

2. **View pod events:**

   ```bash
   kubectl describe pod <pod-name>
   ```

3. **Check container logs:**

   ```bash
   kubectl logs <pod-name> -c <container-name>
   ```

4. **Debug with interactive shell:**
   ```bash
   kubectl exec -it <pod-name> -- /bin/bash
   ```

## Why These Solutions Work

- **String Consistency:** YAML parsing requires consistent data types, especially in arrays
- **Command Preservation:** Using `args` instead of `command` preserves the Docker ENTRYPOINT
- **Proper Override:** Kubernetes `args` correctly overrides Docker `CMD` while maintaining application startup logic
- **Resource Management:** Proper pod configuration ensures predictable resource usage and lifecycle management

## Additional Resources

- [Kubernetes Pod Documentation](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Docker ENTRYPOINT vs CMD](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact)
- [Kubernetes Configuration Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)

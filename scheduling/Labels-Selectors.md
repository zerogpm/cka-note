# Kubernetes Pod Selector Examples

This repository demonstrates how to use Kubernetes selectors to filter and identify resources based on labels. The examples show how to query pods across different environments, business units, and tiers.

## Problem Statement

We have deployed a number of PODs that are labeled with tier, env, and bu (business unit). The tasks include:

1. Finding the number of PODs in the dev environment
2. Finding the number of PODs in the finance business unit
3. Counting objects in the prod environment including PODs, ReplicaSets, and other objects
4. Identifying specific PODs based on multiple label criteria
5. Fixing and creating a ReplicaSet definition

## Environment Configuration

The cluster has various pods deployed with different label combinations:

- Environment labels (env): dev, prod
- Business unit labels (bu): finance and others
- Tier labels (tier): frontend and others

## Commands and Explanations

### 1. Counting PODs in the dev environment

```bash
controlplane ~ ➜  k get pods --selector env=dev
NAME          READY   STATUS    RESTARTS   AGE
app-1-gv647   1/1     Running   0          3m41s
app-1-llfh5   1/1     Running   0          3m41s
app-1-pb4sw   1/1     Running   0          3m41s
db-1-6vb98    1/1     Running   0          3m41s
db-1-b2sg5    1/1     Running   0          3m41s
db-1-r6nmc    1/1     Running   0          3m41s
db-1-z5v8w    1/1     Running   0          3m41s

controlplane ~ ➜  k get pods --selector env=dev --no-headers | wc -l
7
```

**Explanation:**

- `k get pods --selector env=dev`: Lists all pods with the label `env=dev`
- `--no-headers | wc -l`: Removes the headers from the output and counts the number of lines, giving us the total pod count
- Result: 7 pods are running in the dev environment

### 2. Counting PODs in the finance business unit

```bash
controlplane ~ ➜  k get pods --selector bu=finance --no-headers | wc -l
6
```

**Explanation:**

- `k get pods --selector bu=finance`: Lists all pods with the label `bu=finance`
- `--no-headers | wc -l`: Counts the number of lines in the output
- Result: 6 pods belong to the finance business unit

### 3. Counting objects in the prod environment

```bash
controlplane ~ ➜  k get all --selector env=prod
NAME              READY   STATUS    RESTARTS   AGE
pod/app-1-zzxdf   1/1     Running   0          6m45s
pod/app-2-6mpnd   1/1     Running   0          6m46s
pod/auth          1/1     Running   0          6m46s
pod/db-2-hlkvp    1/1     Running   0          6m46s

NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/app-1   ClusterIP   10.43.33.47   <none>        3306/TCP   6m45s

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/app-2   1         1         1       6m46s
replicaset.apps/db-2    1         1         1       6m46s

controlplane ~ ➜  k get all --selector env=prod | wc -l
12

controlplane ~ ➜  k get all --selector env=prod --no-headers | wc -l
7
```

**Explanation:**

- `k get all --selector env=prod`: Lists all Kubernetes resources (not just pods) with the label `env=prod`
- Result shows 4 pods, 1 service, and 2 ReplicaSets
- `| wc -l`: Returns 12 lines with headers (includes headers and blank lines)
- `--no-headers | wc -l`: Returns 7 resources without headers

### 4. Identifying specific PODs with multiple criteria

```bash
kubectl get all --selector env=prod,bu=finance,tier=frontend
NAME              READY   STATUS    RESTARTS   AGE
pod/app-1-zzxdf   1/1     Running   0          10m
```

**Explanation:**

- `kubectl get all --selector env=prod,bu=finance,tier=frontend`: Lists all resources that match all three label criteria
- Result: Only one pod (`app-1-zzxdf`) matches all the conditions:
  - In the prod environment
  - Part of the finance business unit
  - In the frontend tier

### 5. Fixing and creating a ReplicaSet definition

Original YAML with issue:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: front-end
  template:
    metadata:
      labels:
        tier: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
```

**Issue identified:**
The labels in the `template.metadata.labels` section (`tier: nginx`) don't match the selector in `spec.selector.matchLabels` (`tier: front-end`).

**Fixed YAML:**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: front-end
  template:
    metadata:
      labels:
        tier: front-end
    spec:
      containers:
        - name: nginx
          image: nginx
```

**Explanation of the fix:**

- Changed `tier: nginx` to `tier: front-end` in the template's labels section
- This ensures the pod template's labels match the selector criteria
- The ReplicaSet controller uses these labels to identify which pods it manages

**Creating the fixed ReplicaSet:**

```bash
kubectl apply -f replicaset-definition-1.yaml
```

## Key Concepts

### Label Selectors

- **Equality-based selectors**: `--selector key=value` or `-l key=value`
- **Set-based selectors**: `--selector key in (value1,value2)`

### Why Use Labels and Selectors?

1. **Organization**: Group related resources logically
2. **Selection**: Target specific resources for operations
3. **Querying**: Filter resources based on attributes
4. **Service Routing**: Direct traffic to specific pods

### Common Use Cases

- Filtering resources by environment (dev, prod)
- Filtering by application components (frontend, backend, database)
- Filtering by business units or teams
- Combining multiple filters for precise selection

## Summary

This exercise demonstrates how to:

1. Use Kubernetes label selectors to filter and count resources
2. Identify resources matching multiple criteria
3. Debug and fix issues with label selectors in ReplicaSet definitions
4. Understand the importance of matching labels between selectors and pod templates

These skills are essential for managing and operating complex Kubernetes deployments with multiple environments, teams, and application components.

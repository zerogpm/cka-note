# Kubernetes Namespace and Service Discovery

This repository demonstrates Kubernetes namespace operations and service discovery principles. The exercises cover namespace exploration, pod management, and service discovery between namespaces.

## Problem Statement

The exercise involves several tasks related to Kubernetes namespaces:

1. Identifying existing namespaces in the cluster
2. Counting pods in a specific namespace
3. Creating a pod in a designated namespace
4. Locating a specific pod across namespaces
5. Understanding service discovery within the same namespace
6. Understanding service discovery across different namespaces

## Commands and Outputs

### 1. Identifying Existing Namespaces

**Command:**

```bash
controlplane ~ ➜  k get namespace --no-headers
```

**Output:**

```
default           Active   8m25s
dev               Active   75s
finance           Active   75s
kube-node-lease   Active   8m25s
kube-public       Active   8m25s
kube-system       Active   8m25s
manufacturing     Active   74s
marketing         Active   75s
prod              Active   75s
research          Active   74s
```

**Counting the namespaces:**

```bash
controlplane ~ ➜  k get namespace --no-headers | wc -l
```

**Output:**

```
10
```

**Explanation:**

- The `k get namespace --no-headers` command lists all namespaces in the cluster without the headers row
- The `wc -l` command counts the number of lines in the output, giving us the total namespace count
- There are 10 namespaces in the cluster

### 2. Counting Pods in the Research Namespace

**Command:**

```bash
controlplane ~ ➜ k get pod -n research
```

**Output:**

```
NAME    READY   STATUS             RESTARTS      AGE
dna-1   0/1     CrashLoopBackOff   4 (49s ago)   2m11s
dna-2   0/1     CrashLoopBackOff   4 (32s ago)   2m11s
```

**Explanation:**

- The `k get pod -n research` command lists all pods in the research namespace
- There are 2 pods in the research namespace (dna-1 and dna-2)
- Both pods are in CrashLoopBackOff state, indicating they're failing to start properly

### 3. Creating a Pod in the Finance Namespace

**Command:**

```bash
controlplane ~ ➜ k run redis --image=redis -n finance
```

**Output:**

```
pod/redis created
```

**Verification:**

```bash
controlplane ~ ➜ k get pods -n finance
```

**Output:**

```
NAME      READY   STATUS    RESTARTS   AGE
payroll   1/1     Running   0          3m47s
redis     1/1     Running   0          9s
```

**Explanation:**

- The `k run redis --image=redis -n finance` command creates a new pod named "redis" using the redis image in the finance namespace
- The verification command shows both the newly created redis pod and an existing payroll pod in the finance namespace
- Both pods are in Running state, indicating they're functioning correctly

### 4. Locating the Blue Pod

**Command:**

```bash
controlplane ~ ➜ k get all -A | grep -i "blue"
```

**Output:**

```
marketing pod/blue 1/1 Running 0 5m marketing service/blue-service NodePort 10.43.60.49 <none> 8080:30082/TCP 5m
```

**Explanation:**

- The `k get all -A` command lists all resources across all namespaces
- The `grep -i "blue"` filters for resources with "blue" in their name (case-insensitive)
- The blue pod is located in the marketing namespace
- There's also a blue-service in the marketing namespace

### 5. Service Discovery Within the Same Namespace

**Command:**

```bash
controlplane ~ ➜ k get svc -n marketing
```

**Output:**

```
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
blue-service   NodePort   10.43.60.49     <none>        8080:30082/TCP   6m34s
db-service     NodePort   10.43.219.215   <none>        6379:31050/TCP   6m34s
```

**Solution:**
When the Blue application needs to access the database service in its own namespace (marketing), it can use the simple service name: `db-service`

For completeness, the following would also work:

- `db-service.marketing`
- `db-service.marketing.svc`
- `db-service.marketing.svc.cluster.local`

Port to use: 6379

### 6. Service Discovery Across Different Namespaces

**Command:**

```bash
controlplane ~ ➜ k get svc -n dev
```

**Output:**

```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
db-service   ClusterIP   10.43.110.137   <none>        6379/TCP   7m10s
```

**Solution:**
When the Blue application (in marketing namespace) needs to access the db-service in a different namespace (dev), it must use the Fully Qualified Domain Name (FQDN): `db-service.dev.svc.cluster.local`

Alternatively, the shorter form would also work: `db-service.dev`

Port to use: 6379

## Kubernetes YAML Configuration

While no YAML files were explicitly created in this exercise, here's an equivalent YAML representation of the redis pod that was created using the imperative command:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
  namespace: finance
spec:
  containers:
    - name: redis
      image: redis
```

## Explanation of Kubernetes Service Discovery

### How Kubernetes DNS Works

Kubernetes uses a DNS system to enable service discovery. The DNS naming convention follows this pattern:
`<service-name>.<namespace-name>.svc.cluster.local`

- **Within the same namespace**: Services can be accessed using just the service name
- **Across different namespaces**: Services must be accessed using at least `<service-name>.<namespace-name>`

### Why This Solution Works

1. **Namespace isolation**: Namespaces provide logical separation of resources in a Kubernetes cluster
2. **DNS-based service discovery**: Kubernetes automatically assigns DNS names to services
3. **Fully Qualified Domain Names**: The FQDN pattern allows pods in any namespace to discover and access services in any other namespace

This approach allows applications to:

- Find and communicate with other services without knowing their exact IP addresses
- Maintain stable endpoints even as pods are created, destroyed, or moved between nodes
- Access services across namespace boundaries when needed

## Summary

These exercises demonstrate key Kubernetes concepts:

1. **Namespace management**: Creating and exploring namespaces for resource organization
2. **Pod deployment**: Creating pods in specific namespaces
3. **Service discovery**: Understanding how applications can discover and connect to services within and across namespaces
4. **DNS naming conventions**: Learning the pattern for Kubernetes DNS-based service discovery

Understanding these concepts is crucial for properly architecting and managing multi-service applications in Kubernetes.

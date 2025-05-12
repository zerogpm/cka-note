# Kubernetes Kustomization and Patching Guide

This README explains how to use Kubernetes kustomize to modify and patch Kubernetes resource configurations. It addresses specific problems related to deployment scaling, label management, and service configuration.

## Problem Statement

The original question involved several parts:

1. Determining how many nginx pods will be created with a patched deployment
2. Identifying labels applied to the mongo deployment
3. Finding the target port of the mongo-cluster-ip-service
4. Counting containers in the api pod
5. Locating the mount path for the mongo-volume
6. Creating an inline json6902 patch to remove a specific label

## Configuration Files

### kustomization.yaml

```yaml
resources:
  - mongo-depl.yaml
  - nginx-depl.yaml
  - mongo-service.yaml

patches:
  - target:
      kind: Deployment
      name: nginx-deployment
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 3

  - target:
      kind: Deployment
      name: mongo-deployment
    path: mongo-label-patch.yaml

  - target:
      kind: Service
      name: mongo-cluster-ip-service
    patch: |-
      - op: replace
        path: /spec/ports/0/port
        value: 30000

      - op: replace
        path: /spec/ports/0/targetPort
        value: 30000
```

### mongo-depl.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: mongo
  template:
    metadata:
      labels:
        component: mongo
    spec:
      containers:
        - name: mongo
          image: mongo
```

### mongo-label-patch.yaml

```yaml
- op: add
  path: /spec/template/metadata/labels/cluster
  value: staging

- op: add
  path: /spec/template/metadata/labels/feature
  value: db
```

### mongo-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: mongo
  ports:
    - port: 27017
      targetPort: 27017
```

### api-depl.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: api
  template:
    metadata:
      labels:
        component: api
    spec:
      containers:
        - name: nginx
          image: nginx
```

### api-patch.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - name: memcached
          image: memcached
```

### host-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
    type: DirectoryOrCreate
```

### mongo-patch.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
spec:
  template:
    spec:
      containers:
        - name: mongo
          volumeMounts:
            - mountPath: /data/db
              name: mongo-volume
      volumes:
        - name: mongo-volume
          persistentVolumeClaim:
            claimName: host-pvc
```

## Commands and Solutions

### 1. How many nginx pods will get created?

**Command:**

```bash
kubectl apply -k /path/to/directory
kubectl get deployments nginx-deployment
```

**Expected Output:**

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           XX
```

**Explanation:**
The original nginx-deployment has a replicaCount of 1, but the kustomization.yaml file includes a patch that changes it to 3. When kustomize applies this configuration, it will create 3 nginx pods.

### 2. What are the labels that will be applied to the mongo deployment?

**Command:**

```bash
kubectl apply -k /path/to/directory
kubectl get deployment mongo-deployment -o jsonpath='{.spec.template.metadata.labels}'
```

**Expected Output:**

```
{"component":"mongo","cluster":"staging","feature":"db"}
```

**Explanation:**
The mongo deployment originally has the label `component: mongo`. The mongo-label-patch.yaml adds two more labels: `cluster: staging` and `feature: db`. Therefore, the total labels will be:

- component=mongo
- cluster=staging
- feature=db

### 3. What is the target port of the mongo-cluster-ip-service?

**Command:**

```bash
kubectl apply -k /path/to/directory
kubectl get service mongo-cluster-ip-service -o jsonpath='{.spec.ports[0].targetPort}'
```

**Expected Output:**

```
30000
```

**Explanation:**
The original targetPort in mongo-service.yaml is 27017, but the patch in kustomization.yaml replaces it with 30000.

### 4. How many containers are in the api pod?

**Command:**

```bash
kubectl apply -k /path/to/directory
kubectl get pod -l component=api -o jsonpath='{.items[0].spec.containers[*].name}'
```

**Expected Output:**

```
nginx memcached
```

**Explanation:**
The api-depl.yaml file defines one container (nginx), and the api-patch.yaml adds a second container (memcached). Therefore, there are 2 containers in the api pod.

### 5. What path in the mongo container is the mongo-volume volume mounted at?

**Command:**

```bash
kubectl apply -k /path/to/directory
kubectl get deployment mongo-deployment -o jsonpath='{.spec.template.spec.containers[0].volumeMounts[0].mountPath}'
```

**Expected Output:**

```
/data/db
```

**Explanation:**
According to the mongo-patch.yaml file, the mongo-volume is mounted at `/data/db` in the mongo container.

### 6. Create an inline json6902 patch to remove the label org: KodeKloud from the mongo-deployment

**Solution:**

Add the following to kustomization.yaml:

```yaml
resources:
  - mongo-depl.yaml
  - api-depl.yaml
  - mongo-service.yaml

patches:
  - target:
      kind: Deployment
      name: mongo-deployment
    patch: |-
      - op: remove
        path: /spec/template/metadata/labels/org
```

**Command to apply:**

```bash
kubectl apply -k /root/code/k8s/
```

**Explanation:**
This inline patch uses the JSON Patch operation `remove` to delete the label `org: KodeKloud` from the mongo-deployment's pod template metadata.

## Why This Approach Works

Kubernetes kustomize is a powerful tool for managing configuration changes without modifying the original YAML files. This approach works because:

1. **Separation of Concerns**: Base resources are kept separate from the modifications, making it easier to manage and track changes.

2. **Patch-based Modifications**: Kustomize allows for targeted modifications using JSON/YAML patches, which are more precise than manually editing files.

3. **Reusability**: The base resources can be reused across different environments by simply applying different kustomizations.

4. **Declarative Management**: All changes are declared in the kustomization.yaml file, providing a single source of truth for configuration changes.

5. **Reduced Human Error**: Using kustomize reduces the chance of manual errors when modifying configurations.

## Summary

This documentation demonstrates how to use Kubernetes kustomize to manage deployments, services, and persistent volumes. By using patches, we can modify configurations without changing the original YAML files, which is especially useful for:

- Adjusting replica counts
- Adding or removing labels
- Changing service port configurations
- Adding containers to deployments
- Configuring volume mounts

This approach provides a more maintainable and scalable way to manage Kubernetes resources across different environments.

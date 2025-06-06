# Kubernetes Custom Resource Definition (CRD) Setup

## Original Problem Statement

**Question:** "CRD Object can be either namespaced or cluster scoped. Is this statement true or false? We have provided an incomplete CRDs manifest file called crd.yaml under the /root directory. Let's complete it and create a custom resource definition from it. Let's create a custom resource definition called internals.datasets.kodekloud.com. Assign the group to datasets.kodekloud.com and the resource is accessible only from a specific namespace. Make sure the version should be v1 and needed to enable the version so it's being served via REST API. So finally create a custom resource from a given manifest file called custom.yaml."

**Answer to the question:** **TRUE** - CRD objects can indeed be either namespaced or cluster-scoped, depending on the `scope` field in the CRD specification.

## Overview

This guide demonstrates how to create and deploy a Custom Resource Definition (CRD) in Kubernetes that defines a new resource type called `internals.datasets.kodekloud.com`. The CRD will be namespaced-scoped and served via the Kubernetes REST API.

## Files Involved

### 1. Original Incomplete CRD (`crd.yaml`)

```yaml
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name:
spec:
  group:
  versions:
    - name: v2
      served: false
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                internalLoad:
                  type: string
                range:
                  type: integer
                percentage:
                  type: string
  scope:
  names:
    plural: internal
    singular: internal
    kind: Internal
    shortNames:
      - int
```

### 2. Completed CRD (`crd.yaml`)

```yaml
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: internals.datasets.kodekloud.com
spec:
  group: datasets.kodekloud.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                internalLoad:
                  type: string
                range:
                  type: integer
                percentage:
                  type: string
  scope: Namespaced
  names:
    plural: internals
    singular: internal
    kind: Internal
    shortNames:
      - int
```

### 3. Custom Resource Example (`custom.yaml`)

```yaml
---
apiVersion: datasets.kodekloud.com/v1
kind: Internal
metadata:
  name: sample-internal
  namespace: default
spec:
  internalLoad: "high"
  range: 100
  percentage: "75%"
```

## Step-by-Step Implementation

### Step 1: Create the Custom Resource Definition

```bash
# Apply the completed CRD to the cluster
kubectl apply -f crd.yaml
```

**Expected Output:**

```
customresourcedefinition.apiextensions.k8s.io/internals.datasets.kodekloud.com created
```

### Step 2: Verify CRD Creation

```bash
# Check if the CRD was created successfully
kubectl get crd internals.datasets.kodekloud.com
```

**Expected Output:**

```
NAME                               CREATED AT
internals.datasets.kodekloud.com   2024-XX-XXTXX:XX:XXZ
```

### Step 3: Verify API Resources

```bash
# Check if the new resource type is available
kubectl api-resources | grep internals
```

**Expected Output:**

```
internals                         int          datasets.kodekloud.com/v1              true         Internal
```

### Step 4: Create a Custom Resource Instance

```bash
# Create a custom resource from the custom.yaml file
kubectl create -f custom.yaml
```

**Expected Output:**

```
internal.datasets.kodekloud.com/sample-internal created
```

### Step 5: Verify Custom Resource Creation

```bash
# List the created custom resources
kubectl get internals
```

**Expected Output:**

```
NAME              AGE
sample-internal   Xs
```

```bash
# Get detailed information about the custom resource
kubectl describe internal sample-internal
```

**Expected Output:**

```
Name:         sample-internal
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  datasets.kodekloud.com/v1
Kind:         Internal
Metadata:
  Creation Timestamp:  2024-XX-XXTXX:XX:XXZ
  Generation:          1
  Resource Version:    XXXXX
  UID:                 xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Spec:
  Internal Load:  high
  Percentage:     75%
  Range:          100
Events:           <none>
```

## Key Changes Made to Complete the CRD

| Field                | Original Value | Corrected Value                    | Explanation                                   |
| -------------------- | -------------- | ---------------------------------- | --------------------------------------------- |
| `metadata.name`      | Empty          | `internals.datasets.kodekloud.com` | Must match the pattern `<plural>.<group>`     |
| `spec.group`         | Empty          | `datasets.kodekloud.com`           | Defines the API group for the custom resource |
| `versions[0].name`   | `v2`           | `v1`                               | Changed to required version v1                |
| `versions[0].served` | `false`        | `true`                             | Enables the version to be served via REST API |
| `spec.scope`         | Empty          | `Namespaced`                       | Makes the resource namespace-scoped           |
| `names.plural`       | `internal`     | `internals`                        | Corrected to proper plural form               |

## Why This Approach Solves the Problem

### 1. **Namespace Scoping**

Setting `scope: Namespaced` ensures that custom resources are isolated within specific namespaces, providing better resource organization and access control.

### 2. **API Versioning**

Using `v1` with `served: true` enables the custom resource to be accessible via the Kubernetes REST API, allowing standard kubectl operations.

### 3. **Proper Naming Convention**

The CRD name follows the required format `<plural>.<group>`, ensuring proper resource identification and API routing.

### 4. **Schema Validation**

The OpenAPI v3 schema defines the structure and validation rules for custom resource instances, ensuring data consistency.

### 5. **Resource Accessibility**

With the short name `int`, users can use `kubectl get int` as a shorthand for `kubectl get internals`.

## Additional Commands for Management

### Delete Custom Resource

```bash
kubectl delete internal sample-internal
```

### Delete CRD (Warning: This will delete all instances)

```bash
kubectl delete crd internals.datasets.kodekloud.com
```

### View CRD YAML

```bash
kubectl get crd internals.datasets.kodekloud.com -o yaml
```

## Troubleshooting

### Common Issues:

1. **CRD name mismatch**: Ensure the metadata name matches `<plural>.<group>`
2. **Version not served**: Set `served: true` for the version you want to use
3. **Schema validation errors**: Check the OpenAPI v3 schema structure
4. **Namespace issues**: Ensure the target namespace exists when creating namespaced resources

### Validation Commands:

```bash
# Check CRD status
kubectl get crd internals.datasets.kodekloud.com -o jsonpath='{.status}'

# Validate custom resource
kubectl get internals -o yaml
```

This setup successfully creates a functional Custom Resource Definition that extends the Kubernetes API with a new resource type, demonstrating the flexibility and extensibility of the Kubernetes platform.

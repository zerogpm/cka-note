# Kubernetes RBAC (Role-Based Access Control) Notes

## Overview

This document covers key concepts and commands for managing Kubernetes Role-Based Access Control (RBAC).

## Authorization Modes

The cluster is configured with the following authorization modes:

- Node
- RBAC

This can be verified in the kube-apiserver configuration:

```yaml
--authorization-mode=Node,RBAC
```

## Role Management

### Checking Roles

```bash
# Check roles in default namespace
kubectl get role

# Check roles across all namespaces
kubectl get role -A

# Count total roles in all namespaces
kubectl get role -A --no-headers | wc -l
```

### Inspecting Role Permissions

```bash
# View details of a specific role
kubectl describe role <role-name> -n <namespace>

# Example: Inspect kube-proxy role
kubectl describe role kube-proxy -n kube-system
```

### Role Bindings

```bash
# View role bindings
kubectl describe rolebinding <rolebinding-name> -n <namespace>

# Example: Check kube-proxy rolebinding
kubectl describe rolebinding kube-proxy -n kube-system
```

## User Permission Verification

```bash
# Check if a user can perform specific actions
kubectl auth can-i <verb> <resource> --as <username>

# Example: Check if dev-user can list pods
kubectl auth can-i list pods --as dev-user
```

## Creating Roles and Role Bindings

### Create a Role

```bash
kubectl create role <role-name> --verb=<verbs> --resource=<resources>

# Example: Create developer role with pod permissions
kubectl create role developer --verb=list,create,delete --resource=pods
```

### Create a Role Binding

```bash
kubectl create rolebinding <binding-name> --role=<role-name> --user=<username>

# Example: Bind developer role to dev-user
kubectl create rolebinding dev-user-binding --role=developer --user=dev-user
```

## Editing Roles

```bash
kubectl edit role <role-name> -n <namespace>

# Example: Edit developer role in blue namespace
kubectl edit role developer -n blue
```

## Role Configuration Examples

### Basic Pod Management Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: blue
rules:
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - get
      - watch
      - create
      - delete
```

### Role with Resource Name Restrictions

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: blue
rules:
  - apiGroups:
      - ""
    resourceNames:
      - dark-blue-app
    resources:
      - pods
    verbs:
      - get
      - watch
      - create
      - delete
```

### Role with Multiple Resource Types

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: blue
rules:
  - apiGroups:
      - ""
    resourceNames:
      - dark-blue-app
    resources:
      - pods
    verbs:
      - get
      - watch
      - create
      - delete
  - apiGroups:
      - "apps"
    resources:
      - deployments
    verbs:
      - get
      - watch
      - create
      - delete
```

## Common Issues and Troubleshooting

- When a user can't access specific resources, check:
  - If the correct role is created with appropriate permissions
  - If the role binding correctly links the role to the user
  - If the role includes the necessary API groups for the resource
  - If resourceNames restrictions are in place and configured correctly

## Important Notes

- Remember to specify the API group when working with non-core resources:
  - Core resources (pods, services, etc.): `""`
  - Deployments, ReplicaSets, etc.: `"apps"`
  - NetworkPolicies: `"networking.k8s.io"`
- Role bindings can be assigned to:
  - Users
  - Groups
  - Service accounts

# Kubernetes Secrets Configuration Lab

## Original Problem Statement

**How many Secrets exist on the system? In the current(default) namespace.**

**How many secrets are defined in the dashboard-token secret?**

**What is the type of the dashboard-token secret?**

**We are going to deploy an application with the below architecture:**

We have already deployed the required pods and services. Check out the pods and services created. Check out the web application using the Webapp MySQL link above your terminal, next to the Quiz Portal Link.

```
mysql pod -> mysql services -> secret -> webapp pod -> webapp service -> users
```

The reason the application is failed is because we have not created the secrets yet. Create a new secret named db-secret with the data given below.

**Secret Requirements:**

- Secret Name: `db-secret`
- Secret 1: `DB_Host=sql01`
- Secret 2: `DB_User=root`
- Secret 3: `DB_Password=password123`

Configure webapp-pod to load environment variables from the newly created secret. Delete and recreate the pod if required.

---

## Solution Overview

This lab demonstrates how to work with Kubernetes Secrets to securely manage sensitive configuration data for applications. The scenario involves a web application that needs database credentials to connect to a MySQL database.

### Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  MySQL Pod  │───▶│MySQL Service│───▶│   Secret    │───▶│ WebApp Pod  │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                            │                     │
                                            ▼                     ▼
                                    ┌─────────────┐    ┌─────────────┐
                                    │ DB Configs  │    │WebApp Service│
                                    └─────────────┘    └─────────────┘
                                                              │
                                                              ▼
                                                       ┌─────────────┐
                                                       │    Users    │
                                                       └─────────────┘
```

---

## Commands and Analysis

### 1. Check Existing Secrets

**Command:**

```bash
k get secrets
```

**Output:**

```bash
controlplane ~ ✖ k get secrets
NAME              TYPE                                  DATA   AGE
dashboard-token   kubernetes.io/service-account-token   3      43s
```

**Analysis:**

- **Number of secrets in default namespace:** 1
- **Secret name:** `dashboard-token`

### 2. Inspect Dashboard Token Secret

**Command:**

```bash
k describe secrets dashboard-token
```

**Output:**

```bash
controlplane ~ ➜  k describe secrets dashboard-token
Name:         dashboard-token
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-sa
              kubernetes.io/service-account.uid: 5ea50671-e190-48d4-915b-152dd60251dd

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     566 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlkzZVQxX01YMzdMY3lodFE5d3lnd2hXQ0ZuNk1kME9fZjNjWHl3NFZHbE0ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC10b2tlbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQtc2EiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI1ZWE1MDY3MS1lMTkwLTQ4ZDQtOTE1Yi0xNTJkZDYwMjUxZGQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtc2EifQ.KIeaAT9ui9BdoBDCrrBYv2AOGpw08Ig8a2gVx_WI9NtjAmUmm3tg4dM1K2tvbxYwFSttWRY-VrMlIUW0SGgmPHOTBybmOLhayI98I56CYTVe0ypxxTs7rTBhz1kJQ9MH3Ofemo0PjfYjh51MoQGPNsFnoWOkxi2G6w5wRJ14yaQhRJdAkGALRmmv3_SceN0hA_udvAaW3kpqejaYmLb901XSs2WADh6z3s4IhqXkfW1kPsN6mvvFgQTkWWuMcYOLy3IRlwR1qru04LsV5oHMHindoopAGIv2JsAWCH2rKxee8WUFSF5A4P_PxAvaa8E9dkw9Kd_25u1N8o2A9Hup7g
```

**Analysis:**

- **Number of secrets defined in dashboard-token:** 3 (ca.crt, namespace, token)
- **Secret type:** `kubernetes.io/service-account-token`

### 3. Create Database Secret

**Command:**

```bash
k create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
```

**Step-by-step breakdown:**

- `k create secret generic`: Creates a generic type secret
- `db-secret`: Name of the secret
- `--from-literal=DB_Host=sql01`: Creates a key-value pair for database host
- `--from-literal=DB_User=root`: Creates a key-value pair for database username
- `--from-literal=DB_Password=password123`: Creates a key-value pair for database password

---

## YAML Configuration

### WebApp Pod Configuration

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-pod
  name: webapp-pod
  namespace: default
spec:
  containers:
    - image: kodekloud/simple-webapp-mysql
      imagePullPolicy: Always
      name: webapp
      envFrom:
        - secretRef:
            name: db-secret
```

### YAML Configuration Breakdown

**Metadata Section:**

- `apiVersion: v1`: Uses Kubernetes API version 1
- `kind: Pod`: Specifies this is a Pod resource
- `metadata.name`: Pod name is `webapp-pod`
- `metadata.labels`: Adds a label for identification
- `metadata.namespace`: Deploys to the `default` namespace

**Spec Section:**

- `containers[0].image`: Uses the `kodekloud/simple-webapp-mysql` image
- `containers[0].imagePullPolicy: Always`: Always pulls the latest image
- `containers[0].name`: Container name is `webapp`
- `containers[0].envFrom`: Loads environment variables from external sources
- `containers[0].envFrom[0].secretRef.name`: References the `db-secret` secret

---

## Step-by-Step Implementation

### Step 1: Analyze Current State

1. Check existing secrets in the default namespace
2. Examine the dashboard-token secret to understand its structure
3. Identify the secret type and data fields

### Step 2: Create Database Secret

1. Use `kubectl create secret generic` command
2. Add three key-value pairs for database configuration:
   - `DB_Host=sql01`
   - `DB_User=root`
   - `DB_Password=password123`

### Step 3: Configure WebApp Pod

1. Create or update the webapp-pod YAML configuration
2. Add `envFrom` section to load environment variables from the secret
3. Reference the newly created `db-secret`

### Step 4: Deploy/Update Pod

1. If the pod already exists, delete it first: `kubectl delete pod webapp-pod`
2. Apply the new configuration: `kubectl apply -f webapp-pod.yaml`

---

## Why This Approach Solves the Problem

### Security Benefits

- **Separation of Concerns**: Database credentials are stored separately from application code
- **Base64 Encoding**: Secrets are automatically base64 encoded by Kubernetes
- **Access Control**: Secrets can be managed with RBAC policies
- **Audit Trail**: Secret creation and access can be logged

### Operational Benefits

- **Environment Variables**: The application can read database configuration as standard environment variables
- **Dynamic Updates**: Secrets can be updated without rebuilding container images
- **Namespace Isolation**: Secrets are scoped to specific namespaces
- **Reusability**: The same secret can be used by multiple pods

### Problem Resolution

The original application failure was caused by missing database configuration. By creating the `db-secret` and configuring the webapp pod to load environment variables from this secret, the application can now:

1. **Connect to Database**: Access the MySQL database using the provided credentials
2. **Read Configuration**: Load `DB_Host`, `DB_User`, and `DB_Password` as environment variables
3. **Maintain Security**: Keep sensitive data separate from application deployment files
4. **Enable Scalability**: Allow multiple application instances to use the same database configuration

### Environment Variable Mapping

When the pod starts, the secret data becomes available as environment variables:

- `DB_Host` → `sql01`
- `DB_User` → `root`
- `DB_Password` → `password123`

The application code can then read these environment variables to establish database connections, completing the architecture flow from MySQL pod through services and secrets to the webapp pod and finally to end users.

---

## Additional Commands for Verification

```bash
# Verify the secret was created
kubectl get secrets

# Check secret details
kubectl describe secret db-secret

# Verify pod is running
kubectl get pods

# Check pod environment variables (if needed for debugging)
kubectl exec webapp-pod -- env | grep DB_
```

This approach follows Kubernetes best practices for managing sensitive configuration data and ensures secure, scalable application deployment.

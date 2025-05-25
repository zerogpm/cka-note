# Kubernetes Admission Controller Webhook Demo

This repository demonstrates the implementation and usage of Kubernetes Admission Webhooks, specifically focusing on mutating and validating admission controllers for enforcing security policies on pod creation.

## Table of Contents

- [Overview](#overview)
- [Admission Controllers Basics](#admission-controllers-basics)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Testing Scenarios](#testing-scenarios)
- [How It Works](#how-it-works)
- [Troubleshooting](#troubleshooting)

## Overview

This demo implements a webhook that enforces security policies for pods:

- Denies all requests for pods to run as root if no securityContext is provided
- Sets default values: `runAsNonRoot: true` and `runAsUser: 1234` if not specified
- Allows containers to run as root only if `runAsNonRoot` is explicitly set to `false`

## Admission Controllers Basics

### Q: Which of the below combination is correct for Mutating and validating admission controllers?

**A: NamespaceAutoProvision - Mutating, NamespaceExists - Validating**

### Q: What is the flow of invocation of admission controllers?

**A: First Mutating then Validating**

## Prerequisites

- Kubernetes cluster (tested on control plane)
- kubectl configured
- TLS certificates and keys pre-generated at:
  - Certificate: `/root/keys/webhook-server-tls.crt`
  - Key: `/root/keys/webhook-server-tls.key`
- Namespace `webhook-demo` created

## Setup Instructions

### Step 1: Create TLS Secret

Create TLS secret webhook-server-tls for secure webhook communication in webhook-demo namespace.

```bash
kubectl -n webhook-demo create secret tls webhook-server-tls \
    --cert "/root/keys/webhook-server-tls.crt" \
    --key "/root/keys/webhook-server-tls.key"
```

**Explanation**: This command creates a Kubernetes TLS secret that will be used by the webhook server for secure HTTPS communication. The admission controller requires TLS for webhook calls.

### Step 2: Create Webhook Deployment

Deploy the webhook server using the pre-configured deployment definition.

**File: `/root/webhook-deployment.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webhook-server
  namespace: webhook-demo
  labels:
    app: webhook-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webhook-server
  template:
    metadata:
      labels:
        app: webhook-server
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1234
      containers:
        - name: server
          image: stackrox/admission-controller-webhook-demo:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              name: webhook-api
          volumeMounts:
            - name: webhook-tls-certs
              mountPath: /run/secrets/tls
              readOnly: true
      volumes:
        - name: webhook-tls-certs
          secret:
            secretName: webhook-server-tls
```

**Deploy the webhook:**

```bash
kubectl apply -f /root/webhook-deployment.yaml
```

**Explanation**: This deployment runs the webhook server that will intercept and mutate pod creation requests. Notice it follows its own security best practices by running as a non-root user.

### Step 3: Create Webhook Service

Create a service to expose the webhook server to the Kubernetes API server.

**File: `/root/webhook-service.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webhook-server
  namespace: webhook-demo
spec:
  selector:
    app: webhook-server
  ports:
    - port: 443
      targetPort: webhook-api
```

**Deploy the service:**

```bash
kubectl apply -f /root/webhook-service.yaml
```

**Explanation**: This service exposes the webhook server on port 443 (HTTPS) and routes traffic to the webhook-api port (8443) on the pod.

### Step 4: Configure MutatingWebhookConfiguration

**File: `/root/webhook-configuration.yaml`**

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: demo-webhook
webhooks:
  - name: webhook-server.webhook-demo.svc
    clientConfig:
      service:
        name: webhook-server
        namespace: webhook-demo
        path: "/mutate"
      caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURQekNDQWllZ0F3SUJBZ0lVZGNaQmxxbHFtZ3JqTUt2RTIvY1Z5WEUraFpBd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0x6RXRNQ3NHQTFVRUF3d2tRV1J0YVhOemFXOXVJRU52Ym5SeWIyeHNaWElnVjJWaWFHOXZheUJFWlcxdgpJRU5CTUI0WERUSTFNRFV5TlRFM01qRTFOVm9YRFRJMU1EWXlOREUzTWpFMU5Wb3dMekV0TUNzR0ExVUVBd3drClFXUnRhWE56YVc5dUlFTnZiblJ5YjJ4c1pYSWdWMlZpYUc5dmF5QkVaVzF2SUVOQk1JSUJJakFOQmdrcWhraUcKOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXhuRlQrTUt0ekZaeHZUREZhem9JSHAxY0ZSU29lZzlmOVZoLwpwR0pLTnloeVRpc3B5alMzY1RFRmlXY2tSYWFxdnE0UEIyY1QzZXpNZ1R5YjNQeGtKbHFpdHJyTzA4aDVpbTkwCjJoR3VpNjM3NVhHaTNKSzlCTXEzQXdVRTNVdTV3RitTR0xEam9tcHhzM085WXhOOERnc1d6R3A2M2dOY2RQWVMKWTN0bGVacWVHSVJwOTZOd1QzT20xTG5MTU9hdytOL3EwdS9VamJ1MGdRSi8yWnFHN2RESGpja1Z6bmZ6OXlHRwp6aFhRWkZJZWFkL3NiMTRnMUt5U0tuRFNvNVpBbUJmMTA1cjJIZWtFWFoyalVTb3dDamFtNHJWMG85YmxRaFBGClB4ZmZJcktGbVJ3ODRmZm4wRTRjS3RhTHNGbFp0U2Qvb1FZSGlEdlM1ak90RTNWTVV3SURBUUFCbzFNd1VUQWQKQmdOVkhRNEVGZ1FVbjR3Q0hyeGZwRlo2Z3g2VzdMNitpVHpTVURjd0h3WURWUjBqQkJnd0ZvQVVuNHdDSHJ4ZgpwRlo2Z3g2VzdMNitpVHpTVURjd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTkJna3Foa2lHOXcwQkFRc0ZBQU9DCkFRRUFjVG9aSi9PTkdMTk9qalpFNzluOEt6ZG5OL0NKQlNMZTRId2ZWZlQ3dTc2S29GN0lpa1dIalNNR0FlQ1QKVDZZdGdyQ0ZnU2dDVjRTSlFEU0JzRG4vczdvUGZ4RWljcGVISDFtS3FjQkxZS1pZaTB5NXJnV0RHMUYwZ1ljawpkNXQ3TGtHWUVVWmtiRGJEOHVpYUlxWnQ2ZnVCNmxucDRNYjcxOG1vVVhabVpqaFllUzhvZTNFcUNDQnREc3dZClg1S0R4UlZWUnpYZEJ4bmkyUG5GV2VUOW5iaVFiN0Yrb2FzM2w4ckdqdXlYUkQ1UU5Ba0lyNUFpOVNCMS9hRUsKb3hTVjJ1ZjZuOFA2OVB1ZGVRREdXeW42b2ZJanNucEhjWkZmdVNLTE1sdURJWXJIdHJZdkxJd1U0UkxIMzlzUgpNYmFxbkhhUnprSmZSY1JuYnhmOHNxZ2NtUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    rules:
      - operations: ["CREATE"]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
    admissionReviewVersions: ["v1beta1"]
    sideEffects: None
```

**Q: If we apply this configuration which resource and actions it will affect?**

**A: This configuration will affect CREATE operations on pods (v1) in all namespaces.**

**Deploy the webhook configuration:**

```bash
kubectl apply -f /root/webhook-configuration.yaml
```

**Explanation**: This configuration registers the webhook with the Kubernetes API server. It specifies:

- Which operations to intercept (CREATE)
- Which resources to monitor (pods)
- Where to send the admission review requests (webhook-server service)
- The CA bundle for TLS verification

## Testing Scenarios

### Test 1: Pod with No SecurityContext

Deploy a pod with no securityContext specified.

**File: `/root/pod-with-defaults.yaml`**

```yaml
# A pod with no securityContext specified.
# Without the webhook, it would run as user root (0). The webhook mutates it
# to run as the non-root user with uid 1234.
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-defaults
  labels:
    app: pod-with-defaults
spec:
  restartPolicy: OnFailure
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]
```

**Deploy the pod:**

```bash
kubectl apply -f /root/pod-with-defaults.yaml
```

**Verify the mutation:**

```bash
kubectl get po pod-with-defaults -o yaml | grep -A2 " securityContext:"
```

**Expected Output:**

```
  securityContext:
    runAsNonRoot: true
    runAsUser: 1234
```

**Explanation**: The webhook automatically added security context with safe defaults, preventing the pod from running as root.

### Test 2: Pod Explicitly Allowed to Run as Root

Deploy pod with a securityContext explicitly allowing it to run as root.

**File: `/root/pod-with-override.yaml`**

```yaml
# A pod with a securityContext explicitly allowing it to run as root.
# The effect of deploying this with and without the webhook is the same. The
# explicit setting however prevents the webhook from applying more secure
# defaults.
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-override
  labels:
    app: pod-with-override
spec:
  restartPolicy: OnFailure
  securityContext:
    runAsNonRoot: false
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]
```

**Deploy the pod:**

```bash
kubectl apply -f /root/pod-with-override.yaml
```

**Explanation**: This pod is allowed to create because it explicitly sets `runAsNonRoot: false`, indicating the administrator's intent to run as root.

### Test 3: Pod with Conflicting SecurityContext

Deploy a pod with conflicting security settings.

**File: `/root/pod-with-conflict.yaml`**

```yaml
# A pod with a conflicting securityContext setting: it has to run as a non-root
# user, but we explicitly request a user id of 0 (root).
# Without the webhook, the pod could be created, but would be unable to launch
# due to an unenforceable security context leading to it being stuck in a
# 'CreateContainerConfigError' status. With the webhook, the creation of
# the pod is outright rejected.
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-conflict
  labels:
    app: pod-with-conflict
spec:
  restartPolicy: OnFailure
  securityContext:
    runAsNonRoot: true
    runAsUser: 0
  containers:
    - name: busybox
      image: busybox
```

**Deploy the pod (this will fail):**

```bash
kubectl apply -f /root/pod-with-conflict.yaml
```

**Expected Output:**

```
Error from server: error when creating "pod-with-conflict.yaml": admission webhook "webhook-server.webhook-demo.svc" denied the request: runAsNonRoot specified, but runAsUser set to 0 (the root user)
```

**Explanation**: The webhook rejects this pod because it has conflicting settings - it requires non-root execution but specifies root user ID (0).

## How It Works

1. **Interception**: When a pod creation request is made, the Kubernetes API server checks registered admission webhooks
2. **Mutation**: The webhook examines the pod spec and applies security defaults if none are specified
3. **Validation**: The webhook validates that security settings are not conflicting
4. **Response**: The webhook either:
   - Allows the request (possibly with mutations)
   - Denies the request with an error message

## Why This Approach Solves the Problem

This webhook-based approach provides several benefits:

1. **Security by Default**: Pods without explicit security contexts automatically get safe defaults
2. **Prevents Misconfigurations**: Conflicting settings are caught early, preventing pods from being stuck in error states
3. **Flexible Override**: Administrators can still explicitly allow root execution when necessary
4. **Centralized Policy**: Security policies are enforced cluster-wide without modifying individual pod definitions
5. **Early Validation**: Problems are caught at creation time rather than at runtime

## Troubleshooting

- **Webhook not responding**: Check if the webhook-server pod is running in the webhook-demo namespace
- **TLS errors**: Verify the CA bundle in the webhook configuration matches the certificate
- **Pods creating without mutation**: Ensure the MutatingWebhookConfiguration is applied correctly
- **Service communication**: Verify the webhook service is accessible from the API server

## Cleanup

To remove all components:

```bash
kubectl delete -f /root/webhook-configuration.yaml
kubectl delete -f /root/webhook-service.yaml
kubectl delete -f /root/webhook-deployment.yaml
kubectl -n webhook-demo delete secret webhook-server-tls
kubectl delete pod pod-with-defaults pod-with-override pod-with-conflict --ignore-not-found=true
```

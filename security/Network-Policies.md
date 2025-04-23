# Kubernetes Network Policy Guide

## Environment Inspection

First, let's check the pods in our environment:

```bash
k get pods
```

Then, check for services:

```bash
k get svc
```

Now, let's check for network policies:

```bash
k get netpol
```

Output:

```
NAME             POD-SELECTOR    AGE
payroll-policy   name=payroll    3m55s
```

## Network Policy Analysis

### Which pod is the Network Policy applied on?

From the output above, we can see the policy is applied on: `name=payroll`

### What type of traffic is this Network Policy configured to handle?

Let's examine the network policy details:

```bash
k describe netpol payroll-policy
```

Output:

```
Name: payroll-policy
Namespace: default
Created on: 2025-04-23 17:49:59 +0000 UTC
Labels: <none>
Annotations: <none>
Spec:
  PodSelector: name=payroll
  Allowing ingress traffic:
    To Port: 8080/TCP
    From:
      PodSelector: name=internal
  Not affecting egress traffic
```

### What is the impact of the rule configured on this Network Policy?

The rule states: `PodSelector: name=payroll` with `Allowing ingress traffic: To Port: 8080/TCP`.

This means traffic from internal pod to payroll pod is allowed. Specifically, the internal pod can access the payroll pod on port 8080.

## Connectivity Tests

### Testing Internal Application to Payroll-Service

Only the internal application can access the payroll application.

### Testing Internal Application to External-Service

The connection is successful because there is no network policy enforcing restrictions between external and internal pods.

## Creating a New Network Policy

Let's create a network policy to allow traffic from the Internal application only to the payroll-service and db-service.

### Requirements:

- Policy Name: internal-policy
- Policy Type: Egress
- Egress Allow: payroll (Port: 8080)
- Egress Allow: mysql (Port: 3306)
- Allow DNS resolution

### Implementation:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              name: payroll
      ports:
        - protocol: TCP
          port: 8080
    - to:
        - podSelector:
            matchLabels:
              name: mysql
      ports:
        - protocol: TCP
          port: 3306
    # Allow DNS resolution
    - ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

Apply the network policy:

```bash
kubectl apply -f internal-policy.yaml
```

> Note: Don't forget to add the DNS ports (TCP and UDP port 53) to allow DNS resolution from the internal pod.

# Kubernetes Kustomize Configuration Guide

This README explains the Kubernetes configuration using Kustomize to manage different deployment overlays (community, dev, enterprise) and components (auth, db, logging, caching).

## Original Problem

The original task required understanding the current Kustomize configuration and making the following changes:

1. Add the logging component to the community overlay
2. Create a new caching component with required configurations
3. Update the caching component to add environment variables for Redis connection
4. Add the caching component to the enterprise edition

## Project Structure

```
.
├── base/
├── components/
│   ├── auth/
│   ├── db/
│   │   ├── api-patch.yaml
│   │   ├── db-deployment.yaml
│   │   ├── db-service.yaml
│   │   └── kustomization.yaml
│   ├── logging/
│   └── caching/
│       ├── redis-depl.yaml
│       ├── redis-service.yaml
│       ├── api-patch.yaml (to be created)
│       └── kustomization.yaml (to be created)
└── overlays/
    ├── community/
    │   └── kustomization.yaml
    ├── dev/
    │   └── kustomization.yaml
    └── enterprise/
        └── kustomization.yaml
```

## Components Analysis

### Community Overlay (Before Changes)

```yaml
bases:
  - ../../base
components:
  - ../../components/auth
```

### Dev Overlay

```yaml
bases:
  - ../../base
components:
  - ../../components/auth
  - ../../components/db
  - ../../components/logging
```

### DB Component Details

The db component adds 2 environment variables to the api-deployment:

- `DB_CONNECTION`
- `DB_PASSWORD`

**api-patch.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - name: api
          env:
            - name: DB_CONNECTION
              value: postgres-service
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-creds
                  key: password
```

**kustomization.yaml:**

```yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

resources:
  - db-deployment.yaml
  - db-service.yaml

secretGenerator:
  - name: db-creds
    literals:
      - password=password1

patches:
  - path: api-patch.yaml
```

## Required Changes and Implementation

### 1. Adding Logging Component to Community Overlay

**Updated overlays/community/kustomization.yaml:**

```yaml
bases:
  - ../../base
components:
  - ../../components/auth
  - ../../components/logging
```

### 2. Creating Caching Component

**New components/caching/kustomization.yaml:**

```yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

resources:
  - redis-depl.yaml
  - redis-service.yaml

patches:
  - path: api-patch.yaml
```

### 3. Adding Redis Connection Environment Variable

**New components/caching/api-patch.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - name: api
          env:
            - name: REDIS_CONNECTION
              value: redis-service
```

### 4. Adding Caching Component to Enterprise Edition

**Updated overlays/enterprise/kustomization.yaml:**

```yaml
bases:
  - ../../base
components:
  - ../../components/auth
  - ../../components/db
  - ../../components/caching
```

## Deployment Instructions

### Viewing Component Structure

To see the components that make up each overlay:

```bash
# Examine community overlay structure
kustomize build overlays/community --dry-run

# Examine dev overlay structure
kustomize build overlays/dev --dry-run

# Examine enterprise overlay structure
kustomize build overlays/enterprise --dry-run
```

### Applying Configurations

To apply the configurations to your Kubernetes cluster:

```bash
# Apply community overlay
kubectl apply -k overlays/community

# Apply dev overlay
kubectl apply -k overlays/dev

# Apply enterprise overlay
kubectl apply -k overlays/enterprise
```

### Expected Output

When applying the community overlay, you should see:

```
deployment.apps/api-deployment configured
secret/db-creds-<hash> created
service/auth-service created
service/logging-service created
```

When applying the enterprise overlay, you should see:

```
deployment.apps/api-deployment configured
secret/db-creds-<hash> created
service/auth-service created
service/postgres-service created
service/redis-service created
deployment.apps/postgres-deployment created
deployment.apps/redis-deployment created
```

## Understanding Kustomize Components

### What is Kustomize?

Kustomize is a Kubernetes configuration management tool that allows you to customize application configurations without duplicating resource YAML files. It uses:

- **Bases**: Foundation configurations that are shared across multiple environments
- **Overlays**: Environment-specific configurations that extend bases
- **Components**: Reusable configuration units that can be included in overlays

### Benefits of Components

Components in Kustomize provide:

1. **Modularity**: Easily add or remove features like auth, db, or caching
2. **Reusability**: Use the same component across different overlays
3. **Separation of concerns**: Each component manages its own resources and patches

## How This Solution Solves the Original Problem

This approach solves the original problem by:

1. **Maintaining a DRY (Don't Repeat Yourself) codebase**: Common configurations are defined once in base and components
2. **Enabling feature selection by environment**: Different overlays can include different components
3. **Clear organization**: Each component is responsible for its own resources and patches
4. **Separation of concerns**: The caching component contains all Redis-related configurations and its own patch for the API deployment
5. **Scalability**: New components can be easily added without modifying existing ones

By using Kustomize components and overlays, the configuration becomes more maintainable, modular, and easier to understand, especially as the application grows in complexity.

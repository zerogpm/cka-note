# Kubernetes Kustomization Configuration

This README explains the implementation of Kubernetes Kustomization for managing resources across multiple services.

## Problem Statement

The original task was to:

1. Create a single `kustomization.yaml` file in the root of the k8s directory and import all resources defined for db, message-broker, nginx into it.
2. Apply the configuration after creating the kustomization.yaml file.
3. Create individual kustomization.yaml files in each subdirectory and import only the resources within that directory.
4. Update the root kustomization.yaml file to reference these subdirectories.
5. Deploy all resources.

## Directory Structure

```
└── k8s
    ├── db
    │   ├── db-config.yaml
    │   ├── db-depl.yaml
    │   ├── db-service.yaml
    │   └── kustomization.yaml
    ├── kustomization.yaml
    ├── message-broker
    │   ├── rabbitmq-config.yaml
    │   ├── rabbitmq-depl.yaml
    │   ├── rabbitmq-service.yaml
    │   └── kustomization.yaml
    └── nginx
        ├── nginx-depl.yaml
        ├── nginx-service.yaml
        └── kustomization.yaml
```

## Implementation Steps

### 1. Initial Root Kustomization File

The initial approach was to create a single kustomization.yaml file in the root directory that directly referenced all individual resource files:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - db/db-config.yaml
  - db/db-depl.yaml
  - db/db-service.yaml
  - message-broker/rabbitmq-config.yaml
  - message-broker/rabbitmq-depl.yaml
  - message-broker/rabbitmq-service.yaml
  - nginx/nginx-depl.yaml
  - nginx/nginx-service.yaml
```

This configuration allows applying all resources at once with:

```bash
controlplane ~/code/k8s ➜  kubectl apply -k .
```

OR

```bash
controlplane ~/code ➜  kustomize build k8s/ | kubectl apply -f -
```

### 2. Hierarchical Kustomization Structure

The improved approach creates a hierarchical structure with kustomization files in each subdirectory:

#### DB Kustomization (`k8s/db/kustomization.yaml`):

```yaml
resources:
  - db-depl.yaml
  - db-service.yaml
  - db-config.yaml
```

#### Message-broker Kustomization (`k8s/message-broker/kustomization.yaml`):

```yaml
resources:
  - rabbitmq-config.yaml
  - rabbitmq-depl.yaml
  - rabbitmq-service.yaml
```

#### Nginx Kustomization (`k8s/nginx/kustomization.yaml`):

```yaml
resources:
  - nginx-depl.yaml
  - nginx-service.yaml
```

#### Root Kustomization (`k8s/kustomization.yaml`):

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
  - db/
  - message-broker/
  - nginx/
```

### 3. Applying the Configuration

After creating all kustomization files, the configuration is applied using:

```bash
controlplane ~/code ➜ kubectl apply -k /root/code/k8s/
```

OR

```bash
controlplane ~/code ➜ kustomize build /root/code/k8s/ | kubectl apply -f -
```

## Command Explanation

1. `kubectl apply -k .` - Applies all resources defined in the kustomization.yaml file in the current directory.

2. `kustomize build k8s/ | kubectl apply -f -` - First builds the kustomization into a unified YAML file and then pipes that output to kubectl for application.

3. `kubectl apply -k /root/code/k8s/` - Applies all resources defined in the specified directory's kustomization.yaml file.

## Why This Solution Works

The hierarchical kustomization approach provides several advantages:

1. **Modularity**: Each service (db, message-broker, nginx) has its own kustomization file, making it easy to manage resources separately.

2. **Maintainability**: Adding or removing resources for a specific service only requires changes to that service's kustomization file.

3. **Scalability**: New services can be added by creating a new directory with its own kustomization file and adding a reference in the root kustomization file.

4. **Organization**: Resources are logically grouped by service, improving readability and maintenance.

5. **Deployment Flexibility**: The hierarchical structure allows deploying individual services or all services together.

## Expected Console Output

When applying the kustomization with `kubectl apply -k /root/code/k8s/`, you should see output similar to:

```
configmap/db-config created
service/db-service created
deployment.apps/db-deployment created
configmap/rabbitmq-config created
service/rabbitmq-service created
deployment.apps/rabbitmq-deployment created
service/nginx-service created
deployment.apps/nginx-deployment created
```

This indicates that all resources defined in the kustomization files have been successfully applied to the Kubernetes cluster.

## Additional Notes

- The kustomization.yaml files can be extended with additional features like patches, common labels, and namespace definitions.
- For more complex environments, you might want to add environment-specific overlays (e.g., dev, staging, production).
- Kustomize is integrated into kubectl since version 1.14, making it easy to use without installing additional tools.

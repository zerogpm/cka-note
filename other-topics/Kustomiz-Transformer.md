# Kubernetes Kustomize Project

This README explains a Kubernetes project that uses Kustomize for configuration management. The project demonstrates various Kustomize transformers including `commonLabels`, `namePrefix`, `namespace`, `commonAnnotations`, and image transformations.

## Original Questions/Problems

The following questions were posed about a Kubernetes kustomize project:

1. What is the label that will get assigned to every Kubernetes resource within /root/code/k8s/ project?
2. What is the name that will be prefixed before all database resources?
3. What is the namespace that all the monitoring resources will be deployed to?
4. Assign the following annotation to all nginx and monitoring resources: owner: bob@gmail.com
5. Transform all postgres images in the project to mysql.
6. Transform all nginx images in the nginx directory to nginx:1.23.

## Project Structure

```
/root/code/k8s/
├── kustomization.yaml          # Root kustomization file
├── db/
│   ├── kustomization.yaml      # Database kustomization file
│   ├── NoSql/                  # NoSQL database configurations
│   ├── Sql/                    # SQL database configurations
│   └── db-config.yaml          # Database configuration
├── monitoring/
│   ├── kustomization.yaml      # Monitoring kustomization file
│   ├── grafana-depl.yaml       # Grafana deployment
│   └── grafana-service.yaml    # Grafana service
└── nginx/
    ├── kustomization.yaml      # Nginx kustomization file
    ├── nginx-depl.yaml         # Nginx deployment
    └── nginx-service.yaml      # Nginx service
```

## Original Questions and Solutions

### Question 1: What is the label that will get assigned to every Kubernetes resource within /root/code/k8s/ project?

The label `sandbox: dev` will be assigned to every Kubernetes resource in the project.

**Solution**: A `commonLabels` transformer is applied on the root kustomization.yaml file.

```yaml
# k8s/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - db/
  - monitoring/
  - nginx/

commonLabels:
  sandbox: dev
```

### Question 2: What is the name that will be prefixed before all database resources?

The prefix `data-` will be added to all database resources, including both SQL and NoSQL configs.

**Solution**: A `namePrefix` transformer is applied in the db directory's kustomization.yaml file.

```yaml
# k8s/db/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - NoSql/
  - Sql/
  - db-config.yaml

namePrefix: data-
```

### Question 3: What is the namespace that all the monitoring resources will be deployed to?

All monitoring resources will be deployed to the `logging` namespace.

**Solution**: A `namespace` transformer is set in the monitoring directory's kustomization.yaml file.

```yaml
# k8s/monitoring/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - grafana-depl.yaml
  - grafana-service.yaml

namespace: logging
```

### Question 4: Assign the following annotation to all nginx and monitoring resources: owner: bob@gmail.com

**Solution**: `commonAnnotations` are added to both nginx and monitoring kustomization.yaml files:

```yaml
# k8s/nginx/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - nginx-depl.yaml
  - nginx-service.yaml

commonAnnotations:
  owner: bob@gmail.com
```

```yaml
# k8s/monitoring/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - grafana-depl.yaml
  - grafana-service.yaml

namespace: logging

commonAnnotations:
  owner: bob@gmail.com
```

### Question 5: Transform all postgres images in the project to mysql

**Solution**: An `images` transformer is added to the root kustomization.yaml file.

```yaml
# k8s/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - db/
  - monitoring/
  - nginx/

commonLabels:
  sandbox: dev

images:
  - name: postgres
    newName: mysql
```

### Question 6: Transform all nginx images in the nginx directory to nginx:1.23

**Solution**: An `images` transformer is added to the nginx directory's kustomization.yaml file with the `newTag` property.

```yaml
# k8s/nginx/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - nginx-depl.yaml
  - nginx-service.yaml

commonAnnotations:
  owner: bob@gmail.com

images:
  - name: nginx
    newTag: "1.23"
```

## Bash Commands and Step-by-Step Explanation

Here are the commands you would use to work with this Kustomize project:

### 1. Preview the combined resources

```bash
cd /root/code/k8s
kubectl kustomize ./
```

**Step-by-Step Explanation:**

1. Navigate to the root directory of the Kustomize project
2. Run `kubectl kustomize ./` to preview the final YAML that would be applied to the cluster
3. This command processes all kustomization files and applies transformers in the correct order
4. The output shows how resources will look after all transformations but doesn't apply them

**Expected Console Output:**

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    owner: bob@gmail.com
  labels:
    sandbox: dev
  name: data-mysql-service
  namespace: default
spec:
  ports:
    - port: 3306
      targetPort: 3306
  selector:
    app: mysql
    sandbox: dev
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    owner: bob@gmail.com
  labels:
    sandbox: dev
  name: grafana-service
  namespace: logging
spec:
  ports:
    - port: 3000
      targetPort: 3000
  selector:
    app: grafana
    sandbox: dev
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    owner: bob@gmail.com
  labels:
    sandbox: dev
  name: nginx-service
  namespace: default
spec:
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: nginx
    sandbox: dev
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    sandbox: dev
  name: data-mysql-deployment
  namespace: default
spec:
  selector:
    matchLabels:
      app: mysql
      sandbox: dev
  template:
    metadata:
      labels:
        app: mysql
        sandbox: dev
    spec:
      containers:
        - image: mysql
          name: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    owner: bob@gmail.com
  labels:
    sandbox: dev
  name: grafana-deployment
  namespace: logging
spec:
  selector:
    matchLabels:
      app: grafana
      sandbox: dev
  template:
    metadata:
      labels:
        app: grafana
        sandbox: dev
    spec:
      containers:
        - image: grafana
          name: grafana
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    owner: bob@gmail.com
  labels:
    sandbox: dev
  name: nginx-deployment
  namespace: default
spec:
  selector:
    matchLabels:
      app: nginx
      sandbox: dev
  template:
    metadata:
      labels:
        app: nginx
        sandbox: dev
    spec:
      containers:
        - image: nginx:1.23
          name: nginx
```

### 2. Apply the kustomized configuration to the cluster

```bash
cd /root/code/k8s
kubectl apply -k ./
```

**Step-by-Step Explanation:**

1. Navigate to the root directory of the Kustomize project
2. Run `kubectl apply -k ./` to apply all resources with transformations to the cluster
3. The `-k` flag tells kubectl to process kustomization files
4. Kubectl builds the final YAML by applying all transformers and then applies it to the cluster

**Expected Console Output:**

```
service/data-mysql-service created
service/grafana-service created
service/nginx-service created
deployment.apps/data-mysql-deployment created
deployment.apps/grafana-deployment created
deployment.apps/nginx-deployment created
```

### 3. Verify the applied resources

```bash
# Check all resources with labels
kubectl get all --show-labels -A | grep sandbox=dev

# Check that monitoring resources are in the logging namespace
kubectl get all -n logging

# Check that database resources have the data- prefix
kubectl get deployments,services | grep data-

# Verify annotations on nginx resources
kubectl get deployment nginx-deployment -o jsonpath='{.metadata.annotations}'

# Verify mysql image is used instead of postgres
kubectl get deployment data-mysql-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'

# Verify nginx image has the 1.23 tag
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

**Step-by-Step Explanation:**

1. The first command filters all resources to show only those with our `sandbox: dev` label
2. The second command lists all resources in the `logging` namespace (monitoring resources)
3. The third command filters resources to show only those with the `data-` prefix (database resources)
4. The fourth command checks the annotations on the nginx deployment
5. The fifth command verifies the mysql image is used
6. The sixth command verifies the nginx image has the correct tag

**Expected Console Output for the first command:**

```
default     deployment.apps/data-mysql-deployment   1/1     1            1           10m   app=mysql,sandbox=dev
default     service/data-mysql-service              ClusterIP   10.96.x.x    <none>        3306/TCP         10m   app=mysql,sandbox=dev
default     deployment.apps/nginx-deployment        1/1     1            1           10m   app=nginx,sandbox=dev
default     service/nginx-service                   ClusterIP   10.96.x.x    <none>        80/TCP           10m   app=nginx,sandbox=dev
logging     deployment.apps/grafana-deployment      1/1     1            1           10m   app=grafana,sandbox=dev
logging     service/grafana-service                 ClusterIP   10.96.x.x    <none>        3000/TCP         10m   app=grafana,sandbox=dev
```

**Expected Console Output for the fourth command:**

```
{"owner":"bob@gmail.com"}
```

**Expected Console Output for the fifth command:**

```
mysql
```

**Expected Console Output for the sixth command:**

```
nginx:1.23
```

### 4. Clean up the resources

```bash
cd /root/code/k8s
kubectl delete -k ./
```

**Step-by-Step Explanation:**

1. Navigate to the root directory of the Kustomize project
2. Run `kubectl delete -k ./` to remove all resources that were created

**Expected Console Output:**

```
service "data-mysql-service" deleted
service "grafana-service" deleted
service "nginx-service" deleted
deployment.apps "data-mysql-deployment" deleted
deployment.apps "grafana-deployment" deleted
deployment.apps "nginx-deployment" deleted
```

## Why This Approach Solves the Original Problem

Kustomize is a powerful tool for customizing Kubernetes configurations without modifying the original files. Here's why the implemented approach effectively solves the original problem:

### 1. Hierarchical Configuration Management

The project structure follows a hierarchical approach where:

- Root level (`/root/code/k8s/kustomization.yaml`) manages global configurations like common labels and global image transformations
- Component-specific directories (`db/`, `nginx/`, `monitoring/`) manage configurations that only apply to their respective components

This hierarchy ensures that:

- Common configurations are defined once and applied consistently (DRY principle)
- Specific configurations are isolated to relevant components
- Changes at different levels don't interfere with each other

### 2. Declarative Configuration

Each requirement is implemented declaratively using Kustomize transformers:

- `commonLabels` - Adds labels to all resources and updates label selectors automatically
- `namePrefix` - Adds prefixes to resource names without modifying the original YAML
- `namespace` - Sets the namespace for resources without duplicating configurations
- `commonAnnotations` - Adds annotations to resources without modifying the original YAML
- `images` - Updates container images and tags without modifying deployment files

This declarative approach:

- Makes intentions clear and explicit
- Reduces manual errors
- Keeps original configurations intact
- Enables version control of transformations

### 3. Separation of Concerns

Each requirement is implemented at the appropriate level:

- Global labels at the root level affect all resources
- Database prefixing at the database component level
- Namespace settings at the monitoring component level
- Annotations at the component levels (nginx and monitoring)
- Image transformations at the appropriate levels (root for global postgres→mysql, component for nginx versioning)

This separation:

- Prevents conflicts between transformations
- Makes it easy to understand what changes apply where
- Allows for independent management of components

### 4. Advantages Over Alternative Approaches

Compared to alternatives, this approach:

- **Versus manual YAML editing**: Avoids duplicating and manually editing every YAML file, which would be error-prone and difficult to maintain
- **Versus Helm**: More lightweight for simple transformations and doesn't require templates
- **Versus environment variables**: More declarative and doesn't require modifying application code
- **Versus multiple YAML files per environment**: More maintainable as base configurations are kept intact

### 5. Validation and Verification

The provided commands allow for:

- Previewing changes before applying them
- Verifying that transformations are applied correctly
- Easy rollback if needed

This validation process ensures that the implemented solution meets all requirements before affecting the actual Kubernetes cluster.

## Conclusion

Kustomize provides an elegant solution to the configuration management requirements specified in the original problem. By using various transformers at appropriate levels in the configuration hierarchy, the solution ensures that:

1. All resources receive the `sandbox: dev` label
2. All database resources are prefixed with `data-`
3. All monitoring resources are deployed to the `logging` namespace
4. All nginx and monitoring resources have the `owner: bob@gmail.com` annotation
5. All postgres images are transformed to mysql
6. All nginx images are tagged with version 1.23

This approach is maintainable, scalable, and follows Kubernetes best practices for configuration management.

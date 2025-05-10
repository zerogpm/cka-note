# Helm Chart Operations Guide

This README provides a comprehensive guide to working with Helm charts, including searching for packages, managing repositories, and deploying applications on Kubernetes clusters.

## Table of Contents

- [Searching for Charts on Artifact Hub](#searching-for-charts-on-artifact-hub)
- [Managing Helm Repositories](#managing-helm-repositories)
- [Deploying Applications](#deploying-applications)
- [Managing Releases](#managing-releases)
- [Cleanup Operations](#cleanup-operations)

## Searching for Charts on Artifact Hub

### Problem 1: Search for WordPress Helm Chart

**Question:** Which command is used to search for a wordpress helm chart package from the Artifact Hub?

**Solution:**

```bash
helm search hub wordpress
```

**Explanation:** The `helm search hub` command searches for packages in the Artifact Hub, which is a centralized repository for Kubernetes packages including Helm charts, operators, and plugins.

### Problem 2: Search for Consul Helm Chart

**Question:** Search for a consul helm chart package from the Artifact Hub and identify the APP VERSION for the Official HashiCorp Consul Chart.

**Solution:**

```bash
helm search hub consul | grep -i consul
```

**Console Output:**

```bash
controlplane ~ ➜  helm search hub consul | grep -i consul

https://artifacthub.io/packages/helm/hashicorp/...      1.7.0           1.21.0          Official HashiCorp Consul Chart
https://artifacthub.io/packages/helm/warjiang/c...      1.3.0           1.17.0          Official HashiCorp Consul Chart
https://artifacthub.io/packages/helm/bitnami-ak...      10.9.2          1.13.2          HashiCorp Consul is a tool for discovering and ...
https://artifacthub.io/packages/helm/bitnami/co...      11.4.17         1.21.0          HashiCorp Consul is a tool for discovering and ...
https://artifacthub.io/packages/helm/wener/consul       1.7.0           1.21.0          Official HashiCorp Consul Chart
https://artifacthub.io/packages/helm/smo-helm-c...      6.0.0                           ONAP Consul Agent
https://artifacthub.io/packages/helm/wenerme/co...      1.7.0           1.21.0          Official HashiCorp Consul Chart
https://artifacthub.io/packages/helm/intel/consul       0.8.1           1.14.2          A Helm chart for Kubernetes
https://artifacthub.io/packages/helm/zahori/zah...      1.0.1           1.1.2           Consul of Zahori
https://artifacthub.io/packages/helm/prometheus...      1.0.0           0.4.0           A Helm chart for the Prometheus Consul Exporter
https://artifacthub.io/packages/helm/cloudnativ...      0.1.3           0.4.0           A Helm chart for the Prometheus Consul Exporter
https://artifacthub.io/packages/helm/prometheus...      0.4.0           0.4.0           A Helm chart for the Prometheus Consul Exporter
https://artifacthub.io/packages/helm/nativechat...      0.5.1           0.5.0           A Helm chart for the consul-merge-controller
https://artifacthub.io/packages/helm/kubegemsap...      0.5.0           0.4.0           A Helm chart for the Prometheus Consul Exporter
https://artifacthub.io/packages/helm/mintel/hyb...      0.0.3                           A Helm chart for defining Consul CRDs
https://artifacthub.io/packages/helm/consul-hel...      1.1.0           1.1.0           A Helm chart for Deploying the Percona PostgreS...
```

**Answer:** The APP VERSION for the Official HashiCorp Consul Chart is **1.21.0**

**Explanation:**

- The command searches for all consul-related packages on Artifact Hub
- Using `grep -i consul` filters the results to show only consul-related entries
- The output shows multiple versions of Consul charts from different providers
- The Official HashiCorp Consul Chart shows APP VERSION as 1.21.0

## Managing Helm Repositories

### Problem 3: Add Bitnami Repository

**Question:** Add bitnami helm chart repository in the controlplane node. The url for bitnami repository is https://charts.bitnami.com/bitnami

**Solution:**

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

**Explanation:**

- `helm repo add` adds a new chart repository to your local Helm configuration
- First parameter is the repository name (bitnami)
- Second parameter is the repository URL

### Problem 4: Search in Repository

**Question:** Which command is used to search for the wordpress package from the newly added bitnami repository?

**Solution:**

```bash
helm search repo wordpress
```

**Explanation:**

- `helm search repo` searches within repositories that have been added locally
- Unlike `helm search hub`, this searches only in repositories you've explicitly added
- This is faster than searching the entire Artifact Hub

### Problem 5: List Repositories

**Question:** How many helm chart repositories are there in the controlplane node now? We have added a few helm chart repositories in the controlplane node now.

**Solution:**

```bash
helm repo list
```

**Console Output:**

```bash
controlplane ~ ➜  helm repo list
NAME            URL
bitnami         https://charts.bitnami.com/bitnami
puppet          https://puppetlabs.github.io/puppetserver-helm-chart
hashicorp       https://helm.releases.hashicorp.com
```

**Answer:** There are **3** helm chart repositories in the controlplane node

**Explanation:** The command lists all configured Helm repositories with their names and URLs.

## Deploying Applications

### Problem 6: Deploy Apache

**Question:** Deploy the Apache application on the cluster using the apache from the bitnami repository. Set the release Name to: amaze-surf

**Solution:**

```bash
helm install amaze-surf bitnami/apache
```

**Explanation:**

- `helm install` deploys a chart to your Kubernetes cluster
- `amaze-surf` is the release name (a unique identifier for this deployment)
- `bitnami/apache` specifies the chart to install (repository/chart format)

### Problem 7: Check Apache Version

**Question:** What version of apache did we just install on the cluster using the helm chart?

**Solution:**

```bash
helm list
```

**Console Output:**

```bash
controlplane ~ ➜  helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
amaze-surf      default         1               2025-05-10 18:15:12.515795889 +0000 UTC deployed        apache-11.3.2   2.4.63
```

**Answer:** Apache version **2.4.63** was installed

**Explanation:**

- The `helm list` command shows all deployed releases
- The APP VERSION column shows the actual application version (2.4.63)
- The CHART column shows the chart version (apache-11.3.2)

## Managing Releases

### Problem 8: Count Nginx Releases

**Question:** How many releases of nginx charts can you see installed in the cluster now? Note: We just installed some charts

**Solution:**

```bash
helm list
```

**Answer:** **2** releases of nginx charts

**Explanation:** By examining the output of `helm list`, you can count the number of nginx-related releases shown in the results.

### Problem 9: Uninstall Release

**Question:** Uninstall the nginx chart release happy-browse from the cluster.

**Solution:**

```bash
helm uninstall happy-browse
```

**Console Output:**

```bash
controlplane ~ ➜  helm uninstall happy-browse
release "happy-browse" uninstalled
```

**Explanation:**

- `helm uninstall` removes a deployed release from the cluster
- This deletes all Kubernetes resources associated with the release
- The release name is the only required parameter

## Cleanup Operations

### Problem 10: Remove Repository

**Question:** Remove the Hashicorp helm repository from the cluster.

**Solution:**

```bash
helm repo remove hashicorp
```

**Console Output:**

```bash
controlplane ~ ➜  helm repo remove hashicorp
"hashicorp" has been removed from your repositories
```

**Explanation:**

- `helm repo remove` deletes a repository from your local configuration
- This doesn't affect any charts already deployed from this repository
- Only removes the ability to search/install new charts from this repository

## Summary

This guide demonstrates the complete lifecycle of working with Helm charts:

1. **Discovery**: Searching for charts on Artifact Hub
2. **Configuration**: Adding and managing repositories
3. **Deployment**: Installing applications using charts
4. **Management**: Listing and monitoring deployed releases
5. **Cleanup**: Uninstalling releases and removing repositories

These operations provide the foundation for managing applications on Kubernetes using Helm, the package manager for Kubernetes.

## Key Commands Reference

| Command                          | Purpose                          |
| -------------------------------- | -------------------------------- |
| `helm search hub <keyword>`      | Search Artifact Hub for charts   |
| `helm repo add <name> <url>`     | Add a new repository             |
| `helm search repo <keyword>`     | Search in local repositories     |
| `helm repo list`                 | List all configured repositories |
| `helm install <release> <chart>` | Deploy a chart                   |
| `helm list`                      | List all releases                |
| `helm uninstall <release>`       | Remove a deployed release        |
| `helm repo remove <name>`        | Remove a repository              |

## Best Practices

1. Always specify a meaningful release name when installing charts
2. Use `helm list` to verify deployments and check versions
3. Keep your repository list clean by removing unused repositories
4. Use `helm search repo` for faster searches when possible
5. Check both CHART VERSION and APP VERSION when deploying applications

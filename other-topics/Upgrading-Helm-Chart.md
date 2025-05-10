# Helm Nginx Deployment Exercise

This exercise demonstrates how to manage nginx deployments using Helm charts from the Bitnami repository, including upgrading and rolling back versions.

## Prerequisites

- Kubernetes cluster
- Helm 3 installed
- kubectl configured to access the cluster

## Exercise Steps

### Step 1: Add Bitnami Helm Chart Repository

**Question:** Add bitnami helm chart repository to the cluster.  
**Note:** Make sure to add the bitnami chart to the cluster before moving to the next questions.

**Commands:**

```bash
controlplane ~ ✖ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

# Update the repository
helm repo update
```

**Explanation:**

- `helm repo add`: Adds a new Helm chart repository to your local Helm installation
- The Bitnami repository contains production-ready charts for many popular applications
- `helm repo update`: Updates the local cache of available charts from all added repositories

### Step 2: Check Existing nginx Releases

**Question:** How many releases of nginx can you see in the cluster now?

**Command:**

```bash
controlplane ~ ➜  helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
dazzling-web    default         3               2025-05-10 20:44:57.889465718 +0000 UTC deployed        nginx-12.0.4    1.22.0
```

**Answer:** 1 release (dazzling-web)

**Explanation:**

- `helm list`: Shows all deployed Helm releases in the cluster
- The output shows there's one nginx release named "dazzling-web"
- Current chart version is nginx-12.0.4 running nginx application version 1.22.0

### Step 3: Check Revision Count

**Question:** How many revisions of nginx exists in the cluster?

**Answer:** 3

**Explanation:**

- The REVISION column in the helm list output shows "3"
- This means the release has been updated 3 times since initial deployment

### Step 4: Identify Current nginx Version

**Question:** Which version of nginx is currently running in the cluster?

**Answer:** nginx-12.0.4

**Explanation:**

- The CHART column shows the Helm chart version (nginx-12.0.4)
- The APP VERSION column shows the actual nginx application version (1.22.0)

### Step 5: Upgrade nginx Version

**Question:** The DevOps team has decided to upgrade the nginx version to 1.27.x and use the Helm chart version 18.3.6 from the Bitnami repository. Ensure that the nginx version running in the cluster is 1.27.x.

**Command:**

```bash
helm upgrade dazzling-web bitnami/nginx --version 18.3.6
```

**Explanation:**

- `helm upgrade`: Updates an existing release to a new version
- `dazzling-web`: The name of the release to upgrade
- `bitnami/nginx`: The chart to use for the upgrade
- `--version 18.3.6`: Specifies the exact chart version to install

### Step 6: Verify Upgraded Version

**Question:** To which version is the nginx currently upgraded?

**Commands and Output:**

```bash
k get pods
NAME                                 READY   STATUS    RESTARTS   AGE
dazzling-web-nginx-8b4dd764c-4fn6b   1/1     Running   0          54s

controlplane ~ ➜  k describe pods dazzling-web-nginx-8b4dd764c-4fn6b
Name:             dazzling-web-nginx-8b4dd764c-4fn6b
Namespace:        default
Priority:         0
Service Account:  dazzling-web-nginx
Node:             controlplane/192.168.122.145
Start Time:       Sat, 10 May 2025 20:57:09 +0000
Labels:           app.kubernetes.io/instance=dazzling-web
                  app.kubernetes.io/managed-by=Helm
                  app.kubernetes.io/name=nginx
                  app.kubernetes.io/version=1.27.4
                  helm.sh/chart=nginx-18.3.6
                  pod-template-hash=8b4dd764c
Annotations:      cni.projectcalico.org/containerID: 1cf96c0f1ef3618410a1051bc13beeee7e64222f1ca95c3df0f5178534b9ec2f
                  cni.projectcalico.org/podIP: 172.17.0.8/32
                  cni.projectcalico.org/podIPs: 172.17.0.8/32
Status:           Running
IP:               172.17.0.8
IPs:
  IP:           172.17.0.8
Controlled By:  ReplicaSet/dazzling-web-nginx-8b4dd764c
Init Containers:
  preserve-logs-symlinks:
    Container ID:    containerd://620420720de3b3665aed946653026fc265e2dcf424a7d61bb0cb1b5faa75ff58
    Image:           docker.io/bitnami/nginx:1.27.2
    Image ID:        docker.io/bitnami/nginx@sha256:9a0550b139386d419d415f3fd2915e74f59a3738f84f95a196ffe111ec38803a
    Port:            <none>
    Host Port:       <none>
    SeccompProfile:  RuntimeDefault
    Command:
      /bin/bash
    Args:
      -ec
      #!/bin/bash
      . /opt/bitnami/scripts/libfs.sh
      # We copy the logs folder because it has symlinks to stdout and stderr
      if ! is_dir_empty /opt/bitnami/nginx/logs; then
        cp -r /opt/bitnami/nginx/logs /emptydir/app-logs-dir
      fi

    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 10 May 2025 20:57:10 +0000
      Finished:     Sat, 10 May 2025 20:57:10 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:                150m
      ephemeral-storage:  2Gi
      memory:             192Mi
    Requests:
      cpu:                100m
      ephemeral-storage:  50Mi
      memory:             128Mi
    Environment:          <none>
    Mounts:
      /emptydir from empty-dir (rw)
Containers:
  nginx:
    Container ID:    containerd://6fbbb6e9e7a1a9f66085ce8cfbf4ba48c9ae947a4c4fa12322b1e18063ffce11
    Image:           docker.io/bitnami/nginx:1.27.2
    Image ID:        docker.io/bitnami/nginx@sha256:9a0550b139386d419d415f3fd2915e74f59a3738f84f95a196ffe111ec38803a
    Ports:           8080/TCP, 8443/TCP
    Host Ports:      0/TCP, 0/TCP
    SeccompProfile:  RuntimeDefault
    State:           Running
      Started:       Sat, 10 May 2025 20:57:11 +0000
    Ready:           True
    Restart Count:   0
    Limits:
      cpu:                150m
      ephemeral-storage:  2Gi
      memory:             192Mi
```

**Answer:** nginx 1.27.2

**Explanation:**

- The pod description shows the nginx container is using the image `docker.io/bitnami/nginx:1.27.2`
- The label `app.kubernetes.io/version=1.27.4` indicates the application version
- The actual running container shows nginx version 1.27.2

### Step 7: Rollback nginx to Previous Version

**Question:** Oops!.. There seems to be a minor issue in the website and the DevOps Team is asked to rollback the nginx to previous version! Please rollback the nginx to previous version.

**Command:**

```bash
controlplane ~ ➜ helm rollback dazzling-web 1

Rollback was a success! Happy Helming!
```

**Explanation:**

- `helm rollback`: Reverts a release to a previous revision
- `dazzling-web`: The name of the release to rollback
- `1`: The revision number to rollback to (the original deployment)
- This command will restore nginx to the version that was running in revision 1

## Solution Summary

This exercise demonstrates the complete lifecycle of managing applications with Helm:

1. **Repository Management**: Adding external chart repositories (Bitnami)
2. **Release Inspection**: Checking existing deployments and their versions
3. **Upgrades**: Updating to newer versions with specific chart versions
4. **Verification**: Confirming the upgrade was successful
5. **Rollback**: Reverting to previous versions when issues arise

The approach solves the original problem by:

- Providing a controlled way to upgrade nginx from version 1.22.0 to 1.27.x
- Allowing easy rollback when issues are discovered
- Maintaining a history of deployments for audit and recovery purposes
- Using Helm's built-in revision tracking for version management

## Key Takeaways

- Helm manages the entire lifecycle of Kubernetes applications
- Each upgrade creates a new revision that can be referenced later
- Rollbacks are simple one-command operations
- The Bitnami repository provides production-ready charts with sensible defaults
- Always verify upgrades by checking the actual running pods and their versions

## Additional Notes

- The nginx deployment uses resource limits for CPU and memory
- Init containers are used to preserve log symlinks
- The deployment includes both HTTP (8080) and HTTPS (8443) ports
- Service accounts and proper RBAC are configured automatically by the Helm chart

# Kubernetes 2-Tier Application Troubleshooting Guide

This document provides solutions to common issues encountered when deploying a 2-tier web application in Kubernetes. The application consists of a web frontend and MySQL database backend.

## Architecture Overview

The application follows this architecture pattern across all namespaces:

```
user access 30081 -> pod web-service 8080 -> mysql-service 3306 -> mysql 3306
```

## Test 1: Alpha Namespace

### Problem

A simple 2-tier application is deployed in the alpha namespace. It must display a green web page on success, but it's currently failing.

### Investigation

First, let's examine all resources in the cluster:

```bash
controlplane ~ ➜  k get all -A
NAMESPACE     NAME                                          READY   STATUS      RESTARTS   AGE
alpha         pod/mysql                                     1/1     Running     0          10m
alpha         pod/webapp-mysql-5c4c675768-r65vj             1/1     Running     0          10m
kube-system   pod/coredns-ff8999cc5-nz2nx                   1/1     Running     0          15m
kube-system   pod/helm-install-traefik-4gqg2                0/1     Completed   1          15m
kube-system   pod/helm-install-traefik-crd-mh7l6            0/1     Completed   0          15m
kube-system   pod/local-path-provisioner-698b58967b-mhhdc   1/1     Running     0          15m
kube-system   pod/metrics-server-8584b5786c-rcfgp           1/1     Running     0          15m
kube-system   pod/svclb-traefik-ef38021b-7dpfc              2/2     Running     0          15m
kube-system   pod/traefik-857c696d79-jqmbb                  1/1     Running     0          15m

NAMESPACE     NAME                     TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
alpha         service/mysql            ClusterIP      10.43.47.173    <none>           3306/TCP                     10m
alpha         service/web-service      NodePort       10.43.142.254   <none>           8080:30081/TCP               10m
default       service/kubernetes       ClusterIP      10.43.0.1       <none>           443/TCP                      15m
kube-system   service/kube-dns         ClusterIP      10.43.0.10      <none>           53/UDP,53/TCP,9153/TCP       15m
kube-system   service/metrics-server   ClusterIP      10.43.144.143   <none>           443/TCP                      15m
kube-system   service/traefik          LoadBalancer   10.43.240.44    192.168.75.155   80:32529/TCP,443:32171/TCP   15m

NAMESPACE     NAME                                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   daemonset.apps/svclb-traefik-ef38021b   1         1         1       1            1           <none>          15m

NAMESPACE     NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
alpha         deployment.apps/webapp-mysql             1/1     1            1           10m
kube-system   deployment.apps/coredns                  1/1     1            1           15m
kube-system   deployment.apps/local-path-provisioner   1/1     1            1           15m
kube-system   deployment.apps/metrics-server           1/1     1            1           15m
kube-system   deployment.apps/traefik                  1/1     1            1           15m

NAMESPACE     NAME                                                DESIRED   CURRENT   READY   AGE
alpha         replicaset.apps/webapp-mysql-5c4c675768             1         1         1       10m
kube-system   replicaset.apps/coredns-ff8999cc5                   1         1         1       15m
kube-system   replicaset.apps/local-path-provisioner-698b58967b   1         1         1       15m
kube-system   replicaset.apps/metrics-server-8584b5786c           1         1         1       15m
kube-system   replicaset.apps/traefik-857c696d79                  1         1         1       15m

NAMESPACE     NAME                                 STATUS     COMPLETIONS   DURATION   AGE
kube-system   job.batch/helm-install-traefik       Complete   1/1           15s        15m
kube-system   job.batch/helm-install-traefik-crd   Complete   1/1           12s        15m
```

For better focus, let's switch to the alpha namespace:

```bash
controlplane ~ ✖ k config set-context --current --namespace=alpha
Context "default" modified.

k get pods

NAME                            READY   STATUS    RESTARTS   AGE
mysql                           1/1     Running   0          12m
webapp-mysql-5c4c675768-r65vj   1/1     Running   0          12m

controlplane ~ ➜  k get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
webapp-mysql   1/1     1            1           14m

controlplane ~ ➜  k get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
mysql         ClusterIP   10.43.47.173    <none>        3306/TCP         14m
web-service   NodePort    10.43.142.254   <none>        8080:30081/TCP   14m
```

Let's check the deployment configuration:

```bash
controlplane ~ ➜  k describe deploy webapp-mysql
Name:                   webapp-mysql
Namespace:              alpha
CreationTimestamp:      Wed, 14 May 2025 17:06:57 +0000
Labels:                 name=webapp-mysql
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp-mysql
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=webapp-mysql
  Containers:
   webapp-mysql:
    Image:      mmumshad/simple-webapp-mysql
    Port:       8080/TCP
    Host Port:  0/TCP
    Environment:
      DB_Host:      mysql-service #<This is the problem since mysql service name is mysql instead of mysql-service>
      DB_User:      root
      DB_Password:  paswrd
    Mounts:         <none>
  Volumes:          <none>
  Node-Selectors:   <none>
  Tolerations:      <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   webapp-mysql-5c4c675768 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  14m   deployment-controller  Scaled up replica set webapp-mysql-5c4c675768 from 0 to 1
```

### Solution

The issue is a mismatch between the service name. The deployment is looking for `mysql-service` but the actual service is named `mysql`. We have two options:

1. Edit the deployment to use the correct service name:

```bash
controlplane ~ ➜  k edit deploy webapp-mysql
# Change DB_Host: mysql-service to DB_Host: mysql
deployment.apps/webapp-mysql edited
```

2. Or rename the mysql service to match what the deployment expects:

```bash
k edit svc mysql
# Or delete and recreate:
k delete svc mysql
k apply -f tmp/changed.yaml
```

## Test 2: Beta Namespace

### Problem

The same 2-tier application is deployed in the beta namespace but is failing.

### Investigation

Let's check the MySQL service configuration:

```bash
controlplane ~ ✖ k describe svc mysql-service
Name:                     mysql-service
Namespace:                beta
Labels:                   <none>
Annotations:              <none>
Selector:                 name=mysql
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.115.31
IPs:                      10.43.115.31
Port:                     <unset>  3306/TCP
TargetPort:               8080/TCP    # This is wrong - MySQL runs on port 3306, not 8080
Endpoints:                10.22.0.11:8080
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>

controlplane ~ ➜  k get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
mysql                           1/1     Running   0          95s   10.22.0.11   controlplane   <none>           <none>
webapp-mysql-5c4c675768-pvzbt   1/1     Running   0          95s   10.22.0.12   controlplane   <none>           <none>
```

### Solution

The MySQL service is configured with an incorrect target port (8080 instead of 3306). Let's fix it:

```bash
controlplane ~ ➜  k edit svc mysql-service
# Change targetPort from 8080 to 3306
service/mysql-service edited

controlplane ~ ➜  k get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
mysql-service   ClusterIP   10.43.115.31   <none>        3306/TCP         2m19s
web-service     NodePort    10.43.37.158   <none>        8080:30081/TCP   2m19s
```

The service YAML should look like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2025-05-15T02:49:28Z"
  name: mysql-service
  namespace: beta
  resourceVersion: "1268"
  uid: 19980755-b48f-45b2-a910-1ffc9c5fe9da
spec:
  clusterIP: 10.43.115.31
  clusterIPs:
    - 10.43.115.31
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - port: 3306
      protocol: TCP
      targetPort: 3306 # Changed from 8080 to 3306
  selector:
    name: mysql
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

## Test 3: Gamma Namespace

### Problem

The same 2-tier application is deployed in the gamma namespace but is failing or unresponsive.

### Investigation

Let's check the MySQL service:

```bash
controlplane ~ ➜  k describe svc mysql-service
Name:                     mysql-service
Namespace:                gamma
Labels:                   <none>
Annotations:              <none>
Selector:                 name=sql00001    # Incorrect selector, should be name=mysql
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.170.98
IPs:                      10.43.170.98
Port:                     <unset>  3306/TCP
TargetPort:               3306/TCP
Endpoints:                # Empty! No endpoints because selector doesn't match any pods
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

Let's check the MySQL pod:

```bash
controlplane ~ ➜  k describe pod mysql
Name:             mysql
Namespace:        gamma
Priority:         0
Service Account:  default
Node:             controlplane/192.168.75.153
Start Time:       Thu, 15 May 2025 02:59:32 +0000
Labels:           name=mysql    # The label is actually 'name=mysql', not 'name=sql00001'
Annotations:      <none>
Status:           Running
IP:               10.22.0.13
IPs:
  IP:  10.22.0.13
Containers:
  mysql:
    Container ID:   containerd://b2b92df17eabde4a570b39e39f11d4fc3e42e33b6edabd2968b4f1141759da9e
    Image:          mysql:5.6
    Image ID:       docker.io/library/mysql@sha256:20575ecebe6216036d25dab5903808211f1e9ba63dc7825ac20cb975e34cfcae
    Port:           3306/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 15 May 2025 02:59:33 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      MYSQL_ROOT_PASSWORD:  paswrd
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dd8rp (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
```

### Solution

The service selector is configured incorrectly. The service is looking for pods with label `name=sql00001`, but the MySQL pod has label `name=mysql`. Let's fix it:

```bash
controlplane ~ ➜ k edit svc mysql-service
# Change selector from name=sql00001 to name=mysql
```

The corrected service YAML should look like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2025-05-15T02:59:32Z"
  name: mysql-service
  namespace: gamma
  resourceVersion: "1768"
  uid: 4b341aaf-0e31-40fe-8338-e112c007e3c5
spec:
  clusterIP: 10.43.170.98
  clusterIPs:
    - 10.43.170.98
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - port: 3306
      protocol: TCP
      targetPort: 3306
  selector:
    name: mysql # Changed from name=sql00001
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

## Test 4: Delta Namespace

### Problem

The same 2-tier application is deployed in the delta namespace but is failing.

### Investigation

Let's check the webapp deployment:

```bash
controlplane ~ ➜  k describe deploy webapp-mysql
Name:                   webapp-mysql
Namespace:              delta
CreationTimestamp:      Thu, 15 May 2025 14:09:32 +0000
Labels:                 name=webapp-mysql
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp-mysql
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=webapp-mysql
  Containers:
   webapp-mysql:
    Image:      mmumshad/simple-webapp-mysql
    Port:       8080/TCP
    Host Port:  0/TCP
    Environment:
      DB_Host:      mysql-service
      DB_User:      sql-user    # Wrong user - should be 'root' according to requirements
      DB_Password:  paswrd
    Mounts:         <none>
  Volumes:          <none>
  Node-Selectors:   <none>
  Tolerations:      <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   webapp-mysql-7dd5888bd4 (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m28s  deployment-controller  Scaled up replica set webapp-mysql-7dd5888bd4 from 0 to 1
```

### Solution

The deployment environment variables specify incorrect MySQL credentials. According to the requirements, we should use:

- DB_User: root (not sql-user)

Let's fix it:

```bash
controlplane ~ ➜  k edit deploy webapp-mysql
# Change DB_User from sql-user to root
deployment.apps/webapp-mysql edited
```

## Test 5: Epsilon Namespace

### Problem

The same 2-tier application is deployed in the epsilon namespace but is failing.

### Investigation

After checking the deployment and fixing the DB_User to be 'root', the site is still down. Let's check the MySQL pod:

```bash
controlplane ~ ➜  k get pods
NAME                            READY   STATUS    RESTARTS   AGE
mysql                           1/1     Running   0          5m3s
webapp-mysql-5c4c675768-g2zdp   1/1     Running   0          4m3s

controlplane ~ ➜  k describe pod mysql
Name:             mysql
Namespace:        epsilon
Priority:         0
Service Account:  default
Node:             controlplane/192.168.251.203
Start Time:       Fri, 16 May 2025 04:22:57 +0000
Labels:           name=mysql
Annotations:      <none>
Status:           Running
IP:               10.22.0.18
IPs:
  IP:  10.22.0.18
Containers:
  mysql:
    Container ID:   containerd://2d3bd55d98030694fdfea1be7a3bdfdd9699e3c680bc4b7c88acfbbfd4c6e404
    Image:          mysql:5.6
    Image ID:       docker.io/library/mysql@sha256:20575ecebe6216036d25dab5903808211f1e9ba63dc7825ac20cb975e34cfcae
    Port:           3306/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 16 May 2025 04:22:58 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      MYSQL_ROOT_PASSWORD:  passwooooorrddd   # Wrong password - should be 'paswrd'
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-6mlb9 (ro)
```

### Solution

The MySQL pod is configured with the wrong root password. The webapp is trying to connect with password `paswrd`, but MySQL is set up with `passwooooorrddd`. Let's fix the MySQL pod configuration:

```bash
k edit pod mysql
# Change MYSQL_ROOT_PASSWORD from passwooooorrddd to paswrd
```

The corrected section of the YAML should look like this:

```yaml
spec:
  containers:
    - env:
        - name: MYSQL_ROOT_PASSWORD
          value: paswrd # Changed from passwooooorrddd
      image: mysql:5.6
      imagePullPolicy: IfNotPresent
      name: mysql
      ports:
        - containerPort: 3306
          protocol: TCP
```

## Test 6: Zeta Namespace

### Problem

The same 2-tier application is deployed in the zeta namespace but is failing.

### Investigation

Similar to previous tests, we first fix the DB_User and password issues. Then we need to check the web-service:

```bash
k edit svc web-service
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2025-05-16T04:35:01Z"
  name: web-service
  namespace: zeta
  resourceVersion: "2052"
  uid: cced70d0-5b2a-4067-9c46-6c1a92d9662f
spec:
  clusterIP: 10.43.200.252
  clusterIPs:
    - 10.43.200.252
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - nodePort: 30000012 # Incorrect nodePort - should be 30081
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    name: webapp-mysql
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

### Solution

The web-service is configured with an incorrect nodePort (30000012 instead of 30081). Let's fix it:

```bash
k edit svc web-service
# Change nodePort from 30000012 to 30081
service/web-service edited
```

## Summary of Issues and Solutions

1. **Alpha Namespace**:

   - Issue: Service name mismatch
   - Solution: Either edit deployment to use `mysql` instead of `mysql-service` or rename the service

2. **Beta Namespace**:

   - Issue: Incorrect targetPort in mysql-service
   - Solution: Change targetPort from 8080 to 3306

3. **Gamma Namespace**:

   - Issue: Incorrect selector in mysql-service
   - Solution: Change selector from `name=sql00001` to `name=mysql`

4. **Delta Namespace**:

   - Issue: Incorrect MySQL username
   - Solution: Change DB_User from `sql-user` to `root`

5. **Epsilon Namespace**:

   - Issue: Incorrect MySQL root password
   - Solution: Change MYSQL_ROOT_PASSWORD from `passwooooorrddd` to `paswrd`

6. **Zeta Namespace**:
   - Issue: Incorrect nodePort for web-service
   - Solution: Change nodePort from 30000012 to 30081

## General Troubleshooting Approaches

1. **Check Kubernetes objects**:

   ```bash
   k get all -n <namespace>
   ```

2. **Examine pod status**:

   ```bash
   k get pods -n <namespace>
   k describe pod <pod-name> -n <namespace>
   ```

3. **Check service configurations**:

   ```bash
   k get svc -n <namespace>
   k describe svc <service-name> -n <namespace>
   ```

4. **Examine deployments**:

   ```bash
   k describe deploy <deployment-name> -n <namespace>
   ```

5. **Check logs**:

   ```bash
   k logs <pod-name> -n <namespace>
   ```

6. **Edit resources to fix issues**:
   ```bash
   k edit <resource-type> <resource-name> -n <namespace>
   ```

This troubleshooting guide demonstrates common issues that can occur in Kubernetes deployments and how to resolve them effectively.

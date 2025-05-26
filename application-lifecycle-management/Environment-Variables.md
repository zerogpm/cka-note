What is the environment variable name set on the container in the pod?

```bash
controlplane ~ ➜  k describe pod webapp-color
Name:             webapp-color
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.20.135
Start Time:       Mon, 26 May 2025 14:11:31 +0000
Labels:           name=webapp-color
Annotations:      <none>
Status:           Running
IP:               10.22.0.9
IPs:
  IP:  10.22.0.9
Containers:
  webapp-color:
    Container ID:   containerd://44d87f264ba67a325b77077cae00ed0ecbf219eeb310f0bf7736964418969239
    Image:          kodekloud/webapp-color
    Image ID:       docker.io/kodekloud/webapp-color@sha256:99c3821ea49b89c7a22d3eebab5c2e1ec651452e7675af243485034a72eb1423
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 26 May 2025 14:11:35 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      APP_COLOR:  pink
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-6cjk8 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-6cjk8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  36s   default-scheduler  Successfully assigned default/webapp-color to controlplane
  Normal  Pulling    36s   kubelet            Pulling image "kodekloud/webapp-color"
  Normal  Pulled     33s   kubelet            Successfully pulled image "kodekloud/webapp-color" in 2.729s (2.729s including waiting). Image size: 31777918 bytes.
  Normal  Created    33s   kubelet            Created container: webapp-color
  Normal  Started    33s   kubelet            Started container webapp-color
```

What is the value set on the environment variable APP_COLOR on the container in the pod?

Update the environment variable on the POD to display a green background.

Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the Pod.

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
    - env:
        - name: APP_COLOR
          value: green
      image: kodekloud/webapp-color
      name: webapp-color
```

How many ConfigMaps exists in the default namespace?

```bash
controlplane ~ ➜  k get configmaps
NAME               DATA   AGE
db-config          3      14s
kube-root-ca.crt   1      10m
```

Identify the database host from the config map db-config.

```bash
controlplane ~ ➜  k describe configmaps db-config
Name:         db-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DB_HOST:
----
SQL01.example.com

DB_NAME:
----
SQL01

DB_PORT:
----
3306


BinaryData
====

Events:  <none>
```

Create a new ConfigMap for the webapp-color POD. Use the spec given below.

ConfigMap Name: webapp-config-map

Data: APP_COLOR=darkblue

Data: APP_OTHER=disregard

```bash
k create configmap  webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard
```

Update the environment variable on the POD to use only the APP_COLOR key from the newly created ConfigMap.

Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the Pod.

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
    - env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: webapp-config-map
              key: APP_COLOR
      image: kodekloud/webapp-color
      name: webapp-color
```

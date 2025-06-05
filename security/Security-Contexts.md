What is the user used to execute the sleep process within the ubuntu-sleeper pod?

In the current(default) namespace.

```bash
controlplane ~ ➜  k exec ubuntu-sleeper -- whoami
root
```

Edit the pod ubuntu-sleeper to run the sleep process with user ID 1010.

Note: Only make the necessary changes. Do not modify the name or image of the pod.

Pod Name: ubuntu-sleeper

Image Name: ubuntu

SecurityContext: User 1010

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
    - command:
        - sleep
        - "4800"
      image: ubuntu
      name: ubuntu-sleeper
```

A Pod definition file named multi-pod.yaml is given. With what user are the processes in the web container started?

The pod is created with multiple containers and security contexts defined at the Pod and Container level.

```yaml
controlplane ~ ➜  cat multi-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]

```
1002

With what user are the processes in the sidecar container started?

The pod is created with multiple containers and security contexts defined at the Pod and Container level.

1001

Update pod ubuntu-sleeper to run as Root user and with the SYS_TIME capability.


Note: Only make the necessary changes. Do not modify the name of the pod.

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
```

Now update the pod to also make use of the NET_ADMIN capability.


Note: Only make the necessary changes. Do not modify the name of the pod

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      capabilities:
        add: ["SYS_TIME", "NET_ADMIN"]
```
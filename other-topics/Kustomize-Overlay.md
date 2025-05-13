When deploying application to prod environment, what type of image will be used for the api-deployment?

The original api-deployment uses an image of nginx but in the prod overlay it gets patched to memcached as shown below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      component: redis
  template:
    metadata:
      labels:
        component: redis
    spec:
      containers:
        - name: redis
          image: redis
```

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
          image: memcached
```

How many replicas for api-deployment will get deployed in prod?

The prod overlay patch for the api-deployment does not modify the replicas property, so it defaults to the value set in base/

base/kustomization.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      component: api
  template:
    metadata:
      labels:
        component: api
    spec:
      containers:
        - name: api
          image: nginx
          env:
            - name: DB_CONNECTION
              value: db.kodekloud.com
```

What will be the value of the environment variable MONGO_INITDB_ROOT_PASSWORD in the mongo-deployment container in the staging environment?

In the mongo-deployment, mongodb container grabs the value for the MONGO_INITDB_ROOT_PASSWORD env variable from the db-cred configmap.

This value was set to mypassword in the base directory but a patch is applied in the staging to set it to superp@ssword123.

overlays/staging/configMap-patch.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: mongo
  template:
    metadata:
      labels:
        component: mongo
    spec:
      containers:
        - name: mongo
          image: mongo
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                configMapKeyRef:
                  name: db-creds
                  key: username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: db-creds
                  key: password
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-creds
data:
  username: mongo
  password: superp@ssword123
```

When deploying to prod how many total pods are created?

The prod environment will deploy 2 api pods, 2 redis pods and 1 mongo pod.

overlays/prod/kustomization.yaml

resources:

- redis-depl.yaml

overlays/prod/redis-depl.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      component: api
  template:
    metadata:
      labels:
        component: api
    spec:
      containers:
        - name: api
          image: nginx
          env:
            - name: DB_CONNECTION
              value: db.kodekloud.com
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: mongo
  template:
    metadata:
      labels:
        component: mongo
    spec:
      containers:
        - name: mongo
          image: mongo
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                configMapKeyRef:
                  name: db-creds
                  key: username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: db-creds
                  key: password
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 2
```

How many environment variables are set on the nginx container in the api-deployment in dev environment?

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      component: api
  template:
    metadata:
      labels:
        component: api
    spec:
      containers:
        - name: api
          image: nginx
          env:
            - name: DB_CONNECTION
              value: db.kodekloud.com
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      component: api
  template:
    metadata:
      labels:
        component: api
    spec:
      containers:
        - name: api
          image: nginx
          env:
            - name: DB_USERNAME
              valueFrom:
                configMapKeyRef:
                  name: db-creds
                  key: username
            - name: DB_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: db-creds
                  key: password
```

Update the api image in the api-deployment to use caddy docker image in the QA environment.

Perform this using an inline JSON6902 patch.

Note: Please ensure to apply the updated config for QA environment before validation.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

commonLabels:
  environment: QA

patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: caddy
```

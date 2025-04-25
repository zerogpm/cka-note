# Kubernetes Storage Classes and PersistentVolumeClaims

This guide demonstrates how to work with StorageClasses, PersistentVolumeClaims (PVCs), and how volume binding modes affect their behavior.

## Exploring Available StorageClasses

Check what StorageClasses exist in the cluster:

```bash
kubectl get storageclass
# or
kubectl get sc
```

Output:

```
NAME                       PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE     ALLOWVOLUMEEXPANSION   AGE
local-path (default)       rancher.io/local-path          Delete          WaitForFirstConsumer   false                  23m
local-storage              kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   false                  12s
portworx-io-priority-high  kubernetes.io/portworx-volume  Delete          Immediate              false                  12s
```

## StorageClass Properties

### Storage Class Without Dynamic Provisioning

The StorageClass that does not support dynamic volume provisioning:

- **Name**: `local-storage`
- **Provisioner**: `kubernetes.io/no-provisioner`
- **VolumeBindingMode**: `WaitForFirstConsumer`

### Other StorageClass Details

For the storage class `portworx-io-priority-high`:

- **Provisioner**: `kubernetes.io/portworx-volume`
- **VolumeBindingMode**: `Immediate`

## Working with PersistentVolumeClaims

### Checking Existing PVCs

```bash
kubectl get pvc
```

Determined that there is no PVC currently consuming the PersistentVolume called `local-pv`.

### Creating a PVC for the local-pv

Create a new PersistentVolumeClaim to bind to the existing `local-pv` volume:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  resources:
    requests:
      storage: 500Mi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
```

### Checking PVC Status

After creation, the status of the PVC remains `Pending`:

```bash
kubectl describe pvc local-pvc
```

Output:

```
Name: local-pvc
Namespace: default
StorageClass: local-storage
Status: Pending
Volume:
Labels: <none>
Annotations: <none>
Finalizers: [kubernetes.io/pvc-protection]
Capacity:
Access Modes:
VolumeMode: Filesystem
Used By: <none>
Events:
  Type    Reason              Age                From                         Message
  ----    ------              ----               ----                         -------
  Normal  WaitForFirstConsumer 3m16s (x42 over 13m) persistentvolume-controller waiting for first consumer to be created before binding
```

## Understanding WaitForFirstConsumer Binding Mode

**Important Note**: The Storage Class `local-storage` uses `VolumeBindingMode` set to `WaitForFirstConsumer`. This means:

- The binding and provisioning of a PersistentVolume is delayed
- A PVC will remain in `Pending` state until a Pod using the PVC is created
- This is useful for local volumes to ensure the Pod and PV are scheduled on the same node

## Creating a Pod to Use the PVC

Create a new pod that uses the PVC:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx:alpine
      name: nginx
      resources: {}
      volumeMounts:
        - name: nginx
          mountPath: /var/www/html
  volumes:
    - name: nginx
      persistentVolumeClaim:
        claimName: local-pvc
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

After creating the Pod, the status of the PVC changes to `Bound`.

## Creating a Custom StorageClass with WaitForFirstConsumer

Create a new StorageClass that also uses the `WaitForFirstConsumer` binding mode:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

This StorageClass has the following characteristics:

- **Name**: `delayed-volume-sc`
- **Provisioner**: `kubernetes.io/no-provisioner` (no dynamic provisioning)
- **VolumeBindingMode**: `WaitForFirstConsumer` (delays binding until a Pod uses the PVC)

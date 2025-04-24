# Kubernetes Persistent Storage Guide

This guide demonstrates how to work with persistent storage in Kubernetes using Volumes, PersistentVolumes (PV), and PersistentVolumeClaims (PVC).

## Viewing Container Logs

The application stores logs at location `/log/app.log`. To view these logs:

```bash
kubectl exec webapp -- cat /log/app.log
```

**Key Point**: If the pod gets deleted, these logs would be lost without persistent storage.

## Configuring a Volume for Log Storage

### Step 1: Edit the Pod to Add a Volume

```bash
kubectl edit pods webapp
```

Add the necessary volume configuration:

```yaml
containers:
  - env:
      - name: LOG_HANDLERS
        value: file
    image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: event-simulator
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
      - mountPath: /log
        name: webapp
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-tqdr7
        readOnly: true
volumes:
  - hostPath:
      path: /var/log/webapp
      type: ""
    name: webapp
  - name: kube-api-access-tqdr7
    projected:
      defaultMode: 420
      sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
              - key: ca.crt
                path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
```

Since pods cannot be edited directly, apply the changes with:

```bash
kubectl replace --force -f 1212.yaml
```

## Creating a Persistent Volume

Create a Persistent Volume with the following specification:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /pv/log
```

## Creating a Persistent Volume Claim

First attempt at creating a PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

### Checking PVC Status

```bash
kubectl get pvc
```

Output:

```
NAME         STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
claim-log-1  Pending  <unset>                                                                   93s
```

### Checking PV Status

```bash
kubectl get pv
```

Output:

```
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-log   100Mi      RWX           Retain           Available   <unset>                                                  6m14s
```

**Issue Identified**: Access mode mismatch - PV is ReadWriteMany while PVC is ReadWriteOnce

### Fixing the PVC

Delete the existing PVC:

```bash
kubectl delete pvc claim-log-1
```

Create a new PVC with correct access mode (ReadWriteMany):

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
```

Verify the bound status:

```bash
kubectl get pvc
```

Output:

```
NAME         STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
claim-log-1  Bound    pv-log   100Mi      RWX            <unset>                                4s
```

## PV and PVC Capacity Behavior

**Important Note**: Even though the PVC only requested 50Mi, it gets the full 100Mi capacity of the PV.

This happens because:

- PVs are not "partially bound" - a PVC gets assigned the entire PV
- The remaining capacity (50Mi) cannot be used by other PVCs
- This is a limitation of the basic PV/PVC binding system in Kubernetes

For more efficient storage utilization, consider:

- Creating PVs that match exact sizes needed
- Using Storage Classes with dynamic provisioning
- Using cloud provider storage classes that create volumes of exact requested size

## Updating Pod to Use PVC

Update the webapp pod configuration to use the PVC:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: default
spec:
  containers:
    - env:
        - name: LOG_HANDLERS
          value: file
      image: kodekloud/event-simulator
      imagePullPolicy: Always
      name: event-simulator
      volumeMounts:
        - mountPath: /log
          name: webapp
  volumes:
    - name: webapp
      persistentVolumeClaim:
        claimName: claim-log-1
```

## Understanding PV Lifecycle and Reclaim Policy

### What happens when a PVC is deleted?

When attempting to delete a PVC that's in use by a pod:

- The PVC enters a "Terminating" state but cannot be deleted
- It remains in this state until all pods using it are deleted

After the pod is deleted, the PVC is deleted as well.

### What happens to the PV after PVC deletion?

With a Reclaim Policy of "Retain":

```bash
kubectl get pv
```

Output:

```
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                  STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-log   100Mi      RWX           Retain           Released   default/claim-log-1    <unset>                                         24m
```

The PV enters "Released" state, which means:

- The PV is not available for new claims
- The data in the PV is preserved
- An administrator must manually reclaim the storage

## Reclaim Policies

Three possible reclaim policies in Kubernetes:

1. **Retain**: PV is not automatically deleted, and admin must manually reclaim
2. **Delete**: PV and its associated storage are automatically deleted
3. **Recycle**: (Deprecated) Basic scrub is performed before making available again

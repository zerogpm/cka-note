# Kubernetes Storage Setup: StorageClass, PV, and PVC Configuration

## Problem Statement

Your task involves setting up storage components in a Kubernetes cluster. Follow these steps:

**Step 1:** Create a Storage Class named `blue-stc-cka` with the following properties:

- Provisioner: `kubernetes.io/no-provisioner`
- Volume binding mode: `WaitForFirstConsumer`

**Step 2:** Create a Persistent Volume (PV) named `blue-pv-cka` with the following properties:

- Capacity: 100Mi
- Access mode: ReadWriteOnce
- Reclaim policy: Retain
- Storage class: `blue-stc-cka`
- Local path: `/opt/blue-data-cka`
- Node affinity: Set node affinity to create this PV on controlplane

**Step 3:** Create a Persistent Volume Claim (PVC) named `blue-pvc-cka` with the following properties:

- Access mode: ReadWriteOnce
- Storage class: `blue-stc-cka`
- Storage request: 50Mi
- The volume should be bound to `blue-pv-cka`

## Prerequisites

Set the correct Kubernetes context:

```bash
kubectl config use-context kubernetes-admin@kubernetes
```

## Solution Implementation

### Step 1: Create StorageClass

Create a file named `storageclass.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: blue-stc-cka
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

**Apply the StorageClass:**

```bash
kubectl apply -f storageclass.yaml
```

**Verify creation:**

```bash
kubectl get storageclass blue-stc-cka
```

### Step 2: Create Physical Directory on Node

⚠️ **Important:** These commands must be run on the controlplane node itself, NOT inside a pod.

```bash
sudo mkdir -p /opt/blue-data-cka
sudo chmod 755 /opt/blue-data-cka
```

**Verify directory creation:**

```bash
ls -la /opt/ | grep blue-data-cka
```

### Step 3: Create Persistent Volume

Create a file named `persistent-volume.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: blue-pv-cka
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: blue-stc-cka
  local:
    path: /opt/blue-data-cka
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - controlplane
```

**Apply the Persistent Volume:**

```bash
kubectl apply -f persistent-volume.yaml
```

**Verify PV creation:**

```bash
kubectl get pv blue-pv-cka
```

### Step 4: Create Persistent Volume Claim

Create a file named `persistent-volume-claim.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: blue-pvc-cka
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 50Mi
  storageClassName: blue-stc-cka
```

**Apply the Persistent Volume Claim:**

```bash
kubectl apply -f persistent-volume-claim.yaml
```

**Verify PVC binding:**

```bash
kubectl get pvc blue-pvc-cka
kubectl get pv blue-pv-cka
```

### Step 5: Test with a Pod

Create a file named `test-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  tolerations:
    - key: node-role.kubernetes.io/control-plane
      operator: Exists
      effect: NoSchedule
  containers:
    - name: test-container
      image: nginx
      volumeMounts:
        - name: blue-storage
          mountPath: /data
  volumes:
    - name: blue-storage
      persistentVolumeClaim:
        claimName: blue-pvc-cka
```

**Deploy the test pod:**

```bash
kubectl apply -f test-pod.yaml
```

**Verify pod is running:**

```bash
kubectl get pods test-pod
kubectl describe pod test-pod
```

**Test the storage mount:**

```bash
kubectl exec -it test-pod -- df -h /data
kubectl exec -it test-pod -- ls -la /data
```

## Expected Console Output

### StorageClass Creation

```
$ kubectl get storageclass blue-stc-cka
NAME           PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
blue-stc-cka   kubernetes.io/no-provisioner   Retain          WaitForFirstConsumer   false                  1m
```

### Directory Creation

```
$ ls -la /opt/ | grep blue-data-cka
drwxr-xr-x  2 root root 4096 Jun 15 10:30 blue-data-cka
```

### Persistent Volume Status

```
$ kubectl get pv blue-pv-cka
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
blue-pv-cka   100Mi      RWO            Retain           Available           blue-stc-cka            1m
```

### Persistent Volume Claim Binding

```
$ kubectl get pvc blue-pvc-cka
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
blue-pvc-cka   Bound    blue-pv-cka   100Mi      RWO            blue-stc-cka   30s

$ kubectl get pv blue-pv-cka
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
blue-pv-cka   100Mi      RWO            Retain           Bound    default/blue-pvc-cka   blue-stc-cka            2m
```

### Pod Verification

```
$ kubectl get pods test-pod
NAME       READY   STATUS    RESTARTS   AGE
test-pod   1/1     Running   0          1m

$ kubectl exec -it test-pod -- df -h /data
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  5.2G   14G  28% /data
```

## Technical Explanation

### Why This Approach Works

This solution implements **static provisioning** for local storage in Kubernetes. Here's why each component is necessary:

#### 1. StorageClass Configuration

- **`kubernetes.io/no-provisioner`**: Indicates this is static provisioning, not dynamic
- **`WaitForFirstConsumer`**: Delays PV binding until a pod actually needs the storage
- **`Retain` policy**: Preserves data when PVC is deleted

#### 2. Manual Directory Creation

The physical directory creation is **mandatory** because:

- Local storage represents actual filesystem paths on nodes
- Kubernetes cannot create directories on host systems automatically
- The `kubernetes.io/no-provisioner` requires pre-existing storage
- Only cloud provisioners (AWS EBS, GCE PD) can create storage dynamically

#### 3. Node Affinity

```yaml
nodeAffinity:
  required:
    nodeSelectorTerms:
      - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
              - controlplane
```

This ensures the PV is only available on the `controlplane` node where we created the directory.

#### 4. Pod Tolerations

```yaml
tolerations:
  - key: node-role.kubernetes.io/control-plane
    operator: Exists
    effect: NoSchedule
```

Required because controlplane nodes typically have taints that prevent regular pod scheduling.

### Static vs Dynamic Provisioning

| Aspect               | Static (This Solution)         | Dynamic                         |
| -------------------- | ------------------------------ | ------------------------------- |
| **Provisioner**      | `kubernetes.io/no-provisioner` | Cloud provider specific         |
| **Storage Creation** | Manual (admin creates)         | Automatic (provisioner creates) |
| **Use Case**         | Local storage, existing NFS    | Cloud storage (EBS, GCE PD)     |
| **Flexibility**      | Limited, pre-defined           | High, on-demand                 |

## Cleanup Commands

To remove all created resources:

```bash
# Delete the test pod
kubectl delete pod test-pod

# Delete PVC (this will unbind the PV)
kubectl delete pvc blue-pvc-cka

# Delete PV
kubectl delete pv blue-pv-cka

# Delete StorageClass
kubectl delete storageclass blue-stc-cka

# Remove directory from node (run on controlplane)
sudo rm -rf /opt/blue-data-cka
```

## Troubleshooting

### Common Issues

1. **PVC stuck in Pending state**

   - Check if the directory exists on the node
   - Verify node affinity matches actual node names
   - Ensure StorageClass exists

2. **Pod fails to schedule**

   - Add appropriate tolerations for controlplane nodes
   - Check node taints: `kubectl describe node controlplane`

3. **Mount failures**
   - Verify directory permissions: `ls -la /opt/blue-data-cka`
   - Check PVC is bound: `kubectl get pvc`

### Verification Commands

```bash
# Check all storage components
kubectl get storageclass,pv,pvc

# Detailed PV information
kubectl describe pv blue-pv-cka

# Check PVC binding
kubectl describe pvc blue-pvc-cka

# Verify pod storage
kubectl describe pod test-pod | grep -A 10 Volumes
```

## Key Takeaways

- **Static provisioning** requires manual storage creation before Kubernetes resources
- **Local storage** is node-specific and requires proper node affinity
- **WaitForFirstConsumer** binding mode optimizes scheduling for local storage
- **Tolerations** are necessary for pods running on controlplane nodes
- The combination of SC, PV, and PVC creates a complete storage abstraction layer

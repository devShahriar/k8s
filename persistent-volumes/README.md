# Persistent Volumes & Persistent Volume Claims - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What Persistent Volumes (PV) and Persistent Volume Claims (PVC) are
- The difference between static and dynamic provisioning
- How to create and use PVs and PVCs
- Storage classes and access modes
- How to mount persistent storage in pods
- Best practices for persistent storage

## 🎯 What are Persistent Volumes and Claims?

**Persistent Volume (PV)** is a cluster-wide storage resource provisioned by an administrator. Think of it as a "storage pool" available in the cluster.

**Persistent Volume Claim (PVC)** is a request for storage by a user. Think of it as a "reservation" for storage from the pool.

### Real-World Analogy

Think of a parking garage:
- **PV (Persistent Volume)** = The entire parking garage (storage available)
- **PVC (Persistent Volume Claim)** = Your parking ticket (your request for a spot)
- **Pod** = Your car (uses the parking spot)

You request a parking spot (PVC), get assigned one (bound to PV), and your car (pod) uses it.

### The Problem They Solve

**Without PV/PVC:**
- Pods lose data when they restart
- No way to share data between pods
- Storage tied to specific nodes
- Difficult to manage storage

**With PV/PVC:**
- Data persists across pod restarts
- Storage is abstracted from pods
- Storage can be shared (with right access mode)
- Storage managed by cluster administrators

## 🔑 Key Concepts

### Persistent Volume (PV)
- **Cluster resource**: Available to all namespaces
- **Created by admin**: Usually pre-provisioned or dynamically created
- **Has capacity**: e.g., 10Gi, 100Gi
- **Has access modes**: ReadWriteOnce, ReadOnlyMany, ReadWriteMany

### Persistent Volume Claim (PVC)
- **Namespace resource**: Created in a specific namespace
- **Created by user**: Request for storage
- **Specifies requirements**: Size, access mode, storage class
- **Binds to PV**: Automatically or manually

### Binding Process
1. User creates PVC with requirements
2. Kubernetes finds matching PV
3. PVC binds to PV
4. Pod uses PVC to mount storage

## 🏗️ How It Works (Step by Step)

### Step 1: Administrator Creates PV (Static Provisioning)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
```

### Step 2: User Creates PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
```

### Step 3: Kubernetes Binds PVC to PV

```bash
kubectl get pvc
# NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES
# pvc-example   Bound    pv-example   10Gi       RWO
```

### Step 4: Pod Uses PVC

```yaml
volumes:
- name: storage
  persistentVolumeClaim:
    claimName: pvc-example
```

## 📝 Tutorial: Your First PV and PVC

### Step 1: Create a Persistent Volume

```bash
# Apply the PV
kubectl apply -f basic-pv.yaml

# View the PV
kubectl get pv

# Check details
kubectl describe pv pv-example
```

**What you'll see:**
```
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM
pv-example  10Gi       RWO            Retain           Available
```

**Status meanings:**
- **Available**: PV exists but not bound
- **Bound**: PV is bound to a PVC
- **Released**: PVC deleted, PV waiting for reclaim
- **Failed**: PV failed to be provisioned

### Step 2: Create a Persistent Volume Claim

```bash
# Apply the PVC
kubectl apply -f basic-pvc.yaml

# View the PVC
kubectl get pvc

# Check binding status
kubectl get pvc pvc-example
```

**What you'll see:**
```
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES
pvc-example   Bound    pv-example   10Gi       RWO
```

**Status meanings:**
- **Pending**: Waiting for PV to bind
- **Bound**: Successfully bound to PV
- **Lost**: PV was deleted

### Step 3: Verify Binding

```bash
# Check PV status (should be Bound)
kubectl get pv pv-example

# Check PVC details
kubectl describe pvc pvc-example
```

### Step 4: Use PVC in a Pod

```bash
# Apply pod that uses the PVC
kubectl apply -f pod-with-pvc.yaml

# Check pod status
kubectl get pod storage-pod

# Verify storage is mounted
kubectl exec storage-pod -- df -h
kubectl exec storage-pod -- ls -la /data
```

### Step 5: Test Data Persistence

```bash
# Write data to the volume
kubectl exec storage-pod -- sh -c "echo 'Hello from Pod 1' > /data/test.txt"

# Delete the pod
kubectl delete pod storage-pod

# Create a new pod with same PVC
kubectl apply -f pod-with-pvc.yaml

# Verify data persisted
kubectl exec storage-pod -- cat /data/test.txt
# Should show: Hello from Pod 1
```

## 🔌 Access Modes Explained

Access modes determine how the volume can be mounted:

### ReadWriteOnce (RWO)
- **Can be mounted**: As read-write by a single node
- **Use case**: Database, single pod applications
- **Limitation**: Only one pod can write at a time

### ReadOnlyMany (ROX)
- **Can be mounted**: As read-only by many nodes
- **Use case**: Shared configuration, read-only data
- **Limitation**: No write access

### ReadWriteMany (RWX)
- **Can be mounted**: As read-write by many nodes
- **Use case**: Shared file systems, NFS
- **Benefit**: Multiple pods can read and write

### ReadWriteOncePod (RWOP) - Kubernetes 1.22+
- **Can be mounted**: As read-write by a single pod
- **Use case**: Ensures exclusive access
- **Benefit**: Prevents multiple pods from accessing

## 📚 Storage Classes

### What is a Storage Class?

Storage classes define different "classes" of storage (fast SSD, slow HDD, etc.) and enable dynamic provisioning.

### Static vs Dynamic Provisioning

**Static Provisioning:**
- Admin creates PVs manually
- PVC binds to existing PV
- Good for: Specific storage requirements

**Dynamic Provisioning:**
- No PVs pre-created
- Storage class creates PV automatically
- Good for: On-demand storage

### Example Storage Class

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
```

## 📚 Example Files Explained

### Basic PV (`basic-pv.yaml`)

**What it does:**
- Creates a 10Gi persistent volume
- Uses hostPath (local storage)
- Manual storage class
- Retain reclaim policy

**Try it:**
```bash
kubectl apply -f basic-pv.yaml
kubectl get pv
```

### Basic PVC (`basic-pvc.yaml`)

**What it does:**
- Requests 5Gi storage
- Matches PV requirements
- Binds to available PV

**Try it:**
```bash
kubectl apply -f basic-pvc.yaml
kubectl get pvc
```

### Pod with PVC (`pod-with-pvc.yaml`)

**What it does:**
- Mounts PVC as volume
- Shows how to use persistent storage in pods

**Try it:**
```bash
kubectl apply -f pod-with-pvc.yaml
kubectl exec <pod-name> -- ls /data
```

### Dynamic Provisioning (`storage-class.yaml`, `pvc-dynamic.yaml`)

**What it does:**
- Creates storage class for dynamic provisioning
- PVC automatically creates PV

**Try it:**
```bash
kubectl apply -f storage-class.yaml
kubectl apply -f pvc-dynamic.yaml
# PV is created automatically!
```

## 🔄 Reclaim Policies

### Retain
- **What happens**: PV is kept after PVC deletion
- **Data**: Preserved
- **Use case**: Important data, manual cleanup

### Delete
- **What happens**: PV is deleted when PVC is deleted
- **Data**: Lost
- **Use case**: Temporary data, dynamic provisioning

### Recycle (Deprecated)
- **What happens**: Data deleted, PV made available
- **Status**: Deprecated, use Retain or Delete

## 🎓 Common Use Cases

### Use Case 1: Database Storage

**Scenario:** MySQL database needs persistent storage.

**Solution:**
```yaml
# PVC for database
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce  # Database only needs one writer
  resources:
    requests:
      storage: 20Gi
```

### Use Case 2: Shared Configuration

**Scenario:** Multiple pods need to read the same configuration.

**Solution:**
```yaml
# PVC with ReadOnlyMany
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: config-pvc
spec:
  accessModes:
    - ReadOnlyMany  # Multiple pods can read
  resources:
    requests:
      storage: 1Gi
```

### Use Case 3: Application Logs

**Scenario:** Application writes logs that need to persist.

**Solution:**
```yaml
# PVC for logs
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: logs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

## 🔧 Important Fields Explained

### PV Fields

**`capacity.storage`**
```yaml
capacity:
  storage: 10Gi
```
- Size of the volume
- Can be larger than PVC request

**`accessModes`**
```yaml
accessModes:
  - ReadWriteOnce
```
- How volume can be accessed
- Must match PVC requirements

**`persistentVolumeReclaimPolicy`**
```yaml
persistentVolumeReclaimPolicy: Retain
```
- What happens when PVC is deleted
- Options: Retain, Delete, Recycle

**`storageClassName`**
```yaml
storageClassName: manual
```
- Links PV to storage class
- Empty string = no storage class

### PVC Fields

**`resources.requests.storage`**
```yaml
resources:
  requests:
    storage: 5Gi
```
- Amount of storage requested
- Must be ≤ PV capacity

**`accessModes`**
```yaml
accessModes:
  - ReadWriteOnce
```
- Required access mode
- Must match PV access mode

**`storageClassName`**
```yaml
storageClassName: fast-ssd
```
- Requests storage from specific class
- Empty = uses default storage class

## 🐛 Troubleshooting

### Problem: PVC stuck in Pending

```bash
# Check PVC status
kubectl describe pvc <pvc-name>

# Common reasons:
# 1. No PV matches requirements
# 2. Storage class not found
# 3. Insufficient storage
```

**Solution:**
- Check if PV exists: `kubectl get pv`
- Verify storage class: `kubectl get storageclass`
- Check PVC requirements match PV

### Problem: PV not binding to PVC

```bash
# Check PV status
kubectl describe pv <pv-name>

# Check access modes match
kubectl get pv <pv-name> -o yaml | grep accessModes
kubectl get pvc <pvc-name> -o yaml | grep accessModes
```

**Solution:**
- Ensure access modes match
- Check storage class matches
- Verify capacity is sufficient

### Problem: Pod can't mount volume

```bash
# Check pod events
kubectl describe pod <pod-name>

# Check PVC status
kubectl get pvc <pvc-name>
```

**Solution:**
- Verify PVC is Bound
- Check volume mount path
- Verify node has access to storage

## 💡 Best Practices

1. **Use dynamic provisioning when possible**
   - Easier to manage
   - Automatic PV creation
   - Better for cloud environments

2. **Choose right access mode**
   - RWO for databases
   - RWX for shared file systems
   - ROX for read-only data

3. **Set appropriate reclaim policy**
   - Retain for important data
   - Delete for temporary data

4. **Use storage classes**
   - Different classes for different needs
   - Fast SSD for databases
   - Slow HDD for backups

5. **Monitor storage usage**
   - Set resource quotas
   - Monitor PVC usage
   - Clean up unused PVs

6. **Backup important data**
   - PV/PVC don't provide backup
   - Use backup solutions
   - Test restore procedures

## 📋 Quick Reference Commands

```bash
# Create PV
kubectl apply -f basic-pv.yaml

# Create PVC
kubectl apply -f basic-pvc.yaml

# View PVs
kubectl get pv
kubectl describe pv <pv-name>

# View PVCs
kubectl get pvc
kubectl describe pvc <pvc-name>

# View storage classes
kubectl get storageclass

# Delete PVC (PV reclaim depends on policy)
kubectl delete pvc <pvc-name>

# Delete PV
kubectl delete pv <pv-name>
```

## 🎯 Practice Exercises

1. **Create a PV and PVC** and verify binding
2. **Mount PVC in a pod** and write data
3. **Delete the pod** and recreate it - verify data persists
4. **Create a storage class** and use dynamic provisioning
5. **Test different access modes** (RWO, ROX, RWX)

## 📖 Next Steps

Now that you understand PVs and PVCs, learn about:
- **StatefulSets**: Use PVCs for stateful applications
- **Storage Classes**: Dynamic provisioning
- **Volume Snapshots**: Backup and restore


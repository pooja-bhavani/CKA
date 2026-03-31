# Storage Classes and Dynamic Provisioning

Prior to Kubernetes introducing dynamic volume provisioning, cluster administrators had to manually create `PersistentVolume` objects before users could create `PersistentVolumeClaim` (PVC) objects to request storage. This manual process didn't scale well.

## StorageClass (SC)
A **StorageClass** provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels (e.g., SSD vs. HDD on AWS EBS), backup policies, or arbitrarily defined policies determined by the cluster administrators.

### The Provisioner
When a user creates a PVC that requests a specific `storageClassName`, if that class exists and the provisioner plugin is available, Kubernetes will *dynamically provision* a PV that matches the PVC requirements, binding them automatically.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iopsPerGB: "10"
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
```

## Using a StorageClass in a PVC
To dynamically provision storage, a user specifies a `storageClassName` in the PVC. If `storageClassName` is omitted, the *default* storage class (if one exists) is used.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-fast-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 50Gi
```

### Volume Expansion
If the `StorageClass` has `allowVolumeExpansion: true` configured, you can expand a bound PVC simply by editing the PVC definition and changing the `storage` request to a larger value. You cannot decrease the size of a volume.

### Exam Tip 💡
In the CKA exam, you might be asked to change the default StorageClass. To do this, you use the annotation `storageclass.kubernetes.io/is-default-class`. You must first find the current default, change its annotation to `false`, and then annotate the new storage class with `true`.

```bash
kubectl patch storageclass <old-default-class> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
kubectl patch storageclass <new-default-class> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

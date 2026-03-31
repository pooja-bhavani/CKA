# Volumes & Types in Kubernetes

Containers are ephemeral by default. When a container crashes, kubelet will restart it, but the files will be lost. To solve this, Kubernetes provides the concept of a **Volume**.

## Volume Concepts
A Volume is essentially a directory, possibly with some data in it, which is accessible to the containers in a Pod. At its core, a volume outlives any containers that run within the Pod, and data is preserved across container restarts.

### Common Volume Types
1. **emptyDir**: Created when a Pod is assigned to a Node. It exists as long as that Pod is running on that node. If the Pod is evicted, the emptyDir is deleted forever. Use cases include scratch space.
2. **hostPath**: Mounts a file or directory from the host node's filesystem into your Pod. This is rarely used in production because tied to a specific node.
3. **configMap**: Provides a way to inject configuration data into Pods. 
4. **secret**: Similar to configMap but used for sensitive information. Keep in mind secrets are stored in etcd as base64 encoded strings, so RBAC and encryption at rest are necessary.

## Access Modes
When working with Persistent Volumes (PVs), you define how the volume can be mounted. The access modes are:
- `ReadWriteOnce` (RWO): the volume can be mounted as read-write by a single node.
- `ReadOnlyMany` (ROX): the volume can be mounted read-only by many nodes.
- `ReadWriteMany` (RWX): the volume can be mounted as read-write by many nodes.
- `ReadWriteOncePod` (RWOP): the volume can be mounted as read-write by a single Pod.

## Reclaim Policies
When a user is done with their volume, they can delete the PVC objects from the API that allows reclamation of the resource. The reclaim policy for a PersistentVolume tells the cluster what to do with the volume after it has been released of its claim.
- **Retain**: Allows for manual reclamation of the resource. The PV is still considered "released" when the claim is deleted, but the volume is not yet available to another claim.
- **Delete**: Removes both the PersistentVolume object from Kubernetes, as well as the associated storage asset in the external infrastructure (e.g., AWS EBS).
- **Recycle** (Deprecated): Basic scrub (`rm -rf /thevolume/*`) and makes it available again.

### Exam Tip 💡
Expect to be asked to add an `emptyDir` or `hostPath` to an existing Pod manifest, or create a Pod that mounts a `secret` as a file. Always remember to add the `volumeMounts` in the container spec and the `volumes` object in the pod spec.

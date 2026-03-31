# Persistent Volumes (PV) and Persistent Volume Claims (PVC)

Because Pods come and go, relying on node-local storage (like `emptyDir` or `hostPath`) is insufficient for stateful applications like databases. 

Kubernetes solves this by abstracting storage using two API resources: **PersistentVolume** and **PersistentVolumeClaim**.

## PersistentVolume (PV)
A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster just like a node is a cluster resource.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

## PersistentVolumeClaim (PVC)
A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

### Binding
When you create a PVC, the control plane looks for a PV that satisfies the claim's requirements (sufficient storage, matching access mode, matching storage class). If it finds one, it binds them together. **If a matching PV does not exist, the PVC will remain in the `Pending` state indefinitely until a matching PV is created.**

### Using a PVC as a Volume in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```

### Exam Tip 💡
You will heavily be tested on creating and binding PVs and PVCs. Make sure to check the exact capitalization of `storageClassName` and `accessModes` in the documentation during the exam, and always check the STATUS of the PVC (`kubectl get pvc`) to ensure it says `Bound` and not `Pending`.

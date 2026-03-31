# Application Troubleshooting & Output Streams

Troubleshooting application failures in Kubernetes requires methodically inspecting pods, deployments, and their logs.

## The Troubleshooting Flow
If a Pod is failing, what state is it in?
1. **Pending**: Pod cannot be scheduled onto a node. 
   - Check `kubectl describe pod <pod-name>`.
   - Usually caused by insufficient CPU/Memory, unresolved PVCs, or taints/tolerations mismatches.
2. **Waiting / ContainerCreating**: Sometimes stuck pulling images.
   - `ErrImagePull` or `ImagePullBackOff` indicates a typo in the image name, a missing registry secret, or lack of internet access.
3. **CrashLoopBackOff**: The container starts, crashes immediately, and kubelet keeps trying to restart it.
   - You MUST check the container logs to find out why the application code is crashing.

## Evaluating Container Logs
Viewing application standard output (stdout) and standard error (stderr) is essential.

```bash
# View logs for a running pod
kubectl logs <pod-name>

# View logs and follow/tail them
kubectl logs -f <pod-name>

# View logs for a specific container in a multi-container pod
kubectl logs <pod-name> -c <container-name>

# View logs from a previous instantiation of a crashed container
kubectl logs <pod-name> --previous
```

## Monitoring Metrics
To understand if your applications are causing node strain, use the Metrics Server (assuming it's installed in the cluster).

### Top Commands
```bash
# View resource consumption (CPU/Memory) of nodes
kubectl top nodes

# View resource consumption of Pods across all namespaces
kubectl top pods -A --sort-by=cpu
```

If a Pod exceeds its memory limit (`limits.memory`), the kernel will kill it. The pod status will show `OOMKilled` (Out Of Memory Killed). You will see this in `kubectl describe pod`. Note that exceeding CPU limits results in CPU throttling, which degrades performance but does *not* kill the pod. 

## Executing into Containers
If logs do not give you the full picture, you can execute an interactive shell inside the container to inspect config files or test connectivity.

```bash
kubectl exec -it <pod-name> -- /bin/sh
# or /bin/bash depending on the base image
```

From inside the container, you can use `curl` or `nc` to test connections or cat configuration files to ensure config maps were mounted correctly.

### Exam Tip 💡
If a pod is in `CrashLoopBackOff` and you run `kubectl logs <pod-name>`, but it returns nothing, it means the *current* container hasn't produced logs yet before crashing. Always try `kubectl logs <pod-name> --previous` to see what the *last* crashed container printed right before dying!

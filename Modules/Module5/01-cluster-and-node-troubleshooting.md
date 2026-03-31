# Troubleshooting Clusters and Nodes

The Troubleshooting domain accounts for **30%** of the CKA exam. This section focuses on diagnosing issues with the Kubernetes nodes and control plane components.

## Troubleshooting a Worker Node
Nodes will occasionally fail, become network partitioned, or run out of resources. When a node status is `NotReady`, here is your troubleshooting path:

1. **Check Node Status & Events**

   ```bash
   kubectl get nodes
   kubectl describe node <node-name>
   ```
   
   Look at the "Conditions" near the bottom of the describe output. Are there `MemoryPressure` or `DiskPressure` true statements? 

2. **Access the Node**
   If you have SSH access (which you will in the exam), SSH into the node:

   ```bash
   ssh <node-name>
   ```

3. **Check Kubelet Status**
   The `kubelet` service is responsible for communicating with the control plane. If `kubelet` is down, the node becomes `NotReady`.

   ```bash
   sudo systemctl status kubelet
   ```
   If it's inactive/failed:

   ```bash
   sudo systemctl restart kubelet
   sudo systemctl enable kubelet
   ```

4. **Check Kubelet Logs**
   If `kubelet` is running but failing, check journald:

   ```bash
   sudo journalctl -u kubelet -f
   ```

## Troubleshooting Cluster Components (Control Plane)
The control plane components (kube-apiserver, etcd, kube-scheduler, kube-controller-manager) dictate the health of the entire cluster. In a `kubeadm` setup (like the exam), these run as Static Pods.

### Finding Static Pod Manifests
By default, `kubelet` looks for static pod manifests in `/etc/kubernetes/manifests/`.
If you suspect one of the control plane components is misconfigured, look here:

```bash
cd /etc/kubernetes/manifests/
ls -l
```

If you make a typo in these YAML files, `kubelet` will fail to start the pod. **There is no direct validation**. The component will just silently fail to come up.

### Viewing Component Logs
Since control plane components are running as pods (usually in the `kube-system` namespace), you can check their logs via `kubectl`:

```bash
kubectl logs -n kube-system <pod-name>
```

However, if the `kube-apiserver` itself is down, `kubectl` won't work (it will say `The connection to the server <ip>:<port> was refused`).
In this scenario, you must SSH into the control plane node and use `crictl` or `docker` (if using Docker runtime, although CKA uses containerd/crictl now) to find the containers and view logs:

```bash
crictl ps -a
crictl logs <container-id>
```

### Exam Tip 💡
A very common exam task is fixing a broken `kubelet`. It involves SSHing into the node, running `systemctl status kubelet`, finding an issue (often a missing certificate path or an incorrect config argument), correcting it in `/var/lib/kubelet/config.yaml` or `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`, running `systemctl daemon-reload`, and restarting the kubelet.

# Troubleshooting Networking and Services

Network issues account for a large portion of the Troubleshooting domain on the CKA exam.

## Service Connectivity
When a Pod cannot reach another Pod, or external traffic cannot reach a service, follow these steps:

### 1. Verify Endpoints
A `Service` routes traffic to Pods through an intermediate resource called an `Endpoint` (or `EndpointSlice`).
The service selector tries to match pod labels. If labels don't match, the service has no endpoints.

```bash
# Check if the service has endpoints
kubectl get endpoints <service-name>
kubectl describe svc <service-name>
```

If endpoints show `<none>`, the labels on the service selector do not match the labels on the deployed pods.

### 2. DNS Resolution
If pods can communicate by IP address but not by service name, the cluster's DNS provider (usually CoreDNS) is the culprit.

You can spin up a quick debug pod to test DNS resolution:

```bash
kubectl run dns-tester --image=busybox:1.28 --rm -it --restart=Never -- nslookup <service-name>
```

If it fails to resolve:
1. Check the `kube-dns` service in the `kube-system` namespace.
2. Check the `coredns` deployment pods. Are they running or in `CrashLoopBackOff`?
3. Check the logs of the CoreDNS pods for syntax errors in the Corefile config map.

```bash
kubectl logs -n kube-system -l k8s-app=kube-dns
```

### 3. Check Network Policies (CNI)
If endpoints exist and DNS resolves, but `curl` or `ping` requests time out between pods, verify the **NetworkPolicies** in the namespace.

By default, pods accept traffic from any source. Once a `NetworkPolicy` selects a pod, that pod becomes isolated.

```bash
kubectl get networkpolicies -n <namespace>
kubectl describe networkpolicy <policy-name> -n <namespace>
```
Ensure that the namespace label selector, pod selector, and port numbers exactly match the intent.

### 4. Kube-Proxy Issues
`kube-proxy` runs on every node and maintains the iptables or IPVS rules that route traffic for Services.
If services are completely broken cluster-wide, inspect `kube-proxy`:

```bash
kubectl get daemonsets -n kube-system kube-proxy
kubectl logs -n kube-system ds/kube-proxy
```

## Examining the CNI Plugin
In a real-world scenario (and the exam), clusters are provisioned with varied Container Network Interface (CNI) plugins like Calico, Flannel, or Cilium.

If nodes are `Ready` but pods remain in `ContainerCreating` indefinitely, the CNI might be missing or broken.

```bash
# Check for CNI config files on the node
ls -l /etc/cni/net.d/
```

### Exam Tip 💡
Always test connectivity from *inside* the cluster first. Don't try to curl a ClusterIP from your laptop or the control plane node unless you are sure traffic routing works. The easiest way is using `kubectl run temp-test --image=curlimages/curl --rm -it -- curl -m 3 http://service-name`.

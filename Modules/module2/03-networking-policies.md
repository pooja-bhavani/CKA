# Kubernetes Network Policies

## Overview

Network Policies are Kubernetes resources that control traffic flow between pods and network endpoints. They act as a firewall for your cluster, defining rules for ingress (incoming) and egress (outgoing) traffic.

## Why Network Policies Matter

- **Security**: Implements zero-trust networking and microsegmentation
- **Isolation**: Prevents unauthorized access between namespaces and pods
- **Compliance**: Meets regulatory requirements for network segmentation

---

## Prerequisites

Network Policies require a CNI plugin that supports them:
- **Supported**: Calico, Cilium, Weave Net, Romana
- **Not Supported**: Flannel (basic version)

**Check if your CNI supports Network Policies**:
```bash
# Check CNI plugin
kubectl get pods -n kube-system | grep -E 'calico|cilium|weave'

# Test with a simple policy
kubectl apply -f test-netpolicy.yaml
kubectl describe networkpolicy test-netpolicy
```
---

## How Network Policies Work

### Default Behavior
- **Without Network Policies**: All traffic is allowed (open by default)
- **With Network Policies**: Once a pod is selected by any policy, it becomes isolated and only explicitly allowed traffic is permitted

### Policy Types
1. **Ingress**: Controls incoming traffic to pods
2. **Egress**: Controls outgoing traffic from pods

---

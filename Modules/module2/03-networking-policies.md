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

## Basic Network Policy Structure

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-policy
  namespace: default
spec:
  podSelector:          # Which pods this policy applies to
    matchLabels:
      app: myapp
  policyTypes:          # Types of traffic to control
    - Ingress
    - Egress
  ingress:              # Ingress rules
    - from:
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 80
  egress:               # Egress rules
    - to:
        - podSelector:
            matchLabels:
              role: database
      ports:
        - protocol: TCP
          port: 5432
```

---

## Common Use Cases and Patterns

### Pattern 1: Default Deny All Traffic

**Use case**: Start with zero-trust, then explicitly allow needed traffic

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}  # Empty selector = all pods in namespace
  policyTypes:
    - Ingress
    - Egress
```

**What it does**: Blocks all ingress and egress traffic for all pods in the namespace

---

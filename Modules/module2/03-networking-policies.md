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

**What it does**: It blocks all ingress and egress traffic for all pods in the namespace

---

### Pattern 2: Allow Traffic from Specific Pods

**Use case**: Frontend pods can access backend, but nothing else can

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

---

## Troubleshooting Guide

### Examples

#### Example 1: Policy Not Working (Traffic Still Flows)

**Error**:
- Applied Network Policy but pods can still communicate
- No traffic blocking observed

**Debug Steps**:
```bash
# 1. Check if CNI supports Network Policies
kubectl get pods -n kube-system | grep -E 'calico|cilium|weave'

# 2. Verify policy exists
kubectl get networkpolicy -A

# 3. Check policy details
kubectl describe networkpolicy <policy-name> -n <namespace>

# 4. Verify pod labels match policy selector
kubectl get pods --show-labels -n <namespace>

# 5. Check if policy is selecting pods
kubectl get networkpolicy <policy-name> -n <namespace> -o yaml
```

**Solutions**:
- Install a CNI that supports Network Policies (Calico, Cilium)
- Verify pod labels match `podSelector`
- Ensure policy is in the correct namespace
- Check for conflicting policies that might allow traffic

---

### Example 2: Pod Cannot Communicate After Policy Applied

**Error**:
```bash
# From inside pod
wget: can't connect to remote host: Connection timed out
```
**Debug Steps**:
```bash
# 1. Check which policies affect the pod
kubectl get networkpolicy -n <namespace>
kubectl describe networkpolicy -n <namespace>

# 2. Verify pod labels
kubectl get pod <pod-name> -n <namespace> --show-labels

# 3. Test connectivity
kubectl exec -it <pod-name> -n <namespace> -- wget -O- --timeout=5 http://<target-service>

# 4. Check if DNS is allowed
kubectl exec -it <pod-name> -n <namespace> -- nslookup kubernetes.default

# 5. Review policy rules
kubectl get networkpolicy <policy-name> -n <namespace> -o yaml
```

**Solutions**:
```yaml
# Add DNS egress rule (most common fix)
egress:
  - to:
      - namespaceSelector:
          matchLabels:
            name: kube-system
    ports:
      - protocol: UDP
        port: 53

# Or allow all DNS
egress:
  - to:
      - namespaceSelector: {}
    ports:
      - protocol: UDP
        port: 53
```

---

### Example 4: Default Deny Blocks Everything

**Error**:
- Applied default deny policy
- Nothing works, including DNS

**Debug Steps**:
```bash
# Check all policies in namespace
kubectl get networkpolicy -n <namespace>

# Test DNS
kubectl exec -it <pod-name> -n <namespace> -- nslookup google.com
```

**Solution**:
```yaml
# Always allow DNS when using default deny
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector: {}
      ports:
        - protocol: UDP
          port: 53
```

---

## Best Practices

1. **Start with Default Deny**: Apply default deny first, then explicitly allow needed traffic
2. **Always Allow DNS**: Include DNS egress rules in every policy
3. **Use Namespace Labels**: Label namespaces for easier policy management
4. **Test Incrementally**: Apply policies one at a time and test
5. **Use Descriptive Names**: Name policies clearly (e.g., `allow-frontend-to-backend`)
6. **Monitor Impact**: Check logs and metrics after applying policies
7. **Version Control**: Store policies in Git for tracking changes

---

## Exam Tips

1. **Know the structure**: `podSelector`, `policyTypes`, `ingress`, `egress`
2. **Empty selector `{}`**: Matches all pods in namespace
3. **DNS is critical**: Always include DNS egress rules
4. **Label namespaces**: Required for `namespaceSelector` to work
5. **Test with wget/curl**: Use `kubectl exec` to test connectivity
6. **Default behavior**: No policy = all allowed; any policy = default deny
7. **Multiple policies**: Policies are additive (union of all rules)
8. **Practice scenarios**: Default deny, namespace isolation, multi-tier apps

---



















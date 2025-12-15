# CoreDNS Configuration and Troubleshooting

## Overview

CoreDNS is the default DNS server in Kubernetes clusters (since v1.13). It provides service discovery by resolving service names to cluster IPs, enabling pods to communicate using DNS names instead of IP addresses.

## Why CoreDNS Matters

- **Service Discovery**: Pods find services by name (e.g., `backend-service`)
- **Cross-Namespace Communication**: Access services in other namespaces
- **External DNS Resolution**: Resolves external domain names
- **Custom DNS Configuration**: Add custom DNS entries and forwarding rules

## How Kubernetes DNS Works

### DNS Naming Convention

```
<service-name>.<namespace>.svc.cluster.local
```

**Examples**:
```bash
# Same namespace
curl http://backend-service

# Different namespace
curl http://backend-service.production

# Fully qualified domain name (FQDN)
curl http://backend-service.production.svc.cluster.local

# External domain
curl http://google.com
```

### DNS Resolution Flow

1. Pod makes DNS query (e.g., `backend-service`)
2. Query sent to CoreDNS (usually at `10.96.0.10`)
3. CoreDNS checks:
   - Is it a Kubernetes service? → Return ClusterIP
   - Is it external? → Forward to upstream DNS
4. Response returned to pod

---

## Common DNS Patterns

### Pattern 1: Custom DNS Entries

**Use case**: Add custom DNS records for external services

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
        }
        # Custom hosts
        hosts {
           192.168.1.100 custom-db.example.com
           fallthrough
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```
**Apply changes**:
```bash
kubectl edit configmap coredns -n kube-system
# CoreDNS will auto-reload
```
---

### Pattern 2: Increase Cache TTL

**Use case**: Reduce DNS query load for stable services

```yaml
data:
  Corefile: |
    .:53 {
        errors
        health
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 300  # Increase from 30 to 300 seconds
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 300  # Increase cache duration
        loop
        reload
        loadbalance
    }
```

---

## Troubleshooting Guide

### Example 1: DNS Resolution Fails (nslookup fails)

**ERRor**:
```bash
kubectl exec -it test-pod -- nslookup kubernetes.default
# Server:    10.96.0.10
# Address 1: 10.96.0.10
# nslookup: can't resolve 'kubernetes.default'
```

**Debug Steps**:
```bash
# 1. Check if CoreDNS pods are running
kubectl get pods -n kube-system -l k8s-app=kube-dns

# 2. Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# 3. Check CoreDNS service
kubectl get svc -n kube-system kube-dns

# 4. Verify DNS service IP
kubectl get svc -n kube-system kube-dns -o jsonpath='{.spec.clusterIP}'

# 5. Check pod's DNS configuration
kubectl exec -it test-pod -- cat /etc/resolv.conf

# 6. Test DNS from node
nslookup kubernetes.default.svc.cluster.local 10.96.0.10
```

**Solutions**:

**A. CoreDNS pods not running**:
```bash
# Check pod status
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Describe pod for errors
kubectl describe pod -n kube-system <coredns-pod>

# Restart CoreDNS
kubectl rollout restart deployment coredns -n kube-system
```
**B. Wrong DNS service IP in pod**:
```bash
# Check kubelet DNS configuration
# On node:
cat /var/lib/kubelet/config.yaml | grep -A 2 clusterDNS

# Should match kube-dns service IP
kubectl get svc -n kube-system kube-dns -o jsonpath='{.spec.clusterIP}'
```

**C. CoreDNS configuration error**:
```bash
# Check for syntax errors in Corefile
kubectl get configmap coredns -n kube-system -o yaml

# Validate by checking CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns | grep -i error
```

---





















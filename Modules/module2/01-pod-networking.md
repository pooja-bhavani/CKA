# Pod Networking and CNI 

## Table of Contents

1. [Overview](#overview)


## Overview

Pod networking is the foundation of Kubernetes networking. Every pod gets its own IP address, and pods can communicate with each other across nodes without NAT. 
The Container Network Interface (CNI) is the plugin architecture that makes this possible.

## Why Pod Networking Matters

- **Flat Network**: All pods can communicate with all other pods without NAT
- **Unique IPs**: Each pod gets its own IP address
- **Cross-Node Communication**: Pods on different nodes can talk directly
- **Network Isolation**: Network policies to control traffic between pods
- **Service Discovery**: Services rely on pod networking

## CKA Exam Importance

Pod networking and CNI are critical for the exam:
- **10-15% of exam weight** in networking domain
- Must understand how pods communicate across nodes
- Know how to troubleshoot CNI issues
- Understand different CNI plugins and when to use them
- Debug pod connectivity problems

---

## Kubernetes Network Model

Kubernetes imposes the following fundamental requirements on any networking implementation:

1. **Pods can communicate with all other pods** on any node without NAT
2. **Agents on a node** (e.g., kubelet) can communicate with all pods on that node
3. **Pods in the host network** can communicate with all pods on all nodes without NAT

### Network Model Diagram
```
┌─────────────────────────────────────────────────────────────┐
│                         Cluster                             │
│                                                             │
│  ┌──────────────────┐              ┌──────────────────┐     │
│  │   Node 1         │              │   Node 2         │     │
│  │                  │              │                  │     │
│  │  ┌────────────┐  │              │  ┌────────────┐  │     │
│  │  │ Pod A      │  │              │  │ Pod C      │  │     │
│  │  │ 10.244.1.2 │◄─┼──────────────┼─►│ 10.244.2.2 │  │     │
│  │  └────────────┘  │              │  └────────────┘  │     │
│  │                  │              │                  │     │
│  │  ┌────────────┐  │              │  ┌────────────┐  │     │
│  │  │ Pod B      │  │              │  │ Pod D      │  │     │
│  │  │ 10.244.1.3 │  │              │  │ 10.244.2.3 │  │     │
│  │  └────────────┘  │              │  └────────────┘  │     │
│  │                  │              │                  │     │
│  │  CNI Plugin      │              │  CNI Plugin      │     │
│  │  (e.g., Calico)  │              │  (e.g., Calico)  │     │
│  └──────────────────┘              └──────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1. Network Namespace

Each pod gets its own network namespace:

```bash
# On a node, list network namespaces
sudo ip netns list

# Example output:
# cni-12345678-abcd-1234-5678-123456789abc (id: 0)
# cni-87654321-dcba-4321-8765-987654321cba (id: 1)

# Inspect a pod's network namespace
sudo ip netns exec <namespace-id> ip addr
```

### 3. Virtual Ethernet (veth) Pairs

CNI creates veth pairs to connect pod to node:

```
┌─────────────────────────────────────┐
│           Node                      │
│                                     │
│  ┌──────────────────┐               │
│  │  Pod Namespace   │               │
│  │                  │               │
│  │  eth0 ◄──────────┼───► vethXXX   │
│  │  10.244.1.2      │      (bridge) │
│  │                  │               │
│  └──────────────────┘               │
│                                     │
│         cni0 bridge                 │
│         10.244.1.1                  │
│                                     │
└─────────────────────────────────────┘
```


### 4. IP Address Management (IPAM)

CNI plugins manage IP address allocation:

- **host-local**: Maintains local IP address database on each node
- **DHCP**: Uses DHCP server for IP allocation
- **Calico IPAM**: Distributed IP address management

---

## CNI (Container Network Interface)

### What is CNI?

CNI is a specification and set of libraries for configuring network interfaces in Linux containers. It consists of:

1. **CNI Specification**: Defines the interface between container runtime and network plugins
2. **CNI Plugins**: Implement the actual networking functionality

### CNI Configuration

CNI configuration is stored in `/etc/cni/net.d/`:

```bash
# View CNI configuration
cat /etc/cni/net.d/10-calico.conflist
```
**Example CNI Configuration**:
```json
{
  "name": "k8s-pod-network",
  "cniVersion": "1.0.0",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "datastore_type": "kubernetes",
      "nodename": "node1",
      "ipam": {
        "type": "calico-ipam",
        "assign_ipv4": "true",
        "assign_ipv6": "false"
      },
      "policy": {
        "type": "k8s"
      },
      "kubernetes": {
        "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      },
      "container_settings": {
        "allow_ip_forwarding": false
      }
    },
    {
      "type": "bandwidth",
      "capabilities": {
        "bandwidth": true
      }
    }
  ]
}
```

### CNI Plugin Binaries

CNI plugin binaries are stored in `/opt/cni/bin/`:

```bash
# List CNI plugins
ls /opt/cni/bin/

# Common plugins:
# - bridge, host-device, ipvlan, macvlan, ptp, vlan
# - dhcp, host-local, static
# - bandwidth, firewall, portmap, tuning
# - calico, flannel, weave, cilium (third-party)
```

---

## Popular CNI Plugins

### 1. Calico

**What it is**: Layer 3 networking with BGP routing and network policy support

**Features**:
- Pure Layer 3 approach (no overlay)
- BGP (Border Gateway Protocol) routing between nodes
- Network policy enforcement
- IP-in-IP or VXLAN encapsulation (optional)
- High performance

**When to use**:
- In Production environments
- When you need network policies
- Large-scale deployments
- On-premises or cloud

**Installation**:
```bash
# Install Calico
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml

# Verify installation
kubectl get pods -n kube-system | grep calico
```

**Architecture**:
```
┌─────────────────────────────────────────┐
│  Node 1                                 │
│  ┌─────────┐  ┌─────────┐               │
│  │ Pod A   │  │ Pod B   │               │
│  └────┬────┘  └────┬────┘               │
│       │            │                    │
│  ┌────▼────────────▼────┐               │
│  │   calico-node        │               │
│  │   (Felix + BIRD)     │               │
│  └──────────┬───────────┘               │
│             │ BGP                       │
└─────────────┼───────────────────────────┘
              │
              │ BGP Peering
              │
┌─────────────┼───────────────────────────┐
│             │                           │
│  ┌──────────▼───────────┐               │
│  │   calico-node        │               │
│  │   (Felix + BIRD)     │               │
│  └──────────┬───────────┘               │
│       ┌─────▼─────┐                     │
│       │   Pod C   │                     │
│       └───────────┘                     │
│  Node 2                                 │
└─────────────────────────────────────────┘
```

---

### 2. Flannel

**What it is**: Simple overlay network using VXLAN

**Features**:
- Easy to set up
- VXLAN overlay network
- No network policy support (basic version)
- Good for simple clusters

**When to use**:
- During Development/testing
- Simple networking requirements
- Don't need network policies
- Quick setup

**Installation**:
```bash
# Install Flannel
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Verify installation
kubectl get pods -n kube-system | grep flannel
```

### 3. Weave Net

**What it is**: Mesh network with automatic discovery

**Features**:
- Automatic mesh network
- Encryption support
- Network policy support
- Easy setup
- Works across different cloud providers

**When to use**:
- During Multi-cloud deployments
- Need encryption
- Automatic network discovery
- Network policies required

**Installation**:
```bash
# Install Weave Net
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

# Verify installation
kubectl get pods -n kube-system | grep weave
```

---

### 4. Cilium

**What it is**: (extended Berkeley Packet Filter) eBPF-based networking and security

**Features**:
- eBPF for high performance
- Advanced network policies (L7)
- Service mesh capabilities
- Observability and monitoring
- API-aware security

**When to use**:
- Need L7 network policies
- High-performance requirements
- Advanced observability
- Service mesh features

**Installation**:
```bash
# Install Cilium CLI
curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz
sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin

# Install Cilium
cilium install

# Verify installation
cilium status
```

---

## v1.35 Networking Enhancements

### 1. Enhanced Traffic Distribution

v1.35 introduces improved traffic distribution for Services:

```yaml
# v1.35 Enhanced Service with traffic distribution
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
  # NEW in v1.35: Enhanced traffic distribution
  trafficDistribution: PreferClose  # Routes to closest endpoints first
  internalTrafficPolicy: Local      # Keep traffic on same node when possible
```
**Benefits**:
- Reduced latency by routing to closest endpoints
- Better resource utilization
- Improved performance for latency-sensitive applications

### 2. User Namespaces Integration

Pod networking now works with user namespaces for enhanced security:

```yaml
# v1.35 Pod with user namespace isolation
apiVersion: v1
kind: Pod
metadata:
  name: isolated-pod
spec:
  hostUsers: false  # Enable user namespace isolation
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: nginx:1.21
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
```

**Benefits**:
- Enhanced container isolation
- Root inside container maps to unprivileged user on host
- Better security without sacrificing functionality

### 3. In-Place Pod Resource Updates

v1.35 allows updating Pod resources without restart:

```bash
# Update Pod resources without restart (NEW in v1.35)
kubectl patch pod my-pod --type='merge' -p='{
  "spec": {
    "containers": [{
      "name": "app",
      "resources": {
        "limits": {"cpu": "500m", "memory": "1Gi"}
      }
    }]
  }
}'

# Verify Pod didn't restart (check AGE and RESTARTS)
kubectl get pod my-pod -o wide
```

### 4. Enhanced Network Observability

```bash
# v1.35 Network Performance Monitoring

# Check network performance metrics
kubectl top pods --containers
kubectl top nodes

# Monitor CNI plugin performance
kubectl logs -n kube-system -l k8s-app=calico-node --tail=100 | grep -i performance

# Check for network bottlenecks
kubectl get events --field-selector reason=NetworkNotReady
```

---

## Pod-to-Pod Communication Scenarios

### Scenario: Same Node Communication

```
┌─────────────────────────────────────┐
│           Node 1                    │
│                                     │
│  ┌──────────┐      ┌──────────┐     │
│  │  Pod A   │      │  Pod B   │     │
│  │10.244.1.2│      │10.244.1.3│     │
│  └────┬─────┘      └─────┬────┘     │
│       │                  │          │
│       └────────┬─────────┘          │
│                │                    │
│         ┌──────▼──────┐             │
│         │ cni0 bridge │             │
│         └─────────────┘             │
│                                     │
└─────────────────────────────────────┘

Flow:
1. Pod A sends packet to 10.244.1.3
2. Packet goes through veth to cni0 bridge
3. Bridge forwards to Pod B's veth
4. Pod B receives packet
```
## Troubleshooting Pod Networking
### Example 1: Pod Cannot Reach Other Pods

**Error**:
```bash
kubectl exec -it pod-a -- ping 10.244.2.2
# PING 10.244.2.2 (10.244.2.2): 56 data bytes
# Request timeout
```

**Debug Steps**:
```bash
# 1. Check if pod has IP address
kubectl get pod pod-a -o wide

# 2. Check CNI pods are running
kubectl get pods -n kube-system | grep -E 'calico|flannel|weave|cilium'

# 3. Check CNI logs
kubectl logs -n kube-system <cni-pod-name>

# 4. Verify CNI configuration on node
ssh <node>
cat /etc/cni/net.d/*

# 5. Check routing table in pod
kubectl exec -it pod-a -- ip route

# 6. Check node routing
ssh <node>
ip route | grep cni

# 7. Test from node to pod
ssh <node>
ping 10.244.2.2
```

**Solutions**:

**A. CNI pods not running**:
```bash
# Restart CNI daemonset
kubectl rollout restart daemonset <cni-name> -n kube-system

# Example for Calico
kubectl rollout restart daemonset calico-node -n kube-system
```

**B. CNI configuration missing**:
```bash
# Reinstall CNI plugin
kubectl apply -f <cni-manifest-url>
```
---


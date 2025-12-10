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
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "datastore_type": "kubernetes",
      "nodename": "node1",
      "ipam": {
        "type": "calico-ipam"
      },
      "policy": {
        "type": "k8s"
      },
      "kubernetes": {
        "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
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
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml

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
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

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


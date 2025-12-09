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
│                         Cluster                              │
│                                                              │
│  ┌──────────────────┐              ┌──────────────────┐    │
│  │   Node 1         │              │   Node 2         │    │
│  │                  │              │                  │    │
│  │  ┌────────────┐  │              │  ┌────────────┐  │    │
│  │  │ Pod A      │  │              │  │ Pod C      │  │    │
│  │  │ 10.244.1.2 │◄─┼──────────────┼─►│ 10.244.2.2 │  │    │
│  │  └────────────┘  │              │  └────────────┘  │    │
│  │                  │              │                  │    │
│  │  ┌────────────┐  │              │  ┌────────────┐  │    │
│  │  │ Pod B      │  │              │  │ Pod D      │  │    │
│  │  │ 10.244.1.3 │  │              │  │ 10.244.2.3 │  │    │
│  │  └────────────┘  │              │  └────────────┘  │    │
│  │                  │              │                  │    │
│  │  CNI Plugin      │              │  CNI Plugin      │    │
│  │  (e.g., Calico)  │              │  (e.g., Calico)  │    │
│  └──────────────────┘              └──────────────────┘    │
│                                                              │
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

## Overview

Pod networking is the foundation of Kubernetes networking. Every pod gets its own IP address, and pods can communicate with each other across nodes without NAT. 
The Container Network Interface (CNI) is the plugin architecture that makes this possible.

## Why Pod Networking Matters

- **Flat Network**: All pods can communicate with all other pods without NAT
- **Unique IPs**: Each pod gets its own IP address
- **Cross-Node Communication**: Pods on different nodes can talk directly
- **Network Isolation**: Network policies to control traffic between pods
- **Service Discovery**: Services rely on pod networking

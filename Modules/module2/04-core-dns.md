# CoreDNS Configuration and Troubleshooting

## Overview

CoreDNS is the default DNS server in Kubernetes clusters (since v1.13). It provides service discovery by resolving service names to cluster IPs, enabling pods to communicate using DNS names instead of IP addresses.

## Why CoreDNS Matters

- **Service Discovery**: Pods find services by name (e.g., `backend-service`)
- **Cross-Namespace Communication**: Access services in other namespaces
- **External DNS Resolution**: Resolves external domain names
- **Custom DNS Configuration**: Add custom DNS entries and forwarding rules


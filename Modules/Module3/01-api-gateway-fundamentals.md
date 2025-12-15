# Gateway API Fundamentals

## Overview

Gateway API is the next-generation Kubernetes API for managing ingress traffic. It's a more expressive, extensible, and role-oriented successor to the Ingress API. Gateway API provides a standardized way to configure load balancing, traffic routing, and service mesh capabilities.

## Why Gateway API Matters

- **Role-Oriented**: Separates concerns between infrastructure providers, cluster operators, and application developers
- **Expressive**: Supports advanced routing (header-based, weighted, mirroring)
- **Extensible**: Custom resources and vendor-specific features
- **Portable**: Works across different implementations (Envoy, Istio, NGINX, etc.)
- **Type-Safe**: Strongly typed API with better validation

## Importance

Gateway API is increasingly important for modern Kubernetes:
- **Emerging topic** in CKA exam (5-10% weight expected)
- Replaces traditional Ingress in many scenarios
- Understanding migration from Ingress to Gateway API is critical
- Must know core objects: Gateway, HTTPRoute, GatewayClass

---

## Gateway API vs Ingress

| Feature | Ingress | Gateway API |
|---------|---------|-------------|
| **API Maturity** | Stable (v1) | Beta (v1beta1) |
| **Expressiveness** | Basic routing | Advanced routing |
| **Role Separation** | Single resource | Multiple resources |
| **Protocol Support** | HTTP/HTTPS | HTTP, HTTPS, TCP, UDP, gRPC |
| **Traffic Splitting** | Limited | Native support |
| **Header Routing** | Via annotations | Native support |
| **Extensibility** | Annotations | Custom resources |
| **Multi-tenancy** | Limited | Built-in |

---

## Core Concepts

### 1. GatewayClass

**What it is**: Defines the controller that will implement the Gateway (like StorageClass for storage)

**Who manages it**: Cluster administrator / Infrastructure provider

**Example**:
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: GatewayClass
metadata:
  name: istio
spec:
  controllerName: istio.io/gateway-controller
```

**Common GatewayClasses**:
- `istio`: Istio Gateway Controller
- `envoy`: Envoy Gateway
- `nginx`: NGINX Gateway Controller
- `traefik`: Traefik Gateway Controller

---


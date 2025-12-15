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
| **API Maturity** | Stable (v1) | Stable (v1) |
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

### 2. Gateway

**What it is**: Represents a load balancer instance that listens for traffic

**Who manages it**: Cluster operator

**Example**:
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: example-gateway
  namespace: default
spec:
  gatewayClassName: istio
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
```

**Key Fields**:
- `gatewayClassName`: Which GatewayClass to use
- `listeners`: Ports and protocols to listen on
- `allowedRoutes`: Which namespaces can attach routes

---

### 3. HTTPRoute

**What it is**: Defines HTTP routing rules (like Ingress rules)

**Who manages it**: Application developer

**Example**:
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: example-route
  namespace: default
spec:
  parentRefs:
    - name: example-gateway
  hostnames:
    - "example.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api
      backendRefs:
        - name: api-service
          port: 8080
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: web-service
          port: 80
```

---

### 4. Other Route Types

**TCPRoute**: Layer 4 TCP routing
**UDPRoute**: Layer 4 UDP routing
**TLSRoute**: TLS routing based on SNI
**GRPCRoute**: gRPC-specific routing

---


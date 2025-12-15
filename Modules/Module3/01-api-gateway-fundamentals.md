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
apiVersion: gateway.networking.k8s.io/v1
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
apiVersion: gateway.networking.k8s.io/v1
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
apiVersion: gateway.networking.k8s.io/v1
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

## Role-Oriented Design

Gateway API separates responsibilities:

```
┌─────────────────────────────────────────────────┐
│ Infrastructure Provider                         │
│ - Installs Gateway Controller                   │
│ - Creates GatewayClass                          │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ Cluster Operator                                │
│ - Creates Gateway instances                     │
│ - Configures listeners and policies             │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ Application Developer                           │
│ - Creates HTTPRoute/TCPRoute                    │
│ - Defines routing rules                         │
└─────────────────────────────────────────────────┘
```

---

## Basic Setup Example

### Step 1: Install Gateway API CRDs

```bash
# Install Gateway API CRDs 
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.1/standard-install.yaml

# Verify installation (should show v1 and v1beta1 as available versions for backward compatibility)
kubectl get crd | grep gateway
```

### Step 2: Install a Gateway Controller (Envoy Gateway Example)

```bash
# Install Envoy Gateway
kubectl apply --server-side -f https://github.com/envoyproxy/gateway/releases/download/v1.6.1/install.yaml


# Verify installation
kubectl get pods -n envoy-gateway-system
```

### Step 3: Create GatewayClass

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
```

### Step 4: Create Gateway

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
  namespace: default
spec:
  gatewayClassName: envoy
  listeners:
    - name: http
      protocol: HTTP
      port: 80
```

### Step 5: Create HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-route
  namespace: default
spec:
  parentRefs:
    - name: my-gateway
  hostnames:
    - "myapp.example.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: my-service
          port: 80
```

---

## Advanced Routing Scenarios

### Scenario 1: Path-Based Routing

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: path-routing
spec:
  parentRefs:
    - name: my-gateway
  hostnames:
    - "api.example.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /v1
      backendRefs:
        - name: api-v1
          port: 8080
    - matches:
        - path:
            type: PathPrefix
            value: /v2
      backendRefs:
        - name: api-v2
          port: 8080
    - matches:
        - path:
            type: Exact
            value: /health
      backendRefs:
        - name: health-service
          port: 8080
```

---

### Scenario 2: Header-Based Routing

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: header-routing
spec:
  parentRefs:
    - name: my-gateway
  rules:
    - matches:
        - headers:
            - name: X-Version
              value: beta
      backendRefs:
        - name: beta-service
          port: 8080
    - matches:
        - headers:
            - name: X-Version
              value: stable
      backendRefs:
        - name: stable-service
          port: 8080
```

---

### Scenario 3: Host-Based Routing

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: host-routing
spec:
  parentRefs:
    - name: my-gateway
  hostnames:
    - "api.example.com"
  rules:
    - backendRefs:
        - name: api-service
          port: 8080
---
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: web-routing
spec:
  parentRefs:
    - name: my-gateway
  hostnames:
    - "www.example.com"
  rules:
    - backendRefs:
        - name: web-service
          port: 80
```


---

## Real-World Use Cases

### Use Case 1: Multi-Tenant Platform

**Scenario**: SaaS platform with multiple customers, each with their own subdomain

```yaml
# Shared Gateway
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: saas-gateway
  namespace: platform
spec:
  gatewayClassName: envoy
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: Selector
          selector:
            matchLabels:
              tenant: "true"
---
# Customer 1 Route (in customer1 namespace)
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: customer1-route
  namespace: customer1
spec:
  parentRefs:
    - name: saas-gateway
      namespace: platform
  hostnames:
    - "customer1.saas.example.com"
  rules:
    - backendRefs:
        - name: customer1-app
          port: 80
---
# Customer 2 Route (in customer2 namespace)
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: customer2-route
  namespace: customer2
spec:
  parentRefs:
    - name: saas-gateway
      namespace: platform
  hostnames:
    - "customer2.saas.example.com"
  rules:
    - backendRefs:
        - name: customer2-app
          port: 80
```

---

### Use Case 2: Blue-Green Deployment

**Scenario**: Switch traffic between blue and green deployments

```yaml
# Initially: 100% blue
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: blue-green
spec:
  parentRefs:
    - name: my-gateway
  hostnames:
    - "app.example.com"
  rules:
    - backendRefs:
        - name: app-blue
          port: 80
          weight: 100
        - name: app-green
          port: 80
          weight: 0

# After validation: Switch to 100% green
# Just update weights:
#   - app-blue: weight: 0
#   - app-green: weight: 100
```

---

### Use Case 3: API Versioning

**Scenario**: Route API requests based on version in path or header

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api-versioning
spec:
  parentRefs:
    - name: api-gateway
  hostnames:
    - "api.example.com"
  rules:
    # Version in path: /v1/users
    - matches:
        - path:
            type: PathPrefix
            value: /v1
      backendRefs:
        - name: api-v1
          port: 8080
    # Version in path: /v2/users
    - matches:
        - path:
            type: PathPrefix
            value: /v2
      backendRefs:
        - name: api-v2
          port: 8080
    # Version in header: X-API-Version: v3
    - matches:
        - headers:
            - name: X-API-Version
              value: v3
      backendRefs:
        - name: api-v3
          port: 8080
```

---

## Checking Gateway Status

```bash
# Check Gateway status
kubectl get gateway

# Detailed Gateway status
kubectl describe gateway my-gateway

# Check HTTPRoute status
kubectl get httproute

# Describe HTTPRoute
kubectl describe httproute my-route

```

---

## Best Practices

1. **Separate Concerns**: Use different namespaces for infrastructure (Gateway) and applications (HTTPRoute)
2. **Use GatewayClass**: Define clear GatewayClasses for different environments (dev, staging, prod)
3. **Limit Route Attachment**: Use `allowedRoutes` to control which namespaces can attach routes
4. **Monitor Status**: Check Gateway and HTTPRoute status conditions regularly
5. **Use Weights for Rollouts**: Gradually shift traffic using weight-based routing
6. **Validate Before Production**: Test routes in staging with same Gateway configuration
7. **Document Hostnames**: Maintain a registry of hostnames and their owners
8. **Use TLS**: Always use HTTPS in production with proper certificates

---






















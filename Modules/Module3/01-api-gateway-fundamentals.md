# Gateway API Fundamentals

## Overview

Gateway API is the next-generation Kubernetes API for managing ingress traffic. It's a more expressive, extensible, and role-oriented successor to the Ingress API. Gateway API provides a standardized way to configure load balancing, traffic routing, and service mesh capabilities.

## Why Gateway API Matters

- **Role-Oriented**: Separates concerns between infrastructure providers, cluster operators, and application developers
- **Expressive**: Supports advanced routing (header-based, weighted, mirroring)
- **Extensible**: Custom resources and vendor-specific features
- **Portable**: Works across different implementations (Envoy, Istio, NGINX, etc.)
- **Type-Safe**: Strongly typed API with better validation
- Enhanced integration with Service Mesh and improved performance

## Importance

Gateway API is increasingly important for modern Kubernetes:
- **Emerging topic** in CKA exam (5-10% weight expected)
- **Stable API**: Gateway API v1.1+ is production-ready
- **Service Mesh Integration**: Better integration with Istio, Linkerd, and other service meshes
- **Enhanced Traffic Management**: Advanced traffic splitting and routing capabilities
- **Multi-cluster Support**: Cross-cluster traffic management

---

## Gateway API vs Ingress (Updated for v1.35)

| Feature | Ingress | Gateway API v1.35 |
|---------|---------|-------------------|
| **API Maturity** | Stable (v1) | Stable (v1.1) |
| **Expressiveness** | Basic routing | Advanced routing + Service Mesh |
| **Role Separation** | Single resource | Multiple resources |
| **Protocol Support** | HTTP/HTTPS | HTTP, HTTPS, TCP, UDP, gRPC, GRPC-Web |
| **Traffic Splitting** | Limited | Native support with weights |
| **Header Routing** | Via annotations | Native support |
| **Extensibility** | Annotations | Custom resources + Policy API |
| **Multi-tenancy** | Limited | Built-in with namespace isolation |
| **Service Mesh** | External | Native integration |
| **Cross-cluster** | Not supported | Native support |

---

## NEW v1.35: Gateway API v1.1 Features

### 1. Enhanced Service Mesh Integration
- **Native Service Mesh Support**: Direct integration with Istio, Linkerd
- **Mesh-wide Policies**: Traffic policies across service mesh
- **Cross-cluster Routing**: Route traffic between clusters

### 2. Policy API (Beta)
- **Security Policies**: Authentication, authorization at the gateway level
- **Traffic Policies**: Rate limiting, circuit breaking, retries
- **Observability Policies**: Tracing, metrics collection

### 3. Enhanced Protocol Support
- **GRPC-Web**: Native support for GRPC-Web protocol
- **WebSocket**: Better WebSocket handling
- **HTTP/3**: Experimental HTTP/3 support

---

## Installations required

These steps are required because you are installing Envoy Gateway, which uses the Kubernetes Gateway API instead of the traditional Kubernetes Ingress.

**Install CRDs**

```
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml
kubectl get crd | grep gateway
```


**Install Envoy Gateway**

```
kubectl apply -f https://github.com/envoyproxy/gateway/releases/download/v1.5.9/install.yaml
kubectl get pods -n envoy-gateway-system
```

```

```

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
  # NEW in v1.35: Enhanced parameters
  parametersRef:
    group: gateway.istio.io
    kind: IstioGatewayClass
    name: istio-config
```

**Common GatewayClasses**:
- `istio`: Istio Gateway Controller (v1.20+)
- `envoy`: Envoy Gateway (v1.0+)
- `nginx`: NGINX Gateway Controller (v1.1+)
- `traefik`: Traefik Gateway Controller (v3.0+)
- `cilium`: Cilium Gateway Controller (v1.14+)


### Hands‑on (bankapp)

Apply the Envoy GatewayClass 

**gatewayclass-envoy**                
[gatewayclass-envoy.yaml](../../k8s/gateway/01-gatewayclass-envoy.yaml)


```
kubectl apply -f k8s/gateway-api/01-gatewayclass-envoy.yaml
```

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
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: example-com-tls
    allowedRoutes:
      namespaces:
        from: All
  # NEW in v1.35: Enhanced address configuration
  addresses:
  - type: IPAddress
    value: "192.168.1.100"
```

**Key Fields**:
- `gatewayClassName`: Which GatewayClass to use
- `listeners`: Ports and protocols to listen on
- `allowedRoutes`: Which namespaces can attach routes
- `addresses`: Specific IP addresses to bind
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
    # NEW in v1.35: Enhanced backend references
    backendRefs:
    - name: api-service
      port: 8080
      weight: 90
    - name: api-service-canary
      port: 8080
      weight: 10
    # NEW in v1.35: Request/Response filters
    filters:
    - type: RequestHeaderModifier
      requestHeaderModifier:
        add:
        - name: X-Custom-Header
          value: "gateway-api"
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-service
      port: 80
```


### 4. NEW v1.35: Enhanced Route Types

**GRPCRoute** (Stable):
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GRPCRoute
metadata:
  name: grpc-route
spec:
  parentRefs:
  - name: example-gateway
  hostnames:
  - "grpc.example.com"
  rules:
  - matches:
    - method:
        service: "user.UserService"
        method: "GetUser"
    backendRefs:
    - name: user-service
      port: 9090
```

**TCPRoute** (Beta):
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: TCPRoute
metadata:
  name: tcp-route
spec:
  parentRefs:
  - name: example-gateway
    sectionName: tcp
  rules:
  - backendRefs:
    - name: database-service
      port: 5432
```
---

### 4. Other Route Types

**TCPRoute**: Layer 4 TCP routing
**UDPRoute**: Layer 4 UDP routing
**TLSRoute**: TLS routing based on SNI
**GRPCRoute**: gRPC-specific routing

---

## NEW: Policy API Integration

### Security Policy Example
```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: SecurityPolicy
metadata:
  name: api-security
  namespace: default
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: api-route
  authentication:
    jwt:
      providers:
      - name: auth0
        issuer: "https://example.auth0.com/"
        audiences:
        - "api.example.com"
  authorization:
    rules:
    - action: ALLOW
      from:
      - source:
          principals:
          - "user@example.com"
```

### Traffic Policy Example
```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: TrafficPolicy
metadata:
  name: api-traffic-policy
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: api-route
  rateLimit:
    requests: 100
    unit: minute
  retry:
    attempts: 3
    backoff: exponential
  timeout: 30s
```

---


## Role-Oriented Design

Gateway API separates responsibilities:
```
┌─────────────────────────────────────────────────┐
│ Infrastructure Provider                         │
│ - Installs Gateway Controller                   │
│ - Creates GatewayClass                          │
│ - Manages Policy APIs (NEW)                     │
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│ Platform Operator (NEW in v1.35)                │
│ - Creates shared Gateways                       │
│ - Manages cross-cutting policies                │
│ - Configures service mesh integration           │
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│ Cluster Operator                                │
│ - Creates Gateway instances                     │
│ - Configures listeners and policies             │
│ - Manages certificates and TLS                  │
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│ Application Developer                           │
│ - Creates HTTPRoute/GRPCRoute/TCPRoute          │
│ - Defines routing rules                         │
│ - Configures application-specific policies      │
└─────────────────────────────────────────────────┘
```

---

## Setup 

### Step 1: Install Gateway API CRDs

```bash
# Install Gateway API CRDs 
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml


# Verify installation (should show v1 and v1beta1 as available versions for backward compatibility)
kubectl get crd | grep gateway
```

### Step 2: Install a Gateway Controller (Envoy Gateway Example)

```bash
# Install Envoy Gateway
kubectl apply --server-side -f https://github.com/envoyproxy/gateway/releases/download/v1.0.0/install.yaml


# Verify installation
kubectl get pods -n envoy-gateway-system
```

### Step 3: Create GatewayClass

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy-v135
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
  # NEW in v1.35: Enhanced configuration
  parametersRef:
    group: gateway.envoyproxy.io
    kind: EnvoyProxy
    name: envoy-config
  description: "Envoy Gateway optimized for Kubernetes v1.35"
---
# Enhanced Envoy configuration
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: EnvoyProxy
metadata:
  name: envoy-config
spec:
  # NEW: Performance optimizations
  concurrency: 4
  logging:
    level:
      default: info
  telemetry:
    metrics:
      prometheus:
        disable: false
```

### Step 4: Create Gateway

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: production-gateway
  namespace: gateway-system
  annotations:
    # NEW in v1.35: Enhanced annotations
    gateway.networking.k8s.io/bundle-version: "v1.35"
spec:
  gatewayClassName: envoy-v135
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: All
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: wildcard-tls
        namespace: gateway-system
    allowedRoutes:
      namespaces:
        from: All
  # NEW in v1.35: gRPC support
  - name: grpc
    protocol: HTTP
    port: 9090
    allowedRoutes:
      kinds:
      - kind: GRPCRoute
  # NEW in v1.35: TCP support
  - name: tcp
    protocol: TCP
    port: 5432
    allowedRoutes:
      kinds:
      - kind: TCPRoute
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

### Scenario 1: Service Mesh Integration

```yaml
# Gateway with service mesh integration
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: mesh-gateway
  namespace: istio-system
  annotations:
    # Enable Istio service mesh integration
    gateway.istio.io/service-mesh: "enabled"
spec:
  gatewayClassName: istio
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: mesh-tls
    allowedRoutes:
      namespaces:
        from: All
---
# HTTPRoute with mesh policies
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: mesh-route
  namespace: production
spec:
  parentRefs:
  - name: mesh-gateway
    namespace: istio-system
  hostnames:
  - "secure.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: secure-api
      port: 8080
---
# NEW v1.35: Security Policy for mesh
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: SecurityPolicy
metadata:
  name: mesh-security
  namespace: production
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: mesh-route
  authentication:
    mtls:
      mode: STRICT
  authorization:
    rules:
    - action: ALLOW
      from:
      - source:
          namespaces:
          - "production"
          - "staging"
```
---

## Real-World Use Cases

### Use Case 1: Multi-Tenant Platform

**Scenario**: SaaS platform with multiple customers, each with their own subdomain

```yaml
Multi-Tenant SaaS Platform
```yaml
# Shared Gateway with enhanced security
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: saas-gateway
  namespace: platform
spec:
  gatewayClassName: envoy-v135
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: wildcard-saas-tls
    allowedRoutes:
      namespaces:
        from: Selector
        selector:
          matchLabels:
            tenant: "true"
---
# Tenant-specific route with policies
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: tenant-alpha-route
  namespace: tenant-alpha
spec:
  parentRefs:
  - name: saas-gateway
    namespace: platform
  hostnames:
  - "alpha.saas.example.com"
  rules:
  - backendRefs:
    - name: tenant-alpha-app
      port: 80
---
# NEW v1.35: Tenant-specific traffic policy
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: TrafficPolicy
metadata:
  name: tenant-alpha-policy
  namespace: tenant-alpha
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: tenant-alpha-route
  rateLimit:
    requests: 1000
    unit: minute
  circuitBreaker:
    maxConnections: 100
    maxRequests: 200
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

### Use Case 3. Performance Optimization
```yaml
# NEW v1.35: Performance-optimized gateway
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: high-performance-gateway
  annotations:
    # Performance annotations
    gateway.envoyproxy.io/concurrency: "4"
    gateway.envoyproxy.io/buffer-limit: "32KB"
spec:
  gatewayClassName: envoy-v135
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: performance-tls
    allowedRoutes:
      namespaces:
        from: All
---
# Performance-optimized route
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: high-performance-route
spec:
  parentRefs:
  - name: high-performance-gateway
  rules:
  - backendRefs:
    - name: high-performance-service
      port: 8080
    filters:
    # Enable compression
    - type: ResponseHeaderModifier
      responseHeaderModifier:
        add:
        - name: Content-Encoding
          value: gzip
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

### Traditional Best Practices 
1. **Separate Concerns**: Use different namespaces for infrastructure (Gateway) and applications (HTTPRoute)
2. **Use GatewayClass**: Define clear GatewayClasses for different environments
3. **Limit Route Attachment**: Use `allowedRoutes` to control access
4. **Monitor Status**: Check Gateway and Route status conditions regularly
5. **Use Weights for Rollouts**: Gradually shift traffic using weight-based routing
6. **Validate Before Production**: Test routes in staging environments
7. **Document Hostnames**: Maintain a registry of hostnames and ownership
8. **Use TLS**: Always use HTTPS in production with proper certificates

### 5. NEW v1.35 Best Practices
9. **Policy-Driven Security**: Use SecurityPolicy for authentication and authorization
10. **Traffic Management**: Implement TrafficPolicy for rate limiting and circuit breaking
11. **Observability**: Use ObservabilityPolicy for comprehensive monitoring
12. **Cross-Protocol Support**: Leverage gRPC, TCP, and UDP routes appropriately
13. **Service Mesh Integration**: Use native service mesh features when available
14. **Progressive Delivery**: Implement canary deployments with proper observability

---

### Practice Scenarios
1. **Basic Setup**: Install Gateway API, create GatewayClass, Gateway, HTTPRoute
2. **Path-Based Routing**: Route `/api` to one service, `/web` to another
3. **Host-Based Routing**: Different hostnames to different services
4. **Traffic Splitting**: 90/10 split between stable and canary
5. **Cross-Namespace**: Gateway in `gateway-system`, routes in `default`
6. **TLS Termination**: HTTPS gateway with certificate references
7. **NEW**: gRPC service routing
8. **NEW**: Basic security policy application
---

## Migration from Ingress to Gateway API

### Ingress Example
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Equivalent Gateway API
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
spec:
  gatewayClassName: envoy-v135
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: Same
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-route
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

## Summary

Gateway API in Kubernetes v1.35 provides next-generation traffic management:

- **Enhanced API**: Stable v1.1 with Policy API support
- **Multi-Protocol**: HTTP, HTTPS, gRPC, TCP, UDP support
- **Role-Oriented**: Clear separation of concerns
- **Service Mesh**: Native integration with Istio, Linkerd
- **Advanced Features**: Traffic policies, security policies, observability
- **Production Ready**: Stable API with extensive controller ecosystem

---
















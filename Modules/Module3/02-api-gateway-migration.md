# Migrating from Ingress to Gateway API

## Overview

This guide covers the migration process from traditional Kubernetes Ingress to Gateway API. You'll learn how to convert existing Ingress resources to Gateway API equivalents (HTTPRoute, Gateway objects) and understand the benefits of the migration.

## Why Migrate?

**Benefits of Gateway API over Ingress**:
- **More expressive**: Advanced routing (headers, weights, mirroring)
- **Role-oriented**: Clear separation of responsibilities
- **Type-safe**: Better validation and error messages
- **Extensible**: Custom resources without annotations
- **Portable**: Standardized across implementations
- **Future-proof**: Active development and community support

## Why Kubernetes is Evolving Beyond Ingress: The Rise of the Gateway API

***Firstly let's understand what is ingress what are it's limitations***

### What is Ingress in Kubernetes?
It provides a centralized way to manage external access to services running inside a Kubernetes cluster, typically HTTP/HTTPS traffic at the L7 layer. It provides centralizedmechanism to manage all inbound requests to backend applications.
Instead of individual load balancers per service.

### Limitations of Ingress 
- Ingress specifically is designed for HTTP/HTTPS (Layer 7)
- Lack of standardization
- Couldn't handle TCP/UDP traffic or advanced routing

### Gateway API Solution
Gateway API solves the limitations it includes specific resources for these needs: TCPRoute, UDPRoute, TLSRoute, and GRPCRoute, providing comprehensive L4/L7 support 

---

## Migration Strategy

### Phase 1: Preparation
1. Install Gateway API CRDs
2. Install Gateway Controller
3. Create GatewayClass
4. Inventory existing Ingress resources

### Phase 2: Parallel Running
1. Create Gateway resources
2. Convert Ingress to HTTPRoute
3. Test Gateway API routes
4. Validate traffic flow

### Phase 3: Cutover
1. Update DNS or load balancer
2. Monitor traffic
3. Deprecate Ingress resources
4. Clean up old resources

---

## Step-by-Step Migration Examples

### Example 1: Basic Ingress to HTTPRoute

**Original Ingress**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: basic-ingress
  namespace: default
spec:
  ingressClassName: nginx
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```

**Converted to Gateway API**:

**Step 1: Create GatewayClass** (if not exists)
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: k8s-gateway.nginx.org/nginx-gateway-controller
```

**Step 2: Create Gateway**
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: example-gateway
  namespace: default
spec:
  gatewayClassName: nginx
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: Same
```

**Step 3: Create HTTPRoute**
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: basic-route
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
            value: /
      backendRefs:
        - name: web-service
          port: 80
```

---

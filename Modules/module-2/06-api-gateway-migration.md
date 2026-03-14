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
- Policy API integration for security and traffic management
- Enhanced multi-protocol support (gRPC, TCP, UDP)
- Native service mesh integration

## Why Kubernetes is Evolving Beyond Ingress: The Rise of the Gateway API

***Firstly let's understand what is ingress what are it's limitations***

### What is Ingress in Kubernetes?
It provides a centralized way to manage external access to services running inside a Kubernetes cluster, typically HTTP/HTTPS traffic at the L7 layer. It provides centralizedmechanism to manage all inbound requests to backend applications. Instead of individual load balancers per service.

### Limitations of Ingress 
- **Protocol Limitation**: Ingress specifically is designed for HTTP/HTTPS (Layer 7)
- **Lack of standardization**: Heavy reliance on controller-specific annotations
- **Limited routing**: Couldn't handle TCP/UDP traffic or advanced routing
- **Single resource model**: No role separation between infrastructure and application teams
- **Annotation hell**: Complex configurations through non-standard annotations
- **Limited traffic management**: No native support for traffic splitting, mirroring
- **Vendor lock-in**: Controller-specific features not portable

### Gateway API Solution
Gateway API solves these limitations with:
- **Multi-protocol support**: TCPRoute, UDPRoute, TLSRoute, GRPCRoute
- **Comprehensive L4/L7 support**: Full network stack coverage
- **Role-oriented design**: Clear separation of concerns
- **Policy API**: Native security, traffic, and observability policies
- **Service mesh integration**: Native support for Istio, Linkerd, etc.
- **Standardized extensibility**: No more annotation-based configuration

---

## Migration Strategy

### Phase 1: Preparation and Assessment
1. **Audit existing Ingress resources**
2. **Install Gateway API v1.1 CRDs** (latest for v1.35)
3. **Install Gateway Controller** (Envoy, NGINX, Istio, etc.)
4. **Create GatewayClass resources**
5. **Plan role separation** (Infrastructure vs Application teams)

### Phase 2: Parallel Deployment
1. **Create Gateway resources** (Infrastructure team)
2. **Convert Ingress to HTTPRoute** (Application team)
3. **Implement Policy API** (Security team)
4. **Test Gateway API routes** thoroughly
5. **Validate traffic flow** and performance

### Phase 3: Gradual Cutover
1. **Implement traffic splitting** (if needed)
2. **Update DNS or load balancer** configuration
3. **Monitor traffic** and performance metrics
4. **Gradually increase Gateway API traffic**
5. **Deprecate Ingress resources**

### Phase 4: Cleanup and Optimization
1. **Remove old Ingress resources**
2. **Optimize Gateway configurations**
3. **Implement advanced features** (policies, service mesh)
4. **Update monitoring and alerting**
---

## Migration Process

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
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx-v1.4
spec:
  controllerName: k8s-gateway.nginx.org/nginx-gateway-controller
  # NEW: Enhanced parameters
  parametersRef:
    group: gateway.nginx.org
    kind: NginxGateway
    name: nginx-config
```

**Step 2: Create Gateway**
```yaml
apiVersion: gateway.networking.k8s.io/v1
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
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: basic-route
  namespace: default
  annotations:
    # NEW: Migration tracking
    gateway.networking.k8s.io/migrated-from: "basic-ingress"
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

### Example 2: Advanced Migration with Policies

**Step 1: Add Security Policy**

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
  cors:
    allowOrigins:
    - "https://app.example.com"
    allowMethods:
    - GET
    - POST
    - PUT
    - DELETE
    allowHeaders:
    - "Authorization"
    - "Content-Type"
```
**Step 2: Add Traffic Policy** 
```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: TrafficPolicy
metadata:
  name: api-traffic-policy
  namespace: default
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: api-route
  rateLimit:
    requests: 1000
    unit: minute
    clientSelectors:
    - headers:
      - name: X-API-Key
        type: Present
  retry:
    attempts: 3
    backoff: exponential
    baseInterval: 1s
  timeout: 30s
```

---

## Advanced Migration Scenario

### Scenario: Canary Deployment (Not Possible with Ingress)

**Ingress Limitation**: Cannot do true blue-green without external tools

**Gateway API Solution**:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: blue-green-route
  namespace: default
  annotations:
    # NEW: Deployment strategy tracking
    gateway.networking.k8s.io/deployment-strategy: "blue-green"
spec:
  parentRefs:
  - name: production-gateway
  hostnames:
  - "app.example.com"
  rules:
  - backendRefs:
    # Initially: 100% blue, 0% green
    - name: app-blue
      port: 80
      weight: 100
    - name: app-green
      port: 80
      weight: 0
    filters:
    # Track deployment version
    - type: RequestHeaderModifier
      requestHeaderModifier:
        add:
        - name: X-Deployment-Color
          value: "blue"

# After validation, switch to green:
# Update weights: blue: 0, green: 100
# Update header value to "green"
```
---


## Testing Migration

### Pre-Migration Testing

```bash
# Test existing Ingress
curl -H "Host: example.com" http://<INGRESS_IP>/

# Check Ingress status
kubectl get ingress -A
kubectl describe ingress <ingress-name>
```
### Post-Migration Testing

```bash
# Get Gateway address
GATEWAY_IP=$(kubectl get gateway <gateway-name> -o jsonpath='{.status.addresses[0].value}')

# Test HTTPRoute
curl -H "Host: example.com" http://$GATEWAY_IP/

# Compare responses
diff <(curl -s -H "Host: example.com" http://<INGRESS_IP>/) \
     <(curl -s -H "Host: example.com" http://$GATEWAY_IP/)
```

## Troubleshooting Migration Issues

### Issue 1: HTTPRoute Not Attached

**Error**:
```bash
kubectl get httproute
# NAME           HOSTNAMES         AGE
# example-route  ["example.com"]   5m

# But traffic doesn't flow
```

**Debug**:
```bash
# Check HTTPRoute status
kubectl describe httproute example-route

# Check parent status
kubectl get httproute example-route -o jsonpath='{.status.parents}' | jq

# Verify Gateway allows routes from this namespace
kubectl get gateway example-gateway -o yaml | grep -A 10 allowedRoutes
```

**Solution**:
```yaml
# Update Gateway to allow routes
spec:
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All  # or Same, or Selector
```

---

### Issue 2: Path Matching Differences

**Error**: Paths that worked with Ingress don't work with HTTPRoute

**Debug**:
```bash
# Test specific paths
curl -v -H "Host: example.com" http://$GATEWAY_IP/api/v1/users

# Check HTTPRoute matches
kubectl get httproute example-route -o yaml | grep -A 10 matches
```

**Solution**:
- Ingress `Prefix` = Gateway API `PathPrefix`
- Ingress `Exact` = Gateway API `Exact`
- Ingress `ImplementationSpecific` = Gateway API `RegularExpression` (if supported)

---

## Best Practices

### 1. Planning and Preparation
- **Audit existing Ingress resources** thoroughly
- **Identify annotation dependencies** that need Gateway API equivalents
- **Plan role separation** between infrastructure and application teams
- **Choose appropriate Gateway Controller** for your use case
- Plan Policy API adoption strategy

### 2. Incremental Migration Strategy
```bash
# Migration phases
Phase 1: Non-critical services (dev/staging)
Phase 2: Internal services
Phase 3: Customer-facing services
Phase 4: Critical production services

# Per-service migration steps
1. Create Gateway (if shared)
2. Create HTTPRoute
3. Test functionality
4. Implement policies (NEW in v1.35)
5. Switch traffic
6. Monitor and validate
7. Remove Ingress
```

### 3. Testing Strategy
- **Run in parallel**: Keep Ingress running during migration
- **Test thoroughly**: Validate all paths, hosts, and edge cases
- **Performance testing**: Compare latency and throughput
- **Load testing**: Ensure Gateway API handles production load
- Test policy enforcement

---

## Summary

Migrating from Ingress to Gateway API in Kubernetes v1.35 provides:

- **Enhanced Functionality**: Advanced routing, traffic management, and policies
- **Better Architecture**: Role-oriented design with clear separation of concerns
- **Future-Proofing**: Active development and growing ecosystem support
- **Gateway API v1.4 Benefits**: Policy API, service mesh integration, multi-protocol support
- **Improved Operations**: Better observability, debugging, and management
- **Standardization**: Portable configurations across different implementations

The migration requires careful planning but delivers significant long-term benefits for modern Kubernetes networking.
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

## Step-by-Step Migration Example

### Example: Basic Ingress to HTTPRoute

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
  name: nginx
spec:
  controllerName: k8s-gateway.nginx.org/nginx-gateway-controller
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

## Advanced Migration Scenarios

### Scenario 1: Canary Deployment (Not Possible with Ingress)

**Ingress Limitation**: Cannot split traffic by percentage

**Gateway API Solution**:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: canary-route
spec:
  parentRefs:
    - name: example-gateway
  hostnames:
    - "app.example.com"
  rules:
    - backendRefs:
        - name: app-stable
          port: 80
          weight: 90  # 90% to stable
        - name: app-canary
          port: 80
          weight: 10  # 10% to canary
```

---

### Scenario 2: Header-Based Routing (Limited in Ingress)

**Ingress Limitation**: Requires complex annotations, not standardized

**Gateway API Solution**:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: header-route
spec:
  parentRefs:
    - name: example-gateway
  rules:
    - matches:
        - headers:
            - name: X-Version
              value: beta
      backendRefs:
        - name: beta-service
          port: 80
    - matches:
        - headers:
            - name: X-Version
              value: stable
      backendRefs:
        - name: stable-service
          port: 80
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

1. **Migrate incrementally**: Start with non-critical services
2. **Run in parallel**: Keep Ingress running during migration
3. **Test thoroughly**: Validate all paths and hosts
4. **Monitor metrics**: Compare latency and error rates
5. **Document changes**: Keep track of what was converted
6. **Use separate Gateways**: Different Gateways for different environments
7. **Validate TLS**: Ensure certificates work correctly
8. **Plan rollback**: Have a clear rollback procedure

---


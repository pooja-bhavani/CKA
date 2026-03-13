# Kubernetes Services and Endpoints

## Overview

Services in Kubernetes provide stable networking endpoints for accessing pods. They abstract away the ephemeral nature of pod IP addresses and provide load balancing across multiple pod replicas, and how that traffic is controlled and secured using Services, CNI, NetworkPolicies, and now Gateway API.​

## Why Services Matter

- **Stable Endpoints**: Pods are ephemeral and their IPs change; Services provide consistent access
- **Load Balancing**: Distribute traffic across multiple pod replicas
- **Service Discovery**: Enable pods to find and communicate with each other
- **External Access**: Expose applications outside the cluster
- **Traffic Management**: Control how traffic flows to backend pods
---

## Service Types

### 1. ClusterIP (Default)
Exposes the service on an internal cluster IP. Only accessible within the cluster. (App runs only within the cluster and the whole cluster gets an ip address)

**When to use**:
- Internal microservices communication
- Backend services that don't need external access
- Database services accessed only by application pods

**Example**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

**Quick Create**:
```bash
kubectl expose deployment backend --port=80 --target-port=8080 --type=ClusterIP
```

---  

### 2. NodePort
Exposes services on a static port on each Node's IP in the cluster. Static port can be (30000-32767 range). Accessible from outside the cluster via `<NodeIP>:<NodePort>`.

**When to use**:
- Development/testing environments
- When you need external access without a LoadBalancer
- Direct node access scenarios

 **Example**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-nodeport
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080  # Optional, auto-assigned if omitted
```

**Quick Create**:
```bash
kubectl expose deployment frontend --port=80 --target-port=8080 --type=NodePort
```

**Access**:
```bash
# Get node IP
kubectl get nodes -o wide

# Access service
curl http://<NODE_IP>:30080
```

---

### 3. LoadBalancer
Exposes the service externally using a cloud provider's load balancer. Creates a NodePort and ClusterIP automatically. 

**When to use**:
- Production environments on cloud platforms (AWS, GCP, Azure)
- When you need a stable external IP
- Production-grade external access with health checks

**Example**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

**Quick Create**:
```bash
kubectl expose deployment web --port=80 --target-port=8080 --type=LoadBalancer
```

**Check External IP**:
```bash
kubectl get svc web-loadbalancer
# Wait for EXTERNAL-IP to be assigned (shows <pending> initially)
```

---

## Endpoints

**What are Endpoints?**
Endpoints are the actual IP addresses and ports of pods that match a service's selector. Kubernetes automatically creates and manages Endpoints objects.

**To View Endpoints**:
```bash
kubectl get endpoints
kubectl describe endpoints <service-name>
```
### EndpointSlices

EndpointSlices provide a more scalable and extensible alternative to Endpoints:

```bash
# View EndpointSlices (v1.35 preferred method)
kubectl get endpointslices
kubectl describe endpointslice <service-name>-<hash>

# View traditional Endpoints (still supported)
kubectl get endpoints
kubectl describe endpoints <service-name>
```

**EndpointSlice Example**:
```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: web-service-abc123
  labels:
    kubernetes.io/service-name: web-service
addressType: IPv4
endpoints:
- addresses:
  - "10.244.1.5"
  conditions:
    ready: true
  targetRef:
    kind: Pod
    name: web-pod-1
    namespace: default
ports:
- name: http
  port: 8080
  protocol: TCP
```

## v1.35 Enhancements

### 1. Enhanced Traffic Distribution

v1.35 introduces improved traffic distribution options:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
  # NEW in v1.35: Traffic distribution options
  trafficDistribution: PreferClose  # Route to closest endpoints
  internalTrafficPolicy: Local      # Keep traffic on same node when possible
```

**bankapp-topology-aware-service**                
[bankapp-topology-aware-svc.yaml](../../k8s/networking/02-bankapp-topology-aware-svc.yaml)


**Traffic Distribution Options**:
- `PreferClose`: Routes traffic to topologically closer endpoints
- `Cluster` (default): Distributes traffic across all endpoints

**Internal Traffic Policy Options**:
- `Cluster` (default): Routes to all endpoints cluster-wide
- `Local`: Routes only to endpoints on the same node

### 3. Service with Pod Security Standards

Combines a Service and a secure Deployment to demonstrate “Service with Pod Security Standards” + hostUsers: false (user namespaces).

```yaml
# v1.35 Service targeting secure pods
apiVersion: v1
kind: Service
metadata:
  name: secure-service
spec:
  selector:
    app: secure-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: secure-app
  template:
    metadata:
      labels:
        app: secure-app
    spec:
      # v1.35 Pod Security compliance
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault
      hostUsers: false  # v1.35 user namespace isolation
      containers:
      - name: app
        image: nginx:1.21
        ports:
        - name: http
          containerPort: 8080
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop: ["ALL"]
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
```

**secure-app-with-service.yaml**            
[secure-app-with-service.yaml](../../k8s/networking/03-secure-app-with-service.yaml)

What it does:
- Service + secure pods: How Services work with hardened pods, not just default ones.
- User namespaces: hostUsers: false demonstrates v1.35 user‑namespace integration.
- PSS compliant: Good example for future Pod Security admission discussions.

---

## Common Scenarios and Use Cases

### Scenario 1: Exposing a Multi-Tier Application

```yaml
# Frontend - LoadBalancer (external access)
apiVersion: v1
kind: Service
metadata:
  name: frontend
  annotations:
    service.kubernetes.io/topology-aware-hints: auto
spec:
  type: LoadBalancer
  selector:
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: 3000
  trafficDistribution: PreferClose
---
# Backend - ClusterIP (internal only)
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  selector:
    tier: backend
  ports:
  - name: api
    port: 8080
    targetPort: 8080
  internalTrafficPolicy: Local
---
# Database - ClusterIP (internal only)
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  type: ClusterIP
  selector:
    tier: database
  ports:
  - name: postgres
    port: 5432
    targetPort: 5432
  internalTrafficPolicy: Local
```

### Scenario 2: Headless Service (StatefulSets)

Use case: For direct pod-to-pod communication, StatefulSets where each pod needs a stable DNS name.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None  # Headless service
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
```
## Troubleshooting 

### Error 1: Service Not Accessible

**Error**:
```bash
curl: (7) Failed to connect to service
```

**Debug Steps**:
```bash
# 1. Check if service exists
kubectl get svc <service-name>

# 2. Check endpoints (are pods selected?)
kubectl get endpoints <service-name>

# 3. Check pod labels match service selector
kubectl get pods --show-labels
kubectl describe svc <service-name> 

# 4. Check if pods are running
kubectl get pods -l app=<label>

# 5. Test from within cluster
kubectl run test-pod --image=busybox --rm -it -- wget -O- http://<service-name>
```

**Solution**:
- Ensure pod labels match service selector
- Verify pods are in Running state
- Check targetPort matches container port

---

### Error 3: NodePort Not Accessible

**Error**:
```bash
curl: (7) Failed to connect to <NODE_IP>:<NODE_PORT>
```

**Debug Steps**:
```bash
# 1. Verify NodePort service
kubectl get svc <service-name>

# 2. Check firewall rules
# Ensure NodePort range (30000-32767) is open

# 3. Test from node itself
ssh <node>
curl localhost:<nodeport>

# 4. Check kube-proxy
kubectl get pods -n kube-system 
kubectl logs -n kube-system <kube-proxy-pod>
```

**Solution**:
- Open firewall ports for NodePort range
- Verify kube-proxy is running on all nodes
- Check cloud security groups allow traffic

---

### Error 3: NodePort Not Accessible

**Symptoms**:
```bash
curl: (7) Failed to connect to <NODE_IP>:<NODE_PORT>
```

**Debug Steps**:
```bash
# 1. Verify NodePort service
kubectl get svc <service-name>

# 2. Check firewall rules
# Ensure NodePort range (30000-32767) is open

# 3. Test from node itself
ssh <node>
curl localhost:<nodeport>

# 4. Check kube-proxy
kubectl get pods -n kube-system | grep kube-proxy
kubectl logs -n kube-system <kube-proxy-pod>
```

**Solution**:
- Open firewall ports for NodePort range
- Verify kube-proxy is running on all nodes
- Check cloud security groups allow traffic

---

## Best Practices

1. **Use ClusterIP by default** - Only expose externally when necessary
2. **Label consistency** - Ensure pod labels match service selectors
3. **Named ports** - Use named ports for clarity and flexibility
4. **Health checks** - Configure readiness probes so unhealthy pods are removed from endpoints
5. **DNS naming** - Use service names for inter-service communication
6. **Namespace awareness** - Access services in other namespaces: `<service>.<namespace>.svc.cluster.local`

---

# Kubernetes Services and Endpoints

## Overview

Services in Kubernetes provide stable networking endpoints for accessing pods. They abstract away the ephemeral nature of pod IP addresses and provide load balancing across multiple pod replicas, and how that traffic is controlled and secured using Services, CNI, NetworkPolicies, and now Gateway API.​

## Why Services Matter

- **Stable Endpoints**: Pods are ephemeral and their IPs change; Services provide consistent access
- **Load Balancing**: Distribute traffic across multiple pod replicas
- **Service Discovery**: Enable pods to find and communicate with each other
- **External Access**: Expose applications outside the cluster

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



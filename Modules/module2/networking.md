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

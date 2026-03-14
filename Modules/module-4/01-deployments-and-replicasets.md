# Deployments and Replicasets

## Overview

Deployments and ReplicaSets are fundamental workload resources in Kubernetes v1.35, providing declarative updates and scaling capabilities with 
enhanced features for better reliability and performance.

### Enhanced Deployment Features
- **Improved Rolling Updates** - Faster and more reliable rollouts
- **Better Rollback Mechanisms** - Enhanced rollback with detailed history and automatic failure detection
- **Advanced Scaling Strategies** - More sophisticated autoscaling options
- **Resource Optimization** - Better resource utilization during updates
- **Enhanced Status Reporting** - More detailed deployment status information

### ReplicaSet Improvements
- **Optimized Pod Management** - Faster pod creation and deletion
- **Better Label Handling** - Improved selector management
- **Enhanced Monitoring** - Better observability for ReplicaSet operations

### Deployment Strategy Evolution

**v1.34 Deployment Process:**
```bash
# v1.34: Basic deployment with limited control
kubectl create deployment web-app --image=nginx:1.21 --replicas=10
kubectl set image deployment/web-app nginx=nginx:1.22
kubectl rollout status deployment/web-app

# Limitations:
# - No surge control during updates
# - Limited rollback capabilities  
# - Manual monitoring required
# - Resource waste during rollouts
```

**v1.35 Enhanced Process:**
```bash
# v1.35: Intelligent deployment with full control
kubectl create deployment web-app-v135 --image=nginx:1.25 --replicas=10

# Enhanced update with surge control
kubectl patch deployment web-app-v135 -p '{
  "spec": {
    "strategy": {
      "rollingUpdate": {
        "maxUnavailable": "10%",
        "maxSurge": "25%",
        "progressDeadlineSeconds": 600
      }
    },
    "template": {
      "spec": {
        "containers": [{
          "name": "nginx",
          "image": "nginx:1.26"
        }]
      }
    }
  }
}'

# Automatic monitoring and rollback
kubectl rollout status deployment/web-app-v135 --watch=true
# Auto-rollback on failure detection
```
---

## 🌍 Real-World Scenarios

### Scenario 1: E-commerce Platform - Black Friday Traffic Surge

Your e-commerce platform expects 10x traffic during Black Friday. You need to scale from 10 to 100 replicas without any downtime or customer impact.

**v1.34 Challenges:**
```bash
# v1.34: Manual scaling with potential issues
kubectl scale deployment ecommerce-frontend --replicas=100
# Problems:
# - No gradual rollout control
# - Resource constraints could cause failures
# - Limited rollback capabilities
# - Manual monitoring required
```

**v1.35 Solution:**
```yaml
# Enhanced progressive rollout with better control

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-frontend-v135
  annotations:
    deployment.kubernetes.io/strategy-version: "v1.35"
spec:
  replicas: 100
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 10%
      maxSurge: 20%  # Better surge control
      # enhancement: Progressive rollout
      progressDeadlineSeconds: 600
      revisionHistoryLimit: 10
  template:
    spec:
      containers:
      - name: frontend
        image: ecommerce/frontend:v2.1
        # v1.35: Enhanced resource management
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "400m"
        # Improved health checks
        startupProbe:
          httpGet:
            path: /health
            port: 8080
          failureThreshold: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          periodSeconds: 5
```

**Impact:**

- **Zero downtime scaling**: No lost sales during traffic surge
- **Controlled rollout**: 20% surge allows gradual scaling
- **Better monitoring**: Enhanced status reporting prevents issues
- **Cost optimization**: Right-sized resources prevent over-provisioning


### Migration Complexity

```yaml
# BEFORE (v1.34): Basic deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-v134
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    spec:
      containers:
      - name: web
        image: nginx:1.21
        # v1.34: Basic resource allocation
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        # v1.34: Simple health checks
        livenessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 10
```

```yaml
# AFTER (v1.35): Enhanced deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-v135
  annotations:
    deployment.kubernetes.io/strategy-version: "v1.35"
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 10%
      maxSurge: 25%
      # Enhanced rollout control
      progressDeadlineSeconds: 600
      revisionHistoryLimit: 10
  template:
    metadata:
      labels:
        app: web-app
        version: v1.35
    spec:
      containers:
      - name: web
        image: nginx:1.25
        # Optimized resource allocation
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
            ephemeral-storage: "1Gi"
          limits:
            memory: "256Mi"
            cpu: "200m"
            ephemeral-storage: "2Gi"
        # Enhanced health checks
        startupProbe:
          httpGet:
            path: /health
            port: 80
          failureThreshold: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
        # Resource-aware configuration
        env:
        - name: MEMORY_LIMIT
          valueFrom:
            resourceFieldRef:
              resource: limits.memory
              divisor: "1Mi"
        - name: CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              resource: limits.cpu
              divisor: "1m"
```

## ReplicaSets

### Basic ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset-v135
  labels:
    app: nginx
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      tier: frontend
  template:
    metadata:
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Advanced ReplicaSet with v1.35 Features

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: advanced-replicaset-v135
  annotations:
    replicaset.kubernetes.io/version: "v1.35"
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web-server
    matchExpressions:
    - key: environment
      operator: In
      values: ["production", "staging"]
  template:
    metadata:
      labels:
        app: web-server
        environment: production
        version: v1.35
    spec:
      # Enhanced scheduling
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values: ["web-server"]
              topologyKey: kubernetes.io/hostname
      containers:
      - name: web-server
        image: nginx:1.25
        ports:
        - containerPort: 80
        # Improved resource management
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        # Enhanced probes
        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 5
```

---

### Common Exam Scenarios

1. **Create and scale deployments**
2. **Perform rolling updates**
3. **Rollback deployments**
4. **Troubleshoot failed deployments**
5. **Configure resource limits**
6. **Set up health checks**
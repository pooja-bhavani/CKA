## Overview

Deployments and ReplicaSets are core Kubernetes workload resources, managing declarative updates, scaling, and Pod lifecycles. In v1.35 (Timbernetes), key enhancement is the Beta promotion of status.terminatingReplicas for better rollout and scale-down observability.

### Enhanced Deployment Features

- **TerminatingReplicas Field** - Tracks Pods with deletion timestamps in Deployment/ReplicaSet status (Beta, enabled by default via -DeploymentReplicaSetTerminatingReplicas feature gate).
- **Improved Observability** - New metrics like kube_deployment_status_replicas_terminating and kube_replicaset_status_terminating_replicas.​
- **Foundation for Policies** - Enables future Pod replacement logic during terminations.

### ReplicaSet Improvements
- **Optimized Pod Management** - Faster pod creation and deletion
- **Better Label Handling** - Improved selector management
- **Enhanced Monitoring** - Better observability for ReplicaSet operations

### ReplicaSet Role**
ReplicaSets maintain Pod counts; Deployments manage them automatically. v1.35 adds terminatingReplicas tracking for precise replica counts during deletions.

```bash
kubectl get deployment web-app -o yaml | grep -A5 terminatingReplicas
# Output example:
# terminatingReplicas: 2  # Pods terminating but not yet removed
kubectl get rs -o custom-columns=NAME:.metadata.name,REPLICAS:.status.replicas,TERMINATING:.status.terminatingReplicas
```
---

## 🌍 Real-World Scenarios

### Scenario 1: E-commerce Platform - Black Friday Traffic Surge

Your e-commerce platform expects 10x traffic during Black Friday. You need to scale from 10 to 100 replicas without any downtime or customer impact.

**v1.35 Challenges:**
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-frontend
spec:
  replicas: 100
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 10%
      maxSurge: 20%
      progressDeadlineSeconds: 600
  template:
    spec:
      containers:
      - name: frontend
        image: ecommerce/frontend:v2.1
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "400m"
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
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
```

- v1.35 Monitoring
```
kubectl rollout status deployment/ecommerce-frontend --watch
kubectl get deployment ecommerce-frontend -o jsonpath='{.status.terminatingReplicas}'  # Track shutdown progress
```

**Impact:**

- Precise surge control prevents over-provisioning.
- terminatingReplicas visibility during scale-down avoids premature new Pod creation.
- Graceful shutdowns maintain availability.

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
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
```

### Advanced ReplicaSet with v1.35 Features

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web-server
  template:
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: web-server
              topologyKey: kubernetes.io/hostname
      containers:
      - name: web-server
        image: nginx:1.25
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 30
          periodSeconds: 10
```

---

### Common Exam Scenarios

1. **Create and scale deployments**
2. **Perform rolling updates**
3. **Rollback deployments**
4. **Troubleshoot failed deployments**
5. **Configure resource limits**
6. **Set up health checks**

---

## Overview

Deployments and ReplicaSets manage Pod replicas, scaling, and updates in Kubernetes. v1.35 (Timbernetes) promotes status.terminatingReplicas to Beta for precise tracking of Pods during deletion phases in rollouts and scale-down operations.

### Enhanced Deployment Features

- **TerminatingReplicas Field** - Beta feature (enabled by default via DeploymentReplicaSetTerminatingReplicas feature gate) that counts Pods with deletion timestamps in Deployment/ReplicaSet status.
- **Better Rollout Visibility** - New metrics (kube_deployment_status_replicas_terminating, kube_replicaset_status_terminating_replicas) track termination progress.​
- **Scale-Down Precision** - Prevents premature Pod creation by accounting for terminating replicas.

Core Behaviors 
Rolling updates (maxSurge, maxUnavailable), health probes, resource limits, and affinity rules work identically to prior versions.


### ReplicaSet Role**
ReplicaSets maintain Pod counts; Deployments manage them automatically. v1.35 adds terminatingReplicas tracking for precise replica counts during deletions.

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
        image: ecommerce/frontend:v2
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "400m"
            memory: "512Mi"
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
kubectl scale deployment ecommerce-frontend --replicas=10
watch 'kubectl get rs -o custom-columns="RS:.metadata.name,Tot:.status.replicas,Term:.status.terminatingReplicas"'
# Ensures controller waits for terminations before creating new replicas
```

**Impact:**

- Zero downtime during 10x surge
- erminatingReplicas prevents resource spikes during scale-down
- Graceful shutdowns maintain service availability.

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
        image: nginx:1.27
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
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

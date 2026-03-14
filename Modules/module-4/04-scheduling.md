# Pod Scheduling & Affinity 

## Overview

Pod scheduling in Kubernetes v1.35 provides sophisticated mechanisms to control where pods are placed in your cluster, with enhanced features for better resource utilization, performance optimization, and workload distribution.

## 🌍 Real-World Scenario

### Scenario: Machine Learning Platform - GPU Resource Optimization

**Business Context**: Your AI/ML platform serves 500+ data scientists running training jobs that require specific GPU types. You need to optimize GPU utilization while ensuring fair resource allocation and preventing resource conflicts.

**v1.34 Challenges:**
```bash
# v1.34: Manual GPU scheduling (inefficient)
kubectl run ml-training --image=tensorflow/tensorflow:2.8.0-gpu \
  --requests="nvidia.com/gpu=1" \
  --overrides='{"spec":{"nodeSelector":{"accelerator":"nvidia-tesla-v100"}}}'

# Problems:
# - Manual node selection for GPU types
# - No automatic load balancing
# - GPU fragmentation and waste
# - No gang scheduling for distributed training
# - Complex resource conflicts
```

**v1.35 Solution:**
```yaml
# v1.35: GPU scheduling with gang scheduling
apiVersion: scheduling.sigs.k8s.io/v1alpha1
kind: PodGroup
metadata:
  name: distributed-training-group-v135
  namespace: ml-platform
spec:
  scheduleTimeoutSeconds: 300
  minMember: 8  # All 8 GPU workers must be scheduled together
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-training-v135
  namespace: ml-platform
  annotations:
    scheduler.kubernetes.io/gang-scheduling: "enabled"
spec:
  replicas: 8
  selector:
    matchLabels:
      app: ml-training
  template:
    metadata:
      labels:
        app: ml-training
        gang: distributed-training-group-v135
        gpu-type: nvidia-a100
      annotations:
        # Gang scheduling annotations
        scheduler.kubernetes.io/gang-name: "distributed-training-group-v135"
        scheduler.kubernetes.io/gang-min-size: "8"
        scheduler.kubernetes.io/gang-scheduling-timeout: "300s"
    spec:
      # Enhanced scheduling for ML workloads
      schedulerName: gang-scheduler
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: accelerator
                operator: In
                values: ["nvidia-a100", "nvidia-v100"]
              - key: gpu-memory
                operator: In
                values: ["40gb", "80gb"]
        # Enhanced pod anti-affinity for distributed training
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  gang: distributed-training-group-v135
              topologyKey: kubernetes.io/hostname
          - weight: 50
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: ml-training
              topologyKey: topology.kubernetes.io/zone
      # Advanced topology spread for ML workloads
      topologySpreadConstraints:
      - maxSkew: 2
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            gang: distributed-training-group-v135
      containers:
      - name: ml-worker
        image: tensorflow/tensorflow:2.13.0-gpu-v135
        command: ["python", "/app/distributed_training.py"]
        resources:
          requests:
            memory: "16Gi"
            cpu: "8000m"
            nvidia.com/gpu: 2
          limits:
            memory: "32Gi"
            cpu: "16000m"
            nvidia.com/gpu: 2
        env:
        - name: GANG_SIZE
          value: "8"
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: GPU_TYPE
          valueFrom:
            fieldRef:
              fieldPath: metadata.labels['gpu-type']
```
### Migration Complexity

**Scheduling Migration**

```yaml
# BEFORE (v1.34): Basic scheduling
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-training-v134
spec:
  replicas: 8
  template:
    spec:
      # v1.34: Limited scheduling options
      nodeSelector:
        accelerator: nvidia-v100
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: ml-training
              topologyKey: kubernetes.io/hostname
      # No gang scheduling - risk of partial deployment
      containers:
      - name: worker
        image: tensorflow/tensorflow:2.8.0-gpu
        resources:
          requests:
            nvidia.com/gpu: 1
```

```yaml
# AFTER (v1.35): Enhanced scheduling
apiVersion: scheduling.sigs.k8s.io/v1alpha1
kind: PodGroup
metadata:
  name: ml-training-group-v135
spec:
  scheduleTimeoutSeconds: 300
  minMember: 8  # Gang scheduling guarantee
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-training-v135
  annotations:
    scheduler.kubernetes.io/gang-scheduling: "enabled"
spec:
  replicas: 8
  template:
    metadata:
      annotations:
        scheduler.kubernetes.io/gang-name: "ml-training-group-v135"
        scheduler.kubernetes.io/gang-min-size: "8"
    spec:
      schedulerName: gang-scheduler
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: accelerator
                operator: In
                values: ["nvidia-a100", "nvidia-v100"]
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  gang: ml-training-group-v135
              topologyKey: kubernetes.io/hostname
      # v1.35: Enhanced topology spread
      topologySpreadConstraints:
      - maxSkew: 2
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            gang: ml-training-group-v135
        minDomains: 2  # v1.35 feature
      containers:
      - name: worker
        image: tensorflow/tensorflow:2.13.0-gpu-v135
        resources:
          requests:
            nvidia.com/gpu: 1
```

### Enhanced Scheduling Features
- **Gang Scheduling** - Coordinated scheduling for related pods
- **Improved Node Affinity** - More flexible node selection criteria
- **Advanced Pod Affinity/Anti-Affinity** - Better workload distribution controls
- **Enhanced Topology Spread Constraints** - More sophisticated spreading policies
- **Better Resource-Aware Scheduling** - Improved resource allocation decisions

### Performance Improvements
- **Faster Scheduling Decisions** - Optimized scheduler performance
- **Better Preemption Logic** - Smarter pod preemption strategies
- **Enhanced Scheduling Profiles** - More flexible scheduler configurations

---

## Node Affinity in v1.35

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-affinity-pod-v135
  labels:
    app: web-server
    version: v1.35
spec:
  affinity:
    nodeAffinity:
      # Enhanced required affinity
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/arch
            operator: In
            values: ["amd64", "arm64"]
          - key: node-type
            operator: NotIn
            values: ["spot", "preemptible"]
      # Improved preferred affinity
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: instance-type
            operator: In
            values: ["c5.large", "c5.xlarge"]
      - weight: 50
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values: ["us-west-2a", "us-west-2b"]
  containers:
  - name: web-server
    image: nginx:1.25
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
```

### Advanced Node Affinity with v1.35 Features

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: advanced-node-affinity-v135
  annotations:
    scheduler.kubernetes.io/version: "v1.35"
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        # Multiple node selector terms (OR logic)
        - matchExpressions:
          - key: node.kubernetes.io/instance-type
            operator: In
            values: ["m5.large", "m5.xlarge"]
          - key: topology.kubernetes.io/zone
            operator: In
            values: ["us-west-2a"]
        - matchExpressions:
          - key: node.kubernetes.io/instance-type
            operator: In
            values: ["c5.large", "c5.xlarge"]
          - key: topology.kubernetes.io/zone
            operator: In
            values: ["us-west-2b"]
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: workload-optimized
            operator: In
            values: ["true"]
      - weight: 60
        preference:
          matchFields:
          - key: metadata.name
            operator: In
            values: ["node-1", "node-2"]
  containers:
  - name: app
    image: busybox:1.36
    command: ["sleep", "3600"]
```

---

## Pod Affinity and Anti-Affinity in v1.35

### Pod Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-affinity-v135
  labels:
    app: cache
    tier: backend
spec:
  affinity:
    podAffinity:
      # Enhanced required pod affinity
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values: ["database"]
          - key: tier
            operator: In
            values: ["backend"]
        topologyKey: kubernetes.io/hostname
        # Namespace selector for cross-namespace affinity
        namespaceSelector:
          matchLabels:
            environment: production
      # Improved preferred affinity
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: web-server
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: cache
    image: redis:7.2
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "200m"
```

### Pod Anti-Affinity

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment-v135
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
        version: v1.35
    spec:
      affinity:
        podAntiAffinity:
          # Enhanced anti-affinity for HA
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values: ["web-server"]
            topologyKey: kubernetes.io/hostname
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values: ["web-server"]
              topologyKey: topology.kubernetes.io/zone
      containers:
      - name: web-server
        image: nginx:1.25
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
```

---

## Topology Spread Constraints in v1.35

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: advanced-topology-spread-v135
spec:
  replicas: 12
  selector:
    matchLabels:
      app: microservice
  template:
    metadata:
      labels:
        app: microservice
        component: api
    spec:
      topologySpreadConstraints:
      # v1.35: Multi-level topology spreading
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: microservice
        # v1.35: Minimum domains
        minDomains: 3
      - maxSkew: 2
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app: microservice
        minDomains: 6
      - maxSkew: 1
        topologyKey: node.kubernetes.io/instance-type
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            component: api
      containers:
      - name: api
        image: nginx:1.25
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
```

## Taints and Tolerations


```bash
# Enhanced taint management
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key1=value1:PreferNoSchedule

# Taint with effect time
kubectl taint nodes node1 key1=value1:NoExecute --overwrite

# Remove taint
kubectl taint nodes node1 key1=value1:NoSchedule-

# List node taints
kubectl describe nodes node1 | grep Taints
```

### Pod Tolerations

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerations-pod-v135
spec:
  tolerations:
  # Enhanced toleration matching
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
  - key: "key2"
    operator: "Exists"
    effect: "NoExecute"
    # Toleration seconds for graceful eviction
    tolerationSeconds: 300
  - key: "gpu"
    operator: "Equal"
    value: "nvidia"
    effect: "NoSchedule"
  # Wildcard tolerations
  - operator: "Exists"
    effect: "PreferNoSchedule"
  containers:
  - name: app
    image: nginx:1.25
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
        nvidia.com/gpu: 1
      limits:
        memory: "256Mi"
        cpu: "200m"
        nvidia.com/gpu: 1
```

## Gang Scheduling

```yaml
# v1.35: Gang scheduling for ML workloads
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-training-gang-v135
  annotations:
    scheduler.kubernetes.io/gang-scheduling: "enabled"
spec:
  replicas: 4
  selector:
    matchLabels:
      app: ml-training
  template:
    metadata:
      labels:
        app: ml-training
        gang: ml-training-group
      annotations:
        # Gang scheduling annotations
        scheduler.kubernetes.io/gang-name: "ml-training-group"
        scheduler.kubernetes.io/gang-min-size: "4"
        scheduler.kubernetes.io/gang-scheduling-timeout: "300s"
    spec:
      # Enhanced scheduling constraints for gang
      schedulerName: gang-scheduler
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  gang: ml-training-group
              topologyKey: kubernetes.io/hostname
      containers:
      - name: ml-worker
        image: tensorflow/tensorflow:2.13.0-gpu
        command: ["python", "-c", "import time; time.sleep(3600)"]
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
            nvidia.com/gpu: 1
          limits:
            memory: "4Gi"
            cpu: "2000m"
            nvidia.com/gpu: 1
        env:
        - name: GANG_SIZE
          value: "4"
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
```
### Scheduling Best Practices

1. **Use Node Affinity** for hardware-specific requirements
2. **Implement Pod Anti-Affinity** for high availability
3. **Apply Topology Spread Constraints** for even distribution
4. **Set Resource Requests** for proper scheduling decisions
5. **Use Priority Classes** for critical workloads

## Summary

Kubernetes v1.35 brings significant enhancements to pod scheduling:

- **Gang Scheduling** for coordinated workload placement
- **Enhanced Affinity Rules** with more flexible matching
- **Improved Topology Spread Constraints** for better distribution
- **Advanced Scheduler Profiles** for custom scheduling logic
- **Better Performance** with optimized scheduling algorithms
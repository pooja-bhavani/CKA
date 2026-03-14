# Pod Scheduling & Affinity 

## Overview

Pod scheduling in Kubernetes v1.35 provides sophisticated mechanisms to control where pods are placed in your cluster, with enhanced features for better resource utilization, performance optimization, and workload distribution.

## v1.35 Features:

- **Gang Scheduling (Alpha)** – native, workload‑aware gang scheduling via the **Workload API** (`scheduling.k8s.io/v1alpha1`), **disabled by default** and for experimental/advanced use only.
- **Workload‑Aware / Opportunistic Batching (Alpha/Beta)** – scheduler performance optimizations for large, batchy workloads, behind feature gates.
- **`minDomains` in `topologySpreadConstraints`** – now available to enforce a minimum number of failure domains (e.g., zones) used when spreading Pods.

## 🌍 Real-World Scenario

### Scenario: Machine Learning Platform - GPU Resource Optimization

**Business Context**: Your AI/ML platform serves 500+ data scientists running GPU training jobs. You must:

- Pack compatible GPU workloads efficiently.
- Avoid hot‑spot nodes.
- Spread workloads across zones for resilience.

### Deployment with Scheduling Constraints

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-training
  namespace: ml-platform
spec:
  replicas: 8
  selector:
    matchLabels:
      app: ml-training
  template:
    metadata:
      labels:
        app: ml-training
        gpu-type: nvidia-a100
    spec:
      # Standard kube-scheduler (no gang scheduling by default)
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
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: ml-training
              topologyKey: kubernetes.io/hostname
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: ml-training
      containers:
      - name: ml-worker
        image: tensorflow/tensorflow:2.13.0-gpu
        command: ["python", "/app/distributed_training.py"]
        resources:
          requests:
            cpu: "8000m"
            memory: "16Gi"
            nvidia.com/gpu: 2
          limits:
            cpu: "16000m"
            memory: "32Gi"
            nvidia.com/gpu: 2
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
```
**Key points:**

- Node affinity targets specific GPU types and memory sizes.
- Pod anti‑affinity spreads ML workers across nodes.
- Topology spread ensures balanced distribution across zones.


Node Affinity (v1.35 and Earlier)
Node affinity lets you express hard and soft preferences on node labels.

```
apiVersion: v1
kind: Pod
metadata:
  name: node-affinity-pod
  labels:
    app: web-server
spec:
  affinity:
    nodeAffinity:
      # Hard requirement
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/arch
            operator: In
            values: ["amd64", "arm64"]
          - key: node-type
            operator: NotIn
            values: ["spot", "preemptible"]
      # Soft preference
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
          - key: topology.kubernetes.io/zone
            operator: In
            values: ["us-west-2a", "us-west-2b"]
  containers:
  - name: web-server
    image: nginx:1.25
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```

###Pod Affinity and Anti‑Affinity

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-affinity
  labels:
    app: cache
    tier: backend
spec:
  affinity:
    podAffinity:
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
        # Cross-namespace selection
        namespaceSelector:
          matchLabels:
            environment: production
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
        cpu: "100m"
        memory: "256Mi"
      limits:
        cpu: "200m"
        memory: "512Mi"
```


**Pod Anti‑Affinity**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
    spec:
      affinity:
        podAntiAffinity:
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
                matchLabels:
                  app: web-server
              topologyKey: topology.kubernetes.io/zone
      containers:
      - name: web-server
        image: nginx:1.25
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```


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

## Topology Spread Constraints (with minDomains in v1.35)

Topology spread constraints keep Pods evenly spread across failure domains (zones, nodes, instance types).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: advanced-topology-spread
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
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        minDomains: 3  # ensure at least 3 zones are used
        labelSelector:
          matchLabels:
            app: microservice
      - maxSkew: 2
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway
        minDomains: 6
        labelSelector:
          matchLabels:
            app: microservice
      containers:
      - name: api
        image: nginx:1.25
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```

## Taints and Tolerations

```bash
# Add taints
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key2=value2:NoExecute

# Remove taint
kubectl taint nodes node1 key1=value1:NoSchedule-

# List node taints
kubectl describe node node1 | grep -i Taints

```

### Pod Tolerations

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerations-pod
spec:
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
  - key: "key2"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 300
  - operator: "Exists"
    effect: "PreferNoSchedule"
  containers:
  - name: app
    image: nginx:1.25
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```

### Scheduling Best Practices

1. **Use Node Affinity** for hardware-specific workloads (GPUs, ARM/AMD64).
2. **Apply Pod Anti‑Affinity** to spread replicas across nodes and zones for HA.
3. **Apply Topology Spread Constraints** (with minDomains) to enforce multi‑zone distribution.
4. **Set Resource Requests & Limits** so the scheduler can make informed placement decisions
5. **Use Priority Classes** o ensure critical workloads are scheduled first and can preempt lower‑priority Pods.
6. **Treat gang scheduling** and Workload‑aware features as advanced/opt‑in until they are stable.

## Summary

Kubernetes v1.35 brings significant enhancements to pod scheduling:

- **Gang Scheduling** for coordinated workload placement
- **Enhanced Affinity Rules** with more flexible matching
- **Improved Topology Spread Constraints** for better distribution
- **Advanced Scheduler Profiles** for custom scheduling logic
- **Better Performance** with optimized scheduling algorithms
--- 

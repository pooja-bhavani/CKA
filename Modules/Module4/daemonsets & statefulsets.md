## Overview

DaemonSets and StatefulSets are specialized workload controllers in Kubernetes v1.35 that provide unique deployment patterns for system services 
and stateful applications with enhanced features and improved reliability.

### DaemonSet Enhancements
**1. Improved Node Selection**

Advanced node affinity with complex expressions and automatic node discovery

**2. Better Resource Management**

Dynamic resource allocation based on node capacity and workload

### StatefulSet Improvements
**1. Faster Scaling**

Parallel scaling with dependency management

**2. Enhanced Storage Management**

Advanced storage lifecycle with automatic backup integration and corruption detection

**3. Improved Ordering - Flexible Deployment Strategies**

- **v1.34**: Strict sequential ordering only
- **v1.35**: Configurable ordering strategies (parallel, sequential, custom)

---

## DaemonSets in v1.35

### 1. DaemonSet Node Selection

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: gpu-driver-v135
spec:
  selector:
    matchLabels:
      app: gpu-driver
  template:
    spec:
      # v1.35: Advanced node selection
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: accelerator
                operator: In
                values: ["nvidia-tesla-k80", "nvidia-tesla-p100"]
              - key: kubernetes.io/os
                operator: In
                values: ["linux"]
      tolerations:
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
      containers:
      - name: gpu-driver
        image: nvidia/k8s-device-plugin:v0.14.1
        securityContext:
          privileged: true
        volumeMounts:
        - name: device-plugin
          mountPath: /var/lib/kubelet/device-plugins
      volumes:
      - name: device-plugin
        hostPath:
          path: /var/lib/kubelet/device-plugins
```

### 2. StatefulSet Parallel Pod Management

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: parallel-statefulset-v135
spec:
  serviceName: parallel-service
  replicas: 5
  # v1.35: Parallel pod management for faster scaling
  podManagementPolicy: Parallel
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0
      maxUnavailable: 2  # v1.35: Allow multiple pods to be unavailable
  selector:
    matchLabels:
      app: parallel-app
  template:
    spec:
      containers:
      - name: app
        image: nginx:1.25
        # v1.35: Enhanced startup handling
        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 30
          periodSeconds: 10
```

---

## v1.35 Storage Integration

### StatefulSet with Advanced Storage

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch-v135
spec:
  serviceName: elasticsearch
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    spec:
      containers:
      - name: elasticsearch
        image: elasticsearch:8.10.0
        env:
        - name: discovery.type
          value: single-node
        - name: ES_JAVA_OPTS
          value: "-Xms512m -Xmx512m"
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        volumeMounts:
        - name: elasticsearch-data
          mountPath: /usr/share/elasticsearch/data
        - name: elasticsearch-config
          mountPath: /usr/share/elasticsearch/config
  volumeClaimTemplates:
  - metadata:
      name: elasticsearch-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "fast-ssd"
      resources:
        requests:
          storage: 20Gi
  - metadata:
      name: elasticsearch-config
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

---



  
## v1.35 Best Practices

### DaemonSet Best Practices

1. **Resource Limits**: Always set resource requests and limits
2. **Tolerations**: Include appropriate tolerations for system pods
3. **Security Context**: Use non-root users when possible
4. **Health Checks**: Implement proper liveness and readiness probes
5. **Update Strategy**: Use RollingUpdate for zero-downtime updates

### StatefulSet Best Practices

1. **Headless Service**: Always create a headless service
2. **Storage Classes**: Use appropriate storage classes for performance
3. **Pod Anti-Affinity**: Spread pods across nodes for HA
4. **Graceful Shutdown**: Set appropriate terminationGracePeriodSeconds
5. **Backup Strategy**: Implement regular backup procedures

---

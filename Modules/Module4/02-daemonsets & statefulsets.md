## Overview

DaemonSets ensure one Pod per node for system services; StatefulSets manage stateful apps with stable identity, ordering, and storage. Kubernetes v1.35 promotes StatefulSet updateStrategy.rollingUpdate.maxUnavailable to Beta (default-enabled), enabling parallel rolling updates while controlling unavailable Pods.


### DaemonSet Enhancements
**1. Improved Node Selection**

Advanced node affinity with complex expressions and automatic node discovery

**2. Better Resource Management**

Dynamic resource allocation based on node capacity and workload

### v1.35 Improvements

- **StatefulSet maxUnavailable** Specify absolute number/percentage of unavailable Pods during RollingUpdate (default: 1). Speeds large deployments without availability loss.

- **DaemonSets** unchanged; standard nodeAffinity/tolerations available since v1.18+.
​

### Core Features (Stable)

- podManagementPolicy: Parallel/OrderedReady
- VolumeClaimTemplates for persistent storage
- Rolling updates with partition control


---

## DaemonSets in v1.35

### 1. GPU Node DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: gpu-driver
spec:
  selector:
    matchLabels:
      app: gpu-driver
  template:
    spec:
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
        image: nvidia/k8s-device-plugin:v0.14.5
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

### 2. StatefulSet Parallel Scaling with maxUnavailable

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: webapp
spec:
  serviceName: webapp
  replicas: 5
  podManagementPolicy: Parallel  # Stable since v1.7
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0
      maxUnavailable: 20%  # v1.35 Beta: Up to 20% unavailable during update
  selector:
    matchLabels:
      app: webapp
  template:
    spec:
      containers:
      - name: webapp
        image: nginx:1.27
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

Elasticsearch Storage Example

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
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
        image: docker.elastic.co/elasticsearch/elasticsearch:8.12.0
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
      storageClassName: fast-ssd
```

---


  
## v1.35 Best Practices

### DaemonSet 

- Set resources.requests/limits and tolerations for tainted nodes
- Use hostNetwork: true for low-latency services
- Implement livenessProbe to handle node restarts gracefully

### StatefulSet Best Practices
- Use maxUnavailable: 10-25% for faster rollouts (v1.35 Beta)
- Combine podManagementPolicy: Parallel + maxUnavailable for scale
- Always pair with Headless Service for discovery
- Set terminationGracePeriodSeconds: 30+ for DBs

---

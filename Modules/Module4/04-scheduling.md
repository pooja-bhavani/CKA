# Pod Scheduling & Affinity 

## Overview

Pod scheduling in Kubernetes v1.35 provides sophisticated mechanisms to control where pods are placed in your cluster, with enhanced features for better resource utilization, performance optimization, and workload distribution.

## 🌍 Real-World Scenarios

### Scenario 1: Machine Learning Platform - GPU Resource Optimization

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
# v1.35: Intelligent GPU scheduling with gang scheduling
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
        # v1.35: Gang scheduling annotations
        scheduler.kubernetes.io/gang-name: "distributed-training-group-v135"
        scheduler.kubernetes.io/gang-min-size: "8"
        scheduler.kubernetes.io/gang-scheduling-timeout: "300s"
    spec:
      # v1.35: Enhanced scheduling for ML workloads
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
        # v1.35: Enhanced pod anti-affinity for distributed training
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
      # v1.35: Advanced topology spread for ML workloads
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

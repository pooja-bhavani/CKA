# Resource Management in Kubernetes

## Overview

Resource management in Kubernetes ensures that pods get the CPU and memory they need to run effectively, while protecting the cluster from being overwhelmed by resource-hungry applications. It defines boundaries for resource consumption and provides mechanisms to manage resource scarcity.

## Why Resource Management Matters

- **Cluster Stability**: Prevents any single application from consuming all node resources, which could cause node crashes.
- **Fair Allocation**: Ensures multiple tenants and workloads get their fair share of cluster resources.
- **Predictable Performance**: Guarantees baseline resources so applications perform consistently.
- **Cost Efficiency**: Helps pack pods efficiently to maximize hardware utilization.

---

## Core Concepts

### CPU and Memory

Kubernetes manages two primary compute resources:
- **CPU**: Measured in cores or millicores (e.g., `1` core = `1000m`). CPU is a "compressible" resource (pods can be throttled but won't be killed if they use too much).
- **Memory**: Measured in bytes (e.g., `256Mi`, `1Gi`). Memory is an "incompressible" resource (pods will be OOMKilled if they exceed their limits).

### Requests vs Limits

Resource configuration is defined at the container level using two key concepts:

1. **Requests**:
   - The *guaranteed* minimum amount of a resource the container needs.
   - Used by the scheduler to decide which node to place the pod on.
   - Example: If a pod requests `500m` CPU, the scheduler finds a node with at least `500m` unallocated CPU capacity.

2. **Limits**:
   - The *absolute maximum* amount of a resource the container is allowed to use.
   - Enforced by the container runtime (e.g., cgroups in Linux).
   - If a pod exceeds CPU limits, it is throttled. If it exceeds memory limits, it is terminated (`OOMKilled`).

### Example Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx:1.25
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

---

## Quality of Service (QoS) Classes

Kubernetes automatically assigns a QoS class to every pod based on its resource requests and limits. QoS classes determine which pods get evicted first when a node runs out of resources (memory or ephemeral storage).

### 1. Guaranteed
**Definition**: Every container in the pod has *both* CPU and memory requests and limits set, and `requests == limits`.
**Eviction Priority**: Lowest risk. Guaranteed pods are only killed if they exceed their limits or if there's an extreme system-level problem.

```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "1"
  limits:
    memory: "512Mi"
    cpu: "1"
```

### 2. Burstable
**Definition**: At least one container has a memory or CPU request set, but it does not meet the criteria for Guaranteed (e.g., `requests < limits`, or limits are not set for all resources).
**Eviction Priority**: Medium risk. Killed if the node runs out of resources and no BestEffort pods remain.

```yaml
resources:
  requests:
    memory: "128Mi"
  limits:
    memory: "256Mi"
```

### 3. BestEffort
**Definition**: No containers in the pod have any memory or CPU requests or limits set.
**Eviction Priority**: Highest risk. These pods are the first to be killed when the node experiences resource pressure.

```yaml
resources: {} # No resources defined
```

---

## LimitRanges

A `LimitRange` sets default resource requests and limits for pods or containers in a specific namespace. It can also enforce minimum and maximum resource boundaries.

**Use Case**: Prevent users from creating excessively large or excessively small pods in a shared namespace, and automatically inject sensible defaults if users forget to set them.

**Example `LimitRange`**:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: development
spec:
  limits:
  - type: Container
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    default: # This is the default Limit
      cpu: "500m"
      memory: "256Mi"
    min:
      cpu: "50m"
      memory: "64Mi"
    max:
      cpu: "2"
      memory: "1Gi"
```

---

## ResourceQuotas

While `LimitRanges` restrict individual pods/containers, a `ResourceQuota` limits the *aggregate* resource consumption across an entire namespace.

**Use Case**: Prevent a single team or application (running in one namespace) from consuming the entire cluster's capacity.

**Example `ResourceQuota`**:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: development
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
    pods: "20"
```

**What this means:**
- The `development` namespace can request a total of up to 4 CPUs and 8Gi of memory.
- It cannot exceed an aggregate limit of 8 CPUs and 16Gi memory.
- It can run a maximum of 20 pods.

---

## Troubleshooting Resource Issues

### 1. OOMKilled (Out of Memory)
**Symptom**: Pod status shows `OOMKilled`.
**Cause**: The pod tried to allocate more memory than its configured memory **limit**.
**Fix**: Investigate application memory leaks, or increase the memory `limit` in the pod spec.

### 2. CPU Throttling
**Symptom**: Application performance is slow, but the pod is running. Metrics show high CPU throttling.
**Cause**: The pod is constantly hitting its CPU **limit**.
**Fix**: Increase the CPU `limit`, or optimize the application's CPU usage.

### 3. FailedScheduling
**Symptom**: Pod status is `Pending`. `kubectl describe pod` shows `FailedScheduling` with messages like `0/3 nodes are available: 3 Insufficient cpu`.
**Cause**: The pod's resource **requests** are too high. No node in the cluster has enough unallocated capacity to host it.
**Fix**: Reduce the pod's resource requests, or add more/larger nodes to the cluster.

### 4. QuotaExceeded
**Symptom**: Cannot create new pods. Error message mentions `forbidden: exceeded quota`.
**Cause**: The namespace has hit its `ResourceQuota` limit.
**Fix**: Delete unused pods/resources, reduce requests/limits on new pods, or request an increase in the namespace quota.

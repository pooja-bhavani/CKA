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

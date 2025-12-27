# Helm and Kustomize

## Overview

**Helm** and **Kustomize** are two popular tools for managing Kubernetes applications:

- **Helm**: Package manager for Kubernetes 
- **Kustomize**: Template-free configuration management (built into kubectl)

Both tools help deploy and manage applications in Kubernetes v1.35 clusters.

## Helm Fundamentals

### Installation (v1.35 Compatible)

```bash
# Install Helm v3 (latest)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Installing Cluster Components with Helm and Kustomize

# Ubuntu/Debian
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

# Verify installation
helm version
```

### 1. ArgoCD Installation

```bash
# Add ArgoCD repository
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Install ArgoCD
helm install argocd argo/argo-cd \
  -n argocd --create-namespace \
  --set server.securityContext.runAsNonRoot=true \
  --set controller.securityContext.runAsNonRoot=true

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Access ArgoCD UI
kubectl port-forward -n argocd svc/argocd-server 8080:443
```

**Monitoring installation**

### 2. Prometheus Stack with Helm

```bash
# Add Prometheus community repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack (Prometheus + Alertmanager + Grafana)
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace \
  --set prometheus.prometheusSpec.podSecurityContext.runAsNonRoot=true \
  --set prometheus.prometheusSpec.podSecurityContext.runAsUser=1000 \
  --set grafana.securityContext.runAsNonRoot=true \
  --set grafana.securityContext.runAsUser=472

# Check components
kubectl get pods -n monitoring
helm status monitoring -n monitoring

# Access Grafana (default: admin/prom-operator)
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

## Kustomize Fundamentals

### What is Kustomize?

Kustomize is a template-free configuration management tool built into kubectl. It uses a declarative approach to customize Kubernetes configurations.

### Core Concepts

#### 1. Base
Common resources shared across environments.

#### 2. Overlay
Environment-specific customizations applied to the base.

#### 3. Kustomization
A file that describes how to generate or transform Kubernetes resources.


### Example 1: GitOps with ArgoCD and Kustomize

#### ArgoCD Application
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: webapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/webapp-config
    targetRevision: HEAD
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: webapp
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

## Troubleshooting

### Helm Troubleshooting

#### Common Issues and Solutions

**Issue 1: Release Installation Fails**
```bash
# Debug with dry-run
helm install my-app ./chart --dry-run --debug

# Check template rendering
helm template my-app ./chart

# Check values
helm get values my-app

# Check release history
helm history my-app
```

**Issue 2: Upgrade Fails**
```bash
# Check what changed
helm diff upgrade my-app ./chart

# Force upgrade if needed
helm upgrade my-app ./chart --force

# Rollback if upgrade fails
helm rollback my-app 1
```

### Kustomize Troubleshooting

#### Common Issues and Solutions

**Issue 1: Build Fails**
```bash
# Check kustomization syntax
kubectl kustomize ./overlays/prod --dry-run

# Validate YAML
yamllint kustomization.yaml

# Check resource paths
ls -la base/
ls -la overlays/prod/
```

**Issue 2: Patches Not Applied**
```bash
# Check patch format
kubectl kustomize ./overlays/prod | grep -A 10 -B 10 "replicas"

# Verify patch target
kubectl kustomize ./base | grep "name:"
```

**Issue 3: Resource Conflicts**
```bash
# Check for duplicate resources
kubectl kustomize ./overlays/prod | grep "kind:\|name:"

# Use strategic merge patches
patchesStrategicMerge:
- patch.yaml
```

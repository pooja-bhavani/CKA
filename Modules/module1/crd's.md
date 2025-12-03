# Understanding role of CRDs and operators

## Overview

Custom Resource Definitions (CRDs) extend the Kubernetes API by allowing you to define your own custom resources. Operators use
CRDs to manage complex applications by encoding operational knowledge into software.

**What are CRDs?**
CRDs allow you to extend Kubernetes by defining new resource types without modifying the Kubernetes source code.

**Why Use CRDs?**

- Extend Kubernetes API with domain-specific resources (for example databases.example.com), with its own schema, versions, and scope
- Manage complex applications declaratively
- Leverage Kubernetes features (RBAC, kubectl, API server)
- Enable GitOps workflows
- Provide a contract that Operators/controllers can watch and reconcile, encoding operational runbooks into code.

### CRD Architecture

```
┌─────────────────────────────────────┐
│         Kubernetes API              │
│                                     │
│  Built-in Resources    CRDs         │
│  ├── Pod              ├── Database  │
│  ├── Service          ├── Backup    │
│  └── Deployment       └── App       │
└─────────────────────────────────────┘
         │                    │
         ▼                    ▼
    ┌─────────┐         ┌──────────┐
    │  etcd   │         │   etcd   │
    │(built-in)│        │ (custom) │
    └─────────┘         └──────────┘
```
### Understanding Operators

**What is an Operator?**
Operator is a method of packaging, deploying, and managing a Kubernetes application. It extends Kubernetes by using custom resources and controllers to automate operational tasks.

Operator = CRD + Controller + Operational Knowledge

### Operator Pattern

```
┌─────────────────────────────────────────┐
│           Kubernetes API                │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│         Custom Resource (CR)            │
│  apiVersion: example.com/v1             │
│  kind: Database                         │
│  spec:                                  │
│    engine: postgres                     │
│    replicas: 3                          │
└────────────┬────────────────────────────┘
             │
             │ watches
             ▼
┌─────────────────────────────────────────┐
│         Operator (Controller)           │
│                                         │
│  1. Watch for changes                   │
│  2. Compare desired vs actual state     │
│  3. Take action to reconcile            │
│  4. Update status                       │
└────────────┬────────────────────────────┘
             │
             │ creates/manages
             ▼
┌─────────────────────────────────────────┐
│      Kubernetes Resources               │
│  - StatefulSet                          │
│  - Service                              │
│  - ConfigMap                            │
│  - PersistentVolumeClaim                │
└─────────────────────────────────────────┘
```

## Common Operators

### 1. Prometheus Operator

**Purpose**: Manages Prometheus monitoring instances

**CRDs**:
- Prometheus
- ServiceMonitor
- AlertManager
- PrometheusRule

**Example:**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 2
  serviceAccountName: prometheus
  serviceMonitorSelector:
    matchLabels:
      team: frontend
  resources:
    requests:
      memory: 400Mi
```
**Installation:**

```bash
# Install Prometheus Operator
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml

# Verify CRDs
kubectl get crds | grep monitoring.coreos.com
```
### 2. Cert-Manager Operator

**Purpose**: Manages TLS certificates

**CRDs**:
- Certificate
- Issuer
- ClusterIssuer
- CertificateRequest

**Example:**

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-com
  namespace: default
spec:
  secretName: example-com-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - example.com
  - www.example.com
```
**Installation:**

```bash
# Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Verify CRDs
kubectl get crds | grep cert-manager.io
```
### 3. ArgoCD Operator

**Purpose**: Manages GitOps continuous delivery

**CRDs**:
- Application
- AppProject
- ApplicationSet

**Example:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
### 4. Istio Operator

**Purpose**: Manages Istio service mesh

**CRDs**:
- VirtualService
- DestinationRule
- Gateway
- ServiceEntry

**Example:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```



















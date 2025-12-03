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



















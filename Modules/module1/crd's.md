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

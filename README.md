# Complete CKA (Certified Kubernetes Administrator) Guide

<div align="center">

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.35-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](LICENSE)

</div>


> The most comprehensive, hands-on CKA exam preparation guide for Kubernetes v1.35 with real-world projects

## 🎯 What's New in Kubernetes v1.35

This repository is **fully updated for Kubernetes v1.35** with cutting-edge features:

- 🆕 **Gateway API v1.4** - Next-generation ingress with Policy API
- 🆕 **In-Place Pod Updates** - Update resources without pod restart
- 🆕 **User Namespaces** - Enhanced container security isolation
- 🆕 **Gang Scheduling** - Coordinated pod scheduling for ML/AI workloads
- 🆕 **Enhanced Debugging** - Improved troubleshooting capabilities
- 🆕 **Pod Certificates** - Native certificate management for pods

---

### 📚 Learning Sequence - Module Files (Complete Path)

#### **MODULE 1: Cluster Architecture, Installation & Configuration (25%)**

| # | File | Topic | Subtopics |
|---|------|-------|-----------|
| **1** | [`Modules/module-1/01-cluster-lifecycle.md`](./Modules/module-1/01-cluster-lifecycle.md) | **Cluster Lifecycle** | Manage the lifecycle of Kubernetes clusters • Create and manage Kubernetes clusters using kubeadm • Implement and configure a highly-available control plane |
| **2** | [`Modules/module-1/02-pod-security.md`](./Modules/module-1/02-pod-security.md) | **Pod Security** | Prepare underlying infrastructure for installing a Kubernetes cluster |
| **3** | [`Modules/module-1/03-rbac.md`](./Modules/module-1/03-rbac.md) | **RBAC & Access Control** | Manage role based access control (RBAC) |
| **4** | [`Modules/module-1/04-crds.md`](./Modules/module-1/04-crds.md) | **Custom Resources (CRDs)** | Understand CRDs, install and configure operators |
| **5** | [`Modules/module-1/05-helm-and-kustomize.md`](./Modules/module-1/05-helm-and-kustomize.md) | **Helm & Kustomize** | Use Helm and Kustomize to install cluster components • Understand extension interfaces (CNI, CSI, CRI, etc.) |

---

#### **MODULE 2: Services & Networking (20%)**

| # | File | Topic | Subtopics |
|---|------|-------|-----------|
| **6** | [`Modules/module-2/01-pod-networking.md`](./Modules/module-2/01-pod-networking.md) | **Pod Networking Fundamentals** | Understand connectivity between Pods |
| **7** | [`Modules/module-2/02-service-endpoints.md`](./Modules/module-2/02-service-endpoints.md) | **Services & Endpoints** | Use ClusterIP, NodePort, LoadBalancer service types and endpoints |
| **8** | [`Modules/module-2/03-networking-policies.md`](./Modules/module-2/03-networking-policies.md) | **Network Policies** | Define and enforce Network Policies |
| **9** | [`Modules/module-2/04-core-dns.md`](./Modules/module-2/04-core-dns.md) | **CoreDNS Configuration** | Understand and use CoreDNS |
| **10** | [`Modules/module-2/05-api-gateway-fundamentals.md`](./Modules/module-2/05-api-gateway-fundamentals.md) | **Ingress & API Gateway Basics** | Know how to use Ingress controllers and Ingress resources |
| **11** | [`Modules/module-2/06-api-gateway-migration.md`](./Modules/module-2/06-api-gateway-migration.md) | **Advanced API Gateway Patterns** | Use the Gateway API to manage Ingress traffic |

---

#### **MODULE 3: Workloads & Scheduling (15%)**

| # | File | Topic | Subtopics |
|---|------|-------|-----------|
| **12** | [`Modules/module-3/01-deployments-and-replicasets.md`](./Modules/module-3/01-deployments-and-replicasets.md) | **Deployments & ReplicaSets** | Understand application deployments and how to perform rolling update and rollbacks |
| **13** | [`Modules/module-3/02-daemonsets-and-statefulsets.md`](./Modules/module-3/02-daemonsets-and-statefulsets.md) | **DaemonSets & StatefulSets** | Understand the primitives used to create robust, self-healing, application deployments |
| **14** | [`Modules/module-3/03-jobs-and-cronjobs.md`](./Modules/module-3/03-jobs-and-cronjobs.md) | **Jobs & CronJobs** | ConfigMaps and Secrets to configure applications |
| **15** | [`Modules/module-3/04-scheduling.md`](./Modules/module-3/04-scheduling.md) | **Scheduling** | Configure Pod admission and scheduling (limits, node affinity, etc.) |
| **16** | [`Modules/module-3/05-resource-management.md`](./Modules/module-3/05-resource-management.md) | **Resource Management** | Configure workload autoscaling |

---

#### **MODULE 4: Storage (10%)**

| # | File | Topic | Subtopics |
|---|------|-------|-----------|
| **17** | [`Modules/module-4/01-volumes-and-mounts.md`](./Modules/module-4/01-volumes-and-mounts.md) | **Volumes & Types** | Configure volume types, access modes and reclaim policies |
| **18** | [`Modules/module-4/02-persistent-volumes-and-claims.md`](./Modules/module-4/02-persistent-volumes-and-claims.md) | **PVs and PVCs** | Manage persistent volumes and persistent volume claims |
| **19** | [`Modules/module-4/03-storage-classes.md`](./Modules/module-4/03-storage-classes.md) | **Storage Classes** | Implement storage classes and dynamic volume provisioning |

---

#### **MODULE 5: Troubleshooting (30%)**

| # | File | Topic | Subtopics |
|---|------|-------|-----------|
| **20** | [`Modules/module-5/01-cluster-and-node-troubleshooting.md`](./Modules/module-5/01-cluster-and-node-troubleshooting.md) | **Cluster & Nodes** | Troubleshoot clusters and nodes • Troubleshoot cluster components |
| **21** | [`Modules/module-5/02-application-troubleshooting.md`](./Modules/module-5/02-application-troubleshooting.md) | **App Troubleshooting** | Monitor cluster and application resource usage • Manage and evaluate container output streams |
| **22** | [`Modules/module-5/03-networking-troubleshooting.md`](./Modules/module-5/03-networking-troubleshooting.md) | **Network Troubleshooting** | Troubleshoot services and networking |

---

### ✅ Quick Module Navigation

**🔗 Module 1 Files:**
- [01-cluster-lifecycle.md](./Modules/module-1/01-cluster-lifecycle.md)
- [02-pod-security.md](./Modules/module-1/02-pod-security.md)
- [03-rbac.md](./Modules/module-1/03-rbac.md)
- [04-crds.md](./Modules/module-1/04-crds.md)
- [05-helm-and-kustomize.md](./Modules/module-1/05-helm-and-kustomize.md)

**🔗 Module 2 Files:**
- [01-pod-networking.md](./Modules/module-2/01-pod-networking.md)
- [02-service-endpoints.md](./Modules/module-2/02-service-endpoints.md)
- [03-networking-policies.md](./Modules/module-2/03-networking-policies.md)
- [04-core-dns.md](./Modules/module-2/04-core-dns.md)
- [05-api-gateway-fundamentals.md](./Modules/module-2/05-api-gateway-fundamentals.md)
- [06-api-gateway-migration.md](./Modules/module-2/06-api-gateway-migration.md)

**🔗 Module 3 Files:**
- [01-deployments-and-replicasets.md](./Modules/module-3/01-deployments-and-replicasets.md)
- [02-daemonsets-and-statefulsets.md](./Modules/module-3/02-daemonsets-and-statefulsets.md)
- [03-jobs-and-cronjobs.md](./Modules/module-3/03-jobs-and-cronjobs.md)
- [04-scheduling.md](./Modules/module-3/04-scheduling.md)
- [05-resource-management.md](./Modules/module-3/05-resource-management.md)

**🔗 Module 4 Files:**
- [01-volumes-and-mounts.md](./Modules/module-4/01-volumes-and-mounts.md)
- [02-persistent-volumes-and-claims.md](./Modules/module-4/02-persistent-volumes-and-claims.md)
- [03-storage-classes.md](./Modules/module-4/03-storage-classes.md)

**🔗 Module 5 Files:**
- [01-cluster-and-node-troubleshooting.md](./Modules/module-5/01-cluster-and-node-troubleshooting.md)
- [02-application-troubleshooting.md](./Modules/module-5/02-application-troubleshooting.md)
- [03-networking-troubleshooting.md](./Modules/module-5/03-networking-troubleshooting.md)

---

### 📖 How to Study These Modules

1. **Module 1** (Cluster Architecture, Installation & Configuration) - 25% of exam
2. **Module 2** (Services & Networking) - 20% of exam
3. **Module 3** (Workloads & Scheduling) - 15% of exam
4. **Module 4** (Storage) - 10% of exam
5. **Module 5** (Troubleshooting) - 30% of exam
6. Click any file above to navigate directly to it
7. Each file has specific topics and subtopics to master
8. Practice after each module before moving to next

---

## 🚀 Getting Started

To get started with this Kubernetes guide, follow the sequence of modules listed above. Each module contains hands-on labs and theoretical explanations to help you master the CKA exam.

### Prerequisites

- A running Kubernetes cluster (v1.35 recommended)
- `kubectl` installed and configured
- `kind` (optional, for local clusters)

### Create a kind cluster

```bash
kind create cluster --config kind-config.yaml
```

*Verify cluster creation:*
```bash
kubectl get nodes
```

---

## 🧩 Finding it Difficult?

Don't worry — share your doubt as a post or reach out on:
- 💬 **[Discord Community](https://discord.gg/t8bF6Vux88)**
- 💭 **[LinkedIn](https://www.linkedin.com/company/trainwithshubham/)**
- 💬 **[Official Website](https://www.trainwithshubham.com/)** 

<div align="center">
Happy Learning!

**TrainWithShubham**
</div>
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


## Get Started

### Create a kind cluster 

```
kind create cluster --config kind-config.yaml
```

* Verify cluster creation 
```
kubectl get nodes
```


<img width="679" height="292" alt="image" src="https://github.com/user-attachments/assets/bb31e371-d615-4815-8350-7b88787ed3b5" />

* Load image into kind
```
kind load docker-image bankapp:docker --name bankapp
```

* Verify is the pod is visible
```
kubectl run bankapp-test \
  --image=bankapp:docker \
  --restart=Never \
  --port=8080 \
  -n bankapp
```
```
kubectl get pod bankapp-test -n bankapp
```
<img width="1016" height="271" alt="image" src="https://github.com/user-attachments/assets/0b7adc74-0a9a-4c9e-bf94-726941e792e7" />


* Create a namespace

```
kubectl create namespace bankapp
```
* Apply all the yaml files

```
kubectl apply -f <filename>
```

<img width="1015" height="321" alt="image" src="https://github.com/user-attachments/assets/ed51cf44-2682-43aa-819d-e4ee07d702e7" />

* Check the pod status
```
kubectl get pods -n bankapp
```
* Describe the pod for more information
```
kubectl describe pod <podname> -n namespace
```
<img width="803" height="86" alt="image" src="https://github.com/user-attachments/assets/39217fed-7271-4eda-8d21-a4de356889d0" />

Wait until all pods are Running.

<img width="1022" height="294" alt="image" src="https://github.com/user-attachments/assets/550c99d4-582d-4102-a124-4eee4de9e0a2" />

## Ollama
* Build or pull the image locally

```
docker build -t ollama/ollama:local .
```

* Load it into the kind cluster

```
kind load docker-image ollama/ollama:local --name bankapp
```

* Apply ollama yaml files
  
```
kubectl apply -f ollama/ollama-deployment.yaml
```

```
kubectl apply -f ollama/ollama-service.yaml
```

* Once it’s Running, pull tinyllama

```
kubectl exec -it deploy/ollama -n bankapp -- bash
ollama pull tinyllama
ollama list
exit
```

<img width="944" height="254" alt="image" src="https://github.com/user-attachments/assets/fa49c070-2965-4779-8aa2-6bcb5502d218" />
<img width="692" height="115" alt="image" src="https://github.com/user-attachments/assets/a4799687-7260-49b0-9b73-90bb91f46c42" />


Run the app

```
http://localhost:8080
```

<img width="1463" height="915" alt="image" src="https://github.com/user-attachments/assets/6d0788dc-1e27-4fb0-bfaa-1a61d0c0f5bd" />

<img width="1454" height="911" alt="image" src="https://github.com/user-attachments/assets/d6cb7b09-9bec-453e-a4b4-326b8538d918" />




## 🧩 Finding it Difficult?

Don't worry — share your doubt as a post or reach out on:
- 💬 **[Discord Community](https://discord.gg/t8bF6Vux88)**
- 💭 **[LinkedIn](https://www.linkedin.com/company/trainwithshubham/)**
- 💬 **[Official Website](https://www.trainwithshubham.com/)** 

<div align="center">
Happy Learning!

**TrainWithShubham**
</div>
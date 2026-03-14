# Complete CKA (Certified Kubernetes Administrator) Guide

<div align="center">
[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.35-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](LICENSE)
</div>


> The most comprehensive, hands-on CKA exam preparation guide for Kubernetes v1.35 with real-world projects

## 🎯 What's New in Kubernetes v1.35

This repository is **fully updated for Kubernetes v1.35** with cutting-edge features:

- 🆕 **Gateway API v1.1** - Next-generation ingress with Policy API
- 🆕 **In-Place Pod Updates** - Update resources without pod restart
- 🆕 **User Namespaces** - Enhanced container security isolation
- 🆕 **Gang Scheduling** - Coordinated pod scheduling for ML/AI workloads
- 🆕 **Enhanced Debugging** - Improved troubleshooting capabilities
- 🆕 **Pod Certificates** - Native certificate management for pods

---

### 📚 Learning Sequence - Module Files (Complete Path)

#### **MODULE 1: Cluster Architecture & Administration**

| # | File | Topic | Subtopics | Exam Weight |
|---|------|-------|-----------|------------|
| **1** | [`Modules/module-1/01-cluster-lifecycle.md`](./Modules/module-1/01-cluster-lifecycle.md) | **Cluster Lifecycle** | kubeadm init • cluster upgrades • etcd backup/restore • cluster scaling | 25% |
| **2** | [`Modules/module-1/02-pod-security.md`](./Modules/module-1/02-pod-security.md) | **Pod Security** | Pod security policies • certificate management • security standards • network policies | 15% |
| **3** | [`Modules/module-1/03-rbac.md`](./Modules/module-1/03-rbac.md) | **RBAC & Access Control** | Roles • ClusterRoles • RoleBindings • ServiceAccounts • permissions | 15% |
| **4** | [`Modules/module-1/04-crds.md`](./Modules/module-1/04-crds.md) | **Custom Resources (CRDs)** | Custom resource definitions • APIgroups • custom objects • schemas | 25% |
| **5** | [`Modules/module-1/05-helm-and-kustomize.md`](./Modules/module-1/05-helm-and-kustomize.md) | **Kustomize** | Configuration management • overlays • patches • templates • merging | 25% |

---

#### **MODULE 2: Networking & Services**

| # | File | Topic | Subtopics | Exam Weight |
|---|------|-------|-----------|------------|
| **6** | [`Modules/module-2/01-pod-networking.md`](./Modules/module-2/01-pod-networking.md) | **Pod Networking Fundamentals** | Pod-to-pod communication • DNS resolution • Service discovery • CoreDNS | 20% |
| **7** | [`Modules/module-2/02-service-endpoints.md`](./Modules/module-2/02-service-endpoints.md) | **Services & Endpoints** | ClusterIP services • NodePort services • LoadBalancer services • endpoint management | 20% |
| **8** | [`Modules/module-2/03-networking-policies.md`](./Modules/module-2/03-networking-policies.md) | **Network Policies** | Network segmentation • ingress rules • egress rules • traffic filtering | 20% |
| **9** | [`Modules/module-2/04-core-dns.md`](./Modules/module-2/04-core-dns.md) | **CoreDNS Configuration** | DNS resolution • DNS troubleshooting • name resolution • CoreDNS setup | 20% |

---

#### **MODULE 3: Advanced Topics & Ingress**

| # | File | Topic | Subtopics | Exam Weight |
|---|------|-------|-----------|------------|
| **10** | [`Modules/module-3/01-api-gateway-fundamentals.md`](./Modules/module-3/01-api-gateway-fundamentals.md) | **Ingress & API Gateway Basics** | Ingress controller • routing rules • host-based routing • path-based routing | 20% |
| **11** | [`Modules/module-3/02-api-gateway-migration.md`](./Modules/module-3/02-api-gateway-migration.md) | **Advanced API Gateway Patterns** | Traffic management • blue-green deployments • canary deployments • advanced routing | 20% |

---

#### **MODULE 4: Workloads & Scheduling**

| # | File | Topic | Subtopics | Exam Weight |
|---|------|-------|-----------|------------|
| **12** | [`Modules/module-4/01-deployments-and-replicasets.md`](./Modules/module-4/01-deployments-and-replicasets.md) | **Deployments & ReplicaSets** | Deployments • rolling updates • rollbacks • scaling | 15% |
| **13** | [`Modules/module-4/02-daemonsets-and-statefulsets.md`](./Modules/module-4/02-daemonsets-and-statefulsets.md) | **DaemonSets & StatefulSets** | Daemonsets • stateful applications • persistent volumes | 15% |
| **14** | [`Modules/module-4/03-jobs-and-cronjobs.md`](./Modules/module-4/03-jobs-and-cronjobs.md) | **Jobs & CronJobs** | Batch processing • scheduled tasks • parallel executions | 10% |
| **15** | [`Modules/module-4/04-scheduling.md`](./Modules/module-4/04-scheduling.md) | **Scheduling** | nodeSelector • affinity • anti-affinity • taints & tolerations | 15% |
| **16** | [`Modules/module-4/05-resource-management.md`](./Modules/module-4/05-resource-management.md) | **Resource Management** | CPU/Memory requests & limits • ResourceQuotas • LimitRanges | 20% |

---

### ✅ Quick Module Navigation

**🔗 Module 1 Files:**
- [01-cluster-lifecycle.md](./Modules/module-1/01-cluster-lifecycle.md) - kubeadm, upgrades, backups
- [02-pod-security.md](./Modules/module-1/02-pod-security.md) - security policies, certificates
- [03-rbac.md](./Modules/module-1/03-rbac.md) - roles, bindings, permissions
- [04-crds.md](./Modules/module-1/04-crds.md) - custom resources, APIs
- [05-helm-and-kustomize.md](./Modules/module-1/05-helm-and-kustomize.md) - configuration management

**🔗 Module 2 Files:**
- [01-pod-networking.md](./Modules/module-2/01-pod-networking.md) - pod communication, DNS
- [02-service-endpoints.md](./Modules/module-2/02-service-endpoints.md) - services, endpoints
- [03-networking-policies.md](./Modules/module-2/03-networking-policies.md) - network segmentation
- [04-core-dns.md](./Modules/module-2/04-core-dns.md) - DNS troubleshooting

**🔗 Module 3 Files:**
- [01-api-gateway-fundamentals.md](./Modules/module-3/01-api-gateway-fundamentals.md) - ingress basics
- [02-api-gateway-migration.md](./Modules/module-3/02-api-gateway-migration.md) - advanced patterns

**🔗 Module 4 Files:**
- [01-deployments-and-replicasets.md](./Modules/module-4/01-deployments-and-replicasets.md) - deployments, updates
- [02-daemonsets-and-statefulsets.md](./Modules/module-4/02-daemonsets-and-statefulsets.md) - daemonsets, statefulsets
- [03-jobs-and-cronjobs.md](./Modules/module-4/03-jobs-and-cronjobs.md) - jobs, scheduling
- [04-scheduling.md](./Modules/module-4/04-scheduling.md) - taints, tolerations, affinity
- [05-resource-management.md](./Modules/module-4/05-resource-management.md) - quotas, limits

---

### 📖 How to Study These Modules

1. **Start with Module 1** (Cluster Architecture) - Foundation for everything
2. **Then Module 2** (Networking) - Critical for service deployment
3. **Finally Module 3** (Advanced Topics) - Build on networking concepts
4. Click any file above to navigate directly to it
5. Each file has specific topics and subtopics to master
6. Practice after each module before moving to next


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
  --port=8080
```
```
kubectl get pod bankapp-test
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
kind load docker-image ollama/ollama:latest --name
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
- 💭 **[LinkedIn](https://www.linkedin.com/in/shubhamlondhe1996/)**
- 💬 **[Official Website](https://www.trainwithshubham.com/)** 

<div align="center">
Happy Learning!

**TrainWithShubham**
</div>
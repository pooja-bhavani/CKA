# 🚀 Complete CKA (Certified Kubernetes Administrator) Exam Preparation Guide

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.35-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](LICENSE)
[![CKA](https://img.shields.io/badge/CKA-v1.35%20Ready-success?style=for-the-badge)](https://www.cncf.io/certification/cka/)

> The most comprehensive, hands-on CKA exam preparation guide for Kubernetes v1.35 with real-world projects

---

## 📋 Quick Navigation Map

```
CKA Repository Structure
├── 📄 Introduction-to-kubernetes.md      ← START HERE (Fundamentals)
├── 📄 lifecycle-management.md             ← Cluster lifecycle & operations
│
├── 📁 Modules/                            ← Core Learning Path
│   ├── 📁 module1/                        → Cluster Setup & Architecture
│   ├── 📁 module2/                        → Workloads & Scheduling  
│   └── 📁 Module3/                        → Networking & Storage
│
└── 📁 kubernetes-challenge/               ← Hands-On Practice Labs
    ├── 📁 Day1/   - Pod creation & management
    ├── 📁 Day2/   - Deployment & ReplicaSets
    ├── 📁 Day3/   - Services & Networking
    ├── 📁 Day4/   - Storage & Volumes
    ├── 📁 Day5/   - RBAC & Security
    ├── 📁 Day6/   - Troubleshooting
    ├── 📁 Day7/   - Advanced Pod Scheduling
    ├── 📁 Day8/   - Resource Management
    ├── 📁 Day9/   - Cluster Administration
    └── 📁 Day10/  - Mock Exam & Review
```

---

## ✨ How to Use This Repository

### **Learning Path (Recommended Order)**

**Phase 1: Foundations** 
1. Read → [`Introduction-to-kubernetes.md`](Introduction-to-kubernetes.md) 
   - Core Kubernetes concepts, architecture, and components
   - Perfect for refreshing fundamentals

2. Read → [`lifecycle-management.md`](./lifecycle-management.md)
   - Cluster installation, upgrades, backups, and restoration
   - Critical for production environments

**Phase 2: Module Learning**
3. Study → [`Modules/`](./Modules/) folder sequentially:
   - [`module1/`](./Modules/module1/) – Cluster Architecture & Setup
   - [`module2/`](./Modules/module2/) – Workloads & Scheduling
   - [`Module3/`](./Modules/Module3/) – Networking & Storage Solutions

**Phase 3: Hands-On Practice**  
4. Practice → [`kubernetes-challenge/`](./kubernetes-challenge/) labs:
   - Start with [`Day1/`](./kubernetes-challenge/Day1/) and progress through Day10
   - Each day focuses on specific CKA exam domains
   - Complete tasks under realistic time constraints

---

## 📁 Detailed Directory Navigation

### **Root Level Files**

| File | Purpose | When to Use |
|------|---------|-------------|
| [**Introduction-to-kubernetes.md**](./Introduction-to-kubernetes.md) | Kubernetes fundamentals & concepts | Before starting modules |
| [**lifecycle-management.md**](./lifecycle-management.md) | Cluster lifecycle, upgrades, backups | During module1 study |

---

### **📚 Modules/ - Core Learning Content**

#### [`Modules/module1/`](./Modules/module1/) – **Cluster Architecture & Setup**
- Cluster initialization and kubeadm workflow
- Control plane components
- Node management and scaling
- **Exam Weight:** 25% (Cluster Installation & Config)

#### [`Modules/module2/`](./Modules/module2/) – **Workloads & Scheduling** 
- Pod creation and lifecycle
- Deployments, StatefulSets, DaemonSets
- Resource requests and limits
- Scheduling and taints/tolerations
- **Exam Weight:** 15% (Workloads & Scheduling)

#### [`Modules/Module3/`](./Modules/Module3/) – **Networking & Storage**
- Services (ClusterIP, NodePort, LoadBalancer)
- Ingress and network policies
- Storage classes, PersistentVolumes, PersistentVolumeClaims
- ConfigMaps and Secrets
- **Exam Weight:** 30% (Services & Storage combined)

---

### **🏋️ kubernetes-challenge/ - Exam-Style Labs**

Practical, hands-on scenarios matching real CKA exam style:

| Day | Topic | Lab Tasks | Domain |
|-----|-------|-----------|--------|
| [**Day1**](./kubernetes-challenge/Day1/) | Pod Fundamentals | Create, edit, delete pods | Workloads (15%) |
| [**Day2**](./kubernetes-challenge/Day2/) | Deployments | Scale, update, rollback deployments | Workloads (15%) |
| [**Day3**](./kubernetes-challenge/Day3/) | Services & Networking | Expose pods, create services | Services (20%) |
| [**Day4**](./kubernetes-challenge/Day4/) | Storage | Create PV, PVC, mount volumes | Storage (10%) |
| [**Day5**](./kubernetes-challenge/Day5/) | Security & RBAC | Create roles, bindings, service accounts | Troubleshooting (15%) |
| [**Day6**](./kubernetes-challenge/Day6/) | Troubleshooting | Debug failing pods and clusters | Troubleshooting (15%) |
| [**Day7**](./kubernetes-challenge/Day7/) | Pod Scheduling | Use taints, tolerations, node selectors | Workloads (15%) |
| [**Day8**](./kubernetes-challenge/Day8/) | Resource Management | Set quotas, limits, requests | Cluster (25%) |
| [**Day9**](./kubernetes-challenge/Day9/) | Cluster Admin | Manage nodes, etcd, backups | Cluster (25%) |
| [**Day10**](./kubernetes-challenge/Day10/) | Mock Exam | Full exam simulation | All Domains |

---

## 🎯 CKA Exam Domains Mapping

| Exam Domain | Weight | Modules | Challenges |
|-------------|--------|---------|------------|
| **Cluster Architecture, Installation & Config** | 25% | [module1](./Modules/module1/) | [Day8-Day9](./kubernetes-challenge/Day8/) |
| **Workloads & Scheduling** | 15% | [module2](./Modules/module2/) | [Day1-Day2](./kubernetes-challenge/Day1/) |
| **Services & Networking** | 20% | [Module3](./Modules/Module3/) | [Day3](./kubernetes-challenge/Day3/) |
| **Storage** | 10% | [Module3](./Modules/Module3/) | [Day4](./kubernetes-challenge/Day4/) |
| **Troubleshooting** | 15% | All modules | [Day6](./kubernetes-challenge/Day6/) |
| **Other (Security, RBAC)** | 15% | All modules | [Day5](./kubernetes-challenge/Day5/) |

---

## 🚀 Getting Started (Quick Setup)

### **Prerequisites**
- Docker or container runtime installed
- 4GB+ RAM recommended for local clusters
- kubectl CLI installed
- Basic Linux command-line knowledge

### **Set Up Your Practice Cluster**

**Option 1: Using Kind (Kubernetes in Docker)**
```bash
# Install kind: https://kind.sigs.k8s.io/
kind create cluster --name cka-practice
kubectl get nodes  # Verify cluster is running
```

**Option 2: Using Minikube**
```bash
minikube start --cpus=4 --memory=4096
kubectl get nodes
```

**Option 3: Using kubeadm (Multi-node)**
- See individual module docs for detailed kubeadm setup instructions

---

## 📖 Tips for Success

✅ **Do's:**
- Follow the learning path in order (fundamentals → modules → labs)
- Practice each lab **multiple times** until muscle memory kicks in
- Time yourself on challenges (CKA exam is time-limited)
- Keep official Kubernetes docs open while practicing: https://kubernetes.io/docs/
- Reset your cluster between labs for clean state
- Take notes on commands you frequently use

❌ **Don'ts:**
- Skip the fundamentals section
- Move to labs before understanding module concepts
- Copy-paste without understanding what commands do
- Cram the night before – spread practice over weeks
- Only read material – hands-on practice is critical

---

## 🤝 Contributing & Feedback

**Found an issue or have suggestions?**

- Open an **Issue** with:
  - Which module/challenge has the problem
  - What's incorrect or unclear
  - Your Kubernetes version
  
- Submit a **Pull Request** with:
  - Clear description of changes
  - Why the change improves the course
  - Test your changes locally first

**Propose New Challenges:**
- Create a new folder in `kubernetes-challenge/`
- Include scenario description and tasks
- Link to relevant exam domain

---

## 📞 Support & Community

- **Questions?** Open a GitHub Issue
- **Want to contribute?** Fork and submit a PR
- **Found value?** Star ⭐ this repo and share with others studying for CKA

---

## 📅 Recommended Study Timeline

**8-Week Study Plan:**
- **Week 1-2:** Introduction + lifecycle-management.md
- **Week 3-4:** module1 (Cluster Architecture)
- **Week 5:** module2 (Workloads) + module3 (Networking/Storage)  
- **Week 6-7:** kubernetes-challenge/ (Day1-Day8)
- **Week 8:** Day9 + Day10 (Mock Exams)

**Adjust based on your pace and experience level!**

---

## ✅ Before Taking the Real CKA Exam

- [ ] Completed all modules  
- [ ] Scored 80%+ on Day10 mock exam
- [ ] Can perform all tasks within time limits
- [ ] Comfortable with kubectl and bash
- [ ] Reviewed troubleshooting patterns
- [ ] Registered for exam at Linux Foundation

---

**Happy Learning! Good luck with your CKA certification! 🚀**

**Last Updated:** December 2025 | **Kubernetes Version:** v1.35

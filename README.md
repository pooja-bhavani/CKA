# Certified Kubernetes Administrator (CKA) – Course Repo (v1.35)

This repository contains hands-on labs, challenges, and notes for the **Certified Kubernetes Administrator (CKA)** exam, aligned to Kubernetes **v1.35**.

## Repository layout

Current important paths in this branch:

- `Modules/` – main course modules and structured theory + guided labs.
- `kubernetes-challenge/` – exam-style challenges and practice questions.
- `Introduction-to-kubernetes.md` – high-level introduction and core concepts.
- `lifecycle-management.md` – cluster lifecycle, upgrades, backup, restore.

---

## How to use this repo

- Start with **Introduction-to-kubernetes.md** to refresh fundamentals.
- Go through each file in `Modules/` sequentially.
- Use `kubernetes-challenge/` to simulate real CKA exam tasks under time pressure.
- Practice everything on a real Kubernetes cluster (kind/minikube/kubeadm on cloud).

---

## 1. Modules index

> Core learning path (theory + guided practice).

Located under `Modules/`.

As of now:

- `Modules/01-cluster-lifecycle.md`  
  - Cluster lifecycle: installation concepts, upgrades, backups, restores, control plane management.

**Other module files in `Modules/` will cover:**
- Workloads & Scheduling
- Services & Networking
- Storage
- Security
- Troubleshooting

Update this section as new module files are added or renamed.

---

## 2. Kubernetes challenges

> Use these for task-focused, exam-like practice.

Folder: `kubernetes-challenge/`

Recommended structure inside this folder (you can gradually align to this):

- `kubernetes-challenge/basics/` – Pods, Deployments, ReplicaSets, Namespaces.
- `kubernetes-challenge/networking/` – Services, Ingress, NetworkPolicies.
- `kubernetes-challenge/storage/` – PV, PVC, StorageClass, volume modes.
- `kubernetes-challenge/security/` – RBAC, ServiceAccounts, Pod security.
- `kubernetes-challenge/troubleshooting/` – broken workloads, logs, events.

Each challenge should be:

- One markdown file describing the scenario and tasks.
- One or more YAML manifests if required for the setup.
- A short "solution" or hints section (optional for learners, useful for instructors).

---

## 3. Mapping to CKA exam domains

Create a file `CKA-v1.35-mapping.md` in the root of the repo and maintain a mapping like this:

| Exam domain                       | Weight | Modules (example)                 | Challenges folder (example)             |
|-----------------------------------|--------|-----------------------------------|-----------------------------------------|
| Cluster Architecture & Scheduling | 15%    | `Modules/01-cluster-lifecycle.md` | `kubernetes-challenge/basics/`          |
| Cluster Installation & Config     | 25%    | `Modules/01-cluster-lifecycle.md` | `kubernetes-challenge/cluster/`         |
| Workloads & Scheduling            | 15%    | `Modules/workloads.md`            | `kubernetes-challenge/workloads/`       |
| Services & Networking             | 20%    | `Modules/networking.md`           | `kubernetes-challenge/networking/`      |
| Storage                           | 10%    | `Modules/storage.md`              | `kubernetes-challenge/storage/`         |
| Troubleshooting                   | 15%    | `Modules/troubleshooting.md`      | `kubernetes-challenge/troubleshooting/` |

Adjust the filenames and folders as you finalize the structure.

---

## 4. Cluster setup (recommended docs)

Add docs (later) and link them here for learners:

- `docs/cluster-setup-kind.md` – local cluster setup with kind.
- `docs/cluster-setup-kubeadm.md` – multi-node cluster with kubeadm on cloud/VMs.

Each doc should include:

- Prerequisites (CPU, RAM, OS).
- Installation steps.
- Commands to create, view, and delete clusters.
- How to reset quickly between labs.

---

## 5. Branching model

Current branch layout for this repo:

- **`feature/CKA`** – active development branch, new modules and changes land here first.
- **`main`** – stable version of the course (once content is reviewed and ready).

Suggested workflow:

1. Develop and edit content on `feature/CKA`.
2. Once a module/challenge is stable, open a PR from `feature/CKA` to `main`.
3. Tag releases (e.g., `v1.35-course-alpha`, `v1.35-course-stable`) as the material matures.

---

## 6. Feedback and improvements

If you are using this material:

- Note any confusing tasks or missing steps.
- Open issues or PRs with suggestions and corrections.
- Propose new challenges that map clearly to CKA v1.35 domains.

Happy Kubernetes hacking and good luck with your **CKA**!

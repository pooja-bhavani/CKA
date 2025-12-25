# Cluster Architecture, Installation & Configuration (25%) 

This module covers 25% of the CKA exam and focuses on understanding, installing, and configuring Kubernetes clusters.

## Learnings
By completing this module, you will be able to:

- Understand Kubernetes cluster architecture and components 
- Install and configure clusters using kind and kubeadm
- Set up highly available (HA) cluster configurations
- Implement Pod Security standards and troubleshoot admission errors
- Configure RBAC (Role-Based Access Control) for secure access
- Work with Custom Resource Definitions (CRDs) and Operators
- Deploy applications using Helm and Kustomize

## Cluster Architecture

### Control Plane Components
The control plane is the brain of the Kubernetes cluster. It maintains the desired state of the cluster and responds to changes.

### Api-Server
Purpose: The API server is the front-end for the Kubernetes control plane and the central management entity.

**Responsibilities:**

Exposes the Kubernetes API (REST interface)
Validates and processes API requests
Serves as the only component that directly communicates with etcd
Handles authentication, authorization, and admission control
Provides the interface for kubectl and other clients
Key Characteristics:

Horizontally scalable (can run multiple instances)
Stateless (all state stored in etcd)
Listens on port 6443 (default)

Example: API Request 
```
kubectl create deployment nginx --image=nginx
# 1. kubectl sends HTTP POST to kube-apiserver
# 2. API server authenticates and authorizes the request
# 3. Admission controllers validate the request
# 4. API server writes to etcd
# 5. API server returns response to kubectl
```

### etcd
Purpose: Distributed, reliable key-value store that serves as Kubernetes' backing store for all cluster data.

**Responsibilities:**

Stores all cluster state and configuration
Maintains consistency across the cluster
Provides watch functionality for detecting changes
Ensures data persistence and reliability

### kube-scheduler
Purpose: Watches for newly created Pods with no assigned node and selects a node for them to run on.

Responsibilities:

Monitors API server for unscheduled Pods
Evaluates node suitability based on multiple factors
Assigns Pods to appropriate nodes
Respects constraints and requirements

### kube-controller-manager
Purpose: Runs controller processes that regulate the state of the cluster.

**Responsibilities:**

Watches cluster state through API server
Makes changes to move current state toward desired state
Runs multiple controllers as separate processes (compiled into single binary

## Worker Node Components
Worker nodes run the actual application workloads. Each worker node contains the components necessary to run Pods and be managed by the control plane.

### kubelet
Purpose: Primary node agent that runs on each worker node and ensures containers are running in Pods.

**Responsibilities:**

Registers node with API server
Watches API server for Pods assigned to its node
Ensures containers described in PodSpecs are running and healthy
Reports node and Pod status back to API server
Executes liveness and readiness probes
Mounts volumes as specified in Pod specs

### kube-proxy
Purpose: Network proxy that runs on each node and maintains network rules for Pod communication.

**Responsibilities:**

Implements Kubernetes Service abstraction
Maintains network rules on nodes
Performs connection forwarding
Enables Service discovery and load balancing

Proxy Modes:
1. iptables mode (default)
2. IPVS mode
3. userspace mode

How kube-proxy Works:
```
Client Pod → Service IP → kube-proxy rules → Backend Pod
```

### Container Runtime
Purpose: Software responsible for running containers on the node.

**Responsibilities:**

Pulls container images from registries
Unpacks and runs containers
Manages container lifecycle
Provides container isolation

Understanding how components interact is crucial for troubleshooting.

### Pod Creation Flow
```
1. User runs: kubectl create -f pod.yaml
   ↓
2. kubectl → API Server (HTTPS)
   ↓
3. API Server validates and writes to etcd
   ↓
4. Scheduler watches API Server, sees unscheduled Pod
   ↓
5. Scheduler selects node, updates Pod binding in API Server
   ↓
6. API Server writes binding to etcd
   ↓
7. kubelet on selected node watches API Server, sees new Pod
   ↓
8. kubelet tells container runtime to pull image and start container
   ↓
9. kubelet reports Pod status back to API Server
   ↓
10. API Server updates Pod status in etcd
```

### CNI (Container Network Interface) Plugin
- Provides Pod networking
- Examples: Calico, Flannel, Weave, Cilium
- Must be installed for Pod-to-Pod communication

---

# Installation for Kubernetes v1.35

### Kubeadm

## Specifically for kubeadm installation

**Step1: Update System**
```
sudo apt update && sudo apt upgrade -y
sudo reboot
```

**Disable Swap (MANDATORY)**
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Check current cgroup version
stat -fc %T /sys/fs/cgroup/

```

**If not cgroup2fs, enable cgroup v2**
```
sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"
# OR for Ubuntu (if grubby not available)
sudo sed -i 's/GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX="systemd.unified_cgroup_hierarchy=1"/' /etc/default/grub
sudo update-grub
sudo reboot

# Verify after reboot
stat -fc %T /sys/fs/cgroup/  # Should show: cgroup2fs
```

**Load Kernel Modules**
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

**Set sysctl parameters**

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

```

**step2: Install Container Runtime**

```
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y containerd.io

sudo systemctl restart containerd
sudo systemctl enable containerd
systemctl status containerd
```

> Note: If we skip these, kubeadm will fail, nodes stay NotReady, or networking (Pods <-> Pods) breaks.

**Step 3: Install Kubernetes v1.35 Components**

```
# Add GPG Key & Repo
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

# Install exact versions
sudo apt-get install -y kubelet=1.35.0-1.1 kubeadm=1.35.0-1.1 kubectl=1.35.0-1.1

# Hold packages
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl enable kubelet
```

**Verify Installation**
```
kubeadm version
kubelet --version
kubectl version --client

```

**Initialize Control Plane Node**
```
sudo kubeadm init \
  --kubernetes-version=v1.35.0 \
  --pod-network-cidr=10.244.0.0/16
```

**Configure kubectl for your user**
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install a CNI Plugin
```
Option 1: Flannel (simplest, CKA-friendly)
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
or

```
Option 2: Calico (more features)
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```
---

### Kind Installation 

**Step 1: Install Docker**
```
sudo apt update
sudo apt install -y docker.io

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

**step2:Install kind v1.35**
```
curl -Lo kind https://kind.sigs.k8s.io/dl/v0.24.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/
kind version

```

**Step 3: Install kubectl for v1.35**
```
# Download kubectl v1.35
curl -LO "https://dl.k8s.io/release/v1.35.0/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl

# Verify kubectl
kubectl version --client
```
--- 

### Check Cluster Components

```bash
# Check nodes
kubectl get nodes -o wide

# Check system pods
kubectl get pods -A

# Check cluster info
kubectl cluster-info
```
## Kubernetes v1.34 support advanced features 
- Provides **Dynamic Resource Allocation** for GPUs, TPUs, NICs, etc
- **Delayed Job Pod Replacement** -  This policy only creates replacement pods when the original pod is completely terminated
- **Security Tokens** - kubelet can use short-lived, audience-bound ServiceAccount tokens that are automatically rotated
- **Pod-Level Resources** - enable containers to share CPU and memory from a common pod allocation
- **Job Success Policy** - allows jobs to succeed when a subset of pods complete successfully

---

## Cluster Lifecycle Management (kubeadm)

Initialize a Single Control Plane
```
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=<CONTROL_PLANE_IP>
```
**Set up kubectl**
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Install a CNI (e.g. Calico, Flannel, Cilium) so Pods can communicate
#### Weave Net
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
## Join Worker Nodes
```
sudo kubeadm join LOAD_BALANCER_DNS:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --control-plane \
  --certificate-key <cert-key>
```
---

## HA Configuration

**Overview**

A High Availability (HA) Kubernetes cluster eliminates single points of failure by running multiple control plane nodes. This ensures the cluster remains operational even if one or more control plane nodes fail.

### Components

```
                    ┌─────────────────┐
                    │  Load Balancer  │
                    │   (HAProxy/     │
                    │    nginx)       │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────▼─────┐      ┌─────▼─────┐     ┌─────▼─────┐
    │ Control   │      │ Control   │     │ Control   │
    │ Plane 1   │      │ Plane 2   │     │ Plane 3   │
    │           │      │           │     │           │
    │ + etcd    │◄────►│ + etcd    │◄───►│ + etcd    │
    └───────────┘      └───────────┘     └───────────┘
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────▼─────┐      ┌─────▼─────┐     ┌─────▼─────┐
    │  Worker   │      │  Worker   │     │  Worker   │
    │  Node 1   │      │  Node 2   │     │  Node 3   │
    └───────────┘      └───────────┘     └───────────┘
```
### Key Concepts

**Stacked etcd Topology** (Recommended for most cases):
- etcd runs on the same nodes as control plane components
- Simpler to set up and manage
- Requires fewer nodes (minimum 3)
- If a control plane node fails, both control plane and etcd member are lost

**External etcd Topology**:
- etcd runs on separate dedicated nodes
- More resilient (control plane and etcd failures are independent)
- Requires more nodes (3 for etcd + 2+ for control plane)
- More complex to set up and manage

### Infrastructure Requirements

**Minimum for HA:**
- 3 control plane nodes (odd number recommended: 3, 5, 7)
- 3+ worker nodes
- 1 load balancer (can be external or software-based)

**Per Control Plane Node:**
- 2 CPUs (4 recommended)
- 4GB RAM (8GB recommended)
- 50GB disk space
- Network connectivity between all nodes

**Load Balancer:**
- Can be hardware (F5, Citrix) or software (HAProxy, nginx)
- Must support TCP load balancing
- Health checks for API server
  

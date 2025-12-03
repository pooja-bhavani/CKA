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

kube-apiserver
Purpose: The API server is the front-end for the Kubernetes control plane and the central management entity.

Responsibilities:

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

Responsibilities:

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

Responsibilities:

Watches cluster state through API server
Makes changes to move current state toward desired state
Runs multiple controllers as separate processes (compiled into single binary

## Worker Node Components
Worker nodes run the actual application workloads. Each worker node contains the components necessary to run Pods and be managed by the control plane.

### kubelet
Purpose: Primary node agent that runs on each worker node and ensures containers are running in Pods.

Responsibilities:

Registers node with API server
Watches API server for Pods assigned to its node
Ensures containers described in PodSpecs are running and healthy
Reports node and Pod status back to API server
Executes liveness and readiness probes
Mounts volumes as specified in Pod specs

### kube-proxy
Purpose: Network proxy that runs on each node and maintains network rules for Pod communication.

Responsibilities:

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

Responsibilities:

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

### Installation for K8s v1.34

**kubectl:**
```bash
curl -LO "https://dl.k8s.io/release/v1.34.0/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

**kind:**
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Then create v1.34 cluster:
kind create cluster --image kindest/node:v1.34.0
```
**Helm:**
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```







# Cluster Lifecycle Management 

## Overview

Cluster lifecycle management involves maintaining, upgrading, backing up, and restoring Kubernetes clusters. These operations are critical for production environments.

## Table of Contents

1. [Cluster Upgrades](#cluster-upgrades)
2. [etcd Backup and Restore](#etcd-backup-and-restore)
3. [Node Maintenance](#node-maintenance)
4. [Certificate Management](#certificate-management)
5. [Troubleshooting](#troubleshooting)

## Cluster Upgrades

### Upgrade Strategy for v1.35

**Version Skew Policy (Updated for v1.35):**
- Control plane components can be at most one minor version apart
- kubelet can be up to two minor versions behind API server
- kubectl can be ±1 minor version from API server

**Upgrade Order:**
1. Control plane nodes (one at a time if HA)
2. Worker nodes (can be done in batches)

### Pre-Upgrade Checklist

```bash
# 1. Check current versions
kubectl version
kubectl get nodes

# 2. Run v1.35 compatibility check
./v135-pre-upgrade-check.sh

# 3. Review v1.35 release notes
# https://kubernetes.io/releases/notes/

# 4. Backup etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot-pre-v135-$(date +%Y%m%d-%H%M%S).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# 5. Check cluster health
kubectl get nodes
kubectl get pods -A
kubectl get --raw /healthz

# 6. Document current feature gates
kubectl get pods -n kube-system kube-apiserver-$(hostname) -o yaml | grep feature-gates

# 7. Drain control plane node (if HA)
kubectl drain <control-plane-node> --ignore-daemonsets
```

### Upgrade Control Plane (Ubuntu/Debian)

#### Step 1: Upgrade kubeadm

```bash
# Find available v1.35 versions
apt-cache madison kubeadm | grep 1.35

# Upgrade to v1.35.0
sudo apt-mark unhold kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm=1.35.0-1.1
sudo apt-mark hold kubeadm

# Verify version
kubeadm version
```

#### Step 2: Plan the Upgrade

```bash
# Check what will be upgraded to v1.35
sudo kubeadm upgrade plan v1.35.0

# Output shows:
# - Current version (v1.34.x)
# - Target version (v1.35.0)
# - Component versions
# - New feature gates available
# - Breaking changes warnings
```

#### Step 3: Apply the Upgrade

```bash
# For first control plane node
sudo kubeadm upgrade apply v1.35.0 --feature-gates="GenericWorkload=true,UserNamespacesSupport=true"

# For additional control plane nodes (if HA)
sudo kubeadm upgrade node
```

**Expected Output:**
```
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.35.0". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

#### Step 4: Upgrade kubelet and kubectl

```bash
# Ubuntu/Debian
sudo apt-mark unhold kubelet kubectl
sudo apt-get update
sudo apt-get install -y kubelet=1.35.0-1.1 kubectl=1.35.0-1.1
sudo apt-mark hold kubelet kubectl

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Verify versions
kubectl version
kubelet --version
```

#### Step 5: Uncordon the Node

```bash
kubectl uncordon <control-plane-node>

# Verify node is Ready
kubectl get nodes
```

#### Test v1.35 in-place resource updates
```bash
kubectl run test-v135 --image=nginx --requests='cpu=100m,memory=128Mi'
kubectl patch pod test-v135 --type='merge' -p='{"spec":{"containers":[{"name":"test-v135","resources":{"limits":{"cpu":"200m","memory":"256Mi"}}}]}}'
```

### Upgrade Worker Nodes

Repeat for each worker node:

#### Step 1: Drain the Node

```bash
# From control plane
kubectl drain <worker-node> --ignore-daemonsets --delete-emptydir-data

# Verify pods are evicted
kubectl get pods -o wide | grep <worker-node>
```

#### Step 2: Upgrade kubeadm (on worker node)

```bash
# SSH to worker node
ssh <worker-node>

# Upgrade kubeadm
sudo apt-mark unhold kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm=1.35.0-1.1
sudo apt-mark hold kubeadm
```

#### Step 3: Upgrade Node Configuration

```bash
# On worker node
sudo kubeadm upgrade node
```

#### Step 5: Uncordon the Node

```bash
# From control plane
kubectl uncordon <worker-node>

# Verify
kubectl get nodes
```
---

## etcd Backup and Restore

### Understanding etcd

etcd stores all cluster data:
- All Kubernetes objects (Pods, Services, Deployments, etc.)
- Cluster configuration
- Secrets and ConfigMaps
- Resource definitions

**Critical**: Regular backups are essential for disaster recovery.

### etcd Backup

#### Method 1: Using etcdctl (Recommended)

```bash
#!/bin/bash
# v135-etcd-backup.sh - Enhanced backup script for v1.35

BACKUP_DIR="/backup/etcd"
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/etcd-snapshot-v135-${TIMESTAMP}.db"

# Create backup directory
mkdir -p ${BACKUP_DIR}

# Create snapshot with v1.35 metadata
ETCDCTL_API=3 etcdctl snapshot save ${BACKUP_FILE} \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify snapshot
ETCDCTL_API=3 etcdctl snapshot status ${BACKUP_FILE} --write-out=table

# Create metadata file for v1.35
cat > ${BACKUP_FILE}.metadata << EOF
Kubernetes Version: $(kubectl version --short --client | grep Client)
Server Version: $(kubectl version --short | grep Server)
Backup Date: $(date)
Cluster Name: $(kubectl config current-context)
Node Count: $(kubectl get nodes --no-headers | wc -l)
v1.35 Features: In-place updates, Generation tracking, Gang scheduling
EOF

# Backup v1.35 specific configurations
kubectl get configmaps -n kube-system -o yaml > ${BACKUP_DIR}/configmaps-v135-${TIMESTAMP}.yaml
kubectl get secrets -n kube-system -o yaml > ${BACKUP_DIR}/secrets-v135-${TIMESTAMP}.yaml

# Keep only last 7 days of backups
find ${BACKUP_DIR} -name "etcd-snapshot-v135-*.db" -mtime +7 -delete
find ${BACKUP_DIR} -name "*.metadata" -mtime +7 -delete
find ${BACKUP_DIR} -name "*-v135-*.yaml" -mtime +7 -delete

echo "v1.35 backup completed: ${BACKUP_FILE}"
```

### etcd Restore for v1.35

```bash
#!/bin/bash
# v135-etcd-restore.sh

BACKUP_FILE="/backup/etcd/etcd-snapshot-v135-20241225-120000.db"
NODE_NAME=$(hostname)
NODE_IP=$(hostname -I | awk '{print $1}')

echo "⚠️ WARNING: This will restore cluster to backup state"
echo "Backup file: $BACKUP_FILE"
echo "Press Enter to continue or Ctrl+C to abort"
read

# Step 1: Stop API server and etcd
echo "Stopping API server and etcd..."
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
sudo mv /etc/kubernetes/manifests/etcd.yaml /tmp/

# Wait for pods to stop
sleep 30

# Step 2: Backup current etcd data
sudo mv /var/lib/etcd /var/lib/etcd-backup-$(date +%Y%m%d-%H%M%S)

# Step 3: Restore from snapshot
echo "Restoring from snapshot..."
ETCDCTL_API=3 etcdctl snapshot restore $BACKUP_FILE \
  --data-dir=/var/lib/etcd \
  --name=$NODE_NAME \
  --initial-cluster=$NODE_NAME=https://$NODE_IP:2380 \
  --initial-advertise-peer-urls=https://$NODE_IP:2380

# Step 4: Fix ownership
sudo chown -R etcd:etcd /var/lib/etcd

# Step 5: Restart etcd and API server
echo "Restarting etcd and API server..."
sudo mv /tmp/etcd.yaml /etc/kubernetes/manifests/
sleep 20
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

# Step 6: Wait for cluster to be ready
echo "Waiting for cluster to be ready..."
sleep 60

# Step 7: Verify restore
kubectl get nodes
kubectl get pods -A

echo "✅ Restore completed. Verify your v1.35 cluster is working correctly."
```

# On worker node
```bash
sudo apt-mark unhold kubelet kubectl
sudo apt-get update
sudo apt-get install -y kubelet=1.35.0-1.1 kubectl=1.35.0-1.1
sudo apt-mark hold kubelet kubectl

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

## Node Maintenance

### Draining Nodes

Enhanced Node Draining:

```bash
# v1.35 enhanced drain with new options
kubectl drain <node-name> \
  --ignore-daemonsets \
  --delete-emptydir-data \
  --force \
  --grace-period=300 \
  --timeout=600s \
  --skip-wait-for-delete-timeout=60s  # New in v1.35

# Check for pods using v1.35 features
kubectl get pods -o wide | grep <node-name>
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{": Gen="}{.metadata.generation}{", Observed="}{.status.observedGeneration}{"\n"}{end}' | grep -v "Gen=, Observed="
```

---

## Certificate Management

```bash
# Check certificate expiration (includes v1.35 components)
sudo kubeadm certs check-expiration

# Renew all certificates for v1.35
sudo kubeadm certs renew all

# Restart control plane components for v1.35
sudo systemctl restart kubelet

# For static pods, move and restore manifests
sudo mv /etc/kubernetes/manifests/*.yaml /tmp/
sleep 10
sudo mv /tmp/*.yaml /etc/kubernetes/manifests/

# Verify certificates are renewed
sudo kubeadm certs check-expiration

# Update kubeconfig with new certificates
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Test cluster access
kubectl get nodes
```
---

## Troubleshooting

### Upgrade Issues

**Issue**: Upgrade fails with cgroup v1 error
```bash
# Error: "cgroup v1 is not supported in v1.35"
# Solution: Enable cgroup v2 on all nodes
for node in $(kubectl get nodes -o jsonpath='{.items[*].metadata.name}'); do
  kubectl debug node/$node -it --image=busybox -- chroot /host grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"
done
# Reboot all nodes
```

**Issue**: containerd compatibility error
```bash
# Error: "container runtime version not supported"
# Check containerd version on all nodes
for node in $(kubectl get nodes -o jsonpath='{.items[*].metadata.name}'); do
  echo "Node: $node"
  kubectl debug node/$node -it --image=busybox -- chroot /host containerd --version
done

# Upgrade containerd to 1.7+ on affected nodes
kubectl debug node/<node> -it --image=busybox -- chroot /host apt-get install containerd.io
```
**Issue**: v1.35 features not working
```bash
# Check feature gates
kubectl get pods -n kube-system kube-apiserver-$(hostname) -o yaml | grep feature-gates

# Verify API resources
kubectl api-resources | grep -E "(podgroups|storageversionmigrations)"

# Test in-place updates
kubectl run test --image=nginx --requests='cpu=100m'
kubectl patch pod test --type='merge' -p='{"spec":{"containers":[{"name":"test","resources":{"limits":{"cpu":"200m"}}}]}}'
kubectl get pod test -o jsonpath='{.metadata.generation}'
```

**Issue**: Generation tracking not working
```bash
# Check kubelet version
kubectl get nodes -o wide

# Verify Pod has generation fields
kubectl get pod <pod-name> -o yaml | grep -E "(generation|observedGeneration)"

# Check kubelet logs
sudo journalctl -u kubelet -f | grep generation
```

### Migration Issues from v1.34

**Issue**: Services using deprecated PreferClose
```bash
# Find services using old syntax
kubectl get services --all-namespaces -o yaml | grep "trafficDistribution: PreferClose"

# Update to v1.35 syntax
kubectl patch service <service-name> --type='merge' -p='{"spec":{"trafficDistribution":"PreferSameZone"}}'
```

**Issue**: ipvs mode warnings in v1.35
# Check current kube-proxy mode
```bash
kubectl get configmap kube-proxy -n kube-system -o jsonpath='{.data.config\.conf}' | grep mode

# Migrate to nftables mode (recommended for v1.35)
kubectl patch configmap kube-proxy -n kube-system --type merge -p='{
  "data": {
    "config.conf": "mode: nftables\nnftables:\n  masqueradeAll: false\n"
  }
}'

kubectl rollout restart daemonset kube-proxy -n kube-system
```

## Best Practices for v1.35

1. **Pre-Upgrade Validation**:
   - Always run v1.35 compatibility check
   - Verify cgroup v2 on all nodes
   - Check containerd version compatibility
   - Test upgrade in non-production first

2. **Backup Strategy**:
   - Create comprehensive backups before v1.35 upgrade
   - Include v1.35 metadata in backup files
   - Test restore procedures with v1.35 features

3. **Feature Adoption**:
   - Enable v1.35 feature gates gradually
   - Test new features in development first
   - Monitor cluster performance after enabling features

4. **Monitoring**:
   - Monitor generation tracking for update issues
   - Set up alerts for deprecated feature usage
   - Track v1.35 feature adoption metrics

## Exam Tips

1. **Know the Commands**: Memorize upgrade and backup commands
2. **Practice Speed**: Upgrades take time - practice for efficiency
3. **Certificate Locations**: Know where certificates are stored
4. **Backup Verification**: Always verify backups after creation
5. **Drain vs Cordon**: Understand the difference
6. **Version Skew**: Understand Kubernetes version compatibility

## References

- [Kubernetes v1.35 Release Notes](https://kubernetes.io/releases/notes/)
- [Upgrading kubeadm clusters to v1.35](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [v1.35 Feature Gates](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/)
- [cgroup v2 Migration Guide](https://kubernetes.io/docs/concepts/architecture/cgroups/)
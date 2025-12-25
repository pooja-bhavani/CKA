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
sudo kubeadm upgrade apply v1.35.0 --feature-gates="GangScheduling=true,UserNamespacesSupport=true"

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
```
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

**Output:**
```
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 12345678 |    12345 |       1234 |     5.0 MB |
+----------+----------+------------+------------+
```

#### Method 2: Automated Backup Script

```bash
#!/bin/bash
# /usr/local/bin/backup-etcd.sh

BACKUP_DIR="/backup/etcd"
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/etcd-snapshot-${TIMESTAMP}.db"

# Create backup directory
mkdir -p ${BACKUP_DIR}

# Create snapshot
ETCDCTL_API=3 etcdctl snapshot save ${BACKUP_FILE} \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify snapshot
ETCDCTL_API=3 etcdctl snapshot status ${BACKUP_FILE}

# Keep only last 7 days of backups
find ${BACKUP_DIR} -name "etcd-snapshot-*.db" -mtime +7 -delete

echo "Backup completed: ${BACKUP_FILE}"
```

**Schedule with cron:**
```bash
# Edit crontab
sudo crontab -e

# Add daily backup at 2 AM
0 2 * * * /usr/local/bin/backup-etcd.sh >> /var/log/etcd-backup.log 2>&1
```

### etcd Restore

**⚠️ Warning**: Restoring etcd will overwrite all cluster data. Only do this in disaster recovery scenarios.

#### Step 1: Stop API Server and etcd

```bash
# Move manifests to stop static pods
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
sudo mv /etc/kubernetes/manifests/etcd.yaml /tmp/

# Wait for pods to stop
docker ps | grep -E 'kube-apiserver|etcd'
```

#### Step 2: Restore from Snapshot

```bash
# Restore snapshot to new directory
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore \
  --name=<node-name> \
  --initial-cluster=<node-name>=https://<node-ip>:2380 \
  --initial-advertise-peer-urls=https://<node-ip>:2380

# Example:
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore \
  --name=control-plane-1 \
  --initial-cluster=control-plane-1=https://10.0.0.10:2380 \
  --initial-advertise-peer-urls=https://10.0.0.10:2380
```

#### Step 3: Update etcd Configuration

```bash
# Edit etcd manifest
sudo vi /tmp/etcd.yaml

# Update data directory path:
# Change: --data-dir=/var/lib/etcd
# To:     --data-dir=/var/lib/etcd-restore

# Or use sed
sudo sed -i 's|/var/lib/etcd|/var/lib/etcd-restore|g' /tmp/etcd.yaml
```

#### Step 4: Restart etcd and API Server

```bash
# Move manifests back
sudo mv /tmp/etcd.yaml /etc/kubernetes/manifests/
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

# Wait for pods to start
watch kubectl get pods -n kube-system
```

#### Step 5: Verify Restore

```bash
# Check cluster status
kubectl get nodes
kubectl get pods -A

# Verify your data is restored
kubectl get deployments -A
kubectl get services -A
```

---

## Node Maintenance

### Draining Nodes

Safely evict pods before maintenance:

```bash
# Drain node (evict all pods)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Options:
# --ignore-daemonsets: Ignore DaemonSet-managed pods
# --delete-emptydir-data: Delete pods using emptyDir volumes
# --force: Force deletion of pods not managed by controllers
# --grace-period=<seconds>: Grace period for pod termination
# --timeout=<duration>: Timeout for drain operation

# Check pods are evicted
kubectl get pods -o wide | grep <node-name>
```

### Cordoning Nodes

Mark node as unschedulable without evicting pods:

```bash
# Cordon node (mark unschedulable)
kubectl cordon <node-name>

# Verify
kubectl get nodes
# Node will show SchedulingDisabled

# Uncordon when ready
kubectl uncordon <node-name>
```

### Node Removal

```bash
# 1. Drain the node
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# 2. Delete the node from cluster
kubectl delete node <node-name>

# 3. On the node itself, reset kubeadm
sudo kubeadm reset

# 4. Clean up
sudo rm -rf /etc/cni/net.d
sudo rm -rf $HOME/.kube/config
```

### Adding Nodes Back

```bash
# Generate new join command on control plane
kubeadm token create --print-join-command

# On the node, run the join command
sudo kubeadm join <control-plane-ip>:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>

# Verify from control plane
kubectl get nodes
```

---

## Certificate Management

### Check Certificate Expiration

```bash
# Check all certificates
sudo kubeadm certs check-expiration

# Output shows expiration dates for:
# - admin.conf
# - apiserver
# - apiserver-etcd-client
# - apiserver-kubelet-client
# - controller-manager.conf
# - etcd-healthcheck-client
# - etcd-peer
# - etcd-server
# - front-proxy-client
# - scheduler.conf
```

### Renew Certificates

```bash
# Renew all certificates
sudo kubeadm certs renew all

# Renew specific certificate
sudo kubeadm certs renew apiserver

# Restart control plane components
sudo systemctl restart kubelet

# For static pods, move and restore manifests
sudo mv /etc/kubernetes/manifests/*.yaml /tmp/
sleep 10
sudo mv /tmp/*.yaml /etc/kubernetes/manifests/
```

### Manual Certificate Renewal

```bash
# Backup current certificates
sudo cp -r /etc/kubernetes/pki /etc/kubernetes/pki.backup

# Renew certificates
sudo kubeadm certs renew all

# Update kubeconfig
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verify
kubectl get nodes
```

---

## Troubleshooting

### Upgrade Issues

**Issue**: kubeadm upgrade fails

```bash
# Check kubeadm version
kubeadm version

# Check for version skew
kubectl version

# Review upgrade plan
sudo kubeadm upgrade plan

# Check logs
sudo journalctl -u kubelet -f
```

**Issue**: Pods not starting after upgrade

```bash
# Check pod status
kubectl get pods -A
kubectl describe pod <pod-name> -n <namespace>

# Check node status
kubectl get nodes
kubectl describe node <node-name>

# Check kubelet
sudo systemctl status kubelet
sudo journalctl -u kubelet -f
```

### Backup/Restore Issues

**Issue**: etcdctl command not found

```bash
# Install etcdctl
ETCD_VER=v3.5.9
wget https://github.com/etcd-io/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzf etcd-${ETCD_VER}-linux-amd64.tar.gz
sudo mv etcd-${ETCD_VER}-linux-amd64/etcdctl /usr/local/bin/
```

**Issue**: Backup fails with certificate errors

```bash
# Verify certificate paths
ls -la /etc/kubernetes/pki/etcd/

# Use correct paths in etcdctl command
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

**Issue**: Restore fails

```bash
# Check snapshot integrity
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db

# Ensure cluster is stopped
sudo mv /etc/kubernetes/manifests/*.yaml /tmp/

# Verify no etcd process running
ps aux | grep etcd

# Try restore again with correct parameters
```

## Best Practices

1. **Regular Backups**:
   - Automate daily etcd backups
   - Store backups off-cluster
   - Test restore procedures regularly

2. **Upgrade Strategy**:
   - Test upgrades in non-production first
   - Upgrade one minor version at a time
   - Keep detailed upgrade logs

3. **Maintenance Windows**:
   - Schedule maintenance during low-traffic periods
   - Communicate with stakeholders
   - Have rollback plan ready

4. **Monitoring**:
   - Monitor cluster health before/after operations
   - Set up alerts for certificate expiration
   - Track upgrade progress

5. **Documentation**:
   - Document cluster configuration
   - Keep runbooks for common operations
   - Record all changes

## Exam Tips

1. **Know the Commands**: Memorize upgrade and backup commands
2. **Practice Speed**: Upgrades take time - practice for efficiency
3. **Certificate Locations**: Know where certificates are stored
4. **Backup Verification**: Always verify backups after creation
5. **Drain vs Cordon**: Understand the difference
6. **Version Skew**: Understand Kubernetes version compatibility

## References

- [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Operating etcd clusters](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
- [Certificate Management](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)
- [Safely Drain a Node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)

---

[← Back to Installation](README.md) | [Next: HA Configuration →](04-ha-installation.md)

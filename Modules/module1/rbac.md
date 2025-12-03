# RBAC (Role‑Based Access Control)

### Let's understand Role, RoleBinding, ClusterRole, ClusterRoleBinding, ServiceAccount creation/config/troubleshooting

## Overview
RBAC (Role‑Based Access Control) is the authorization mechanism in K8s. That allows you to control who can perform what actions on which resources.

**Core Components:**
- **Role**: Defines permissions within a namespace
- **ClusterRole**: Defines permissions cluster-wide or resuable set of permissions
- **RoleBinding**: Grants Role permissions to subjects in a namespace
- **ClusterRoleBinding**: Grants ClusterRole permissions to subjects cluster-wide
- **ServiceAccount**: Provides identity for processes running in Pods

### ServiceAccount creation and use
```
# Create namespace and ServiceAccount
kubectl create namespace dev
kubectl create serviceaccount app-sa -n dev
```
```
# Inspect
kubectl get sa -n dev
kubectl describe sa app-sa -n dev
```

Use this in a Pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
spec:
  serviceAccountName: app-sa
  containers:
  - name: nginx
    image: nginx
```

### Examles

### Role and RoleBinding (namespace‑scoped)
Motive: Allow app-sa to list/get/watch Pods only in dev namespace.

**Roles**
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```
**RoleBinding**
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: dev
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
```
```
kubectl apply -f <filename.yaml>
kubectl apply -f <filename.yaml>
```









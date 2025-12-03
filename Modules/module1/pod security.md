# Pod Security Standards and Admission Control

## Overview

Kubernetes uses Pod Security Admission (PSA) to enforce the built‑in Pod Security Standards (PSS) at namespace level to enforce security best practices. 
PSA evaluates Pod create/update requests at admission time and can enforce, warn, or audit policy violations before Pods ever run.

Pod Security Standards (PSS) levels
The three built‑in profiles:

1. Privileged (Unrestricted)
Purpose: No restrictions - allows known privilege escalations

- Allows privileged Pods, host networking, hostPath volumes, Running as root and all other capabilities.

2. Baseline

Purpose: Prevents known privilege escalations while minimizing restrictions

- Minimally restrictive prevents known privilege escalations while allowing most default Pod specs.
- Disallows privileged containers, some host namespaces, and unsafe capabilities.​

3. Restricted
Purpose: Follows pod hardening best practices

- Most secure, based on current Pod hardening best practices.
- Security-critical applications
- Enforces non‑root, seccomp, and limited capabilities and restricts host access patterns.​

## Pod Security Admission

### Admission Modes

Pod Security Admission operates in three modes per namespace:

#### 1. enforce
- **Behavior**: Rejects pods that violate the policy
- **Use**: Production namespaces
- **Effect**: Pod creation fails

#### 2. audit
- **Behavior**: Allows pods but logs violations
- **Use**: Monitoring and gradual rollout
- **Effect**: Pod created, event logged

#### 3. warn
- **Behavior**: Allows pods but shows warning to user
- **Use**: Development and testing
- **Effect**: Pod created, warning displayed

### Namespace Labels

Configure Pod Security using namespace labels:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    # Enforce restricted standard
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: v1.34
    
    # Audit baseline standard
    pod-security.kubernetes.io/audit: baseline
    pod-security.kubernetes.io/audit-version: v1.34
    
    # Warn on privileged violations
    pod-security.kubernetes.io/warn: baseline
    pod-security.kubernetes.io/warn-version: v1.34
```

Namespaces are labeled to select profile + mode, for example:
```
kubectl label namespace team-a \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=baseline \
  pod-security.kubernetes.io/warn=baseline
```

## Implementing Pod Security

### Step 1: Check Current Configuration

```bash
# Check if Pod Security Admission is enabled
kubectl api-resources | grep podsecurity

# Check existing namespace labels
kubectl get namespaces --show-labels

# Check specific namespace
kubectl get namespace default -o yaml
```

## Admission Controllers

### What are Admission Controllers?

Admission controllers are plugins that intercept requests to the Kubernetes API server before object persistence. They can:
- **Validate**: Check if request meets requirements
- **Mutate**: Modify the request
- **Reject**: Deny the request

### Common Admission Controllers

#### 1. PodSecurity (Validating)
- Enforces Pod Security Standards

#### 2. NamespaceLifecycle (Validating)
- Prevents creation of objects in terminating namespaces
- Ensures system namespaces cannot be deleted

#### 3. LimitRanger (Validating)
- Enforces resource limits on pods and containers
- Applies default limits if not specified

#### 4. ResourceQuota (Validating)
- Enforces resource quotas per namespace
- Prevents resource exhaustion

#### 5. ServiceAccount (Mutating)
- Automatically adds ServiceAccount to pods
- Mounts ServiceAccount token

#### 6. DefaultStorageClass (Mutating)
- Adds default StorageClass to PVCs
- Only if no StorageClass specified

#### 7. MutatingAdmissionWebhook (Mutating)
- Calls external webhooks to mutate objects
- Used by service meshes, policy engines

#### 8. ValidatingAdmissionWebhook (Validating)
- Calls external webhooks to validate objects
- Used for custom policies

### Checking Enabled Admission Controllers
```
# Check enabled admission controllers
kubectl exec -n kube-system kube-apiserver-<node> -- kube-apiserver -h | grep enable-admission-plugins
```

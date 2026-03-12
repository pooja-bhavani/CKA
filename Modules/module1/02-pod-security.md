# Pod Security Standards and Admission Control

## Overview

Kubernetes uses Pod Security Admission (PSA) to enforce the built‑in Pod Security Standards (PSS) at namespace level to enforce security best practices. 
PSA evaluates Pod create/update requests at admission time and can enforce, warn, or audit policy violations before Pods ever run.

### Key Concepts

Pod Security Standards (PSS) levels
The three built‑in profiles:

**1. Privileged (Unrestricted)**
Purpose: No restrictions - allows known privilege escalations

- Allows privileged Pods, host networking, hostPath volumes, Running as root and all other capabilities.

**2. Baseline**

Purpose: Prevents known privilege escalations while minimizing restrictions

- Minimally restrictive prevents known privilege escalations while allowing most default Pod specs.
- Disallows privileged containers, some host namespaces, and unsafe capabilities.​

**3. Restricted**

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
  name: production-namespace
  labels:
    # Enforce restricted standard
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: v1.35
    
    # Audit baseline violations
    pod-security.kubernetes.io/audit: baseline
    pod-security.kubernetes.io/audit-version: v1.35
    
    # Warn on baseline violations
    pod-security.kubernetes.io/warn: baseline
    pod-security.kubernetes.io/warn-version: v1.35
```
**Namespace-Pod-security**              
[namespace-pod-security.yaml](../../k8s/security/01-namespaces-pod-security.yaml)

```
kubectl apply -f 01-namespaces-pod-security.yaml
kubectl get ns --show-labels
```
<img width="1318" height="242" alt="image" src="https://github.com/user-attachments/assets/7f26c46d-1c47-428b-9b0d-eaba30f43127" />

**What this does:**
* bankapp-dev is relaxed – good for experimentation.
* bankapp-staging enforces baseline and monitors for restricted.
* bankapp-prod fully enforces restricted:v1.35, so only hardened Pods are admitted.

### Pod Security – Bad vs Good Pod
put yaml link
<img width="1007" height="173" alt="image" src="https://github.com/user-attachments/assets/66d3bcdf-dab3-4912-b647-40cac182d3f5" />


* violates PodSecurity "restricted:v1.35": privileged containers, hostPath volumes are not allowed                   
The bankapp-prod namespace, which enforces restricted:v1.35. When I try to run this privileged Pod with a hostPath to /var/log, the API server rejects it at admission time. It tells me why: the container is privileged, it uses hostPath, it can escalate privileges, it has all capabilities, and it doesn’t set runAsNonRoot. None of these Pods will ever start on my nodes.



## v1.35 Security Enhancements

### 1. User Namespaces (Beta)

**What's New**: Enhanced container isolation using Linux user namespaces.

**Benefits**:
- Root inside container maps to unprivileged user on host
- Better isolation without sacrificing functionality
- Reduces attack surface

**Configuration**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: user-namespace-pod
spec:
  securityContext:
    runAsUser: 0      # Root inside container
  hostUsers: false    # NEW in v1.35: Enable user namespace isolation
  containers:
  - name: app
    image: nginx
    securityContext:
      runAsUser: 0    # Maps to unprivileged user on host
```
---

### 2. Pod Certificates 

**What's New**: Native certificate generation and rotation without external tools.

**Benefits**:
- No need for cert-manager or SPIFFE/SPIRE
- Automatic certificate rotation
- Built-in workload identity

**Configuration**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-certs
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: workload-certs
      mountPath: /var/run/secrets/workload-identity
      readOnly: true
  volumes:
  - name: workload-certs
    projected:
      sources:
      - podCertificate:  # NEW in v1.35
          commonName: "app.default.svc.cluster.local"
          duration: "24h"
          renewBefore: "8h"
          dnsNames:
          - "app.default.svc.cluster.local"
          - "app"
```
---


## Admission Controllers

### What are Admission Controllers?


Admission controllers are plugins that intercept requests to the Kubernetes API server before object persistence. They operate in two phases:

1. **Mutating Phase**: Can modify the request
2. **Validating Phase**: Can accept or reject the request

### Common Admission Controllers in v1.35

#### 1. PodSecurity (Validating)
- **Purpose**: Enforces Pod Security Standards
- **New in v1.35**: Enhanced user namespace support
- **Configuration**: Namespace labels

#### 2. NamespaceLifecycle (Validating)
- **Purpose**: Prevents operations on terminating namespaces
- **Behavior**: Ensures system namespaces cannot be deleted

#### 3. LimitRanger (Validating)
- **Purpose**: Enforces resource limits and defaults

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

#### 9. CertificateApproval (NEW in v1.35)
- **Purpose**: Manages Pod certificate requests
- **Behavior**: Approves/denies certificate requests

### Checking Enabled Admission Controllers
```
# Check enabled admission controllers
kubectl exec -n kube-system kube-apiserver-<node> -- kube-apiserver -h | grep enable-admission-plugins
```
---

## Troubleshooting Admission Errors

### Common Error Patterns

#### Example 1 – HostPath blocked by restricted policy

Error:
```
Error from server (Forbidden): error when creating "pod.yaml": 
pods "nginx" is forbidden: violates PodSecurity "restricted:latest": 
hostPath volumes are not allowed
```
Cause: restricted profile disallows hostPath because it can expose the node filesystem.

Problem Pod:
```yaml
spec:
  volumes:
  - name: host-logs
    hostPath:
      path: /var/log
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: host-logs
      mountPath: /logs
```

**Solution:**
```yaml
spec:
  volumes:
  - name: app-logs
    persistentVolumeClaim:
      claimName: app-logs-pvc
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: app-logs
      mountPath: /logs
```

**Debugging Steps:**

```bash
# Check namespace Pod Security labels
kubectl get namespace <namespace> -o yaml | grep pod-security

# Try with warn mode first
kubectl label namespace <namespace> \
  pod-security.kubernetes.io/enforce=privileged \
  pod-security.kubernetes.io/warn=restricted \
  --overwrite

# Create pod and see warnings
kubectl apply -f pod.yaml
```

#### Example 2 : ResourceQuota Exceeded

**Error Message:**
```
Error from server (Forbidden): pods "my-pod" is forbidden: 
exceeded quota: compute-quota, requested: requests.cpu=2, used: requests.cpu=8, limited: requests.cpu=10
```

**Solution:**
```bash
# Check quota
kubectl get resourcequota -n <namespace>
kubectl describe resourcequota compute-quota -n <namespace>

# Reduce pod resources or increase quota
kubectl edit resourcequota compute-quota -n <namespace>
```
---

#### Example 3: LimitRange Violation

**Error Message:**
```
Error from server (Forbidden): pods "my-pod" is forbidden: 
maximum cpu usage per Container is 2, but limit is 4
```

**Solution:**
```bash
# Check LimitRange
kubectl get limitrange -n <namespace>
kubectl describe limitrange <limitrange-name> -n <namespace>

# Adjust pod resources
spec:
  containers:
  - name: nginx
    resources:
      limits:
        cpu: "2"
        memory: "2Gi"
```
### Debugging Workflow

```bash
# 1. Check admission error details
kubectl apply -f pod.yaml --dry-run=server -o yaml

# 2. Check namespace Pod Security labels
kubectl get namespace <namespace> -o yaml

# 3. Check events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'

# 4. Check API server logs
kubectl logs -n kube-system kube-apiserver-<node>

# 5. Test with different security levels
kubectl label namespace <namespace> \
  pod-security.kubernetes.io/enforce=privileged \
  --overwrite

# 6. Gradually increase restrictions
kubectl label namespace <namespace> \
  pod-security.kubernetes.io/enforce=baseline \
  --overwrite
```

#### Example 4: Running as Root Blocked (In v1.35)
**Error Message:**
```
Error from server (Forbidden): pods "app" is forbidden: violates PodSecurity "restricted:latest": runAsNonRoot != true
```

**Solution:**
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  hostUsers: false  # NEW in v1.35: Enable user namespace isolation
  containers:
  - name: app
    image: nginx:1.21
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
      runAsNonRoot: true
      runAsUser: 1000
```

**What are Pod Certificates?**
Pod Certificates is a beta feature in v1.35 that allows Kubernetes to automatically generate and manage TLS certificates for individual Pods without requiring external tools like cert-manager or SPIFFE/SPIRE.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-certs
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: workload-certs
      mountPath: /var/run/secrets/workload-identity
      readOnly: true
  volumes:
  - name: workload-certs
    projected:
      sources:
      - podCertificate:  # NEW in v1.35
          commonName: "app.default.svc.cluster.local"
          duration: "24h"
          renewBefore: "8h"
          dnsNames:
          - "app.default.svc.cluster.local"
          - "app"
```


#### Pod Certificate Issues (NEW in v1.35)
**Error:** "podCertificate volume source not supported"

**Diagnosis:**
```bash
# Check if certificate APIs are available:
kubectl api-resources | grep certificates

# Check feature gates on API server
kubectl get pods -n kube-system kube-apiserver-$(hostname) -o yaml | grep feature-gates

```

**Solution:**
```bash
# Enable Pod certificates feature gate
sudo kubeadm upgrade apply v1.35.0 --feature-gates="PodCertificates=true"
```

- This enables the Pod Certificates feature gate during cluster upgrade
- After this, Pods can use podCertificate volume sources

**Real-World Use Cases:**
1. Service Mesh: Pods get automatic mTLS certificates
2. Microservices: Secure service-to-service communication
3. API Authentication: Pods authenticate to external APIs using certificates
4. Compliance: Meet requirements for certificate-based workload identity

**Benefits Over External Tools:**
1. Simpler: No need to install cert-manager or SPIFFE
2. Native: Built into Kubernetes core
3. Automatic: Handles certificate lifecycle automatically
4. Secure: Certificates are Pod-specific and short-lived
---

## Exam Tips

1. **Know the Three Standards**: Privileged, Baseline, Restricted
2. **Understand Modes**: enforce, audit, warn
3. **Label Format**: `pod-security.kubernetes.io/<mode>: <level>`
4. **Common Fixes**: runAsNonRoot, drop capabilities, seccomp profile
5. **Debugging**: Use `--dry-run=server` to test
6. **Quick Fix**: Temporarily set to privileged, then fix and restore
7. **Check Events**: `kubectl get events` shows admission errors

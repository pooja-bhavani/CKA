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

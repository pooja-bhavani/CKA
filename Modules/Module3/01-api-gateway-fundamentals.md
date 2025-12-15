# Gateway API Fundamentals

## Overview

Gateway API is the next-generation Kubernetes API for managing ingress traffic. It's a more expressive, extensible, and role-oriented successor to the Ingress API. Gateway API provides a standardized way to configure load balancing, traffic routing, and service mesh capabilities.

## Why Gateway API Matters

- **Role-Oriented**: Separates concerns between infrastructure providers, cluster operators, and application developers
- **Expressive**: Supports advanced routing (header-based, weighted, mirroring)
- **Extensible**: Custom resources and vendor-specific features
- **Portable**: Works across different implementations (Envoy, Istio, NGINX, etc.)
- **Type-Safe**: Strongly typed API with better validation

## Importance

Gateway API is increasingly important for modern Kubernetes:
- **Emerging topic** in CKA exam (5-10% weight expected)
- Replaces traditional Ingress in many scenarios
- Understanding migration from Ingress to Gateway API is critical
- Must know core objects: Gateway, HTTPRoute, GatewayClass

---

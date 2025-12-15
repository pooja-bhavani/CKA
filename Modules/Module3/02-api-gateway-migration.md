# Migrating from Ingress to Gateway API

## Overview

This guide covers the migration process from traditional Kubernetes Ingress to Gateway API. You'll learn how to convert existing Ingress resources to Gateway API equivalents (HTTPRoute, Gateway objects) and understand the benefits of the migration.

## Why Migrate?

**Benefits of Gateway API over Ingress**:
- **More expressive**: Advanced routing (headers, weights, mirroring)
- **Role-oriented**: Clear separation of responsibilities
- **Type-safe**: Better validation and error messages
- **Extensible**: Custom resources without annotations
- **Portable**: Standardized across implementations
- **Future-proof**: Active development and community support

## Why Kubernetes is Evolving Beyond Ingress: The Rise of the Gateway API

***Firstly let's understand what is ingress what are it's limitations***

### What is Ingress in Kubernetes?
It provides a centralized way to manage external access to services running inside a Kubernetes cluster, typically HTTP/HTTPS traffic at the L7 layer. It provides centralizedmechanism to manage all inbound requests to backend applications.
Instead of individual load balancers per service.

 

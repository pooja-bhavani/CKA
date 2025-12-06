# Lab: Basic RBAC with ServiceAccount

## Objective

Give a ServiceAccount permission to list Pods only in the `dev` namespace, and verify it with `kubectl auth can-i`.

## Steps

### 1. Create namespace and ServiceAccount

```
kubectl apply -f sa-app.yaml
kubectl get ns
kubectl get sa -n dev
```

### 2. Check permissions BEFORE RBAC

```
kubectl auth can-i list pods -n dev \
  --as=system:serviceaccount:dev:app-sa
```

You should see: `no`.

### 3. Create Role and RoleBinding

```
kubectl apply -f role-pod-reader.yaml
kubectl apply -f rolebinding-pod-reader.yaml
```

### 4. Check permissions AFTER RBAC

```
kubectl auth can-i list pods -n dev \
  --as=system:serviceaccount:dev:app-sa
```

Now you should see: `yes`.

### 5. (Optional) Try to list Pods

```
kubectl get pods -n dev \
  --as=system:serviceaccount:dev:app-sa
```

If there are no Pods, you'll see "No resources found", but not a Forbidden error.

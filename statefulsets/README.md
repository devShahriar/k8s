# StatefulSets

## Overview

A **StatefulSet** manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods. Unlike Deployments, StatefulSets maintain a sticky identity for each of their Pods.

## Key Features

- **Stable Network Identity**: Each pod gets a stable hostname based on its ordinal index
- **Stable Storage**: Each pod gets persistent storage that persists across rescheduling
- **Ordered Deployment**: Pods are created and terminated in a predictable order
- **Ordered Scaling**: Pods are scaled up/down in order

## How It Works

1. Pods are created sequentially (0, 1, 2, ...)
2. Each pod gets a unique, stable network identity: `<statefulset-name>-<ordinal>`
3. Each pod can have its own PersistentVolumeClaim
4. Pods are terminated in reverse order
5. StatefulSets require a Headless Service for network identity

## Common Use Cases

- Databases (MySQL, PostgreSQL, MongoDB)
- Distributed systems requiring stable identities
- Applications needing stable storage per pod
- Applications requiring ordered deployment/scaling

## Examples

### Basic StatefulSet
- `basic-statefulset.yaml` - Simple StatefulSet with persistent storage

### StatefulSet with Headless Service
- `statefulset-with-service.yaml` - Complete example with required service

## Commands

```bash
# Create StatefulSet
kubectl apply -f basic-statefulset.yaml

# View StatefulSets
kubectl get statefulsets

# View pods (notice the ordered naming)
kubectl get pods -l app=web

# Scale StatefulSet
kubectl scale statefulset/web --replicas=5

# View persistent volume claims
kubectl get pvc

# Delete StatefulSet (pods deleted in reverse order)
kubectl delete statefulset web
```

## Important Fields

- **serviceName**: Name of the headless service used for stable network identity
- **volumeClaimTemplates**: Template for creating PersistentVolumeClaims
- **podManagementPolicy**: Parallel or Ordered (default)
- **updateStrategy**: RollingUpdate or OnDelete

## Differences from Deployments

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| Pod Identity | Random | Stable (ordinal-based) |
| Storage | Shared | Individual per pod |
| Scaling | Parallel | Ordered |
| Use Case | Stateless | Stateful |


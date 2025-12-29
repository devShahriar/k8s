# Affinity and Anti-Affinity

## Overview

**Affinity and Anti-Affinity** provide advanced pod scheduling capabilities beyond simple node selectors. They allow you to express complex scheduling rules including soft preferences, pod-to-pod relationships, and zone distribution.

## Key Concepts

### Node Affinity
- Similar to node selectors but more powerful
- Supports soft preferences (preferredDuringScheduling)
- Supports hard requirements (requiredDuringScheduling)
- Supports operators: In, NotIn, Exists, DoesNotExist, Gt, Lt

### Pod Affinity
- Schedule pods based on other pods' labels
- **Affinity**: Prefer to run with certain pods
- **Anti-Affinity**: Prefer to avoid certain pods

### Pod Anti-Affinity
- Spread pods across nodes/zones
- Prevent pods from co-locating
- Useful for high availability

## How It Works

1. Define affinity rules in pod spec
2. Scheduler evaluates rules during scheduling
3. Hard requirements (required) must be met
4. Soft preferences (preferred) are weighted and considered
5. Pods are placed according to best match

## Common Use Cases

- **High Availability**: Spread pods across nodes/zones
- **Co-location**: Run related pods on same node
- **Isolation**: Prevent pods from running together
- **Zone Distribution**: Distribute pods across availability zones
- **Resource Optimization**: Place pods near their data

## Examples

### Node Affinity
- `node-affinity-required.yaml` - Hard node affinity requirement
- `node-affinity-preferred.yaml` - Soft node affinity preference

### Pod Anti-Affinity
- `pod-anti-affinity.yaml` - Prevent pods from co-locating
- `deployment-anti-affinity.yaml` - Deployment with anti-affinity

### Pod Affinity
- `pod-affinity.yaml` - Co-locate pods together

## Commands

```bash
# Apply deployment with affinity
kubectl apply -f deployment-anti-affinity.yaml

# View pod distribution
kubectl get pods -o wide

# Check which nodes pods are on
kubectl get pods -l app=nginx -o wide

# View node labels (for zone/node selection)
kubectl get nodes --show-labels
```

## Affinity Types

### RequiredDuringSchedulingIgnoredDuringExecution
- **Hard requirement**: Must be satisfied for pod to be scheduled
- If not satisfied, pod remains Pending
- Similar to node selector but more flexible

### PreferredDuringSchedulingIgnoredDuringExecution
- **Soft preference**: Scheduler tries to satisfy but not required
- Has a weight (1-100) for prioritization
- Pod can still be scheduled if not satisfied

## Operators

- **In**: Value is in the list
- **NotIn**: Value is not in the list
- **Exists**: Label key exists
- **DoesNotExist**: Label key does not exist
- **Gt**: Greater than (for numeric values)
- **Lt**: Less than (for numeric values)

## Topology Keys

Common topology keys:
- `kubernetes.io/hostname` - Node level
- `topology.kubernetes.io/zone` - Availability zone
- `topology.kubernetes.io/region` - Region
- Custom labels

## Examples Explained

### Node Affinity (Required)
```yaml
requiredDuringSchedulingIgnoredDuringExecution:
  nodeSelectorTerms:
  - matchExpressions:
    - key: disktype
      operator: In
      values:
      - ssd
```

### Node Affinity (Preferred)
```yaml
preferredDuringSchedulingIgnoredDuringExecution:
- weight: 100
  preference:
    matchExpressions:
    - key: zone
      operator: In
      values:
      - us-west-2a
```

### Pod Anti-Affinity
```yaml
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
  - labelSelector:
      matchExpressions:
      - key: app
        operator: In
        values:
        - nginx
    topologyKey: kubernetes.io/hostname
```

## Differences from Node Selectors

| Feature | Node Selector | Affinity |
|---------|---------------|----------|
| Hard Requirements | ✅ | ✅ |
| Soft Preferences | ❌ | ✅ |
| Operators | Exact match only | In, NotIn, Exists, etc. |
| Pod Relationships | ❌ | ✅ |
| Zone Distribution | ❌ | ✅ |
| Weighting | ❌ | ✅ |

## Best Practices

1. **High Availability**: Use pod anti-affinity to spread pods across nodes
2. **Zone Distribution**: Use topology keys to distribute across zones
3. **Soft First**: Prefer soft rules (preferred) when possible for flexibility
4. **Combine Rules**: Use both node and pod affinity for complex scenarios
5. **Test Thoroughly**: Affinity rules can prevent scheduling if too restrictive

## Common Patterns

### Spread Pods Across Nodes
```yaml
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
  - topologyKey: kubernetes.io/hostname
```

### Spread Pods Across Zones
```yaml
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
  - topologyKey: topology.kubernetes.io/zone
```

### Co-locate Related Pods
```yaml
podAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
  - topologyKey: kubernetes.io/hostname
```


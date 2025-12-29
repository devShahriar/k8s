# Node Selectors

## Overview

A **Node Selector** is a simple way to constrain pods to run on specific nodes. It's a field in the pod specification that specifies key-value pairs that must match labels on the target node.

## Key Features

- **Simple Scheduling**: Easy way to target specific nodes
- **Label-Based**: Uses node labels for selection
- **Hard Constraint**: Pods will not be scheduled if no matching node exists
- **Single Value**: Can only specify one value per key

## How It Works

1. Label nodes with key-value pairs
2. Specify nodeSelector in pod spec with matching labels
3. Scheduler only places pods on nodes with matching labels
4. If no node matches, pod remains in Pending state

## Common Use Cases

- Run pods on nodes with specific hardware (GPU, SSD)
- Separate workloads by environment (dev, staging, prod)
- Route pods to specific availability zones
- Use nodes with specific software installed

## Examples

### Basic Node Selector
- `pod-node-selector.yaml` - Pod that runs only on nodes with specific label

### Deployment with Node Selector
- `deployment-node-selector.yaml` - Deployment using node selector

## Commands

```bash
# Label a node
kubectl label nodes <node-name> disktype=ssd

# View node labels
kubectl get nodes --show-labels

# View specific node labels
kubectl describe node <node-name>

# Remove a label
kubectl label nodes <node-name> disktype-

# Apply pod with node selector
kubectl apply -f pod-node-selector.yaml

# Check pod scheduling
kubectl get pods -o wide
```

## Important Fields

- **nodeSelector**: Map of key-value pairs that must match node labels
- **nodeName**: Directly specify node name (bypasses scheduler)

## Limitations

- **Simple Matching**: Only supports exact label matching
- **AND Logic**: All labels must match (no OR logic)
- **No Soft Constraints**: Cannot express preferences, only hard requirements
- **Single Values**: Cannot specify multiple values for the same key

## When to Use Node Selectors

✅ **Use Node Selectors when:**
- You have simple, hard requirements
- You need to target specific node types
- You want a straightforward solution

❌ **Use Affinity/Anti-Affinity when:**
- You need soft preferences
- You need complex scheduling rules
- You need pod-to-pod relationships

## Example Node Labels

Common node labels you might use:
- `node-role.kubernetes.io/master` - Master node
- `kubernetes.io/arch` - Architecture (amd64, arm64)
- `kubernetes.io/os` - Operating system (linux, windows)
- `node.kubernetes.io/instance-type` - Instance type (cloud providers)
- Custom labels: `environment=production`, `disktype=ssd`, `gpu=true`


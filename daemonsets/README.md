# DaemonSets

## Overview

A **DaemonSet** ensures that a copy of a Pod runs on all (or some) nodes in the cluster. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected.

## Key Features

- **One Pod Per Node**: Automatically runs one pod on each node
- **Node Addition**: Automatically creates pods when new nodes join
- **Node Removal**: Automatically deletes pods when nodes leave
- **Selective Deployment**: Can target specific nodes using node selectors

## How It Works

1. DaemonSet controller watches for nodes
2. When a node is added, it creates a pod on that node
3. When a node is removed, it deletes the pod
4. Pods are managed directly by DaemonSet (not through ReplicaSet)
5. Each pod has a unique identity tied to its node

## Common Use Cases

- **Log Collection**: Run log collection agents (Fluentd, Filebeat) on every node
- **Monitoring**: Run monitoring agents (Prometheus node exporter) on every node
- **Network Plugins**: Run network components (CNI plugins) on every node
- **Storage**: Run storage daemons (GlusterFS, Ceph) on every node
- **Security**: Run security scanning tools on every node

## Examples

### Basic DaemonSet
- `basic-daemonset.yaml` - Simple DaemonSet running on all nodes

### DaemonSet with Node Selector
- `daemonset-node-selector.yaml` - DaemonSet running only on specific nodes

## Commands

```bash
# Create DaemonSet
kubectl apply -f basic-daemonset.yaml

# View DaemonSets
kubectl get daemonsets

# View pods created by DaemonSet
kubectl get pods -l app=fluentd-logging

# View pods across nodes
kubectl get pods -l app=fluentd-logging -o wide

# Update DaemonSet
kubectl set image daemonset/fluentd-logging fluentd=fluentd:v2.0

# Delete DaemonSet
kubectl delete daemonset fluentd-logging
```

## Important Fields

- **selector**: Labels to identify pods managed by this DaemonSet
- **template**: Pod template used to create pods on nodes
- **updateStrategy**: RollingUpdate or OnDelete
- **nodeSelector**: Select specific nodes to run pods on

## Differences from Deployments

| Feature | Deployment | DaemonSet |
|---------|-----------|-----------|
| Pod Count | User-defined replicas | One per node |
| Use Case | Application workloads | System-level services |
| Scaling | Manual scaling | Automatic (based on nodes) |
| Node Targeting | Can use node selectors | Always runs on nodes |

## Update Strategies

- **RollingUpdate** (default): Updates pods one at a time
- **OnDelete**: Updates pods only when manually deleted


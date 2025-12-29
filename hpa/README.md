# Horizontal Pod Autoscaler (HPA)

## Overview

The **Horizontal Pod Autoscaler (HPA)** automatically scales the number of pods in a deployment, replica set, or stateful set based on observed CPU utilization, memory usage, or custom metrics.

## Key Features

- **Automatic Scaling**: Scales pods up or down based on metrics
- **CPU/Memory Based**: Uses resource utilization metrics
- **Custom Metrics**: Supports custom application metrics
- **Multiple Metrics**: Can scale based on multiple metrics simultaneously
- **Target Utilization**: Maintains target resource utilization

## How It Works

1. HPA controller queries metrics from Metrics Server or custom metrics API
2. Calculates desired replica count based on current metrics and target
3. Updates the deployment/replicaset to scale pods
4. Continuously monitors and adjusts as needed
5. Respects min and max replica limits

## Scaling Formula

```
desiredReplicas = ceil[currentReplicas * (currentMetricValue / desiredMetricValue)]
```

Example:
- Current replicas: 3
- Current CPU: 90%
- Target CPU: 50%
- Desired: ceil[3 * (90/50)] = ceil[5.4] = 6 replicas

## Common Use Cases

- Web applications with variable traffic
- API services with fluctuating load
- Batch processing jobs
- Applications with predictable scaling patterns

## Prerequisites

- Metrics Server must be installed in the cluster
- For custom metrics: Custom Metrics API adapter
- Deployment/ReplicaSet/StatefulSet with resource requests defined

## Examples

### Basic HPA (CPU)
- `hpa-cpu.yaml` - HPA scaling based on CPU utilization

### HPA with CPU and Memory
- `hpa-cpu-memory.yaml` - HPA using both CPU and memory metrics

### HPA with Min/Max Replicas
- `hpa-min-max.yaml` - HPA with explicit min and max replica limits

## Commands

```bash
# Create HPA
kubectl apply -f hpa-cpu.yaml

# View HPAs
kubectl get hpa

# View HPA details
kubectl describe hpa <hpa-name>

# View HPA metrics
kubectl get hpa <hpa-name> -o yaml

# Delete HPA
kubectl delete hpa <hpa-name>

# Manually scale (HPA will adjust)
kubectl scale deployment nginx-deployment --replicas=5

# Check metrics server
kubectl top nodes
kubectl top pods
```

## Important Fields

- **scaleTargetRef**: Reference to the deployment/replicaset to scale
- **minReplicas**: Minimum number of replicas (default: 1)
- **maxReplicas**: Maximum number of replicas (required)
- **metrics**: List of metrics to use for scaling
- **targetCPUUtilizationPercentage**: Target CPU utilization (deprecated, use metrics)
- **targetMemoryUtilizationPercentage**: Target memory utilization (deprecated, use metrics)

## Metric Types

### Resource Metrics
- **CPU**: `resource.cpu`
- **Memory**: `resource.memory`

### Custom Metrics
- Application-specific metrics
- Requires custom metrics API adapter

### External Metrics
- Metrics from external systems
- Requires external metrics API adapter

## HPA Behavior

### Scale Up
- Happens quickly (default: 15 seconds)
- Can scale by multiple pods at once
- Respects maxReplicas limit

### Scale Down
- Happens slowly (default: 5 minutes)
- Prevents flapping (rapid up/down)
- Respects minReplicas limit

### Cooldown Periods
- **scaleUpPeriod**: How quickly to scale up (default: 0s)
- **scaleDownPeriod**: How quickly to scale down (default: 5m)

## Best Practices

1. **Set Resource Requests**: HPA needs resource requests to calculate utilization
2. **Set Reasonable Limits**: Define min and max replicas appropriately
3. **Monitor HPA**: Watch HPA behavior and adjust targets
4. **Test Scaling**: Test how your application handles scaling
5. **Consider Custom Metrics**: Use application metrics for better scaling decisions
6. **Avoid Over-Scaling**: Set appropriate max replicas to control costs

## Troubleshooting

### HPA Shows `<unknown>` for metrics
- Check if Metrics Server is installed: `kubectl get deployment metrics-server -n kube-system`
- Verify pods have resource requests defined
- Check HPA events: `kubectl describe hpa <hpa-name>`

### HPA Not Scaling
- Check if min/max replicas allow scaling
- Verify metrics are available
- Check HPA status: `kubectl get hpa <hpa-name> -o yaml`

### HPA Scaling Too Aggressively
- Increase scaleDownPeriod
- Adjust target utilization percentage
- Review application resource requests

## Example Scenarios

### High Traffic Website
- Target: 70% CPU
- Min: 2 replicas
- Max: 20 replicas

### API Service
- Target: 50% CPU, 60% memory
- Min: 3 replicas
- Max: 10 replicas

### Background Jobs
- Target: 80% CPU
- Min: 1 replica
- Max: 5 replicas


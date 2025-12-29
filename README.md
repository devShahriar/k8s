# Kubernetes Practice Repository

Welcome to the Kubernetes practice repository! This repository contains hands-on examples and detailed explanations for various Kubernetes concepts.

## 📚 Topics Covered

This repository covers the following Kubernetes topics with practical examples:

1. **[Deployments](./deployments/README.md)** - Manage stateless applications with rolling updates
2. **[StatefulSets](./statefulsets/README.md)** - Manage stateful applications with stable network identities
3. **[Services](./services/README.md)** - Expose applications running in pods
4. **[DaemonSets](./daemonsets/README.md)** - Run a pod on every node in the cluster
5. **[ConfigMaps](./configmaps/README.md)** - Store non-sensitive configuration data
6. **[Secrets](./secrets/README.md)** - Store sensitive data like passwords and tokens
7. **[Node Selectors](./node-selectors/README.md)** - Schedule pods on specific nodes
8. **[Taints and Tolerations](./taints-tolerations/README.md)** - Control pod scheduling with node taints
9. **[Affinity and Anti-Affinity](./affinity-anti-affinity/README.md)** - Advanced pod scheduling rules
10. **[Horizontal Pod Autoscaler (HPA)](./hpa/README.md)** - Automatically scale pods based on metrics

## 🚀 Getting Started

### Prerequisites

- A Kubernetes cluster (minikube, kind, or cloud-based)
- `kubectl` configured to connect to your cluster

### How to Use This Repository

1. Navigate to the topic you want to learn about
2. Read the README.md file in that directory for detailed explanations
3. Review the YAML examples provided
4. Apply the examples to your cluster using `kubectl apply -f <yaml-file>`
5. Experiment and modify the examples to understand the concepts better

## 📖 Example Usage

```bash
# Apply a deployment
kubectl apply -f deployments/basic-deployment.yaml

# Check the status
kubectl get deployments

# View detailed information
kubectl describe deployment <deployment-name>
```

## 🎯 Learning Path

We recommend following this order for beginners:

1. **Deployments** - Start here to understand basic pod management
2. **Services** - Learn how to expose your applications
3. **ConfigMaps & Secrets** - Understand configuration management
4. **Node Selectors** - Learn basic scheduling
5. **Taints & Tolerations** - Advanced scheduling concepts
6. **Affinity & Anti-Affinity** - Fine-grained scheduling control
7. **StatefulSets** - For stateful applications
8. **DaemonSets** - For system-level pods
9. **HPA** - Automatic scaling

## 📝 Notes

- All examples are designed to work in a standard Kubernetes cluster
- Some examples may require specific cluster configurations (e.g., multiple nodes for node selectors)
- Always review YAML files before applying them to production environments

## 🤝 Contributing

Feel free to improve examples, add more use cases, or fix any issues you find!

## 📄 License

This repository is for educational purposes.


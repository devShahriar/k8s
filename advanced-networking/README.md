# Advanced Networking - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- Network Policies for pod-to-pod communication control
- DNS configuration and service discovery
- Endpoints and EndpointSlices
- Headless Services
- Network plugins (CNI)
- Advanced Service configurations

## 🎯 Network Policies

**Network Policies** are Kubernetes resources that control traffic flow between pods. They act as a firewall for your pods.

### Real-World Analogy

Think of Network Policies as security guards:
- **Pods** = Buildings
- **Network Policy** = Security guard with rules
- **Allowed traffic** = People with proper credentials
- **Blocked traffic** = People without credentials

### The Problem They Solve

**Without Network Policies:**
- All pods can communicate with all pods (default)
- No network segmentation
- Security risk if one pod is compromised

**With Network Policies:**
- Control which pods can communicate
- Network segmentation
- Defense in depth

## 📝 Tutorial: Your First Network Policy

### Step 1: Create a Network Policy

```bash
# Apply network policy
kubectl apply -f network-policy-basic.yaml

# View network policies
kubectl get networkpolicies

# Check details
kubectl describe networkpolicy deny-all
```

### Step 2: Test Network Isolation

```bash
# Create test pods
kubectl run test-pod --image=busybox --rm -it --restart=Never -- sh

# Try to access another pod
wget -qO- http://<pod-ip>
```

## 🔑 Key Concepts

### Network Policy Selectors
- **podSelector**: Selects pods this policy applies to
- **namespaceSelector**: Selects namespaces
- **ipBlock**: Selects IP ranges

### Policy Types
- **Ingress**: Incoming traffic rules
- **Egress**: Outgoing traffic rules

### Default Behavior
- **No policy**: All traffic allowed
- **Policy exists**: Only explicitly allowed traffic

## 📚 Example Files Explained

### Basic Network Policy (`network-policy-basic.yaml`)
- Denies all traffic by default
- Allows only specified traffic

### Allow Specific Pods (`network-policy-allow-pods.yaml`)
- Allows communication between specific pods

### Namespace Isolation (`network-policy-namespace.yaml`)
- Isolates namespaces from each other

## 🎓 Common Use Cases

### Use Case 1: Database Isolation
Only allow specific pods to access database.

### Use Case 2: Frontend-Backend Separation
Frontend can talk to backend, but not directly to database.

### Use Case 3: Multi-Tenant Isolation
Different tenants can't access each other's pods.

## 💡 Best Practices

1. Start with deny-all, then allow specific traffic
2. Use labels for pod selection
3. Test policies in non-production first
4. Document why each rule exists


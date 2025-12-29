# Taints and Tolerations - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What taints and tolerations are and why they exist
- How to apply taints to nodes
- How to add tolerations to pods
- The three different taint effects
- Real-world use cases and best practices

## 🎯 What are Taints and Tolerations?

**Taints and Tolerations** are Kubernetes' way of saying "this node doesn't want most pods, but some special pods are welcome."

### Real-World Analogy

Think of it like a VIP section in a club:
- **Taint** = "VIP Only" sign on the door (applied to the node)
- **Toleration** = VIP pass (applied to the pod)
- Regular pods = Regular customers (can't enter)
- Pods with toleration = VIP customers (can enter)

### Why Do We Need This?

**Problem**: You have special nodes (like master nodes or GPU nodes) that should only run specific workloads.

**Solution**: Taint the node so only pods with matching tolerations can run there.

## 🔑 Key Concepts

### Taints
- Applied to **nodes** (like a "keep out" sign)
- Prevents most pods from being scheduled
- Has three effects (severity levels)

### Tolerations
- Applied to **pods** (like a "VIP pass")
- Allows pods to ignore node taints
- Must match the taint's key, value, and effect

## 🏗️ How It Works (Step by Step)

### Step 1: Admin Taints a Node

```bash
kubectl taint nodes worker-node-1 dedicated=gpu:NoSchedule
```

**What happens:**
- Node `worker-node-1` is now "tainted"
- It has a taint: `dedicated=gpu:NoSchedule`
- Most pods will be rejected from this node

### Step 2: Scheduler Tries to Place a Pod

When a pod needs to be scheduled:
1. Scheduler looks at all nodes
2. Finds `worker-node-1` has a taint
3. Checks if the pod has a matching toleration
4. **If NO toleration**: Pod is rejected ❌
5. **If YES toleration**: Pod can be scheduled ✅

### Step 3: Pod with Toleration

A pod with this toleration can run on the tainted node:

```yaml
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "gpu"
  effect: "NoSchedule"
```

## 📝 Tutorial: Hands-On Practice

### Step 1: Check Your Cluster

First, let's see what nodes you have:

```bash
# List all nodes
kubectl get nodes

# View node details and existing taints
kubectl describe node <node-name>
```

**Note**: Master nodes usually have taints by default:
```
Taints: node-role.kubernetes.io/master:NoSchedule
```

### Step 2: Apply a Taint to a Node

Let's taint a worker node (replace with your node name):

```bash
# Taint a node for GPU workloads
kubectl taint nodes worker-node-1 dedicated=gpu:NoSchedule

# Verify the taint
kubectl describe node worker-node-1 | grep Taints
# Should show: Taints: dedicated=gpu:NoSchedule
```

**What this means:**
- This node is now reserved for GPU workloads
- Regular pods cannot be scheduled here
- Only pods with matching toleration can run

### Step 3: Try to Schedule a Regular Pod

```bash
# Create a regular pod (no toleration)
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: regular-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
EOF

# Check pod status
kubectl get pod regular-pod

# You'll see: Pending (can't be scheduled)
kubectl describe pod regular-pod | grep -A 5 Events
# Will show: "0/1 nodes are available: 1 node(s) had taint..."
```

### Step 4: Create a Pod with Toleration

Now let's create a pod that can tolerate the taint:

```bash
kubectl apply -f pod-toleration.yaml

# Check pod status
kubectl get pod nginx-tolerated

# Should show: Running (scheduled on tainted node)
kubectl get pod nginx-tolerated -o wide
# Shows which node it's running on
```

### Step 5: Remove the Taint

When you're done:

```bash
# Remove the taint (note the minus sign at the end)
kubectl taint nodes worker-node-1 dedicated=gpu:NoSchedule-

# Verify removal
kubectl describe node worker-node-1 | grep Taints
# Should show: Taints: <none>
```

## 🎚️ Understanding Taint Effects

There are three levels of "strictness":

### 1. NoSchedule (Strict - No New Pods)

```bash
kubectl taint nodes node-1 dedicated=special:NoSchedule
```

**What it does:**
- ❌ New pods **cannot** be scheduled (without toleration)
- ✅ Existing pods **are NOT affected**
- **Use case**: Reserve node for specific workloads

**Example scenario:**
- You have a GPU node
- You want only ML workloads to use it
- Taint it with `NoSchedule`
- Only ML pods with toleration can run there

### 2. PreferNoSchedule (Soft - Try to Avoid)

```bash
kubectl taint nodes node-1 dedicated=special:PreferNoSchedule
```

**What it does:**
- ⚠️ Scheduler **tries to avoid** scheduling pods here
- ✅ But it's **not required** - pods can still be scheduled if needed
- ✅ Existing pods **are NOT affected**
- **Use case**: Soft preference, not a hard rule

**Example scenario:**
- You prefer certain nodes for certain workloads
- But if cluster is full, it's okay to use them
- Less strict than `NoSchedule`

### 3. NoExecute (Very Strict - Evict Existing Pods)

```bash
kubectl taint nodes node-1 maintenance=true:NoExecute
```

**What it does:**
- ❌ New pods **cannot** be scheduled (without toleration)
- ❌ Existing pods **ARE EVICTED** (without toleration)
- **Use case**: Node maintenance, draining nodes

**Example scenario:**
- Node needs maintenance
- Taint with `NoExecute`
- All regular pods are evicted
- Only maintenance pods (with toleration) can stay

## 📚 Example Files Explained

### Pod with Toleration (`pod-toleration.yaml`)

This pod can run on tainted nodes because it has tolerations:

```yaml
tolerations:
# Tolerate master node taint (common pattern)
- key: "node-role.kubernetes.io/master"
  operator: "Exists"        # Key exists (value doesn't matter)
  effect: "NoSchedule"

# Tolerate custom taint
- key: "dedicated"
  operator: "Equal"          # Key and value must match exactly
  value: "special-workload"
  effect: "NoSchedule"
```

**Key points:**
- **operator: "Exists"**: Only checks if key exists (ignores value)
- **operator: "Equal"**: Key AND value must match exactly
- **effect**: Must match the taint's effect

### Deployment with Toleration (`deployment-toleration.yaml`)

Same concept, but for a Deployment (all pods will have the toleration).

## 🔍 Understanding Operators

### Operator: "Equal"

```yaml
tolerations:
- key: "dedicated"
  operator: "Equal"      # Key AND value must match
  value: "gpu"          # Must be exactly "gpu"
  effect: "NoSchedule"
```

**Matches taint:**
- ✅ `dedicated=gpu:NoSchedule`
- ❌ `dedicated=cpu:NoSchedule` (value doesn't match)
- ❌ `special=gpu:NoSchedule` (key doesn't match)

### Operator: "Exists"

```yaml
tolerations:
- key: "dedicated"
  operator: "Exists"     # Only checks if key exists
  effect: "NoSchedule"  # Value is ignored
```

**Matches taint:**
- ✅ `dedicated=gpu:NoSchedule`
- ✅ `dedicated=cpu:NoSchedule`
- ✅ `dedicated=anything:NoSchedule`
- ❌ `special=gpu:NoSchedule` (key doesn't exist)

## 🎓 Common Use Cases

### Use Case 1: Master Node Protection

**Problem**: Master nodes should only run system pods, not user workloads.

**Solution**:
```bash
# Master nodes are tainted by default
# System pods (like kube-proxy) have tolerations
# User pods don't, so they can't run on master
```

**Check master node taints:**
```bash
kubectl describe node <master-node> | grep Taints
```

### Use Case 2: Dedicated GPU Nodes

**Problem**: You have expensive GPU nodes that should only run ML workloads.

**Solution**:
```bash
# Taint GPU nodes
kubectl taint nodes gpu-node-1 hardware=gpu:NoSchedule

# ML pods have toleration
# Regular pods cannot use GPU nodes
```

### Use Case 3: Node Maintenance

**Problem**: You need to perform maintenance on a node.

**Solution**:
```bash
# Taint with NoExecute to evict pods
kubectl taint nodes worker-node-1 maintenance=true:NoExecute

# Pods are evicted and rescheduled elsewhere
# Perform maintenance
# Remove taint when done
kubectl taint nodes worker-node-1 maintenance=true:NoExecute-
```

### Use Case 4: Cost Optimization

**Problem**: You have expensive nodes (high-memory) that should only run specific workloads.

**Solution**:
```bash
# Taint expensive nodes
kubectl taint nodes highmem-node-1 cost=high:PreferNoSchedule

# Important workloads have toleration
# Scheduler prefers cheaper nodes, but can use expensive ones if needed
```

## 🔄 Differences from Node Selectors

| Feature | Node Selector | Taints & Tolerations |
|---------|---------------|---------------------|
| **Direction** | Pod → Node ("I want this node") | Node → Pod ("This node rejects pods") |
| **Purpose** | Positive selection | Negative selection |
| **Logic** | Hard requirement | Can be soft (PreferNoSchedule) |
| **Use Case** | "Run on SSD nodes" | "Don't run on master nodes" |

**When to use which:**
- **Node Selector**: "I want pods on specific nodes"
- **Taints & Tolerations**: "I don't want most pods on specific nodes"

## 💡 Best Practices

1. **Always taint master nodes**
   - Kubernetes does this by default
   - Prevents user workloads from running on control plane

2. **Use descriptive taint keys**
   - Good: `dedicated=gpu`, `environment=production`
   - Bad: `x=1`, `test=true`

3. **Document your taints**
   - Keep a list of which nodes have which taints
   - Document which workloads need which tolerations

4. **Use NoExecute for maintenance**
   - Evicts pods cleanly
   - Better than manually deleting pods

5. **Combine with Node Selectors**
   - Use both for fine-grained control
   - Node selector: "I want GPU nodes"
   - Taint: "But only if I have the right toleration"

## 🐛 Troubleshooting

### Problem: Pod stuck in Pending

```bash
# Check pod events
kubectl describe pod <pod-name> | grep -A 10 Events

# Common message:
# "0/1 nodes are available: 1 node(s) had taint {dedicated: gpu}, that the pod didn't tolerate"
```

**Solution**: Add matching toleration to pod

### Problem: Pod was evicted

```bash
# Check why pod was evicted
kubectl get pod <pod-name> -o yaml | grep -A 5 reason

# Common reason: Node tainted with NoExecute
```

**Solution**: Add toleration or remove taint

### Problem: Can't remove taint

```bash
# Make sure you include the minus sign
kubectl taint nodes <node-name> key=value:NoSchedule-

# Check if removed
kubectl describe node <node-name> | grep Taints
```

## 📋 Quick Reference

### Common Taints

```bash
# Master node (Kubernetes default)
node-role.kubernetes.io/master:NoSchedule

# Control plane (newer Kubernetes)
node-role.kubernetes.io/control-plane:NoSchedule

# Custom dedicated node
dedicated=special-workload:NoSchedule

# Maintenance mode
maintenance=true:NoExecute
```

### Common Tolerations

```yaml
# Tolerate master node
tolerations:
- key: "node-role.kubernetes.io/master"
  operator: "Exists"
  effect: "NoSchedule"

# Tolerate any taint with key "dedicated"
tolerations:
- key: "dedicated"
  operator: "Exists"
  effect: "NoSchedule"

# Tolerate specific taint
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "gpu"
  effect: "NoSchedule"
```

## 🎯 Practice Exercises

1. **Taint a node** and try to schedule a regular pod (should fail)
2. **Create a pod with toleration** (should succeed)
3. **Use NoExecute** to evict existing pods
4. **Remove the taint** and verify pods can schedule normally
5. **Check master node taints** in your cluster

## 📖 Next Steps

Now that you understand Taints and Tolerations, learn about:
- **Node Selectors**: Positive selection (complement to taints)
- **Affinity/Anti-Affinity**: More advanced scheduling rules

## Examples

### Node with Taint
- `taint-node.yaml` - Example of tainting a node

### Pod with Toleration
- `pod-toleration.yaml` - Pod that can tolerate node taints

### Deployment with Toleration
- `deployment-toleration.yaml` - Deployment with tolerations

## Commands

```bash
# Taint a node
kubectl taint nodes <node-name> key=value:NoSchedule

# View node taints
kubectl describe node <node-name>

# Remove a taint
kubectl taint nodes <node-name> key=value:NoSchedule-

# Remove all taints
kubectl taint nodes <node-name> key-  # Remove by key

# Apply pod with toleration
kubectl apply -f pod-toleration.yaml
```

## Taint Effects

### NoSchedule
- Pods without toleration **cannot** be scheduled
- Existing pods are **not** affected
- Use case: Reserve node for specific workloads

### PreferNoSchedule
- Scheduler **tries to avoid** scheduling pods without toleration
- Not a hard requirement
- Use case: Soft preference for node usage

### NoExecute
- Pods without toleration **cannot** be scheduled
- Existing pods **are evicted** if they don't have toleration
- Use case: Maintenance mode, node draining

## Important Fields

### Taint Format
```
<key>=<value>:<effect>
```

### Toleration Format
```yaml
tolerations:
- key: "key"
  operator: "Equal"  # or "Exists"
  value: "value"
  effect: "NoSchedule"
```

## Operator Types

- **Equal**: Key and value must match exactly
- **Exists**: Key must exist (value is ignored)

## Differences from Node Selectors

| Feature | Node Selector | Taints & Tolerations |
|---------|---------------|---------------------|
| Direction | Pod → Node | Node → Pod |
| Purpose | "I want this node" | "This node rejects pods" |
| Logic | Positive selection | Negative selection |
| Flexibility | Hard requirement | Can be soft (PreferNoSchedule) |

## Best Practices

1. **Master Nodes**: Always taint master nodes to prevent regular workloads
2. **Dedicated Nodes**: Use taints for nodes reserved for specific workloads
3. **Maintenance**: Taint nodes before maintenance with NoExecute
4. **Combine with Node Selectors**: Use both for fine-grained control

## Common Taints

- `node-role.kubernetes.io/master:NoSchedule` - Master node (Kubernetes default)
- `node-role.kubernetes.io/control-plane:NoSchedule` - Control plane node
- `dedicated=special-workload:NoSchedule` - Custom dedicated node
- `maintenance=true:NoExecute` - Node under maintenance


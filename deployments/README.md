# Deployments - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What Deployments are and why they're important
- How to create and manage Deployments
- How rolling updates work
- How to scale and rollback Deployments
- Best practices for using Deployments

## 🎯 What is a Deployment?

A **Deployment** is a Kubernetes resource that manages your application's pods. Think of it as a "smart pod manager" that:
- Keeps your desired number of pods running
- Updates your application without downtime
- Can rollback if something goes wrong
- Automatically replaces failed pods

### Real-World Analogy

Imagine you run a restaurant:
- **Pods** = Individual waiters
- **Deployment** = The manager who ensures you always have 3 waiters working
- If a waiter gets sick (pod fails), the manager hires a replacement
- When you want to update uniforms (new app version), the manager replaces waiters one by one so service never stops

## 🔑 Key Features

- **Rolling Updates**: Update pods incrementally without downtime
- **Rollback**: Revert to a previous version if something goes wrong
- **Scaling**: Scale the number of replicas up or down
- **Self-healing**: Automatically replaces failed pods

## 🏗️ How It Works (Step by Step)

Let's understand the lifecycle:

1. **You create a Deployment** with a YAML file specifying:
   - How many replicas you want (e.g., 3 pods)
   - What container image to use
   - Resource limits

2. **Kubernetes creates a ReplicaSet**:
   - The Deployment doesn't directly manage pods
   - It creates a ReplicaSet, which actually manages the pods
   - This abstraction allows for rolling updates

3. **ReplicaSet creates Pods**:
   - The ReplicaSet ensures the desired number of pods are running
   - If a pod dies, ReplicaSet creates a new one

4. **When you update the Deployment**:
   - A new ReplicaSet is created with the new configuration
   - Old ReplicaSet gradually scales down
   - New ReplicaSet gradually scales up
   - This is called a "rolling update"

5. **If something goes wrong**:
   - You can rollback to the previous ReplicaSet
   - Kubernetes keeps a history of changes

## 📝 Tutorial: Your First Deployment

### Step 1: Create a Basic Deployment

Let's start with the simplest example:

```bash
# Apply the basic deployment
kubectl apply -f basic-deployment.yaml
```

**What happens?**
- Kubernetes reads the YAML file
- Creates a Deployment object
- Deployment creates a ReplicaSet
- ReplicaSet creates 3 pods running nginx

### Step 2: Verify Your Deployment

```bash
# Check if deployment was created
kubectl get deployments

# You should see:
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   3/3     3            3           10s
```

**Understanding the output:**
- **READY**: 3/3 means all 3 desired pods are running
- **UP-TO-DATE**: 3 pods are running the latest version
- **AVAILABLE**: 3 pods are ready to serve traffic

### Step 3: View the Pods

```bash
# View pods created by the deployment
kubectl get pods -l app=nginx

# You should see 3 pods:
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-7d4f8b9c5d-abc12   1/1     Running   0          15s
# nginx-deployment-7d4f8b9c5d-def34   1/1     Running   0          15s
# nginx-deployment-7d4f8b9c5d-ghi56   1/1     Running   0          15s
```

**Notice the naming:**
- Format: `<deployment-name>-<replicaset-hash>-<pod-hash>`
- The hash ensures unique names

### Step 4: Understand the Relationship

```bash
# View the ReplicaSet created by Deployment
kubectl get replicasets

# View detailed information
kubectl describe deployment nginx-deployment
```

## 🔄 Tutorial: Rolling Updates

### Step 1: Update the Deployment

Let's update nginx to a newer version:

```bash
# Method 1: Using kubectl command
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Method 2: Edit the YAML and reapply
kubectl edit deployment nginx-deployment
# Change image: nginx:1.21 to nginx:1.22
```

### Step 2: Watch the Rolling Update

```bash
# Watch the rollout in real-time
kubectl rollout status deployment/nginx-deployment

# In another terminal, watch pods
watch kubectl get pods -l app=nginx
```

**What you'll see:**
1. New pods start with new image (nginx:1.22)
2. Old pods terminate gradually
3. Service never goes down (always pods available)

### Step 3: Check the History

```bash
# View rollout history
kubectl rollout history deployment/nginx-deployment

# View details of a specific revision
kubectl rollout history deployment/nginx-deployment --revision=2
```

## ⏪ Tutorial: Rollback

### Step 1: Rollback to Previous Version

```bash
# Rollback to the previous version
kubectl rollout undo deployment/nginx-deployment

# Rollback to a specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

### Step 2: Verify Rollback

```bash
# Check the current image
kubectl describe deployment nginx-deployment | grep Image

# Should show nginx:1.21 (the previous version)
```

## 📈 Tutorial: Scaling

### Scale Up

```bash
# Scale to 5 replicas
kubectl scale deployment/nginx-deployment --replicas=5

# Verify
kubectl get pods -l app=nginx
# You should now see 5 pods
```

### Scale Down

```bash
# Scale to 2 replicas
kubectl scale deployment/nginx-deployment --replicas=2

# Verify
kubectl get pods -l app=nginx
# You should now see 2 pods (others will be terminated)
```

## 📚 Example Files Explained

### 1. Basic Deployment (`basic-deployment.yaml`)
**What it does:**
- Creates 3 nginx pods
- Sets basic resource requests and limits
- Perfect for learning the basics

**Try it:**
```bash
kubectl apply -f basic-deployment.yaml
kubectl get pods -w  # Watch pods being created
```

### 2. Rolling Update Deployment (`rolling-update-deployment.yaml`)
**What it does:**
- Configures how rolling updates happen
- Sets `maxSurge` and `maxUnavailable`
- Includes health checks (liveness and readiness probes)

**Key concepts:**
- **maxSurge**: How many extra pods can be created during update (default: 25%)
- **maxUnavailable**: How many pods can be unavailable (default: 25%)

**Try it:**
```bash
kubectl apply -f rolling-update-deployment.yaml
kubectl set image deployment/nginx-rolling-update nginx=nginx:1.22
kubectl rollout status deployment/nginx-rolling-update
```

### 3. Resource Limits Deployment (`resource-limits-deployment.yaml`)
**What it does:**
- Sets CPU and memory requests and limits
- Important for resource management

**Understanding resources:**
- **Requests**: Guaranteed resources (scheduler uses this)
- **Limits**: Maximum resources (container can't exceed this)

**Try it:**
```bash
kubectl apply -f resource-limits-deployment.yaml
kubectl describe pod <pod-name>  # See resource limits
```

## 🎓 Common Use Cases

### ✅ When to Use Deployments

- **Stateless applications**: Web servers, APIs, microservices
- **Applications needing updates**: Any app that changes over time
- **High availability**: Applications that need multiple replicas
- **Production workloads**: Most production applications use Deployments

### ❌ When NOT to Use Deployments

- **Stateful applications**: Use StatefulSets instead (databases, etc.)
- **One pod per node**: Use DaemonSets instead (logging, monitoring)
- **Single-run jobs**: Use Jobs or CronJobs instead

## 🔧 Important Fields Explained

### `replicas`
```yaml
replicas: 3
```
- Number of pod replicas to maintain
- Kubernetes will ensure this many pods are always running

### `selector`
```yaml
selector:
  matchLabels:
    app: nginx
```
- Labels used to identify pods managed by this deployment
- Must match labels in pod template

### `template`
```yaml
template:
  metadata:
    labels:
      app: nginx
  spec:
    containers: [...]
```
- The pod template used to create pods
- Every pod created will have these labels and this spec

### `strategy`
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 2
    maxUnavailable: 1
```
- **RollingUpdate**: Update pods gradually (default, recommended)
- **Recreate**: Kill all pods, then create new ones (downtime)

## 🐛 Troubleshooting

### Problem: Pods not starting

```bash
# Check pod status
kubectl get pods

# Check pod events
kubectl describe pod <pod-name>

# Check pod logs
kubectl logs <pod-name>
```

**Common issues:**
- Image pull errors: Check image name and registry access
- Resource constraints: Check if cluster has enough resources
- Configuration errors: Check YAML syntax

### Problem: Deployment stuck

```bash
# Check deployment status
kubectl describe deployment <deployment-name>

# Check rollout status
kubectl rollout status deployment/<deployment-name>

# View events
kubectl get events --sort-by='.lastTimestamp'
```

### Problem: Can't rollback

```bash
# Check if history exists
kubectl rollout history deployment/<deployment-name>

# If no history, check revisionHistoryLimit
kubectl get deployment <deployment-name> -o yaml | grep revisionHistoryLimit
```

## 💡 Best Practices

1. **Always set resource requests and limits**
   - Helps scheduler make better decisions
   - Prevents resource starvation

2. **Use rolling updates for production**
   - Zero-downtime deployments
   - Can rollback if needed

3. **Set appropriate replica counts**
   - Too few: Single point of failure
   - Too many: Wasted resources

4. **Use health checks**
   - Liveness probe: Is the app running?
   - Readiness probe: Is the app ready for traffic?

5. **Label your resources properly**
   - Makes it easier to manage and query
   - Essential for services and selectors

## 📋 Quick Reference Commands

```bash
# Create
kubectl apply -f basic-deployment.yaml

# View
kubectl get deployments
kubectl get pods -l app=nginx
kubectl describe deployment nginx-deployment

# Update
kubectl set image deployment/nginx-deployment nginx=nginx:1.22
kubectl edit deployment nginx-deployment

# Scale
kubectl scale deployment/nginx-deployment --replicas=5

# Rollback
kubectl rollout undo deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment

# Delete
kubectl delete deployment nginx-deployment
```

## 🎯 Practice Exercises

1. **Create a deployment** with 5 replicas of nginx
2. **Update the image** to nginx:1.23 and watch the rolling update
3. **Scale down** to 2 replicas, then **scale up** to 10
4. **Rollback** to the previous version
5. **Delete** the deployment and recreate it

## 📖 Next Steps

Now that you understand Deployments, learn about:
- **Services**: Expose your deployments to the network
- **ConfigMaps**: Manage configuration for your deployments
- **HPA**: Automatically scale deployments based on load


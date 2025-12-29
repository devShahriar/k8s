# ConfigMaps - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What ConfigMaps are and why they're useful
- How to create ConfigMaps from different sources
- How to use ConfigMaps in pods (environment variables, volumes)
- Best practices for configuration management
- When to use ConfigMaps vs Secrets

## 🎯 What is a ConfigMap?

A **ConfigMap** is a Kubernetes object that stores configuration data in key-value pairs. It allows you to separate configuration from your application code, making your applications more flexible and easier to manage.

### Real-World Analogy

Think of a restaurant menu:
- **Application code** = The kitchen (fixed recipes)
- **ConfigMap** = The menu board (can change without changing recipes)
- **Chefs (pods)** = Read the menu board to know what to cook

You can change the menu (ConfigMap) without rebuilding the kitchen (container image)!

### The Problem ConfigMaps Solve

**Without ConfigMaps:**
- Configuration is hardcoded in container images
- Need to rebuild image for every config change
- Same image can't be used in different environments
- Configuration mixed with code

**With ConfigMaps:**
- Configuration stored separately
- Change config without rebuilding images
- Same image works in dev, staging, production
- Configuration managed by Kubernetes

## 🔑 Key Features

- **Non-Sensitive Data**: Store configuration (not secrets!)
- **Multiple Formats**: Key-value pairs, files, directories
- **Dynamic Updates**: Can be updated without pod restart (with volumes)
- **Environment-Specific**: Different ConfigMaps for different environments

## 🏗️ How It Works (Step by Step)

### Step 1: Create a ConfigMap

You create a ConfigMap with your configuration data:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://localhost:5432/mydb"
  log_level: "info"
```

### Step 2: Reference in Pod

You reference the ConfigMap in your pod:

```yaml
env:
- name: DATABASE_URL
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: database_url
```

### Step 3: Pod Uses Config

When the pod starts, the ConfigMap data is injected as environment variables or files, and your application can use it.

## 📝 Tutorial: Your First ConfigMap

### Step 1: Create a Basic ConfigMap

```bash
# Create ConfigMap from YAML
kubectl apply -f basic-configmap.yaml

# Verify it was created
kubectl get configmaps

# View the data
kubectl get configmap app-config -o yaml
```

**What you'll see:**
```yaml
data:
  database_url: postgresql://localhost:5432/mydb
  log_level: info
  max_connections: "100"
```

### Step 2: View ConfigMap Details

```bash
# Describe the ConfigMap
kubectl describe configmap app-config

# Get specific value
kubectl get configmap app-config -o jsonpath='{.data.log_level}'
```

### Step 3: Use ConfigMap in a Pod

```bash
# Apply pod that uses the ConfigMap
kubectl apply -f pod-env-configmap.yaml

# Check the pod
kubectl get pod app-pod

# View environment variables in the pod
kubectl exec app-pod -- env | grep -E "DATABASE_URL|LOG_LEVEL"
```

**You should see:**
```
DATABASE_URL=postgresql://localhost:5432/mydb
LOG_LEVEL=info
```

## 📚 Creating ConfigMaps

### Method 1: From YAML File (Recommended)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  key1: value1
  key2: value2
```

**Apply:**
```bash
kubectl apply -f basic-configmap.yaml
```

### Method 2: From Literal Values

```bash
kubectl create configmap my-config \
  --from-literal=key1=value1 \
  --from-literal=key2=value2
```

### Method 3: From File

```bash
# Create from a single file
kubectl create configmap my-config --from-file=config.properties

# Create from multiple files
kubectl create configmap my-config \
  --from-file=config1.properties \
  --from-file=config2.properties
```

### Method 4: From Directory

```bash
# Creates ConfigMap with all files in directory
kubectl create configmap my-config --from-file=./config-dir/
```

## 🔌 Using ConfigMaps in Pods

### Method 1: As Environment Variables (Individual Keys)

**Best for:** Single configuration values

```yaml
env:
- name: DATABASE_URL
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: database_url
- name: LOG_LEVEL
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: log_level
```

**Try it:**
```bash
kubectl apply -f pod-env-configmap.yaml
kubectl exec <pod-name> -- env | grep DATABASE
```

### Method 2: As Environment Variables (All Keys)

**Best for:** Loading all ConfigMap keys as environment variables

```yaml
envFrom:
- configMapRef:
    name: app-config
```

**Note:** All keys become environment variables. Key names must be valid env var names.

### Method 3: As Volume (Files)

**Best for:** Configuration files, multiple files, file-based configs

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config
volumeMounts:
- name: config-volume
  mountPath: /etc/config
```

**Try it:**
```bash
kubectl apply -f pod-volume-configmap.yaml
kubectl exec <pod-name> -- ls /etc/nginx/conf.d
kubectl exec <pod-name> -- cat /etc/nginx/conf.d/nginx.conf
```

**Benefits:**
- Files are created in the pod
- Can mount specific keys as files
- Supports directory structures

## 📚 Example Files Explained

### Basic ConfigMap (`basic-configmap.yaml`)

**What it contains:**
- Simple key-value pairs
- Multi-line configuration
- Different data types

**Try it:**
```bash
kubectl apply -f basic-configmap.yaml
kubectl get configmap app-config -o yaml
```

### Pod Using ConfigMap as Env Vars (`pod-env-configmap.yaml`)

**What it does:**
- Shows both individual key reference and ConfigMap reference
- Demonstrates how to use ConfigMap values in pods

**Try it:**
```bash
kubectl apply -f pod-env-configmap.yaml
kubectl exec app-pod -- printenv
```

### Pod Using ConfigMap as Volume (`pod-volume-configmap.yaml`)

**What it does:**
- Mounts ConfigMap as files in the pod
- Shows how to use file-based configuration

**Try it:**
```bash
kubectl apply -f pod-volume-configmap.yaml
kubectl exec nginx-pod -- cat /etc/nginx/conf.d/nginx.conf
```

## 🔄 Updating ConfigMaps

### Update ConfigMap

```bash
# Method 1: Edit directly
kubectl edit configmap app-config

# Method 2: Apply updated YAML
kubectl apply -f updated-configmap.yaml

# Method 3: Replace
kubectl create configmap app-config --from-literal=key=newvalue --dry-run=client -o yaml | kubectl apply -f -
```

### How Updates Work

**Environment Variables:**
- ❌ **NOT updated automatically**
- Pod must be restarted to pick up changes
- Use Deployment rolling update

**Volumes:**
- ✅ **Updated automatically** (after sync period, ~10-60 seconds)
- No pod restart needed
- Changes reflected in mounted files

**To force update with env vars:**
```bash
# Restart pods to pick up new env vars
kubectl rollout restart deployment <deployment-name>
```

## 🎓 Common Use Cases

### Use Case 1: Application Configuration

**Scenario:** Your app needs database URL, API endpoints, feature flags.

**Solution:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/app"
  api_endpoint: "https://api.example.com"
  feature_flags: "new-ui,beta-feature"
```

### Use Case 2: Configuration Files

**Scenario:** Your app reads config from files (nginx.conf, app.properties).

**Solution:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
        listen 80;
        location / {
            root /usr/share/nginx/html;
        }
    }
```

Mount as volume in pod.

### Use Case 3: Environment-Specific Configs

**Scenario:** Different configs for dev, staging, production.

**Solution:**
- Create separate ConfigMaps: `app-config-dev`, `app-config-prod`
- Reference appropriate ConfigMap in each environment
- Use same container image everywhere

## ⚠️ Important Limitations

1. **Size Limit**: 1MB per ConfigMap
2. **Not for Secrets**: Use Secrets for sensitive data
3. **Plain Text**: Data is not encrypted
4. **Namespace Scoped**: ConfigMaps are namespace-specific

## 🔧 Important Fields Explained

### `data`
```yaml
data:
  key1: value1
  key2: value2
```
- Key-value pairs stored as strings
- Values can be multi-line (use `|`)

### `binaryData`
```yaml
binaryData:
  binary-file: <base64-encoded-data>
```
- For binary data
- Must be base64 encoded
- Less common, usually use Secrets for binary

## 🐛 Troubleshooting

### Problem: ConfigMap not found

```bash
# Check if ConfigMap exists
kubectl get configmap <name>

# Check namespace
kubectl get configmap <name> -n <namespace>

# Verify name in pod spec matches
kubectl describe pod <pod-name> | grep -A 5 ConfigMap
```

### Problem: Environment variable not set

```bash
# Check pod spec
kubectl get pod <pod-name> -o yaml | grep -A 10 env

# Verify ConfigMap key exists
kubectl get configmap <name> -o yaml

# Check pod logs for errors
kubectl logs <pod-name>
```

### Problem: ConfigMap update not reflected

**For environment variables:**
- Restart the pod/deployment
- Env vars are set at pod creation time

**For volumes:**
- Wait 10-60 seconds for sync
- Check if volume is mounted correctly
- Verify file permissions

## 💡 Best Practices

1. **Use ConfigMaps for non-sensitive data**
   - Application settings
   - Feature flags
   - File paths
   - **NOT** passwords, tokens, keys

2. **Keep ConfigMaps small**
   - 1MB limit per ConfigMap
   - Split large configs into multiple ConfigMaps

3. **Use descriptive names**
   - Good: `app-config`, `database-config`
   - Bad: `config1`, `cm1`

4. **Version your ConfigMaps**
   - Include version in name: `app-config-v1`
   - Or use labels for versioning

5. **Use volumes for file-based configs**
   - Better for configuration files
   - Supports automatic updates

6. **Document your ConfigMaps**
   - Add comments in YAML
   - Document what each key is for

## 📋 Quick Reference Commands

```bash
# Create
kubectl apply -f basic-configmap.yaml
kubectl create configmap my-config --from-literal=key=value

# View
kubectl get configmaps
kubectl get configmap <name> -o yaml
kubectl describe configmap <name>

# Edit
kubectl edit configmap <name>

# Delete
kubectl delete configmap <name>

# Use in pod
# Reference in pod spec as shown in examples
```

## 🎯 Practice Exercises

1. **Create a ConfigMap** with your application settings
2. **Use it in a pod** as environment variables
3. **Mount it as a volume** and verify files are created
4. **Update the ConfigMap** and see how it affects the pod
5. **Create ConfigMaps** from files and directories

## 📖 Next Steps

Now that you understand ConfigMaps, learn about:
- **Secrets**: For sensitive configuration data
- **Deployments**: How to use ConfigMaps with Deployments
- **StatefulSets**: Configuration for stateful applications

# Custom Resource Definitions (CRDs) - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What CRDs are and why they're powerful
- How to create and manage CRDs
- Custom resources and controllers
- How to extend Kubernetes API
- Real-world use cases

## 🎯 What are CRDs?

**Custom Resource Definitions (CRDs)** allow you to extend the Kubernetes API with your own resource types. Think of them as creating new "kinds" of Kubernetes resources.

### Real-World Analogy

Think of Kubernetes as a restaurant:
- **Built-in resources** = Standard menu items (Deployment, Service, Pod)
- **CRDs** = Custom menu items you create
- **Custom Controller** = Chef who knows how to prepare your custom dish

### The Problem They Solve

**Without CRDs:**
- Limited to built-in Kubernetes resources
- Can't model domain-specific concepts
- Need to use generic resources (ConfigMaps, Annotations)

**With CRDs:**
- Define your own resource types
- Model domain-specific concepts
- Cleaner, more intuitive APIs
- Enable operator patterns

## 🔑 Key Concepts

### Custom Resource Definition (CRD)
- Defines a new resource type
- Specifies schema (fields, types, validation)
- Extends Kubernetes API

### Custom Resource (CR)
- Instance of a CRD
- Like a Pod is an instance of Pod resource

### Custom Controller
- Watches Custom Resources
- Implements business logic
- Creates/updates other resources

## 🏗️ How It Works (Step by Step)

### Step 1: Define CRD

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: websites.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema: {...}
  scope: Namespaced
  names:
    plural: websites
    singular: website
    kind: Website
```

### Step 2: Create Custom Resource

```yaml
apiVersion: example.com/v1
kind: Website
metadata:
  name: my-website
spec:
  domain: example.com
  replicas: 3
```

### Step 3: Controller Watches and Acts

Controller watches for Website resources and creates Deployments, Services, etc.

## 📝 Tutorial: Your First CRD

### Step 1: Create CRD

```bash
# Apply CRD
kubectl apply -f website-crd.yaml

# Verify CRD created
kubectl get crd

# Check CRD details
kubectl describe crd websites.example.com
```

### Step 2: Create Custom Resource

```bash
# Create Website resource
kubectl apply -f website-example.yaml

# View custom resources
kubectl get websites

# Check details
kubectl describe website my-website
```

### Step 3: Verify API Extension

```bash
# Check API resources
kubectl api-resources | grep website

# Get custom resource
kubectl get website my-website -o yaml
```

## 📚 Example Files Explained

### Basic CRD (`website-crd.yaml`)
- Defines Website custom resource
- Includes schema validation
- Shows versioning

### Custom Resource (`website-example.yaml`)
- Instance of Website CRD
- Shows how to use custom resources

## 🎓 Common Use Cases

### Use Case 1: Application Operators
Define resources like `Database`, `Cache`, `Application`

### Use Case 2: Infrastructure as Code
Define `Cluster`, `NodePool`, `Network` resources

### Use Case 3: CI/CD Resources
Define `Pipeline`, `Build`, `Deployment` resources

## 💡 Best Practices

1. Use meaningful group names
2. Version your CRDs
3. Add validation schemas
4. Document your CRDs
5. Use subresources (status, scale)


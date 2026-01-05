# Service Accounts - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What Service Accounts are and why they exist
- How Service Accounts relate to authentication
- RBAC (Role-Based Access Control)
- How pods use Service Accounts
- Image pull secrets
- Best practices for security

## 🎯 What is a Service Account?

A **Service Account** is an identity for processes running in pods. It's used for authentication and authorization when pods interact with the Kubernetes API.

### Real-World Analogy

Think of Service Accounts as employee badges:
- **User Account** = Your personal ID badge (for humans)
- **Service Account** = Service/robot ID badge (for applications)
- **Permissions** = What doors you can access

### The Problem They Solve

**Without Service Accounts:**
- Pods can't authenticate to Kubernetes API
- No way to control what pods can do
- All pods have same permissions (or none)

**With Service Accounts:**
- Pods have identity
- Fine-grained permissions (RBAC)
- Secure API access
- Image pull authentication

## 🔑 Key Concepts

### Service Account
- Identity for pods
- Namespace-scoped
- Can have secrets attached

### Default Service Account
- Every namespace has `default` service account
- Created automatically
- Limited permissions

### RBAC (Role-Based Access Control)
- Roles: Define permissions
- RoleBindings: Grant roles to Service Accounts
- ClusterRoles/ClusterRoleBindings: Cluster-wide

## 🏗️ How It Works (Step by Step)

### Step 1: Service Account Created

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
```

### Step 2: Role Defined

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

### Step 3: Role Bound to Service Account

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: my-service-account
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Step 4: Pod Uses Service Account

```yaml
spec:
  serviceAccountName: my-service-account
```

## 📝 Tutorial: Your First Service Account

### Step 1: Create Service Account

```bash
# Create service account
kubectl apply -f basic-service-account.yaml

# View service accounts
kubectl get serviceaccounts
kubectl get sa  # Short form

# Check details
kubectl describe sa my-service-account
```

### Step 2: Create Role and RoleBinding

```bash
# Create role
kubectl apply -f role-pod-reader.yaml

# Create role binding
kubectl apply -f rolebinding-pod-reader.yaml

# Verify
kubectl get role,rolebinding
```

### Step 3: Use in Pod

```bash
# Create pod with service account
kubectl apply -f pod-with-sa.yaml

# Test API access from pod
kubectl exec pod-with-sa -- sh -c "curl -k https://kubernetes/api/v1/namespaces/default/pods"
```

## 📚 Example Files Explained

### Basic Service Account (`basic-service-account.yaml`)
- Simple service account
- Can attach secrets

### Role and RoleBinding (`role-pod-reader.yaml`, `rolebinding-pod-reader.yaml`)
- Defines permissions
- Grants to service account

### Pod with Service Account (`pod-with-sa.yaml`)
- Pod using custom service account
- Has API access

## 🎓 Common Use Cases

### Use Case 1: Pod Needs API Access
Service account with permissions to read pods, create services, etc.

### Use Case 2: Image Pull from Private Registry
Service account with imagePullSecrets.

### Use Case 3: Different Permissions per Namespace
Different service accounts with different roles in each namespace.

## 💡 Best Practices

1. Create dedicated service accounts (don't use default)
2. Follow principle of least privilege
3. Use namespaced roles when possible
4. Regularly audit permissions
5. Document why each permission is needed


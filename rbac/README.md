# Kubernetes RBAC - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What RBAC is and why it's critical for security
- Roles vs ClusterRoles
- RoleBindings vs ClusterRoleBindings
- Service Accounts and their relationship to RBAC
- How to create and manage RBAC resources
- Best practices for Kubernetes security

## 🎯 What is RBAC?

**RBAC (Role-Based Access Control)** is a method of regulating access to Kubernetes resources based on roles assigned to users and service accounts.

### Real-World Analogy

Think of RBAC like a company's access control system:
- **User/Service Account** = Employee
- **Role** = Job title (Developer, Admin, Viewer)
- **RoleBinding** = Assignment of job title to employee
- **Resources** = What they can access (files, systems)
- **Verbs** = What they can do (read, write, delete)

### The Problem RBAC Solves

**Without RBAC:**
- All users have full access (security risk!)
- No way to limit what users can do
- Can't delegate permissions
- No audit trail

**With RBAC:**
- Fine-grained permissions
- Principle of least privilege
- Separation of duties
- Audit and compliance

## 🔑 Key Concepts

### Role
- **Namespaced resource**
- Defines permissions in a namespace
- Can only grant access to resources in that namespace

### ClusterRole
- **Cluster-wide resource**
- Defines permissions across all namespaces
- Can grant access to cluster-scoped resources

### RoleBinding
- **Namespaced resource**
- Binds a Role to subjects (users, groups, service accounts)
- Only affects the namespace it's created in

### ClusterRoleBinding
- **Cluster-wide resource**
- Binds a ClusterRole to subjects
- Affects all namespaces

### Subjects
- **Users**: Human users
- **Groups**: Collections of users
- **ServiceAccounts**: Pod identities

### Resources & Verbs
- **Resources**: What you can access (pods, services, deployments)
- **Verbs**: What you can do (get, list, create, update, delete, watch)

## 🏗️ How It Works (Step by Step)

### Step 1: Define Permissions (Role)

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

### Step 2: Grant Permissions (RoleBinding)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: my-sa
roleRef:
  kind: Role
  name: pod-reader
```

### Step 3: Use in Pod

```yaml
spec:
  serviceAccountName: my-sa
```

## 📝 Tutorial: Your First RBAC Setup

### Step 1: Create Service Account

```bash
# Create service account
kubectl apply -f service-account.yaml

# View service accounts
kubectl get serviceaccounts
```

### Step 2: Create Role

```bash
# Create role
kubectl apply -f role-pod-reader.yaml

# View roles
kubectl get roles

# Check role details
kubectl describe role pod-reader
```

### Step 3: Create RoleBinding

```bash
# Create role binding
kubectl apply -f rolebinding-pod-reader.yaml

# View role bindings
kubectl get rolebindings

# Check what permissions are granted
kubectl describe rolebinding read-pods
```

### Step 4: Test Permissions

```bash
# Create pod with service account
kubectl apply -f pod-with-sa.yaml

# Test API access from pod
kubectl exec pod-with-sa -- sh -c "curl -k -H 'Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)' https://kubernetes/api/v1/namespaces/default/pods"
```

## 📚 Example Files Explained

### Service Account (`service-account.yaml`)
- Identity for pods
- Used in RoleBindings

### Role (`role-pod-reader.yaml`)
- Defines what can be done
- Namespace-scoped

### ClusterRole (`clusterrole-admin.yaml`)
- Cluster-wide permissions
- Can access cluster-scoped resources

### RoleBinding (`rolebinding-pod-reader.yaml`)
- Grants role to service account
- Namespace-scoped

### ClusterRoleBinding (`clusterrolebinding-admin.yaml`)
- Grants cluster role
- Cluster-wide effect

## 🎓 Common Use Cases

### Use Case 1: Read-Only Access
Role that allows reading pods but not modifying them.

### Use Case 2: Developer Access
Role that allows creating/updating deployments in dev namespace.

### Use Case 3: Admin Access
ClusterRole that allows full access to all resources.

### Use Case 4: Namespace Isolation
Different roles in different namespaces for multi-tenancy.

## 💡 Best Practices

1. **Principle of Least Privilege**
   - Grant minimum permissions needed
   - Start restrictive, add as needed

2. **Use Namespaced Roles When Possible**
   - More secure than ClusterRoles
   - Limits blast radius

3. **Separate Service Accounts**
   - Don't use default service account
   - One service account per application

4. **Regular Audits**
   - Review permissions regularly
   - Remove unused roles/bindings

5. **Document Permissions**
   - Document why each permission is needed
   - Review during security audits

## 🔍 Common Verbs

- **get**: Read a specific resource
- **list**: List all resources of a type
- **create**: Create new resources
- **update**: Update existing resources
- **patch**: Partially update resources
- **delete**: Delete resources
- **watch**: Watch for changes
- *****: All verbs

## 📋 Quick Reference Commands

```bash
# Check permissions
kubectl auth can-i create pods --as=system:serviceaccount:default:my-sa

# View roles
kubectl get roles,clusterroles

# View bindings
kubectl get rolebindings,clusterrolebindings

# Describe permissions
kubectl describe role pod-reader
kubectl describe rolebinding read-pods
```


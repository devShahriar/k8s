# Secrets - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What Secrets are and when to use them
- How to create and manage Secrets
- How to use Secrets in pods securely
- Different types of Secrets
- Security best practices

## 🎯 What is a Secret?

A **Secret** is a Kubernetes object that stores sensitive data like passwords, tokens, and keys. It's similar to ConfigMaps but designed specifically for confidential information.

### Real-World Analogy

Think of a safe:
- **ConfigMap** = Public notice board (anyone can read)
- **Secret** = Safe with combination (only authorized people can access)
- **Your app** = Person with the combination

### The Problem Secrets Solve

**Without Secrets:**
- Passwords hardcoded in images (security risk!)
- Secrets in environment variables (visible in process list)
- No way to manage sensitive data properly
- Secrets mixed with code

**With Secrets:**
- Sensitive data stored separately
- Base64 encoded (not encrypted, but better than plain text)
- RBAC can control access
- Can be rotated without code changes

## ⚠️ Important Security Note

**Secrets are base64 encoded, NOT encrypted!**

- Base64 is encoding (like translating to another language)
- Anyone with access can decode it
- **Always use RBAC** to restrict access
- **Enable encryption at rest** for etcd
- Consider external secret managers for production

## 🔑 Key Features

- **Sensitive Data Storage**: Passwords, tokens, certificates
- **Base64 Encoding**: Data is stored as base64-encoded strings
- **Type Safety**: Different types for different use cases
- **Access Control**: Can be restricted using RBAC

## 🏗️ How It Works (Step by Step)

### Step 1: Create a Secret

You create a Secret with sensitive data:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  username: admin
  password: secret123
```

### Step 2: Reference in Pod

You reference the Secret in your pod:

```yaml
env:
- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: password
```

### Step 3: Pod Uses Secret

When the pod starts, the Secret data is injected, and your application can use it securely.

## 📝 Tutorial: Your First Secret

### Step 1: Create a Basic Secret

```bash
# Create Secret from YAML
kubectl apply -f basic-secret.yaml

# Verify it was created
kubectl get secrets

# View the secret (data is base64 encoded)
kubectl get secret app-secret -o yaml
```

**What you'll see:**
```yaml
data:
  username: YWRtaW4=        # base64 encoded
  password: c2VjcmV0MTIz    # base64 encoded
```

### Step 2: Decode Secret Data

```bash
# Decode a specific value
kubectl get secret app-secret -o jsonpath='{.data.password}' | base64 -d
echo

# Decode username
kubectl get secret app-secret -o jsonpath='{.data.username}' | base64 -d
echo
```

**You should see:**
```
secret123
admin
```

### Step 3: Use Secret in a Pod

```bash
# Apply pod that uses the Secret
kubectl apply -f pod-secret-env.yaml

# Check the pod
kubectl get pod app-pod

# View environment variables (values are visible in process)
kubectl exec app-pod -- env | grep -E "DB_USERNAME|DB_PASSWORD"
```

**Note:** Environment variables are visible in the process, so be careful!

## 📚 Creating Secrets

### Method 1: From YAML File (Using stringData)

**Recommended for learning** - values in plain text, automatically encoded:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:  # Plain text (automatically base64 encoded)
  username: admin
  password: secret123
```

**Apply:**
```bash
kubectl apply -f basic-secret.yaml
```

### Method 2: From YAML File (Using data - Pre-encoded)

**For production** - values already base64 encoded:

```bash
# Encode values first
echo -n 'admin' | base64
# Output: YWRtaW4=

echo -n 'secret123' | base64
# Output: c2VjcmV0MTIz
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=
  password: c2VjcmV0MTIz
```

### Method 3: From Literal Values

```bash
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

### Method 4: From File

```bash
# Create from files
kubectl create secret generic my-secret \
  --from-file=username.txt \
  --from-file=password.txt
```

### Method 5: TLS Secret

```bash
# Create TLS secret from certificate files
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key
```

### Method 6: Docker Registry Secret

```bash
# Create secret for private Docker registry
kubectl create secret docker-registry regcred \
  --docker-server=docker.io \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=myemail@example.com
```

## 🔌 Using Secrets in Pods

### Method 1: As Environment Variables (Individual Keys)

**Best for:** Single secret values

```yaml
env:
- name: DB_USERNAME
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: username
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: password
```

**Try it:**
```bash
kubectl apply -f pod-secret-env.yaml
kubectl exec <pod-name> -- env | grep DB_
```

### Method 2: As Environment Variables (All Keys)

```yaml
envFrom:
- secretRef:
    name: app-secret
```

**Note:** All keys become environment variables. Key names must be valid env var names.

### Method 3: As Volume (Files)

**Best for:** Certificates, keys, file-based secrets

```yaml
volumes:
- name: secret-volume
  secret:
    secretName: app-secret
volumeMounts:
- name: secret-volume
  mountPath: /etc/secrets
  readOnly: true
```

**Benefits:**
- Files are created in the pod
- Can mount specific keys as files
- More secure than environment variables (not in process list)

### Method 4: For Image Pull (Docker Registry)

```yaml
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: private-registry.io/myapp:latest
```

## 📚 Secret Types Explained

### 1. Opaque (Default)

**Use for:** Generic user-defined secrets

```yaml
type: Opaque
```

**Example:** Passwords, API keys, tokens

### 2. kubernetes.io/tls

**Use for:** TLS certificates

```yaml
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

**Created with:**
```bash
kubectl create secret tls tls-secret --cert=cert.crt --key=cert.key
```

### 3. kubernetes.io/dockerconfigjson

**Use for:** Docker registry credentials

```yaml
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-config>
```

**Created with:**
```bash
kubectl create secret docker-registry regcred \
  --docker-server=registry.io \
  --docker-username=user \
  --docker-password=pass
```

### 4. kubernetes.io/basic-auth

**Use for:** Basic authentication credentials

### 5. kubernetes.io/ssh-auth

**Use for:** SSH authentication

## 📚 Example Files Explained

### Basic Secret (`basic-secret.yaml`)

**What it contains:**
- Username and password
- Shows both `stringData` (plain text) and `data` (base64)

**Try it:**
```bash
kubectl apply -f basic-secret.yaml
kubectl get secret app-secret -o yaml
```

### TLS Secret (`tls-secret.yaml`)

**What it contains:**
- TLS certificate and key
- Template for production use

**Note:** Replace with actual certificates in production!

### Pod Using Secret (`pod-secret-env.yaml`)

**What it does:**
- Shows how to use Secret values as environment variables
- Demonstrates secure secret injection

**Try it:**
```bash
kubectl apply -f pod-secret-env.yaml
kubectl exec app-pod -- env | grep DB_
```

## 🔄 Updating Secrets

### Update Secret

```bash
# Method 1: Edit directly
kubectl edit secret app-secret

# Method 2: Apply updated YAML
kubectl apply -f updated-secret.yaml

# Method 3: Replace
kubectl create secret generic app-secret \
  --from-literal=password=newpassword \
  --dry-run=client -o yaml | kubectl apply -f -
```

### How Updates Work

**Environment Variables:**
- ❌ **NOT updated automatically**
- Pod must be restarted to pick up changes

**Volumes:**
- ✅ **Updated automatically** (after sync period)
- No pod restart needed

**To force update:**
```bash
# Restart pods to pick up new secret values
kubectl rollout restart deployment <deployment-name>
```

## 🔒 Security Best Practices

### 1. Use RBAC to Restrict Access

```yaml
# Role that allows reading secrets
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
```

### 2. Enable Encryption at Rest

**For etcd (where secrets are stored):**
- Configure etcd encryption
- Use cloud provider encryption
- Encrypt etcd backups

### 3. Use External Secret Managers

**For production:**
- HashiCorp Vault
- AWS Secrets Manager
- Google Secret Manager
- Azure Key Vault

### 4. Never Commit Secrets

- Use `.gitignore` for secret files
- Use CI/CD secret management
- Rotate secrets regularly

### 5. Prefer Volumes Over Environment Variables

**Why:**
- Env vars visible in process list
- Volumes more secure
- Better for certificates and keys

### 6. Use Separate Secrets per Environment

- `app-secret-dev`
- `app-secret-staging`
- `app-secret-prod`

## 🎓 Common Use Cases

### Use Case 1: Database Credentials

**Scenario:** Your app needs database username and password.

**Solution:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  username: dbuser
  password: dbpassword123
```

Use in pod as environment variables.

### Use Case 2: TLS Certificates

**Scenario:** Your app needs SSL/TLS certificates.

**Solution:**
```bash
kubectl create secret tls app-tls \
  --cert=cert.crt \
  --key=cert.key
```

Mount as volume in pod.

### Use Case 3: Private Docker Registry

**Scenario:** You need to pull images from private registry.

**Solution:**
```bash
kubectl create secret docker-registry regcred \
  --docker-server=registry.io \
  --docker-username=user \
  --docker-password=pass
```

Reference in pod spec as `imagePullSecrets`.

## 🔧 Important Fields Explained

### `type`
```yaml
type: Opaque  # or kubernetes.io/tls, kubernetes.io/dockerconfigjson, etc.
```
- Defines the structure of the secret
- Opaque is default for generic secrets

### `data`
```yaml
data:
  key: <base64-encoded-value>
```
- Base64-encoded key-value pairs
- Use for pre-encoded values

### `stringData`
```yaml
stringData:
  key: plain-text-value
```
- Plain text values (automatically encoded)
- Easier to use, but less secure in YAML files

## 🐛 Troubleshooting

### Problem: Secret not found

```bash
# Check if Secret exists
kubectl get secret <name>

# Check namespace
kubectl get secret <name> -n <namespace>

# Verify name in pod spec
kubectl describe pod <pod-name> | grep -A 5 Secret
```

### Problem: Can't decode secret

```bash
# Check if value is base64 encoded
kubectl get secret <name> -o jsonpath='{.data.key}'

# Decode it
kubectl get secret <name> -o jsonpath='{.data.key}' | base64 -d
```

### Problem: Permission denied

```bash
# Check RBAC permissions
kubectl auth can-i get secrets --as=system:serviceaccount:default:default

# Check service account
kubectl get pod <pod-name> -o jsonpath='{.spec.serviceAccountName}'
```

## 📋 Quick Reference Commands

```bash
# Create
kubectl apply -f basic-secret.yaml
kubectl create secret generic my-secret --from-literal=key=value
kubectl create secret tls tls-secret --cert=cert.crt --key=cert.key

# View
kubectl get secrets
kubectl get secret <name> -o yaml
kubectl describe secret <name>

# Decode
kubectl get secret <name> -o jsonpath='{.data.key}' | base64 -d

# Edit
kubectl edit secret <name>

# Delete
kubectl delete secret <name>
```

## 🎯 Practice Exercises

1. **Create a Secret** with database credentials
2. **Use it in a pod** as environment variables
3. **Mount it as a volume** and verify files are created
4. **Create a TLS secret** (use self-signed cert for testing)
5. **Update the Secret** and restart pods to pick up changes

## 📖 Next Steps

Now that you understand Secrets, learn about:
- **ConfigMaps**: For non-sensitive configuration
- **RBAC**: To secure access to Secrets
- **Deployments**: How to use Secrets with Deployments

## ⚠️ Security Reminder

Remember:
- Secrets are **base64 encoded, NOT encrypted**
- Always use **RBAC** to restrict access
- Enable **encryption at rest** for etcd
- Consider **external secret managers** for production
- **Never commit secrets** to version control

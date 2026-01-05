# Helm - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What Helm is and why it's the "package manager for Kubernetes"
- Charts, Releases, and Repositories
- How to create and use Helm charts
- Helm templates and values
- Best practices for Helm deployments
- Helm 3 vs Helm 2 differences

## 🎯 What is Helm?

**Helm** is a package manager for Kubernetes. It helps you manage Kubernetes applications by packaging them into charts that can be easily installed, upgraded, and shared.

### Real-World Analogy

Think of Helm like package managers:
- **Helm** = apt/yum/homebrew for Kubernetes
- **Chart** = Package (like .deb or .rpm)
- **Release** = Installed package instance
- **Repository** = Package repository (like apt repos)

### The Problem Helm Solves

**Without Helm:**
- Managing multiple YAML files manually
- No versioning of deployments
- Difficult to share configurations
- Hard to manage dependencies
- No rollback mechanism

**With Helm:**
- Single command deployments
- Version management
- Easy configuration with values
- Template system for reusability
- Built-in rollback

## 🔑 Key Concepts

### Chart
- Package of Kubernetes resources
- Contains templates and values
- Can have dependencies

### Release
- Instance of a chart deployed to cluster
- Has a name and version
- Can be upgraded or rolled back

### Repository
- Collection of charts
- Can be public or private
- Examples: Helm Hub, Artifact Hub

### Template
- YAML files with Go templating
- Parameterized with values
- Generates Kubernetes manifests

### Values
- Configuration for charts
- Can override defaults
- Passed via values.yaml or --set

## 🏗️ How It Works (Step by Step)

### Step 1: Install Helm

```bash
# Install Helm 3
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify installation
helm version
```

### Step 2: Create or Use a Chart

```bash
# Create new chart
helm create my-app

# Or use existing chart
helm install my-release stable/nginx
```

### Step 3: Customize Values

```bash
# Edit values.yaml
# Or override with --set
helm install my-release ./my-chart --set image.tag=1.21
```

### Step 4: Install Release

```bash
# Install chart
helm install my-release ./my-chart

# Check status
helm status my-release

# List releases
helm list
```

## 📝 Tutorial: Your First Helm Chart

### Step 1: Create a Chart

```bash
# Create new chart
helm create my-first-chart

# Explore structure
tree my-first-chart
```

**Chart Structure:**
```
my-first-chart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default values
├── templates/          # Template files
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl    # Helper templates
└── charts/             # Chart dependencies
```

### Step 2: Customize Chart

```bash
# Edit values.yaml
vim my-first-chart/values.yaml

# Edit templates
vim my-first-chart/templates/deployment.yaml
```

### Step 3: Install Chart

```bash
# Dry run (see what would be created)
helm install my-release ./my-first-chart --dry-run --debug

# Install
helm install my-release ./my-first-chart

# Check status
helm status my-release
```

### Step 4: Upgrade Release

```bash
# Modify values
vim my-first-chart/values.yaml

# Upgrade
helm upgrade my-release ./my-first-chart

# Check history
helm history my-release
```

### Step 5: Rollback

```bash
# List revisions
helm history my-release

# Rollback to previous
helm rollback my-release

# Rollback to specific revision
helm rollback my-release 2
```

## 📚 Chart Structure Explained

### Chart.yaml
- Chart metadata
- Name, version, description
- Dependencies

### values.yaml
- Default configuration
- Can be overridden
- Used in templates

### templates/
- Kubernetes manifest templates
- Go templating syntax
- Generated during install

### _helpers.tpl
- Reusable template functions
- Shared across templates

## 🎓 Common Use Cases

### Use Case 1: Deploy Application
Package your application as a Helm chart for easy deployment.

### Use Case 2: Share Configurations
Share Helm charts via repositories.

### Use Case 3: Environment Management
Different values files for dev, staging, production.

### Use Case 4: Dependency Management
Charts can depend on other charts.

## 💡 Best Practices

1. **Version Your Charts**
   - Use semantic versioning
   - Update Chart.yaml version

2. **Use Values Files**
   - Separate values for environments
   - Document all values

3. **Template Best Practices**
   - Use helpers for common patterns
   - Validate required values
   - Add comments

4. **Test Your Charts**
   - Use `helm lint`
   - Test with `--dry-run`
   - Test in dev first

5. **Security**
   - Don't hardcode secrets
   - Use secrets management
   - Review generated manifests

## 📋 Quick Reference Commands

```bash
# Create chart
helm create my-chart

# Install
helm install my-release ./my-chart

# Upgrade
helm upgrade my-release ./my-chart

# Rollback
helm rollback my-release

# List releases
helm list

# Uninstall
helm uninstall my-release

# Lint
helm lint ./my-chart

# Package
helm package ./my-chart

# Search
helm search repo nginx
```


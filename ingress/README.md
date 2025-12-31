# Ingress - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What Ingress is and why it's better than NodePort/LoadBalancer
- How to create and configure Ingress resources
- Different Ingress controllers and their features
- Path-based and host-based routing
- TLS/SSL termination with Ingress
- Annotations and their purposes
- Best practices for production use

## 🎯 What is Ingress?

**Ingress** is a Kubernetes resource that provides HTTP and HTTPS routing to services based on hostname or path. It acts as a reverse proxy and load balancer for your Kubernetes services.

### Real-World Analogy

Think of a hotel concierge:
- **Services** = Different hotel services (restaurant, spa, gym)
- **Ingress** = The concierge desk (routes guests to the right service)
- **Ingress Controller** = The actual concierge (does the routing)
- **Rules** = Instructions for the concierge (path-based, host-based)

### The Problem Ingress Solves

**Without Ingress:**
- Need NodePort for each service (many ports to manage)
- Need LoadBalancer for each service (expensive, one IP per service)
- No path-based routing (can't route `/api` to one service, `/web` to another)
- No hostname-based routing (can't use multiple domains)
- No SSL/TLS termination at cluster level

**With Ingress:**
- Single entry point (one IP/domain)
- Path-based routing (multiple services, one domain)
- Hostname-based routing (multiple domains)
- SSL/TLS termination
- Cost-effective (one LoadBalancer, many services)

## 🔑 Key Concepts

### Ingress Resource
- **Kubernetes object**: Defines routing rules
- **Doesn't do routing**: Just defines what should happen
- **Needs controller**: Ingress Controller does the actual work

### Ingress Controller
- **Actual implementation**: Does the routing
- **Examples**: NGINX, Traefik, HAProxy, AWS ALB
- **Must be installed**: Not included by default

### Ingress vs Service Types

| Feature | NodePort | LoadBalancer | Ingress |
|---------|----------|--------------|---------|
| External Access | ✅ | ✅ | ✅ |
| Path Routing | ❌ | ❌ | ✅ |
| Host Routing | ❌ | ❌ | ✅ |
| SSL/TLS | Manual | Manual | ✅ Built-in |
| Cost | Low | High (per service) | Low (one LB) |

## 🏗️ How It Works (Step by Step)

### Step 1: Install Ingress Controller

```bash
# Example: Install NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

### Step 2: Create Services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - port: 80
```

### Step 3: Create Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Step 4: Ingress Controller Routes Traffic

1. External request arrives at Ingress Controller
2. Controller checks Ingress rules
3. Matches hostname and path
4. Routes to appropriate service
5. Service forwards to pods

## 📝 Tutorial: Your First Ingress

### Step 1: Install Ingress Controller

**For Minikube:**
```bash
minikube addons enable ingress
```

**For other clusters:**
```bash
# NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Wait for controller to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

### Step 2: Create a Deployment and Service

```bash
# Create deployment
kubectl apply -f ../deployments/basic-deployment.yaml

# Create service
kubectl apply -f ../services/clusterip-service.yaml
```

### Step 3: Create Basic Ingress

```bash
# Apply basic ingress
kubectl apply -f basic-ingress.yaml

# View ingress
kubectl get ingress

# Check ingress details
kubectl describe ingress basic-ingress
```

**What you'll see:**
```
NAME            CLASS   HOSTS              ADDRESS        PORTS
basic-ingress   nginx   example.com        192.168.1.100  80
```

### Step 4: Test the Ingress

**Get Ingress IP:**
```bash
# Get external IP
kubectl get ingress basic-ingress

# Or for minikube
minikube ip
```

**Test with curl:**
```bash
# Add to /etc/hosts (or use actual domain)
echo "$(minikube ip) example.com" | sudo tee -a /etc/hosts

# Test
curl http://example.com
```

## 🌐 Ingress Rules Explained

### Path-Based Routing

Routes different paths to different services:

```yaml
rules:
- http:
    paths:
    - path: /api
      pathType: Prefix
      backend:
        service:
          name: api-service
          port:
            number: 80
    - path: /web
      pathType: Prefix
      backend:
        service:
          name: web-service
          port:
            number: 80
```

**Example:**
- `http://example.com/api/users` → api-service
- `http://example.com/web/dashboard` → web-service

### Host-Based Routing

Routes different hostnames to different services:

```yaml
rules:
- host: api.example.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: api-service
          port:
            number: 80
- host: web.example.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: web-service
          port:
            number: 80
```

**Example:**
- `http://api.example.com` → api-service
- `http://web.example.com` → web-service

### Path Types

**Exact:**
- Matches exact path
- `/api` matches only `/api`

**Prefix:**
- Matches path prefix
- `/api` matches `/api`, `/api/users`, `/api/v1`
- Longest match wins

**ImplementationSpecific:**
- Depends on Ingress Controller
- Use with caution

## 🔒 TLS/SSL Configuration

### Create TLS Secret

```bash
# Create TLS certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=example.com"

# Create Kubernetes secret
kubectl create secret tls example-tls \
  --cert=tls.crt \
  --key=tls.key
```

### Use TLS in Ingress

```yaml
spec:
  tls:
  - hosts:
    - example.com
    secretName: example-tls
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

## 📚 Example Files Explained

### Basic Ingress (`basic-ingress.yaml`)

**What it does:**
- Simple host-based routing
- Routes all traffic to one service
- Perfect for learning

**Try it:**
```bash
kubectl apply -f basic-ingress.yaml
kubectl get ingress
```

### Path-Based Ingress (`path-based-ingress.yaml`)

**What it does:**
- Routes different paths to different services
- `/api` → api-service
- `/web` → web-service

**Try it:**
```bash
kubectl apply -f path-based-ingress.yaml
curl http://example.com/api
curl http://example.com/web
```

### Host-Based Ingress (`host-based-ingress.yaml`)

**What it does:**
- Routes different hostnames to different services
- `api.example.com` → api-service
- `web.example.com` → web-service

**Try it:**
```bash
kubectl apply -f host-based-ingress.yaml
# Add to /etc/hosts or use DNS
curl http://api.example.com
curl http://web.example.com
```

### TLS Ingress (`tls-ingress.yaml`)

**What it does:**
- HTTPS with TLS termination
- SSL certificate management

**Try it:**
```bash
# Create TLS secret first
kubectl create secret tls example-tls --cert=tls.crt --key=tls.key
kubectl apply -f tls-ingress.yaml
curl https://example.com
```

## 🎨 Ingress Annotations

Annotations provide controller-specific configuration:

### NGINX Ingress Annotations

```yaml
metadata:
  annotations:
    # Rewrite target
    nginx.ingress.kubernetes.io/rewrite-target: /
    
    # SSL redirect
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "100"
    
    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    
    # Authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
```

### Common Annotations

**Rewrite:**
```yaml
nginx.ingress.kubernetes.io/rewrite-target: /$2
```
Rewrites `/api/v1/users` to `/users` if path is `/api/v1(/|$)(.*)`

**SSL Redirect:**
```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "true"
```
Redirects HTTP to HTTPS

**Rate Limiting:**
```yaml
nginx.ingress.kubernetes.io/limit-rps: "100"
```
Limits requests per second

## 🎓 Common Use Cases

### Use Case 1: Multiple Services, One Domain

**Scenario:** You have API and web frontend, want one domain.

**Solution:**
```yaml
rules:
- host: example.com
  http:
    paths:
    - path: /api
      pathType: Prefix
      backend:
        service:
          name: api-service
    - path: /
      pathType: Prefix
      backend:
        service:
          name: web-service
```

### Use Case 2: Multiple Domains

**Scenario:** Different domains for different services.

**Solution:**
```yaml
rules:
- host: api.example.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: api-service
- host: web.example.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: web-service
```

### Use Case 3: SSL/TLS Termination

**Scenario:** HTTPS for your application.

**Solution:**
```yaml
tls:
- hosts:
  - example.com
  secretName: example-tls
rules:
- host: example.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: web-service
```

## 🔧 Important Fields Explained

### `spec.rules`
```yaml
rules:
- host: example.com
  http:
    paths: [...]
```
- List of routing rules
- Each rule can have host and paths

### `spec.rules[].host`
```yaml
host: example.com
```
- Hostname to match
- Optional (empty = all hosts)
- Wildcards supported by some controllers

### `spec.rules[].http.paths`
```yaml
paths:
- path: /api
  pathType: Prefix
  backend: {...}
```
- List of path rules
- Evaluated in order
- First match wins

### `spec.rules[].http.paths[].pathType`
```yaml
pathType: Prefix
```
- **Exact**: Exact match
- **Prefix**: Prefix match
- **ImplementationSpecific**: Controller-specific

### `spec.rules[].http.paths[].backend`
```yaml
backend:
  service:
    name: web-service
    port:
      number: 80
```
- Service to route to
- Must exist in same namespace

### `spec.tls`
```yaml
tls:
- hosts:
  - example.com
  secretName: example-tls
```
- TLS configuration
- Secret must contain `tls.crt` and `tls.key`

## 🐛 Troubleshooting

### Problem: Ingress not getting IP

```bash
# Check ingress controller
kubectl get pods -n ingress-nginx

# Check ingress status
kubectl describe ingress <ingress-name>

# Check service for ingress controller
kubectl get svc -n ingress-nginx
```

**Solution:**
- Ensure Ingress Controller is running
- Check LoadBalancer service (if cloud)
- For minikube: `minikube tunnel`

### Problem: 404 Not Found

```bash
# Check ingress rules
kubectl get ingress <ingress-name> -o yaml

# Check backend service
kubectl get svc <service-name>

# Check service endpoints
kubectl get endpoints <service-name>
```

**Solution:**
- Verify service exists and has endpoints
- Check path matches correctly
- Verify pathType is correct

### Problem: TLS not working

```bash
# Check TLS secret
kubectl get secret <secret-name>

# Check secret contents
kubectl describe secret <secret-name>
```

**Solution:**
- Verify secret exists
- Check secret has `tls.crt` and `tls.key`
- Ensure hosts match in TLS and rules

## 💡 Best Practices

1. **Use Ingress Class**
   - Specify ingress class explicitly
   - Allows multiple controllers

2. **Use Path Prefix Correctly**
   - Understand Exact vs Prefix
   - Order matters (longest first)

3. **Enable TLS**
   - Use TLS for production
   - Consider cert-manager for automatic certs

4. **Use Annotations Wisely**
   - Controller-specific
   - Document why you use them

5. **Monitor Ingress**
   - Watch for 5xx errors
   - Monitor latency
   - Check controller logs

6. **Resource Limits**
   - Set limits on Ingress Controller
   - Monitor resource usage

## 📋 Quick Reference Commands

```bash
# Create ingress
kubectl apply -f basic-ingress.yaml

# View ingress
kubectl get ingress
kubectl describe ingress <ingress-name>

# View ingress controller
kubectl get pods -n ingress-nginx

# Delete ingress
kubectl delete ingress <ingress-name>

# Test ingress
curl -H "Host: example.com" http://<ingress-ip>
```

## 🎯 Practice Exercises

1. **Create basic ingress** and route to a service
2. **Create path-based routing** for multiple services
3. **Create host-based routing** for multiple domains
4. **Add TLS** to your ingress
5. **Test different path types** (Exact vs Prefix)

## 📖 Next Steps

Now that you understand Ingress, learn about:
- **Ingress Controllers**: Different controller options
- **cert-manager**: Automatic TLS certificate management
- **Service Mesh**: Advanced traffic management (Istio, Linkerd)


# Services - Complete Tutorial

## 📖 What You'll Learn

By the end of this tutorial, you will understand:
- What Services are and why they're essential
- The four types of Services and when to use each
- How service discovery works in Kubernetes
- How to expose your applications internally and externally
- How load balancing works with Services

## 🎯 What is a Service?

A **Service** is a stable network endpoint that provides access to your pods. Think of it as a "phone number" that always works, even when individual pods come and go.

### Real-World Analogy

Imagine a restaurant:
- **Pods** = Individual waiters (they come and go)
- **Service** = The host desk (always there, always the same)
- **Customers** = Other applications (they call the host desk, not individual waiters)
- The host desk routes customers to available waiters

### The Problem Services Solve

**Without Services:**
- Pods have IP addresses that change when they restart
- You'd need to update IP addresses everywhere
- No way to load balance across multiple pods
- No stable endpoint for other applications

**With Services:**
- Stable IP address and DNS name
- Automatic load balancing
- Pods can be replaced without breaking connections
- Simple service discovery

## 🔑 Key Features

- **Service Discovery**: Stable DNS name for your application
- **Load Balancing**: Distributes traffic across multiple pods
- **Abstraction**: Decouples frontend from backend pods
- **Multiple Types**: Different ways to expose your service

## 🏗️ How It Works (Step by Step)

### Step 1: You Have Pods Running

```bash
# You have a deployment with pods
kubectl get pods -l app=nginx
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-xxx-abc            1/1     Running   0          5m
# nginx-deployment-xxx-def            1/1     Running   0          5m
# nginx-deployment-xxx-ghi            1/1     Running   0          5m
```

Each pod has its own IP address (e.g., 10.244.1.5, 10.244.2.3, 10.244.3.7)

### Step 2: You Create a Service

```bash
kubectl apply -f clusterip-service.yaml
```

**What happens:**
1. Service gets a stable ClusterIP (e.g., 10.96.0.1)
2. Service uses label selector to find matching pods
3. Service creates endpoints pointing to pod IPs
4. kube-proxy sets up routing rules

### Step 3: Traffic Routing

When traffic arrives at the Service IP:
1. kube-proxy intercepts the traffic
2. Load balances across all pod IPs
3. Routes to a healthy pod
4. If a pod dies, it's automatically removed from the pool

## 📝 Tutorial: Your First Service

### Step 1: Create a Deployment

First, let's create pods to expose:

```bash
# Create a deployment
kubectl apply -f ../deployments/basic-deployment.yaml

# Verify pods are running
kubectl get pods -l app=nginx
```

### Step 2: Create a ClusterIP Service

```bash
# Apply the service
kubectl apply -f clusterip-service.yaml

# View the service
kubectl get service nginx-service
```

**Output:**
```
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   10.96.0.1       <none>        80/TCP    10s
```

**Understanding the output:**
- **CLUSTER-IP**: The stable IP address (10.96.0.1)
- **PORT(S)**: Port 80 is exposed
- **EXTERNAL-IP**: `<none>` means it's only internal

### Step 3: Verify Service Endpoints

```bash
# Check which pods the service is targeting
kubectl get endpoints nginx-service

# You should see all your pod IPs:
# NAME            ENDPOINTS                                          AGE
# nginx-service   10.244.1.5:80,10.244.2.3:80,10.244.3.7:80        15s
```

### Step 4: Test Service Discovery

```bash
# Create a test pod to access the service
kubectl run test-pod --image=busybox --rm -it --restart=Never -- sh

# Inside the pod, test DNS resolution
nslookup nginx-service

# Test HTTP access
wget -qO- http://nginx-service
```

**What you'll see:**
- DNS resolves `nginx-service` to the ClusterIP
- HTTP requests are load-balanced across pods

## 🌐 Service Types Explained

### 1. ClusterIP (Default) - Internal Only

**What it does:**
- Creates a stable IP inside the cluster
- Only accessible from within the cluster
- Perfect for internal communication

**When to use:**
- Backend services (databases, APIs)
- Internal microservices
- Services that shouldn't be exposed externally

**Example:**
```bash
kubectl apply -f clusterip-service.yaml
```

**Access from:**
- Other pods in the cluster
- Via DNS: `http://nginx-service` or `http://nginx-service.default.svc.cluster.local`

### 2. NodePort - External Access via Node IP

**What it does:**
- Exposes service on each node's IP at a static port (30000-32767)
- Accessible from outside the cluster
- Creates a ClusterIP automatically

**When to use:**
- Development and testing
- Simple external access
- When you don't have a cloud load balancer

**Example:**
```bash
kubectl apply -f nodeport-service.yaml
```

**Access from:**
- Outside cluster: `http://<any-node-ip>:30080`
- Inside cluster: `http://nginx-nodeport` (works like ClusterIP)

**Important:**
- Port range: 30000-32767
- If you don't specify `nodePort`, Kubernetes assigns one randomly

### 3. LoadBalancer - Cloud Load Balancer

**What it does:**
- Creates an external load balancer (cloud provider)
- Automatically creates NodePort and ClusterIP
- Gets an external IP address

**When to use:**
- Production environments
- Cloud providers (AWS, GCP, Azure)
- When you need a proper load balancer

**Example:**
```bash
kubectl apply -f loadbalancer-service.yaml
```

**Access from:**
- External IP provided by cloud provider
- Example: `http://35.123.45.67`

**Note:**
- Requires cloud provider support
- In local clusters (minikube, kind), may not work
- In minikube: `minikube service nginx-loadbalancer` to get URL

### 4. ExternalName - External Service Alias

**What it does:**
- Maps service to external DNS name
- No proxying, just DNS CNAME
- Useful for accessing external services

**When to use:**
- Accessing external databases
- External APIs
- Services outside the cluster

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
```

**Access:**
- Pods can access `external-db` as if it's internal
- DNS resolves to `database.example.com`

## 📚 Example Files Explained

### ClusterIP Service (`clusterip-service.yaml`)

**What it does:**
- Creates an internal service
- Selects pods with label `app=nginx`
- Exposes port 80

**Try it:**
```bash
kubectl apply -f clusterip-service.yaml
kubectl get svc nginx-service
kubectl get endpoints nginx-service
```

### NodePort Service (`nodeport-service.yaml`)

**What it does:**
- Exposes service on node port 30080
- Accessible from outside cluster
- Also works internally like ClusterIP

**Try it:**
```bash
kubectl apply -f nodeport-service.yaml
# Get node IP
kubectl get nodes -o wide
# Access: http://<node-ip>:30080
```

### LoadBalancer Service (`loadbalancer-service.yaml`)

**What it does:**
- Requests external load balancer
- Gets external IP (in cloud environments)

**Try it:**
```bash
kubectl apply -f loadbalancer-service.yaml
kubectl get svc nginx-loadbalancer
# Wait for EXTERNAL-IP to be assigned
```

## 🔍 Understanding Service Discovery

### DNS Resolution

Services are automatically registered in Kubernetes DNS:

**Full DNS name:**
```
<service-name>.<namespace>.svc.cluster.local
```

**Short names:**
- Same namespace: `<service-name>`
- Different namespace: `<service-name>.<namespace>`

**Examples:**
```bash
# From default namespace
curl http://nginx-service
curl http://nginx-service.default.svc.cluster.local

# From another namespace
curl http://nginx-service.default
```

### How to Test DNS

```bash
# Create a test pod
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup nginx-service

# You'll see:
# Server:    10.96.0.10
# Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
# Name:      nginx-service
# Address 1: 10.96.0.1 nginx-service.default.svc.cluster.local
```

## ⚖️ Load Balancing

### How Load Balancing Works

1. **Round Robin** (default): Requests distributed evenly
2. **Session Affinity**: Can be enabled for sticky sessions

**Enable Session Affinity:**
```yaml
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
```

### Testing Load Balancing

```bash
# Create multiple requests
for i in {1..10}; do
  kubectl run test-$i --image=busybox --rm -i --restart=Never -- wget -qO- http://nginx-service
done

# Each request may hit a different pod
```

## 🎓 Common Use Cases

### Use Case 1: Expose Web Application

**Scenario:** You have a web app deployment and want to expose it.

**Solution:**
```bash
# Create ClusterIP for internal access
kubectl apply -f clusterip-service.yaml

# Or NodePort for external access
kubectl apply -f nodeport-service.yaml
```

### Use Case 2: Microservices Communication

**Scenario:** Frontend needs to call backend API.

**Solution:**
```yaml
# Backend service (ClusterIP)
apiVersion: v1
kind: Service
metadata:
  name: backend-api
spec:
  selector:
    app: backend
  ports:
  - port: 8080
```

Frontend can access: `http://backend-api:8080`

### Use Case 3: Database Connection

**Scenario:** Application needs to connect to database.

**Solution:**
```yaml
# Database service (ClusterIP - internal only)
apiVersion: v1
kind: Service
metadata:
  name: postgres-db
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
```

Application connects to: `postgres-db:5432`

## 🔧 Important Fields Explained

### `selector`
```yaml
selector:
  app: nginx
```
- **Purpose**: Selects which pods are part of this service
- **Must match**: Pod labels
- **Important**: If no pods match, service has no endpoints

### `ports`
```yaml
ports:
- port: 80        # Port exposed by service
  targetPort: 80  # Port on the pod
  protocol: TCP   # TCP or UDP
  name: http      # Optional name
```
- **port**: Port the service listens on
- **targetPort**: Port on the pod (defaults to `port` if not specified)
- **protocol**: TCP (default) or UDP

### `type`
```yaml
type: ClusterIP  # or NodePort, LoadBalancer, ExternalName
```
- Determines how the service is exposed
- Default: ClusterIP

## 🐛 Troubleshooting

### Problem: Service has no endpoints

```bash
# Check service endpoints
kubectl get endpoints <service-name>

# If empty, check:
# 1. Do pods have matching labels?
kubectl get pods --show-labels

# 2. Are pods running?
kubectl get pods

# 3. Check service selector
kubectl describe service <service-name> | grep Selector
```

### Problem: Can't access service

```bash
# Check service exists
kubectl get svc <service-name>

# Check endpoints
kubectl get endpoints <service-name>

# Test from a pod
kubectl run test --image=busybox --rm -it --restart=Never -- wget -qO- http://<service-name>
```

### Problem: NodePort not accessible

```bash
# Check service type
kubectl get svc <service-name>

# Check node port
kubectl get svc <service-name> -o yaml | grep nodePort

# Check firewall rules (if applicable)
# Verify you're using correct node IP and port
```

## 💡 Best Practices

1. **Use ClusterIP for internal services**
   - More secure
   - Better performance

2. **Use descriptive service names**
   - Good: `user-api-service`, `payment-service`
   - Bad: `svc1`, `test`

3. **Match service and deployment names**
   - Makes it easier to understand relationships
   - Example: `nginx-deployment` → `nginx-service`

4. **Use NodePort for development only**
   - Not suitable for production
   - Use LoadBalancer or Ingress for production

5. **Always set resource labels**
   - Makes service selection easier
   - Better organization

## 📋 Quick Reference Commands

```bash
# Create
kubectl apply -f clusterip-service.yaml

# View
kubectl get services
kubectl get svc  # Short form
kubectl describe service <service-name>

# Check endpoints
kubectl get endpoints <service-name>

# Expose deployment as service
kubectl expose deployment nginx-deployment --port=80 --type=NodePort

# Port forward (for testing)
kubectl port-forward service/nginx-service 8080:80

# Delete
kubectl delete service <service-name>
```

## 🎯 Practice Exercises

1. **Create a ClusterIP service** and verify it works internally
2. **Create a NodePort service** and access it from outside
3. **Test service discovery** using DNS from another pod
4. **Verify load balancing** by making multiple requests
5. **Check endpoints** and see how they change when pods restart

## 📖 Next Steps

Now that you understand Services, learn about:
- **Ingress**: More advanced external access (better than NodePort)
- **ConfigMaps**: Configuration for your services
- **Deployments**: What services expose

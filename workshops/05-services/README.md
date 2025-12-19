# Workshop 5: Services and Networking

## Overview

This workshop covers Kubernetes networking, service types, DNS, network policies, and ingress controllers.

## Kubernetes Networking Model

### Core Principles

1. **Pod-to-Pod Communication**: All pods can communicate without NAT
2. **Pod-to-Service Communication**: Services provide stable endpoints
3. **External-to-Service Communication**: Expose services externally
4. **Network Policies**: Control traffic between pods

### Pod Networking

Each pod gets its own IP address:

```bash
# Create test pods
kubectl run pod1 --image=nginx --labels="app=pod1"
kubectl run pod2 --image=nginx --labels="app=pod2"

# Get pod IPs
kubectl get pods -o wide

# Test connectivity
kubectl exec pod1 -- curl <pod2-ip>
```

## Service Types

### 1. ClusterIP (Default)

Internal-only service:

```yaml
# clusterip-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
```

```bash
# Create service
kubectl apply -f clusterip-service.yaml

# View service
kubectl get svc backend-service

# Get service details
kubectl describe svc backend-service

# Test from within cluster
kubectl run test --image=busybox -it --rm -- wget -O- backend-service
```

### 2. NodePort

Exposes service on each node's IP:

```yaml
# nodeport-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-nodeport
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # Optional: 30000-32767
    protocol: TCP
```

```bash
# Access service
curl http://<node-ip>:30080

# For Minikube
minikube service frontend-nodeport

# For Docker Desktop
curl http://localhost:30080
```

### 3. LoadBalancer

Creates external load balancer (cloud providers):

```yaml
# loadbalancer-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
```

```bash
# Get external IP
kubectl get svc web-loadbalancer

# For Minikube (requires tunnel)
minikube tunnel
```

### 4. ExternalName

Maps service to external DNS:

```yaml
# externalname-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  type: ExternalName
  externalName: api.example.com
```

## Service Discovery

### DNS-Based Discovery

Kubernetes provides DNS for all services:

```bash
# Service DNS format:
# <service-name>.<namespace>.svc.cluster.local

# Within same namespace
curl http://backend-service

# Cross-namespace
curl http://backend-service.default.svc.cluster.local

# Short form
curl http://backend-service.default
```

### Environment Variables

```bash
# Kubernetes automatically creates env vars for services
# Format: <SERVICE_NAME>_SERVICE_HOST
#         <SERVICE_NAME>_SERVICE_PORT

kubectl exec <pod> -- env | grep SERVICE
```

## Headless Services

For direct pod access without load balancing:

```yaml
# headless-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: database-headless
spec:
  clusterIP: None  # Makes it headless
  selector:
    app: database
  ports:
  - port: 3306
```

```bash
# DNS returns all pod IPs
nslookup database-headless.default.svc.cluster.local

# Useful for:
# - StatefulSets
# - Database clusters
# - Custom service discovery
```

## Session Affinity

Sticky sessions for stateful applications:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sticky-service
spec:
  selector:
    app: webapp
  sessionAffinity: ClientIP  # or None
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800  # 3 hours
  ports:
  - port: 80
```

## Ingress

### What is Ingress?

Ingress manages external HTTP/HTTPS access to services:

- HTTP/HTTPS routing
- SSL/TLS termination
- Name-based virtual hosting
- Path-based routing

### Install Ingress Controller

#### Nginx Ingress Controller

```bash
# For Minikube
minikube addons enable ingress

# For Kind
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# For general use
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Verify installation
kubectl get pods -n ingress-nginx
```

### Basic Ingress

```yaml
# basic-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: basic-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.local
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

```bash
# Add to /etc/hosts
echo "<minikube-ip> myapp.local" | sudo tee -a /etc/hosts

# Or get ingress IP
kubectl get ingress

# Access application
curl http://myapp.local
```

### Path-Based Routing

```yaml
# path-based-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 3000
```

### Name-Based Virtual Hosting

```yaml
# vhost-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vhost-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: api.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
  - host: www.myapp.com
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

### TLS/HTTPS Ingress

```bash
# Create self-signed certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=myapp.local/O=myapp"

# Create secret
kubectl create secret tls myapp-tls \
  --cert=tls.crt \
  --key=tls.key
```

```yaml
# tls-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.local
    secretName: myapp-tls
  rules:
  - host: myapp.local
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

## Network Policies

Control traffic flow between pods:

### Default Deny All

```yaml
# deny-all-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### Allow Specific Traffic

```yaml
# allow-frontend-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Allow Namespace Traffic

```yaml
# namespace-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-namespace
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: production
    ports:
    - protocol: TCP
      port: 3306
```

### Egress Policy

```yaml
# egress-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: egress-policy
spec:
  podSelector:
    matchLabels:
      app: webapp
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 3306
  - to:  # Allow DNS
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

## Complete Example: Multi-Tier Application

```yaml
# multi-tier-app.yaml
---
# Frontend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      tier: web
  template:
    metadata:
      labels:
        app: frontend
        tier: web
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
---
# Frontend Service
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - port: 80
  type: ClusterIP
---
# Backend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
      tier: api
  template:
    metadata:
      labels:
        app: backend
        tier: api
    spec:
      containers:
      - name: api
        image: hashicorp/http-echo
        args:
        - -text=API Response
        ports:
        - containerPort: 5678
---
# Backend Service
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 5678
  type: ClusterIP
---
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
```

## Hands-On Exercises

### Exercise 1: Service Types

```bash
# Create deployment
kubectl create deployment web --image=nginx:alpine --replicas=3

# Expose as ClusterIP
kubectl expose deployment web --port=80 --type=ClusterIP

# Test internally
kubectl run test --image=busybox -it --rm -- wget -O- http://web

# Change to NodePort
kubectl delete svc web
kubectl expose deployment web --port=80 --type=NodePort

# Access externally
kubectl get svc web
```

### Exercise 2: Ingress Setup

```bash
# Enable ingress (Minikube)
minikube addons enable ingress

# Create deployment and service
kubectl create deployment nginx --image=nginx:alpine
kubectl expose deployment nginx --port=80

# Create ingress
kubectl create ingress nginx --rule="nginx.local/*=nginx:80"

# Add to hosts file
echo "$(minikube ip) nginx.local" | sudo tee -a /etc/hosts

# Test
curl http://nginx.local
```

### Exercise 3: Network Policies

```bash
# Create two deployments
kubectl create deployment frontend --image=nginx:alpine
kubectl create deployment backend --image=nginx:alpine

# Create services
kubectl expose deployment frontend --port=80
kubectl expose deployment backend --port=80

# Test connectivity (should work)
kubectl exec -it deployment/frontend -- curl backend

# Apply deny-all policy
kubectl apply -f deny-all-policy.yaml

# Test again (should fail)
kubectl exec -it deployment/frontend -- curl backend

# Apply allow policy
kubectl apply -f allow-frontend-policy.yaml

# Label frontend pods
kubectl label deployment frontend app=frontend
kubectl label deployment backend app=backend

# Test (should work)
kubectl exec -it deployment/frontend -- curl backend
```

## Troubleshooting

### Service Not Accessible

```bash
# Check service
kubectl describe svc <service-name>

# Check endpoints
kubectl get endpoints <service-name>

# Verify pod labels
kubectl get pods --show-labels

# Test DNS
kubectl run test --image=busybox -it --rm -- nslookup <service-name>
```

### Ingress Not Working

```bash
# Check ingress controller
kubectl get pods -n ingress-nginx

# Check ingress
kubectl describe ingress <ingress-name>

# Check ingress logs
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller

# Verify DNS
host myapp.local
```

### Network Policy Issues

```bash
# Check if network plugin supports policies
kubectl get pods -n kube-system

# Describe policy
kubectl describe networkpolicy <policy-name>

# Check pod labels
kubectl get pods --show-labels
```

## Next Steps

1. **[Workshop 6: ConfigMaps and Secrets](../06-configuration/)** - Configuration management
2. **[Workshop 7: Persistent Storage](../07-storage/)** - Volumes and storage
3. Implement service mesh (Istio/Linkerd)
4. Explore advanced ingress features

---

**Next**: [Workshop 6: ConfigMaps and Secrets →](../06-configuration/)
# Workshop 3: Your First Kubernetes Application

## Overview

In this workshop, you'll deploy your first application to Kubernetes. We'll cover creating pods, deployments, services, and accessing your application.

## Prerequisites

- Kubernetes cluster running (from Workshop 2)
- kubectl installed and configured
- Basic understanding of containers

## The Application

We'll deploy a simple nginx web server that serves a custom HTML page.

## Part 1: Understanding Pods

### What is a Pod?

A Pod is the smallest deployable unit in Kubernetes. It can contain one or more containers that share:
- Network namespace (same IP address)
- Storage volumes
- Configuration

### Create Your First Pod

```yaml
# nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: development
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
      name: http
```

### Deploy the Pod

```bash
# Create the pod
kubectl apply -f nginx-pod.yaml

# Check pod status
kubectl get pods

# Watch pod creation
kubectl get pods -w

# Detailed pod information
kubectl describe pod nginx-pod

# View pod logs
kubectl logs nginx-pod

# Follow logs in real-time
kubectl logs -f nginx-pod
```

### Access the Pod

```bash
# Port forward to access locally
kubectl port-forward pod/nginx-pod 8080:80

# Open in browser: http://localhost:8080
# Or use curl
curl http://localhost:8080
```

### Execute Commands in Pod

```bash
# Open shell in pod
kubectl exec -it nginx-pod -- /bin/bash

# Inside the pod:
root@nginx-pod:/# cat /etc/nginx/nginx.conf
root@nginx-pod:/# curl localhost
root@nginx-pod:/# exit

# Run single command
kubectl exec nginx-pod -- ls -la /usr/share/nginx/html
```

### Delete the Pod

```bash
kubectl delete pod nginx-pod

# Or delete using file
kubectl delete -f nginx-pod.yaml
```

## Part 2: Deployments - Production-Ready Apps

### Why Deployments?

Pods alone have limitations:
- No automatic restart if they fail
- No scaling capabilities
- No rolling updates
- No rollback mechanism

**Deployments solve all these problems!**

### Create a Deployment

```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Deploy the Application

```bash
# Create deployment
kubectl apply -f nginx-deployment.yaml

# Check deployment status
kubectl get deployments

# Check replica sets
kubectl get replicasets

# Check pods created by deployment
kubectl get pods -l app=nginx

# Describe deployment
kubectl describe deployment nginx-deployment
```

### Scale the Deployment

```bash
# Scale to 5 replicas using kubectl
kubectl scale deployment nginx-deployment --replicas=5

# Verify scaling
kubectl get pods -l app=nginx

# Scale down to 2
kubectl scale deployment nginx-deployment --replicas=2

# Or edit the deployment file and reapply
kubectl edit deployment nginx-deployment
```

### Test Self-Healing

```bash
# List pods
kubectl get pods -l app=nginx

# Delete a pod
kubectl delete pod <pod-name>

# Watch Kubernetes recreate it
kubectl get pods -l app=nginx -w
```

## Part 3: Services - Networking

### Why Services?

Pods are ephemeral and get new IP addresses when recreated. Services provide:
- Stable network endpoint
- Load balancing across pods
- Service discovery

### Service Types

1. **ClusterIP** (default) - Internal cluster access only
2. **NodePort** - Exposes service on each node's IP
3. **LoadBalancer** - Creates external load balancer (cloud)
4. **ExternalName** - Maps to external DNS

### Create a ClusterIP Service

```yaml
# nginx-service-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

```bash
# Create service
kubectl apply -f nginx-service-clusterip.yaml

# Check service
kubectl get services
kubectl get svc  # Short form

# Describe service
kubectl describe service nginx-service

# See endpoints
kubectl get endpoints nginx-service
```

### Test Service Internally

```bash
# Create a test pod
kubectl run test-pod --image=busybox --rm -it --restart=Never -- sh

# Inside the pod:
wget -O- http://nginx-service
exit
```

### Create a NodePort Service

```yaml
# nginx-service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080  # Optional: 30000-32767
  type: NodePort
```

```bash
# Create NodePort service
kubectl apply -f nginx-service-nodeport.yaml

# Get service details
kubectl get svc nginx-nodeport

# Access the service
# Minikube:
minikube service nginx-nodeport

# Docker Desktop:
curl http://localhost:30080

# Kind: 
curl http://localhost:30080
```

### Create a LoadBalancer Service (Cloud)

```yaml
# nginx-service-lb.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

```bash
# Create LoadBalancer service
kubectl apply -f nginx-service-lb.yaml

# Get external IP (may take a few minutes)
kubectl get svc nginx-loadbalancer

# For Minikube, use:
minikube tunnel  # In separate terminal
kubectl get svc nginx-loadbalancer
```

## Part 4: Custom Application

Let's deploy a custom HTML page.

### Create ConfigMap

```yaml
# nginx-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-html
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>My First K8s App</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                display: flex;
                justify-content: center;
                align-items: center;
                height: 100vh;
                margin: 0;
                background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                color: white;
            }
            .container {
                text-align: center;
                padding: 50px;
                background: rgba(255,255,255,0.1);
                border-radius: 20px;
                backdrop-filter: blur(10px);
            }
            h1 { font-size: 3em; margin-bottom: 20px; }
            p { font-size: 1.2em; }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>🚀 Hello Kubernetes!</h1>
            <p>My First Kubernetes Application</p>
            <p>Deployed with ❤️ by $(whoami)</p>
        </div>
    </body>
    </html>
```

### Update Deployment with ConfigMap

```yaml
# nginx-deployment-custom.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-custom
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-custom
  template:
    metadata:
      labels:
        app: nginx-custom
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html
        configMap:
          name: nginx-html
```

### Deploy Custom Application

```bash
# Create ConfigMap
kubectl apply -f nginx-configmap.yaml

# Create Deployment
kubectl apply -f nginx-deployment-custom.yaml

# Create Service
kubectl expose deployment nginx-custom --type=NodePort --port=80

# Access the application
minikube service nginx-custom  # Minikube
# Or
kubectl port-forward deployment/nginx-custom 8080:80
```

## Part 5: Complete Application Stack

Let's deploy a real-world multi-tier application.

### Frontend Deployment

```yaml
# frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
      tier: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
  - port: 80
  type: LoadBalancer
```

### Backend API Deployment

```yaml
# backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
      tier: backend
  template:
    metadata:
      labels:
        app: backend
        tier: backend
    spec:
      containers:
      - name: api
        image: hashicorp/http-echo
        args:
        - "-text=Hello from Backend API"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 5678
  type: ClusterIP
```

### Deploy Complete Stack

```bash
# Deploy everything
kubectl apply -f frontend-deployment.yaml
kubectl apply -f backend-deployment.yaml

# Check everything
kubectl get all

# Check by labels
kubectl get pods -l tier=frontend
kubectl get pods -l tier=backend
```

## Hands-On Exercises

### Exercise 1: Deploy Hello World

```bash
# 1. Create a deployment
kubectl create deployment hello --image=gcr.io/google-samples/hello-app:1.0

# 2. Expose it
kubectl expose deployment hello --type=NodePort --port=8080

# 3. Access it
kubectl get service hello
# Visit the NodePort

# 4. Scale it
kubectl scale deployment hello --replicas=5

# 5. Check pods
kubectl get pods -l app=hello

# 6. Clean up
kubectl delete deployment hello
kubectl delete service hello
```

### Exercise 2: Deploy with Labels

```bash
# Deploy with labels
kubectl create deployment nginx --image=nginx:alpine
kubectl label deployment nginx env=production
kubectl label deployment nginx version=v1.0

# Query by labels
kubectl get deployments -l env=production
kubectl get deployments -l version=v1.0
kubectl get deployments -l env=production,version=v1.0
```

### Exercise 3: Rolling Update

```bash
# Deploy v1
kubectl create deployment app --image=gcr.io/google-samples/hello-app:1.0
kubectl scale deployment app --replicas=5

# Update to v2
kubectl set image deployment/app hello-app=gcr.io/google-samples/hello-app:2.0

# Watch rollout
kubectl rollout status deployment app

# Check rollout history
kubectl rollout history deployment app

# Rollback if needed
kubectl rollout undo deployment app
```

## Useful kubectl Commands

### Get Resources

```bash
# List all resources
kubectl get all
kubectl get all -A  # All namespaces

# Specific resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get nodes

# With labels
kubectl get pods -l app=nginx
kubectl get pods --show-labels

# Wide output
kubectl get pods -o wide

# JSON/YAML output
kubectl get pod <pod-name> -o json
kubectl get pod <pod-name> -o yaml
```

### Describe Resources

```bash
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
kubectl describe service <service-name>
kubectl describe node <node-name>
```

### Logs and Debugging

```bash
# View logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # Follow
kubectl logs <pod-name> --previous  # Previous instance

# Multi-container pod
kubectl logs <pod-name> -c <container-name>

# Execute commands
kubectl exec <pod-name> -- <command>
kubectl exec -it <pod-name> -- /bin/bash
```

### Edit Resources

```bash
# Edit directly
kubectl edit deployment <name>
kubectl edit service <name>

# Apply changes
kubectl apply -f <file.yaml>

# Patch resources
kubectl patch deployment <name> -p '{"spec":{"replicas":5}}'
```

### Delete Resources

```bash
# Delete by name
kubectl delete pod <pod-name>
kubectl delete deployment <deployment-name>

# Delete by file
kubectl delete -f <file.yaml>

# Delete by label
kubectl delete pods -l app=nginx

# Delete all in namespace
kubectl delete all --all -n <namespace>
```

## Best Practices

1. **Always use Deployments** instead of bare Pods
2. **Use labels** for organization and selection
3. **Set resource requests and limits** for containers
4. **Use liveness and readiness probes** for health checks
5. **Never use latest tag** in production
6. **Use namespaces** for isolation
7. **Version your manifests** in Git
8. **Use ConfigMaps and Secrets** for configuration

## Common Errors and Solutions

### ImagePullBackOff

```bash
# Check event details
kubectl describe pod <pod-name>

# Common causes:
# - Wrong image name
# - Image doesn't exist
# - No credentials for private registry
```

### CrashLoopBackOff

```bash
# Check logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Common causes:
# - Application error
# - Wrong command/args
# - Missing dependencies
```

### Service Not Accessible

```bash
# Check service
kubectl get svc
kubectl describe svc <service-name>

# Check endpoints
kubectl get endpoints <service-name>

# Verify labels match
kubectl get pods --show-labels
```

## Cleanup

```bash
# Delete specific resources
kubectl delete deployment nginx-deployment
kubectl delete service nginx-service
kubectl delete configmap nginx-html

# Delete everything with label
kubectl delete all -l app=nginx

# Delete all resources
kubectl delete all --all
```

## Next Steps

Now that you can deploy applications:

1. **[Workshop 4: Pods and Deployments](../04-pods-deployments/)** - Deep dive
2. **[Workshop 5: Services and Networking](../05-services/)** - Advanced networking
3. Experiment with different application types
4. Try deploying your own containerized apps

## Additional Resources

- [Kubernetes Workloads](https://kubernetes.io/docs/concepts/workloads/)
- [Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

**Next**: [Workshop 4: Pods and Deployments Deep Dive →](../04-pods-deployments/)
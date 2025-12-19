# Workshop 4: Pods and Deployments Deep Dive

## Overview

This workshop provides an in-depth look at Pods and Deployments, covering advanced features, patterns, and best practices for production deployments.

## Understanding Pods

### Pod Lifecycle

Pods go through several phases:

1. **Pending** - Pod accepted but not yet running
2. **Running** - Pod bound to node, containers created
3. **Succeeded** - All containers terminated successfully
4. **Failed** - All containers terminated, at least one failed
5. **Unknown** - Cannot determine pod state

```bash
# Watch pod lifecycle
kubectl get pods -w

# Check pod status
kubectl get pod <pod-name> -o jsonpath='{.status.phase}'
```

### Multi-Container Pods

#### Sidecar Pattern

```yaml
# sidecar-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-with-sidecar
spec:
  containers:
  # Main application container
  - name: webapp
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs
      mountPath: /var/log/nginx
  
  # Sidecar container for log processing
  - name: log-processor
    image: busybox:latest
    command: ['sh', '-c']
    args:
    - while true; do
        echo "Processing logs at $(date)";
        cat /logs/access.log 2>/dev/null | tail -5;
        sleep 10;
      done
    volumeMounts:
    - name: logs
      mountPath: /logs
  
  volumes:
  - name: logs
    emptyDir: {}
```

#### Init Containers

```yaml
# init-container-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-with-init
spec:
  # Init containers run before main containers
  initContainers:
  - name: init-db
    image: busybox:latest
    command: ['sh', '-c']
    args:
    - |
      echo "Waiting for database...";
      until nslookup db-service; do
        echo "Database not ready, waiting...";
        sleep 2;
      done;
      echo "Database is ready!";
  
  - name: init-cache
    image: busybox:latest
    command: ['sh', '-c']
    args:
    - |
      echo "Warming up cache...";
      sleep 5;
      echo "Cache ready!";
  
  containers:
  - name: webapp
    image: nginx:alpine
    ports:
    - containerPort: 80
```

### Resource Management

```yaml
# pod-with-resources.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    resources:
      requests:
        memory: "64Mi"    # Minimum memory guaranteed
        cpu: "250m"       # 0.25 CPU cores
      limits:
        memory: "128Mi"   # Maximum memory allowed
        cpu: "500m"       # 0.5 CPU cores
```

**Important Concepts:**

- **Requests**: Guaranteed resources, used for scheduling
- **Limits**: Maximum resources, container throttled/killed if exceeded
- **CPU**: Measured in cores (1000m = 1 core)
- **Memory**: Measured in bytes (Ki, Mi, Gi)

### Health Checks

```yaml
# pod-with-probes.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-probes
spec:
  containers:
  - name: webapp
    image: nginx:alpine
    ports:
    - containerPort: 80
    
    # Liveness Probe - Is the container alive?
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    
    # Readiness Probe - Is the container ready to serve traffic?
    readinessProbe:
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      successThreshold: 1
      failureThreshold: 3
    
    # Startup Probe - Has the application started?
    startupProbe:
      httpGet:
        path: /startup
        port: 80
      initialDelaySeconds: 0
      periodSeconds: 10
      timeoutSeconds: 3
      failureThreshold: 30
```

**Probe Types:**

1. **HTTP Probe**: GET request to endpoint
2. **TCP Probe**: TCP connection check
3. **Exec Probe**: Command execution in container

```yaml
# Different probe types
livenessProbe:
  # HTTP Probe
  httpGet:
    path: /health
    port: 8080

# or TCP Probe
livenessProbe:
  tcpSocket:
    port: 3306

# or Exec Probe
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
```

### Environment Variables

```yaml
# pod-with-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: app
    image: busybox:latest
    command: ['sh', '-c', 'env && sleep 3600']
    env:
    # Direct value
    - name: DATABASE_HOST
      value: "mysql.default.svc.cluster.local"
    
    # From ConfigMap
    - name: APP_CONFIG
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: app.properties
    
    # From Secret
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    
    # From Pod metadata
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
```

## Deployments Deep Dive

### Deployment Strategy

#### Rolling Update (Default)

```yaml
# rolling-update-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-update-app
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods above desired during update
      maxUnavailable: 1  # Max pods unavailable during update
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v1
        ports:
        - containerPort: 8080
```

**Example Update:**

```bash
# Deploy v1
kubectl apply -f rolling-update-deployment.yaml

# Update to v2
kubectl set image deployment/rolling-update-app app=myapp:v2

# Watch rollout
kubectl rollout status deployment/rolling-update-app

# Check rollout history
kubectl rollout history deployment/rolling-update-app

# Rollback to previous version
kubectl rollout undo deployment/rolling-update-app

# Rollback to specific revision
kubectl rollout undo deployment/rolling-update-app --to-revision=2
```

#### Recreate Strategy

```yaml
# recreate-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recreate-app
spec:
  replicas: 3
  strategy:
    type: Recreate  # Kill all pods, then create new ones
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v1
```

### Advanced Deployment Features

#### Blue-Green Deployment

```yaml
# blue-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: app
        image: myapp:v1
---
# green-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: app
        image: myapp:v2
---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: myapp
    version: blue  # Switch to 'green' to cutover
  ports:
  - port: 80
    targetPort: 8080
```

```bash
# Switch from blue to green
kubectl patch service app-service -p '{"spec":{"selector":{"version":"green"}}}'
```

#### Canary Deployment

```yaml
# stable-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-stable
spec:
  replicas: 9  # 90% of traffic
  selector:
    matchLabels:
      app: myapp
      track: stable
  template:
    metadata:
      labels:
        app: myapp
        track: stable
    spec:
      containers:
      - name: app
        image: myapp:v1
---
# canary-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-canary
spec:
  replicas: 1  # 10% of traffic
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp
        track: canary
    spec:
      containers:
      - name: app
        image: myapp:v2
---
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: myapp  # Matches both stable and canary
  ports:
  - port: 80
    targetPort: 8080
```

### Pod Disruption Budgets

```yaml
# pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2  # or: maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
```

PDB ensures minimum number of pods during:
- Voluntary disruptions (node drain, deployments)
- Node maintenance
- Cluster upgrades

### Horizontal Pod Autoscaling

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

```bash
# Create HPA using kubectl
kubectl autoscale deployment myapp --cpu-percent=70 --min=2 --max=10

# View HPA
kubectl get hpa

# Describe HPA
kubectl describe hpa app-hpa
```

## Hands-On Exercises

### Exercise 1: Multi-Container Pod

Create a pod with nginx and a log forwarder:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: log-forwarder
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
    - name: logs
      mountPath: /var/log/nginx
  - name: log-agent
    image: fluent/fluent-bit:latest
    volumeMounts:
    - name: logs
      mountPath: /logs
  volumes:
  - name: logs
    emptyDir: {}
```

### Exercise 2: Init Container Setup

Create a pod that waits for a service:

```bash
# Create the service first
kubectl create service clusterip db-service --tcp=3306:3306

# Deploy pod with init container
kubectl apply -f init-container-pod.yaml

# Watch pod startup
kubectl get pods -w

# Check init container logs
kubectl logs webapp-with-init -c init-db
```

### Exercise 3: Rolling Update Practice

```bash
# Create deployment
kubectl create deployment webapp --image=nginx:1.24 --replicas=5

# Expose it
kubectl expose deployment webapp --port=80 --type=NodePort

# Update image
kubectl set image deployment/webapp nginx=nginx:1.25 --record

# Watch update
kubectl rollout status deployment/webapp

# Check history
kubectl rollout history deployment/webapp

# Rollback
kubectl rollout undo deployment/webapp
```

### Exercise 4: Implement Health Checks

Add health checks to running deployment:

```bash
kubectl edit deployment webapp

# Add liveness and readiness probes
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 5

readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 3
```

## Production Best Practices

### 1. Always Set Resource Limits

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

### 2. Implement Health Checks

Always use liveness and readiness probes.

### 3. Use Pod Disruption Budgets

Protect against voluntary disruptions.

### 4. Never Use :latest Tag

```yaml
# Bad
image: nginx:latest

# Good
image: nginx:1.25.0
```

### 5. Set Pod Priorities

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
---
apiVersion: v1
kind: Pod
metadata:
  name: critical-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: myapp:v1
```

### 6. Use Affinity and Anti-Affinity

```yaml
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - myapp
        topologyKey: kubernetes.io/hostname
```

## Troubleshooting

### Pod Stuck in Pending

```bash
# Check events
kubectl describe pod <pod-name>

# Common causes:
# - Insufficient resources
# - Node selector mismatch
# - PVC not bound
# - Image pull errors

# Check node resources
kubectl describe nodes
```

### CrashLoopBackOff

```bash
# Check logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Check events
kubectl describe pod <pod-name>

# Common causes:
# - Application errors
# - Missing configuration
# - Failed health checks
# - Incorrect command
```

### ImagePullBackOff

```bash
# Check image name
kubectl describe pod <pod-name>

# Test image pull
docker pull <image-name>

# Check image pull secrets
kubectl get secrets
```

## Useful Commands

```bash
# Get deployment details
kubectl get deployment <name> -o yaml

# Scale deployment
kubectl scale deployment <name> --replicas=5

# Update image
kubectl set image deployment/<name> <container>=<new-image>

# Edit deployment
kubectl edit deployment <name>

# Rollout commands
kubectl rollout status deployment/<name>
kubectl rollout pause deployment/<name>
kubectl rollout resume deployment/<name>
kubectl rollout restart deployment/<name>
kubectl rollout undo deployment/<name>

# Pod operations
kubectl logs <pod-name> -f
kubectl exec -it <pod-name> -- /bin/bash
kubectl port-forward <pod-name> 8080:80
kubectl top pod <pod-name>
```

## Next Steps

1. **[Workshop 5: Services and Networking](../05-services/)** - Learn advanced networking
2. **[Workshop 6: ConfigMaps and Secrets](../06-configuration/)** - Configuration management
3. Practice deployment strategies
4. Implement autoscaling

---

**Next**: [Workshop 5: Services and Networking →](../05-services/)
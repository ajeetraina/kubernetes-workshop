# Workshop 1: Introduction to Kubernetes

## What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. Originally developed by Google and now maintained by the Cloud Native Computing Foundation (CNCF), Kubernetes has become the de facto standard for container orchestration.

## Why Kubernetes?

### The Problem Kubernetes Solves

Imagine you have:
- 100 microservices running in containers
- Each service needs to scale independently
- Services need to communicate with each other
- Failed containers need to restart automatically
- Traffic needs to be load balanced
- Updates need to happen with zero downtime

Doing this manually is impossible at scale. That's where Kubernetes comes in.

### Key Benefits

1. **Automated Deployment**: Deploy applications consistently across environments
2. **Self-Healing**: Automatically restart failed containers
3. **Horizontal Scaling**: Scale applications up or down based on demand
4. **Service Discovery**: Containers can find and communicate with each other
5. **Load Balancing**: Distribute traffic across multiple containers
6. **Rolling Updates**: Update applications without downtime
7. **Resource Optimization**: Efficiently pack containers onto nodes
8. **Multi-cloud**: Run on any cloud provider or on-premises

## Kubernetes Architecture

### Control Plane Components

The control plane manages the Kubernetes cluster:

#### 1. API Server (`kube-apiserver`)
- Front-end for the Kubernetes control plane
- Exposes the Kubernetes API
- All communication goes through the API server

#### 2. etcd
- Consistent and highly-available key-value store
- Stores all cluster data
- Backing store for all cluster data

#### 3. Scheduler (`kube-scheduler`)
- Watches for newly created Pods
- Assigns Pods to nodes based on resource requirements
- Considers constraints and available resources

#### 4. Controller Manager (`kube-controller-manager`)
- Runs controller processes
- Node controller: Notices when nodes go down
- Replication controller: Maintains correct number of pods
- Endpoints controller: Populates endpoints object
- Service Account controller: Creates default accounts

#### 5. Cloud Controller Manager
- Integrates with cloud provider APIs
- Manages cloud-specific resources
- Routes, load balancers, volumes

### Node Components

Each worker node runs:

#### 1. Kubelet
- Agent that runs on each node
- Ensures containers are running in pods
- Communicates with API server
- Reports node and pod status

#### 2. Kube-proxy
- Network proxy on each node
- Maintains network rules
- Handles service abstraction
- Enables communication between pods

#### 3. Container Runtime
- Software responsible for running containers
- Examples: Docker, containerd, CRI-O
- Pulls images and runs containers

## Core Kubernetes Objects

### 1. Pod
- Smallest deployable unit
- One or more containers
- Shared network and storage
- Ephemeral (temporary)

### 2. Deployment
- Manages a set of identical Pods
- Ensures desired number of Pods are running
- Handles rolling updates
- Maintains replica count

### 3. Service
- Stable network endpoint
- Load balances traffic to Pods
- Provides service discovery
- Types: ClusterIP, NodePort, LoadBalancer

### 4. Namespace
- Virtual cluster within Kubernetes
- Provides isolation
- Resource quota management
- Useful for multi-tenancy

### 5. ConfigMap
- Store configuration data
- Key-value pairs
- Environment variables
- Configuration files

### 6. Secret
- Store sensitive data
- Passwords, tokens, keys
- Base64 encoded
- Can be encrypted at rest

## Kubernetes vs Docker

### Docker
- Container runtime
- Builds and runs containers
- Single host focused
- Docker Compose for multi-container apps

### Kubernetes
- Container orchestrator
- Manages containers at scale
- Multi-host cluster
- Production-grade features

**They work together**: Docker builds containers, Kubernetes orchestrates them.

## Real-World Use Cases

### 1. Microservices Architecture
```
E-commerce Application:
- Frontend Service (React)
- Product Catalog Service
- Shopping Cart Service
- Payment Service
- Order Service
- Inventory Service
```

### 2. CI/CD Pipelines
- Jenkins running in Kubernetes
- Dynamic build agents
- Auto-scaling based on build load
- Infrastructure as Code

### 3. Machine Learning
- Model training jobs
- Model serving (inference)
- GPU resource management
- Batch processing

### 4. Big Data Processing
- Spark clusters
- Kafka message brokers
- Elasticsearch clusters
- Data pipelines

## Kubernetes Distributions

### Production Distributions
- **Google Kubernetes Engine (GKE)** - Google Cloud
- **Amazon Elastic Kubernetes Service (EKS)** - AWS
- **Azure Kubernetes Service (AKS)** - Microsoft Azure
- **Red Hat OpenShift** - Enterprise Kubernetes
- **Rancher** - Multi-cluster management

### Development/Learning
- **Minikube** - Single-node cluster for learning
- **Kind** - Kubernetes in Docker
- **Docker Desktop** - Built-in Kubernetes
- **K3s** - Lightweight Kubernetes
- **MicroK8s** - Minimal Kubernetes

## Kubernetes Ecosystem

### Essential Tools
- **kubectl** - Command-line tool
- **Helm** - Package manager
- **kustomize** - Configuration management
- **Lens** - IDE for Kubernetes
- **k9s** - Terminal UI

### Monitoring & Observability
- **Prometheus** - Metrics collection
- **Grafana** - Visualization
- **Jaeger** - Distributed tracing
- **Fluentd** - Log aggregation
- **ELK Stack** - Logging solution

### Security
- **Falco** - Runtime security
- **Aqua Security** - Container security
- **Twistlock** - Cloud security
- **OPA** - Policy enforcement

### Service Mesh
- **Istio** - Most popular
- **Linkerd** - Lightweight
- **Consul** - HashiCorp solution

## Key Concepts to Remember

### Desired State vs Current State
- You declare desired state
- Kubernetes maintains it
- Self-healing behavior
- Reconciliation loop

### Declarative Configuration
```yaml
# You don't say HOW to do something
# You declare WHAT you want
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3  # I want 3 replicas
```

### Labels and Selectors
```yaml
labels:
  app: frontend
  tier: web
  environment: production

selector:
  matchLabels:
    app: frontend
```

## Common Terminology

- **Cluster**: Set of nodes running containerized applications
- **Node**: Worker machine (VM or physical)
- **Pod**: Group of one or more containers
- **Replica**: Copy of a pod
- **Service**: Networking abstraction
- **Ingress**: HTTP/HTTPS routing
- **Volume**: Storage abstraction
- **Manifest**: YAML/JSON configuration file

## Kubernetes API Versions

```yaml
apiVersion: v1              # Core API
apiVersion: apps/v1         # Deployments, StatefulSets
apiVersion: batch/v1        # Jobs, CronJobs
apiVersion: networking.k8s.io/v1  # Ingress
apiVersion: rbac.authorization.k8s.io/v1  # RBAC
```

## Quick Command Reference

```bash
# Cluster info
kubectl cluster-info
kubectl version
kubectl get nodes

# Working with resources
kubectl get pods
kubectl get services
kubectl get deployments

# Describe resources
kubectl describe pod <pod-name>
kubectl describe node <node-name>

# Logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # Follow logs

# Execute commands
kubectl exec -it <pod-name> -- /bin/bash
```

## Hands-On Exercise

### Exercise 1: Explore Your Cluster

```bash
# Get cluster information
kubectl cluster-info

# List all nodes
kubectl get nodes

# Get detailed node information
kubectl describe node <node-name>

# Check available API resources
kubectl api-resources

# Check API versions
kubectl api-versions
```

### Exercise 2: Understanding Namespaces

```bash
# List all namespaces
kubectl get namespaces

# See pods in all namespaces
kubectl get pods --all-namespaces

# See pods in kube-system namespace
kubectl get pods -n kube-system
```

## Common Patterns

### 1. Sidecar Pattern
```
Main Container + Helper Container
- Main: Application
- Sidecar: Logging agent, proxy
```

### 2. Ambassador Pattern
```
Main Container + Ambassador Container
- Main: Application
- Ambassador: Handles external connections
```

### 3. Adapter Pattern
```
Main Container + Adapter Container
- Main: Application  
- Adapter: Transforms output format
```

## Best Practices

1. **Always use namespaces** for logical separation
2. **Set resource limits** to prevent resource exhaustion
3. **Use labels** consistently for organization
4. **Implement health checks** (liveness/readiness probes)
5. **Use ConfigMaps/Secrets** for configuration
6. **Version your images** - avoid `:latest` tag
7. **Implement RBAC** for security
8. **Monitor everything** with metrics and logs

## Common Pitfalls

1. ❌ Not setting resource requests/limits
2. ❌ Using `latest` image tag
3. ❌ Running containers as root
4. ❌ Not implementing health checks
5. ❌ Storing secrets in ConfigMaps
6. ❌ Not using persistent volumes for stateful apps
7. ❌ Ignoring security contexts
8. ❌ Not implementing RBAC

## Next Steps

Now that you understand Kubernetes fundamentals, you're ready to:

1. **[Workshop 2: Installing Kubernetes](../02-installation/)** - Set up your local environment
2. **[Workshop 3: Your First Application](../03-first-app/)** - Deploy your first app

## Additional Resources

- [Official Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [CNCF Cloud Native Glossary](https://glossary.cncf.io/)
- [Kubernetes Patterns Book](https://k8spatterns.io/)

## Quiz

Test your understanding:

1. What is the smallest deployable unit in Kubernetes?
2. Which component schedules Pods to nodes?
3. What is the difference between a Deployment and a Pod?
4. Name three types of Services in Kubernetes
5. What is the purpose of kubectl?

<details>
<summary>Answers</summary>

1. Pod
2. kube-scheduler
3. A Deployment manages multiple Pods, provides rolling updates, and maintains desired state. A Pod is just a container wrapper.
4. ClusterIP, NodePort, LoadBalancer
5. kubectl is the command-line tool for interacting with Kubernetes clusters
</details>

---

**Next**: [Workshop 2: Installing Kubernetes →](../02-installation/)
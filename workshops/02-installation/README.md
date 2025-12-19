# Workshop 2: Installing Kubernetes

## Overview

This workshop covers multiple ways to install and run Kubernetes for development, learning, and production purposes. We'll explore various tools and their use cases.

## Prerequisites

- Linux, macOS, or Windows operating system
- At least 4GB RAM (8GB recommended)
- 20GB free disk space
- Virtualization enabled in BIOS (for VM-based solutions)
- Docker installed (for some methods)

## Installation Options

### Quick Comparison

| Tool | Best For | Cluster Type | Resource Usage |
|------|----------|--------------|----------------|
| Docker Desktop | Beginners, Mac/Windows | Single-node | Medium |
| Minikube | Learning, Testing | Single-node | Low-Medium |
| Kind | CI/CD, Quick testing | Multi-node | Low |
| K3s | IoT, Edge, Learning | Single/Multi | Very Low |
| MicroK8s | Ubuntu users | Single/Multi | Low |
| Kubeadm | Production, Learning | Multi-node | Medium |

## Method 1: Docker Desktop (Easiest for Beginners)

### What is Docker Desktop?

Docker Desktop includes a standalone Kubernetes server that runs on your local machine. It's the easiest way to get started on Windows and macOS.

### Installation Steps

#### For macOS

```bash
# Download Docker Desktop from:
# https://www.docker.com/products/docker-desktop

# Or using Homebrew
brew install --cask docker
```

#### For Windows

```powershell
# Download from https://www.docker.com/products/docker-desktop
# Or using Chocolatey
choco install docker-desktop
```

#### Enable Kubernetes

1. Open Docker Desktop
2. Go to **Settings** → **Kubernetes**
3. Check **Enable Kubernetes**
4. Click **Apply & Restart**
5. Wait for Kubernetes to start (green indicator)

### Verify Installation

```bash
# Check kubectl is installed
kubectl version --client

# Check cluster is running
kubectl cluster-info

# View nodes
kubectl get nodes

# Expected output:
# NAME             STATUS   ROLES           AGE   VERSION
# docker-desktop   Ready    control-plane   1m    v1.28.2
```

### Resource Configuration

```bash
# Docker Desktop Settings → Resources
# Recommended:
# CPUs: 4
# Memory: 8GB
# Disk: 60GB
```

## Method 2: Minikube (Best for Learning)

### What is Minikube?

Minikube creates a single-node Kubernetes cluster inside a VM or container. It's perfect for learning and development.

### Installation

#### Linux

```bash
# Download Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Install
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Verify
minikube version
```

#### macOS

```bash
# Using Homebrew
brew install minikube

# Or download directly
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

#### Windows

```powershell
# Using Chocolatey
choco install minikube

# Or using winget
winget install Kubernetes.minikube
```

### Start Minikube

```bash
# Start with default settings
minikube start

# Start with specific driver
minikube start --driver=docker
minikube start --driver=virtualbox
minikube start --driver=hyperkit  # macOS
minikube start --driver=hyperv    # Windows

# Start with custom resources
minikube start --cpus=4 --memory=8192 --disk-size=40g

# Start with specific Kubernetes version
minikube start --kubernetes-version=v1.28.0
```

### Useful Minikube Commands

```bash
# Check status
minikube status

# Stop cluster
minikube stop

# Delete cluster
minikube delete

# SSH into node
minikube ssh

# Open dashboard
minikube dashboard

# Get cluster IP
minikube ip

# List addons
minikube addons list

# Enable addon
minikube addons enable ingress
minikube addons enable metrics-server
minikube addons enable dashboard
```

### Verify Minikube

```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

## Method 3: Kind (Kubernetes in Docker)

### What is Kind?

Kind runs Kubernetes clusters using Docker containers as nodes. It's fast, lightweight, and perfect for CI/CD.

### Installation

#### Linux

```bash
# Download Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64

# Install
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

#### macOS

```bash
# Using Homebrew
brew install kind

# Or download directly
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-darwin-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

#### Windows

```powershell
# Using Chocolatey
choco install kind

# Or using curl
curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.20.0/kind-windows-amd64
move kind-windows-amd64.exe C:\Windows\System32\kind.exe
```

### Create Clusters

#### Single-Node Cluster

```bash
# Create basic cluster
kind create cluster --name workshop

# Verify
kubectl cluster-info --context kind-workshop
kubectl get nodes
```

#### Multi-Node Cluster

```yaml
# Create config file: kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
```

```bash
# Create cluster with config
kind create cluster --name multi-node --config kind-config.yaml

# Verify nodes
kubectl get nodes
```

#### Cluster with Ingress

```yaml
# kind-ingress.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```

```bash
kind create cluster --config kind-ingress.yaml

# Install Nginx Ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

### Useful Kind Commands

```bash
# List clusters
kind get clusters

# Delete cluster
kind delete cluster --name workshop

# Load Docker image into cluster
kind load docker-image my-image:tag --name workshop

# Export cluster logs
kind export logs --name workshop
```

## Method 4: K3s (Lightweight Kubernetes)

### What is K3s?

K3s is a lightweight Kubernetes distribution perfect for edge, IoT, and resource-constrained environments.

### Installation

#### Single Server (Master)

```bash
# Install K3s
curl -sfL https://get.k3s.io | sh -

# Check status
sudo systemctl status k3s

# Get kubeconfig
sudo cat /etc/rancher/k3s/k3s.yaml

# Copy to your kubeconfig
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
```

#### With Custom Options

```bash
# Install without Traefik
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable traefik" sh -

# Install with specific version
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.3+k3s1 sh -
```

#### Add Worker Nodes

```bash
# On master, get token
sudo cat /var/lib/rancher/k3s/server/node-token

# On worker node
curl -sfL https://get.k3s.io | K3S_URL=https://master-ip:6443 K3S_TOKEN=<token> sh -
```

### Verify K3s

```bash
kubectl get nodes
kubectl get pods -A
```

### Uninstall K3s

```bash
# On server
/usr/local/bin/k3s-uninstall.sh

# On agent
/usr/local/bin/k3s-agent-uninstall.sh
```

## Method 5: MicroK8s (Canonical/Ubuntu)

### What is MicroK8s?

MicroK8s is a lightweight Kubernetes from Canonical, optimized for Ubuntu.

### Installation (Ubuntu/Linux)

```bash
# Install via snap
sudo snap install microk8s --classic

# Add user to group
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube

# Re-enter session
su - $USER

# Check status
microk8s status --wait-ready
```

### Enable Addons

```bash
# Enable DNS
microk8s enable dns

# Enable dashboard
microk8s enable dashboard

# Enable storage
microk8s enable storage

# Enable ingress
microk8s enable ingress

# Enable registry
microk8s enable registry

# List available addons
microk8s status
```

### Use kubectl with MicroK8s

```bash
# Use microk8s kubectl
microk8s kubectl get nodes

# Or create alias
alias kubectl='microk8s kubectl'

# Or export kubeconfig
microk8s config > ~/.kube/config
```

## Install kubectl (if needed)

### Linux

```bash
# Download latest version
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Install
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify
kubectl version --client
```

### macOS

```bash
# Using Homebrew
brew install kubectl

# Or download directly
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### Windows

```powershell
# Using Chocolatey
choco install kubernetes-cli

# Or using winget
winget install Kubernetes.kubectl
```

## Kubectl Configuration

### Understanding Kubeconfig

```bash
# View current context
kubectl config current-context

# View all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context docker-desktop
kubectl config use-context minikube
kubectl config use-context kind-workshop

# View config file
kubectl config view

# Set namespace
kubectl config set-context --current --namespace=my-namespace
```

### Kubeconfig File Structure

```yaml
# ~/.kube/config
apiVersion: v1
kind: Config
clusters:
- cluster:
    server: https://127.0.0.1:6443
  name: docker-desktop
contexts:
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
current-context: docker-desktop
users:
- name: docker-desktop
  user:
    client-certificate-data: ...
    client-key-data: ...
```

## Hands-On Exercises

### Exercise 1: Install and Verify

```bash
# Choose one method and install
# For example, using Minikube:
minikube start

# Verify installation
kubectl cluster-info
kubectl get nodes
kubectl version

# Check all system pods
kubectl get pods -n kube-system
```

### Exercise 2: Create Multiple Clusters

```bash
# Create first cluster with Kind
kind create cluster --name cluster1

# Create second cluster
kind create cluster --name cluster2

# List contexts
kubectl config get-contexts

# Switch between clusters
kubectl config use-context kind-cluster1
kubectl get nodes

kubectl config use-context kind-cluster2
kubectl get nodes
```

### Exercise 3: Explore System Components

```bash
# View all system pods
kubectl get pods -n kube-system

# Describe control plane components
kubectl describe pod -n kube-system <kube-apiserver-pod>
kubectl describe pod -n kube-system <kube-scheduler-pod>
kubectl describe pod -n kube-system <etcd-pod>

# View logs
kubectl logs -n kube-system <pod-name>
```

## Troubleshooting

### Common Issues

#### Docker Desktop Kubernetes Not Starting

```bash
# Reset Kubernetes
# Docker Desktop → Troubleshoot → Reset Kubernetes Cluster

# Check Docker is running
docker ps

# Restart Docker Desktop
```

#### Minikube Won't Start

```bash
# Delete and recreate
minikube delete
minikube start

# Check driver
minikube start --driver=docker --alsologtostderr

# Check virtualization
egrep -q 'vmx|svm' /proc/cpuinfo && echo yes || echo no
```

#### kubectl Connection Refused

```bash
# Check cluster is running
kubectl cluster-info

# Check correct context
kubectl config current-context

# View and fix kubeconfig
kubectl config view
```

#### Permission Issues

```bash
# Fix kubeconfig permissions
chmod 600 ~/.kube/config

# Add user to docker group (Linux)
sudo usermod -aG docker $USER
newgrp docker
```

## Choosing the Right Tool

### Use Docker Desktop if:
- ✅ You're on macOS or Windows
- ✅ You want the simplest setup
- ✅ You're just learning Kubernetes
- ✅ You want GUI management

### Use Minikube if:
- ✅ You need addons and features
- ✅ You want to learn cluster operations
- ✅ You need to test different K8s versions
- ✅ You want persistent development environment

### Use Kind if:
- ✅ You're building CI/CD pipelines
- ✅ You need quick cluster creation/deletion
- ✅ You want multi-node testing
- ✅ You prefer container-based clusters

### Use K3s if:
- ✅ You have limited resources
- ✅ You're deploying to edge/IoT
- ✅ You want production-ready lightweight K8s
- ✅ You need fast startup times

### Use MicroK8s if:
- ✅ You're on Ubuntu/Linux
- ✅ You want easy addon management
- ✅ You prefer snap packages
- ✅ You need production-ready clusters

## Next Steps

Now that you have Kubernetes installed:

1. **[Workshop 3: Your First Application](../03-first-app/)** - Deploy your first app
2. Practice kubectl commands
3. Explore the Kubernetes dashboard
4. Try different installation methods

## Additional Resources

- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [Kind Documentation](https://kind.sigs.k8s.io/)
- [K3s Documentation](https://docs.k3s.io/)
- [MicroK8s Documentation](https://microk8s.io/docs)

## Quick Reference

```bash
# Start clusters
minikube start
kind create cluster
docker desktop # Enable K8s in GUI

# Check cluster
kubectl cluster-info
kubectl get nodes

# Stop clusters
minikube stop
kind delete cluster

# Switch contexts
kubectl config use-context <context-name>
```

---

**Next**: [Workshop 3: Your First Kubernetes Application →](../03-first-app/)
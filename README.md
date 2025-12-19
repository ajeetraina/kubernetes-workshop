# Kubernetes Workshop - Complete Guide from Beginner to Advanced

[![GitHub stars](https://img.shields.io/github/stars/ajeetraina/kubernetes-workshop)](https://github.com/ajeetraina/kubernetes-workshop/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/ajeetraina/kubernetes-workshop)](https://github.com/ajeetraina/kubernetes-workshop/network)
[![Docker](https://img.shields.io/badge/Docker-Enabled-blue)](https://docker.com)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.28+-326CE5.svg?logo=kubernetes)](https://kubernetes.io)

Welcome to the most comprehensive Kubernetes workshop! This repository contains hands-on labs, tutorials, and real-world examples to help you master Kubernetes from scratch.

## 🎯 What You'll Learn

- **Kubernetes Fundamentals** - Core concepts, architecture, and components
- **Container Orchestration** - Deploy, scale, and manage containerized applications
- **Production Best Practices** - Security, monitoring, networking, and storage
- **Real-World Applications** - Microservices, CI/CD, and cloud-native patterns
- **Advanced Topics** - Custom controllers, operators, and service mesh

## 📚 Workshop Structure

### Part 1: Getting Started with Kubernetes
- [Introduction to Kubernetes](./workshops/01-introduction/)
- [Installing Kubernetes](./workshops/02-installation/)
- [Your First Kubernetes Application](./workshops/03-first-app/)

### Part 2: Core Concepts
- [Pods and Deployments](./workshops/04-pods-deployments/)
- [Services and Networking](./workshops/05-services/)
- [ConfigMaps and Secrets](./workshops/06-configuration/)
- [Persistent Storage](./workshops/07-storage/)

### Part 3: Advanced Kubernetes
- [StatefulSets and DaemonSets](./workshops/08-advanced-workloads/)
- [Ingress and Load Balancing](./workshops/09-ingress/)
- [Helm Package Manager](./workshops/10-helm/)
- [Monitoring and Logging](./workshops/11-observability/)

### Part 4: Production Ready
- [Security Best Practices](./workshops/12-security/)
- [High Availability Setup](./workshops/13-ha-setup/)
- [CI/CD with Kubernetes](./workshops/14-cicd/)
- [Troubleshooting Guide](./workshops/15-troubleshooting/)

## 🚀 Quick Start

### Prerequisites

- Docker installed ([Install Docker](https://docs.docker.com/get-docker/))
- Basic understanding of containers
- A text editor (VS Code recommended)
- Terminal/Command line familiarity

### Option 1: Kubernetes with Docker Desktop

```bash
# Enable Kubernetes in Docker Desktop
# Settings → Kubernetes → Enable Kubernetes
```

### Option 2: Minikube

```bash
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster
minikube start
```

### Option 3: Kind (Kubernetes in Docker)

```bash
# Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create cluster
kind create cluster --name workshop
```

## 🎓 Learning Path

### For Beginners
Start here if you're new to Kubernetes:
1. Complete workshops 01-03 (Introduction, Installation, First App)
2. Practice with workshops 04-07 (Core Concepts)
3. Build a sample application using what you've learned

### For Intermediate Users
If you know Kubernetes basics:
1. Skip to workshops 08-11 (Advanced topics)
2. Explore Helm charts and operators
3. Set up monitoring and logging

### For Advanced Users
Looking for production knowledge:
1. Focus on workshops 12-15 (Production best practices)
2. Explore custom resources and operators
3. Implement full CI/CD pipelines

## 💡 Real-World Examples

This workshop includes practical examples based on production scenarios:

- **E-commerce Microservices** - Deploy a complete shopping application
- **ML Model Serving** - Run machine learning inference at scale
- **Stateful Applications** - Databases and persistent workloads
- **Multi-tenant Clusters** - Namespace isolation and resource quotas

## 🛠️ Tools and Technologies

- **Kubernetes** - Container orchestration platform
- **Docker** - Container runtime
- **Helm** - Kubernetes package manager
- **Kubectl** - Kubernetes command-line tool
- **Prometheus & Grafana** - Monitoring stack
- **Istio** - Service mesh (advanced)

## 📖 Additional Resources

- [Official Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes Blog](https://kubernetes.io/blog/)
- [CNCF Landscape](https://landscape.cncf.io/)
- [Collabnix Community](https://collabnix.com)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes:

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 Workshop Format

Each workshop includes:
- **Theory** - Concepts explained clearly
- **Hands-on Labs** - Step-by-step exercises
- **Real Examples** - Production-ready configurations
- **Best Practices** - Industry standards
- **Troubleshooting** - Common issues and solutions

## 🌟 Why This Workshop?

- ✅ **Beginner Friendly** - Start from zero knowledge
- ✅ **Hands-On** - Learn by doing, not just reading
- ✅ **Production Ready** - Real-world best practices
- ✅ **Continuously Updated** - Latest Kubernetes features
- ✅ **Community Driven** - Active support and contributions

## 📊 Workshop Progress

Track your progress through the workshops:

- [ ] Part 1: Getting Started (Workshops 1-3)
- [ ] Part 2: Core Concepts (Workshops 4-7)
- [ ] Part 3: Advanced Topics (Workshops 8-11)
- [ ] Part 4: Production Ready (Workshops 12-15)

## 🔗 Related Projects

- [Docker Workshop](https://github.com/collabnix/dockerlabs)
- [Helm Charts Repository](https://github.com/ajeetraina/helm-charts)
- [Kubernetes Security](https://github.com/ajeetraina/k8s-security)

## 📧 Contact & Support

- **Author**: Ajeet Singh Raina ([@ajeetraina](https://github.com/ajeetraina))
- **Community**: [Collabnix](https://collabnix.com)
- **Twitter**: [@ajeetsraina](https://twitter.com/ajeetsraina)

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## ⭐ Star This Repository

If you find this workshop helpful, please give it a star! It helps others discover the content.

---

**Ready to start your Kubernetes journey? Begin with [Workshop 1: Introduction to Kubernetes](./workshops/01-introduction/)**
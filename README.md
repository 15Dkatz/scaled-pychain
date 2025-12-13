# Scaled Pychain - Kubernetes Module

![Python Blockchain Logo](./python_blockchain_logo.png)

## Overview

This repository extends the **Docker capstone project** from the Docker course, taking the containerized Python and React blockchain application and deploying it to a Kubernetes cluster. This module demonstrates how to:

- Deploy containerized applications to Kubernetes using Deployments and Services
- Automate development workflows with Skaffold
- Configure networking with Ingress and port forwarding
- Expose applications publicly using Cloudflare Tunnel
- Implement horizontal pod autoscaling based on CPU metrics

**Prerequisites:** This module assumes you've completed the Docker capstone project where you containerized the Python and React blockchain application. If you haven't, please refer to the Docker course materials first.

## Course Context

This module is part of a comprehensive course series:
1. **Python Blockchain Tutorial** - Build the blockchain and cryptocurrency application (optional to do from scratch - completed application cloned in the Docker course)
2. **Docker Capstone** - Containerize the application with Docker and Docker Compose
3. **Kubernetes Module (This Repository)** - Deploy and scale the application on Kubernetes

![Docker Course Logo](./docker_endorsed_course.png)

## Repository Structure

- `backend/` - Python Flask API with blockchain logic (Kubernetes-ready version)
- `frontend/` - React frontend application
- `k8s/` - Kubernetes manifests (Deployments, Services, Ingress, HPA)
- `skaffold.yaml` - Skaffold configuration for automated development workflows
- `docker-compose.yml` - Reference Docker Compose configuration (from Docker capstone)

## Getting Started

### 1. Clone and Setup

```bash
git clone https://github.com/15Dkatz/scaled-pychain.git
cd scaled-pychain
git checkout start-here
```

### 2. Install Required Tools

See the [Module Repository and Installation Instructions](../../new_content/k8s_pychain_showcase/5_module_repo_article_installation_instructions.md) for detailed installation steps for:
- minikube (v1.32.0)
- kubectl (v1.29.0)
- skaffold (v2.9.0)
- cloudflared (2024.3.0)

### 3. Start Your Kubernetes Cluster

```bash
minikube start
```

### 4. Verify Cluster Status

```bash
minikube status
kubectl get nodes
minikube profile list
```

## Command Reference

### Cluster Management (minikube)

**Start the cluster:**
```bash
minikube start
```

**Check cluster status:**
```bash
minikube status
```

**List all minikube profiles:**
```bash
minikube profile list
```

**Get minikube cluster IP:**
```bash
minikube ip
```

**Enable minikube addons:**
```bash
minikube addons enable ingress
minikube addons enable metrics-server
minikube addons list
```

**Start minikube tunnel (for LoadBalancer services):**
```bash
sudo minikube tunnel
```

**Access a service via minikube:**
```bash
minikube service frontend
```

**Clean up minikube (for fresh start):**
```bash
minikube delete --all --purge
```

---

### Building and Loading Docker Images

**Build all application images:**
```bash
docker build -f backend/Dockerfile -t backend:latest .
docker build -f backend/Dockerfile -t backend-peer:latest .
docker build -f backend/Dockerfile -t backend-seed:latest .
docker build -f frontend/Dockerfile -t frontend:latest .
```

**Load images into minikube:**
```bash
minikube image load backend:latest
minikube image load backend-peer:latest
minikube image load backend-seed:latest
minikube image load frontend:latest
```

---

### Kubernetes Secrets and Configuration

**Create a Secret from environment file:**
```bash
kubectl create secret generic backend-secret --from-env-file=backend/config/.env
```

**View secrets:**
```bash
kubectl get secrets
kubectl describe secret backend-secret
```

---

### Deploying with kubectl

**Apply all Kubernetes manifests:**
```bash
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-peer-deployment.yaml
kubectl apply -f k8s/backend-seed-deployment.yaml
kubectl apply -f k8s/frontend-deployment.yaml
```

**Apply Ingress configuration:**
```bash
kubectl apply -f k8s/ingress.yaml
```

**Apply metrics-server configuration:**
```bash
kubectl apply -f k8s/metrics-server.yaml
```

**Apply HPA configuration:**
```bash
kubectl apply -f k8s/backend-peer-hpa.yaml
```

---

### Inspecting Kubernetes Resources

**View all resources:**
```bash
kubectl get deployments
kubectl get pods
kubectl get svc
kubectl get ingress
kubectl get hpa
```

**View resources across all namespaces:**
```bash
kubectl get pods -A
kubectl get svc -A
kubectl get deployment -A
```

**View specific resource details:**
```bash
kubectl get deployment backend-peer
kubectl describe pod <pod-name>
kubectl describe svc frontend
kubectl describe ingress frontend-ingress
```

**View resource YAML:**
```bash
kubectl get deployment backend-peer -o yaml
kubectl get hpa backend-peer -o yaml
```

**Export resource to file:**
```bash
kubectl get deployment metrics-server -n kube-system -o yaml > k8s/metrics-server.yaml
kubectl get hpa backend-peer -o yaml > k8s/backend-peer-hpa.yaml
```

---

### Skaffold Development Workflow

**Initialize Skaffold (first time setup):**
```bash
skaffold init
```

**Start Skaffold development mode (watches files and auto-redeploys):**
```bash
skaffold dev
```

**Check Skaffold version:**
```bash
skaffold version
```

---

### Networking and Port Forwarding

**Port forward to a Service:**
```bash
kubectl port-forward svc/backend 5050:5050
kubectl port-forward svc/frontend 8080:80
```

**Port forward to a specific Pod:**
```bash
kubectl port-forward <pod-name> 5050:5050
```

**Test backend API via port forward:**
```bash
curl http://localhost:5050/api/blockchain
curl http://localhost:5050/api/wallet/info
```

**View Ingress controller services:**
```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

**Patch Ingress controller to LoadBalancer type:**
```bash
kubectl patch svc ingress-nginx-controller \
  -n ingress-nginx \
  -p '{"spec":{"type":"LoadBalancer"}}'
```

**Check Service external IPs:**
```bash
kubectl get svc -A
```

---

### Cloudflare Tunnel (Public Access)

**Install cloudflared (macOS):**
```bash
brew install cloudflared
```

**Create temporary public tunnel:**
```bash
cloudflared tunnel --url http://127.0.0.1
```

**Verify cloudflared installation:**
```bash
cloudflared --version
```

---

### Horizontal Pod Autoscaling (HPA)

**Create HPA using kubectl:**
```bash
kubectl autoscale deployment backend-peer \
  --cpu-percent=50 \
  --min=1 \
  --max=10
```

**Watch HPA in real-time:**
```bash
kubectl get hpa -w
```

**View HPA details:**
```bash
kubectl get hpa
kubectl describe hpa backend-peer
```

**Delete HPA:**
```bash
kubectl delete hpa backend-peer
```

---

### Load Testing

**Run load generator Pod:**
```bash
kubectl run -i --tty load-generator \
  --rm \
  --image=busybox:1.28 \
  --restart=Never -- \
  /bin/sh -c "while sleep 0.01; do wget -q -O- http://backend-peer:5050/api/wallet/info; done"
```

---

### Debugging and Logs

**View Pod logs:**
```bash
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # Follow logs
kubectl logs <pod-name> -c <container-name>  # Multi-container pods
```

**View logs from all Pods in a Deployment:**
```bash
kubectl logs -l app=backend
kubectl logs -l app=frontend
```

**Execute commands in a running Pod:**
```bash
kubectl exec -it <pod-name> -- /bin/sh
kubectl exec <pod-name> -- env
```

**Describe Pod for troubleshooting:**
```bash
kubectl describe pod <pod-name>
```

**View events:**
```bash
kubectl get events
kubectl get events --sort-by='.lastTimestamp'
```

---

### Cleanup and Resource Management

**Delete specific resources:**
```bash
kubectl delete deployment frontend
kubectl delete svc frontend
kubectl delete pod <pod-name>
kubectl delete ingress frontend-ingress
kubectl delete hpa backend-peer
```

**Delete all resources from manifests:**
```bash
kubectl delete -f k8s/backend-deployment.yaml
kubectl delete -f k8s/frontend-deployment.yaml
```

**Delete Secret:**
```bash
kubectl delete secret backend-secret
```

**Stop minikube:**
```bash
minikube stop
```

**Delete minikube cluster:**
```bash
minikube delete
```

---

## Important Branches

- `start-here` - Starting point for the module (check this out first!)
- `k8s-deployments` - Contains Kubernetes Deployment and Service manifests (Video 6)
- `skaffold-setup` - Includes Skaffold configuration (Video 7)
- `main` - Completed, production-ready version of all features

## Key Differences from Docker Capstone

This Kubernetes-ready version includes several important changes:

1. **Modularized Backend Structure** - Code organized into focused modules (`context.py`, `factory.py`, `routes.py`, `logging.py`, `polling.py`)
2. **API Prefixes** - All backend routes now use `/api/` prefix for easier Kubernetes routing
3. **Structured Logging** - Replaced `print` statements with proper logging to stdout (Kubernetes-friendly)
4. **Graceful Shutdown** - Proper signal handling for Pod termination
5. **Environment-Based Configuration** - Centralized config in `backend/config/` package

## Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [Skaffold Documentation](https://skaffold.dev/docs/)
- [Cloudflare Tunnel Documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)

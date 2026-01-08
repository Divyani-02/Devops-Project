# Kubernetes Manifests

This directory contains Kubernetes manifests for deploying the Hello DevOps application.

## Directory Structure

```
k8s/
├── deployment.yaml      # Production-ready deployment (for AWS EKS)
├── service.yaml         # LoadBalancer service (for AWS EKS)
└── minikube/           # Minikube-specific configurations
    ├── deployment.yaml  # Local deployment
    └── service.yaml     # NodePort service
```

## Deployment Options

### Option 1: AWS EKS (Cloud Production)

Use the main manifests in `k8s/` directory:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

**Features:**
- LoadBalancer service (public IP automatically provisioned)
- Pulls image from Docker Hub
- Production-ready configuration

**Access:**
```bash
kubectl get service hello-devops-service
# Wait for EXTERNAL-IP
# Visit: http://<EXTERNAL-IP>
```

---

### Option 2: Minikube (Local Development)

Use the manifests in `k8s/minikube/` directory:

**Step 1:** Build image in Minikube's Docker
```bash
eval $(minikube docker-env)
docker build -t divyania02/hello-devops:latest -f docker/Dockerfile .
```

**Step 2:** Deploy
```bash
kubectl apply -f k8s/minikube/deployment.yaml
kubectl apply -f k8s/minikube/service.yaml
```

**Step 3:** Access
```bash
minikube service hello-devops-service
# Opens browser automatically
```

**Features:**
- NodePort service (no external LoadBalancer needed)
- Uses local Docker image
- No cloud costs

---

## Key Differences

| Feature | AWS EKS | Minikube |
|---------|---------|----------|
| Service Type | LoadBalancer | NodePort |
| Image Pull Policy | Always (default) | Never |
| Public Access | Yes (External IP) | No (localhost only) |
| Cost | ~$180/month | Free |
| Use Case | Production | Development/Testing |

---

## Current Configuration

The main `k8s/` manifests are configured for **AWS EKS deployment**.

To test locally, use the `k8s/minikube/` versions.

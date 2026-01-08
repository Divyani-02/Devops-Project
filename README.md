# Automated Kubernetes Deployment

## Overview

This project automates the deployment of a simple web application to a Kubernetes cluster using modern DevOps principles including Infrastructure as Code (Terraform), containerization (Docker), and CI/CD automation (GitHub Actions).

## Project Structure

```
Devops-Project/
├── app/
│   ├── app.py              # Flask web application
│   └── requirements.txt    # Python dependencies
├── docker/
│   └── Dockerfile          # Container image definition
├── terraform/
│   ├── main.tf            # AWS EKS infrastructure
│   ├── variables.tf       # Configuration variables
│   └── outputs.tf         # Output values
├── k8s/
│   ├── deployment.yaml    # Kubernetes Deployment
│   └── service.yaml       # Kubernetes Service (LoadBalancer)
└── .github/
    └── workflows/
        └── deploy.yml     # CI/CD pipeline
```

## Core Requirements

### 1. Infrastructure as Code (IaC)

Terraform provisions a managed Kubernetes cluster on AWS EKS with:
- VPC with public/private subnets
- EKS cluster with managed node groups
- Auto-scaling configuration
- All networking components (NAT Gateway, Internet Gateway, Load Balancer)

```bash
cd terraform
terraform init
terraform apply
```

### 2. Containerization

Dockerfile packages the Flask application:
- Optimized Python 3.11-slim base image
- Application runs on port 5000
- Published to Docker Hub: `divyania02/hello-devops:latest`

```bash
docker build -t divyania02/hello-devops:latest -f docker/Dockerfile .
docker push divyania02/hello-devops:latest
```

### 3. Kubernetes Manifests

YAML manifests define the application deployment:
- **Deployment:** 2 replicas with resource limits
- **Service:** LoadBalancer type for public access

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### 4. CI/CD Pipeline

GitHub Actions automates the deployment:
- **Trigger:** Push to main branch
- **Build:** Creates Docker image from Dockerfile
- **Push:** Uploads image to Docker Hub
- **Deploy:** Applies Kubernetes manifests to EKS cluster

### 5. Documentation

**Brief overview:** Automated Kubernetes deployment using Terraform for infrastructure, Docker for containerization, and GitHub Actions for CI/CD.

**Instructions:**
```bash
# Deploy infrastructure
cd terraform && terraform apply

# Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name hello-devops-cluster

# Push code (CI/CD auto-deploys)
git push origin main

# Get application URL
kubectl get service hello-devops-service
```

**Design choices:**
- **AWS EKS:** Managed Kubernetes reduces operational overhead
- **Terraform:** Infrastructure as Code for reproducible deployments
- **GitHub Actions:** Free CI/CD for automated deployments
- **LoadBalancer Service:** Makes application publicly accessible

**Live URL:** Get the EXTERNAL-IP from `kubectl get service hello-devops-service`

---

**Repository:** https://github.com/Divyani-02/Devops-Project  
**Docker Image:** divyania02/hello-devops:latest

# DevOps Engineer Take-Home Task: Automated Kubernetes Deployment

## 🎯 Overview

This project demonstrates automated deployment of a Flask web application to AWS EKS (Elastic Kubernetes Service) using modern DevOps practices including Infrastructure as Code (Terraform), containerization (Docker), and CI/CD automation (GitHub Actions).

**Live Application URL:** `http://<EKS-LoadBalancer-URL>` (Available after deployment)

## 📋 Table of Contents

- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Deployment Process](#deployment-process)
- [Design Choices](#design-choices)
- [Clean Up](#clean-up)

## 🏗️ Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   GitHub    │────▶│ GitHub       │────▶│   AWS EKS   │
│  Repository │     │   Actions    │     │   Cluster   │
└─────────────┘     └──────────────┘     └─────────────┘
                           │                     │
                           ▼                     ▼
                    ┌──────────────┐     ┌─────────────┐
                    │  Docker Hub  │     │ Application │
                    │    Image     │     │   Pods      │
                    └──────────────┘     └─────────────┘
```

## 🛠️ Technologies Used

- **Application:** Python Flask
- **Containerization:** Docker
- **Container Registry:** Docker Hub
- **IaC Tool:** Terraform
- **Cloud Provider:** AWS (EKS, VPC, EC2)
- **Orchestration:** Kubernetes
- **CI/CD:** GitHub Actions
- **Version Control:** Git/GitHub

## 📁 Project Structure

```
Devops-Project/
├── app/
│   ├── app.py              # Flask application
│   └── requirements.txt    # Python dependencies
├── docker/
│   └── Dockerfile          # Docker image definition
├── terraform/
│   ├── main.tf            # Main Terraform configuration
│   ├── variables.tf       # Variable definitions
│   ├── outputs.tf         # Output values
│   └── terraform.tfvars.example  # Example variables
├── k8s/
│   ├── deployment.yaml    # Kubernetes Deployment (AWS/Production)
│   ├── service.yaml       # Kubernetes Service (LoadBalancer for AWS)
│   └── minikube/
│       ├── deployment.yaml # Minikube-specific deployment
│       └── service.yaml    # NodePort service for local testing
├── .github/
│   └── workflows/
│       └── deploy.yml     # CI/CD pipeline
└── README.md
```

## 🔧 Kubernetes Configurations

This project includes **two sets** of Kubernetes manifests for different environments:

### **Production/AWS EKS (`k8s/`)**

Primary configuration for cloud deployment:

- **`deployment.yaml`**
  - Pulls image from Docker Hub
  - Standard `imagePullPolicy` (IfNotPresent)
  - Production-ready resource limits
  
- **`service.yaml`**
  - Type: `LoadBalancer`
  - Creates AWS Elastic Load Balancer
  - Provides public IP for external access

**Usage:**
```bash
kubectl apply -f k8s/
```

### **Local Development (`k8s/minikube/`)**

Optimized configuration for local Minikube testing:

- **`deployment.yaml`**
  - Uses `imagePullPolicy: Never`
  - Uses locally built Docker image
  - Identical resource configuration
  
- **`service.yaml`**
  - Type: `NodePort`
  - Fixed port: 30080
  - Works with Minikube port forwarding

**Usage:**
```bash
# Build image in Minikube's Docker
eval $(minikube docker-env)
docker build -t divyania02/hello-devops:latest -f docker/Dockerfile .

# Deploy
kubectl apply -f k8s/minikube/

# Access
minikube service hello-devops-service
```

### **Key Differences:**

| Feature | AWS/Production | Minikube/Local |
|---------|---------------|----------------|
| Image Source | Docker Hub | Local Docker |
| Service Type | LoadBalancer | NodePort |
| External Access | Public IP/DNS | localhost tunnel |
| Use Case | Cloud deployment | Local testing |

## ✅ Prerequisites

Before starting, ensure you have:

1. **AWS Account** with appropriate permissions
2. **AWS CLI** installed and configured
3. **Terraform** (v1.0+)
4. **Docker** installed locally
5. **kubectl** installed
6. **Git** installed
7. **Docker Hub account**
8. **GitHub account**

## 🚀 Setup Instructions

### Step 1: Clone the Repository

```bash
git clone https://github.com/Divyani-02/Devops-Project.git
cd Devops-Project
```

### Step 2: Configure AWS Credentials

```bash
aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Default region: us-east-1
```

### Step 3: Set Up Terraform

```bash
cd terraform

# Copy example variables
cp terraform.tfvars.example terraform.tfvars

# Edit terraform.tfvars with your preferences (optional)

# Initialize Terraform
terraform init

# Review the plan
terraform plan

# Apply the configuration (creates EKS cluster - takes ~15 minutes)
terraform apply
```

### Step 4: Configure kubectl

After Terraform completes, configure kubectl:

```bash
aws eks update-kubeconfig --region us-east-1 --name hello-devops-cluster

# Verify connection
kubectl cluster-info
kubectl get nodes
```

### Step 5: Configure GitHub Secrets

Add these secrets in your GitHub repository (Settings → Secrets and variables → Actions):

- `AWS_ACCESS_KEY_ID`: Your AWS access key
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret key
- `DOCKER_USERNAME`: Your Docker Hub username (divyania02)
- `DOCKER_PASSWORD`: Your Docker Hub password or token

### Step 6: Deploy the Application

**Option A: Automatic (via GitHub Actions)**
- Push code to the `main` branch
- GitHub Actions will automatically build, push, and deploy

**Option B: Manual Deployment**

```bash
# Build and push Docker image
docker build -t divyania02/hello-devops:latest -f docker/Dockerfile .
docker push divyania02/hello-devops:latest

# Deploy to Kubernetes
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Check deployment status
kubectl get deployments
kubectl get pods
kubectl get services
```

### Step 7: Access the Application

```bash
# Get the LoadBalancer URL
kubectl get service hello-devops-service

# The EXTERNAL-IP column shows your application URL
# Visit: http://<EXTERNAL-IP>
```

You should see: **"Hello DevOps Engineer!"**

## ⚠️ AWS Deployment Cost Consideration

This project includes production-ready Terraform code for AWS EKS deployment. 
However, **AWS EKS incurs significant costs** (~$180/month for cluster + nodes).

### Current Demonstration:
- **Platform:** Minikube (local Kubernetes)
- **Reason:** Cost-effective demonstration without cloud charges
- **Functionality:** Identical to AWS EKS deployment
- **Access:** `minikube service hello-devops-service`

### AWS EKS Deployment (When Ready):

All infrastructure code is prepared and tested. To deploy to AWS:

```bash
# 1. Configure AWS credentials
aws configure

# 2. Deploy infrastructure (⚠️ Costs ~$0.25/hour)
cd terraform
terraform init
terraform apply

# 3. Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name hello-devops-cluster

# 4. Push to GitHub - Actions will automatically deploy
git push origin main

# 5. IMPORTANT: Destroy when done testing
terraform destroy
```

**Estimated AWS Costs:**
- EKS Control Plane: $0.10/hour ($72/month)
- EC2 Nodes (2× t3.medium): $0.083/hour ($60/month)
- Load Balancer: $0.025/hour ($18/month)
- **Total: ~$0.20/hour or $150-180/month**

### Design Decision:
Used Minikube for local demonstration to avoid unnecessary AWS costs while 
maintaining production-grade infrastructure code and DevOps best practices.

## 🔄 Deployment Process

### CI/CD Pipeline Flow

1. **Trigger:** Push to `main` branch
2. **Build Stage:**
   - Checkout code
   - Build Docker image from Dockerfile
   - Tag with commit SHA and `latest`
   - Push to Docker Hub
3. **Deploy Stage (when AWS configured):**
   - Configure AWS credentials
   - Update kubectl config
   - Apply Kubernetes manifests
   - Verify deployment rollout
   - Display LoadBalancer URL

## 💡 Design Choices

### Infrastructure Decisions

1. **AWS EKS:** 
   - Managed Kubernetes service reduces operational overhead
   - Automatic updates and patching
   - Integrated with AWS services

2. **Terraform Modules:**
   - Used community-maintained modules for VPC and EKS
   - Reduces code complexity and follows best practices
   - Easy to maintain and update

3. **Node Configuration:**
   - t3.medium instances (2 vCPU, 4GB RAM)
   - Min 1, Max 3 nodes for cost optimization
   - Auto-scaling based on workload

4. **Networking:**
   - Private subnets for worker nodes (security)
   - Public subnets for LoadBalancer
   - Single NAT Gateway (cost optimization for dev)

### Application Decisions

1. **Flask Framework:**
   - Lightweight and simple
   - Perfect for demonstrating concepts
   - Easy to containerize

2. **Docker Image:**
   - Based on `python:3.11-slim` (minimal size)
   - Multi-stage build could be added for optimization
   - Resource limits defined in K8s manifest

3. **Kubernetes Configuration:**
   - 2 replicas for high availability
   - LoadBalancer service for external access
   - Resource requests/limits for efficient scheduling

4. **CI/CD:**
   - GitHub Actions (free for public repos)
   - Automated on every push to main
   - Build caching for faster builds

## 🧹 Clean Up

To avoid AWS charges, destroy resources when done:

```bash
# Delete Kubernetes resources
kubectl delete -f k8s/

# Destroy Terraform infrastructure
cd terraform
terraform destroy

# Confirm with 'yes'
```

## 📊 Monitoring & Observability (Optional Enhancement)

For production, consider adding:
- **Prometheus & Grafana** for metrics
- **EFK Stack** (Elasticsearch, Fluentd, Kibana) for logs
- **AWS CloudWatch** integration
- **Health checks** and liveness probes

## 🔐 Security Considerations

- Secrets stored in GitHub Secrets (encrypted)
- IAM roles with least privilege
- Private subnets for worker nodes
- Security groups properly configured
- Regular updates of base images

## 📝 Testing

```bash
# Test locally with Docker
docker run -p 5000:5000 divyania02/hello-devops:latest
curl http://localhost:5000

# Test on Kubernetes
kubectl port-forward service/hello-devops-service 8080:80
curl http://localhost:8080
```

## 🤝 Contributing

This is a take-home assignment project. Fork and modify as needed.

## 📄 License

MIT License

## 👤 Author

**Divyani Aher**
- GitHub: [@Divyani-02](https://github.com/Divyani-02)
- Docker Hub: [divyania02](https://hub.docker.com/u/divyania02)

---

**Project Status:** ✅ Complete and Deployed

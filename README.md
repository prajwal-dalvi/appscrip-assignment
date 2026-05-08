# DevOps Assignment — EKS + ArgoCD + NGINX

## Project Overview

This project demonstrates a complete DevOps pipeline for deploying an NGINX web server on Amazon EKS (Elastic Kubernetes Service) using Infrastructure as Code (IaC) with Terraform and GitOps with ArgoCD. The setup provisions AWS infrastructure, deploys ArgoCD for continuous deployment, and uses GitOps principles to manage application deployments.

## Project Flow

1. **Infrastructure Provisioning**: Terraform creates a VPC, subnets, and an EKS cluster in AWS.
2. **ArgoCD Installation**: ArgoCD is installed on the EKS cluster to enable GitOps deployments.
3. **Application Deployment**: ArgoCD monitors the repository and automatically deploys NGINX using Kubernetes manifests stored in the `manifests` directory.
4. **Access**: The NGINX service is exposed and can be accessed via port forwarding.

## Directory Structure

### `terraform/`
Contains Terraform configuration files for provisioning AWS infrastructure:
- `main.tf`: Defines the AWS provider, VPC module, and EKS cluster module. Creates a VPC with public and private subnets, NAT gateway, and an EKS cluster with managed node groups.
- `variables.tf`: Defines input variables like AWS region and cluster name with default values.
- `outputs.tf`: Outputs important information like cluster name, endpoint, and kubeconfig command for accessing the cluster.

### `argocd/`
Contains ArgoCD application configuration:
- `nginx-app.yaml`: ArgoCD Application manifest that defines how to deploy the NGINX application. It points to the `manifests` directory in this repository and syncs changes automatically to the EKS cluster.

### `manifests/`
Contains Kubernetes manifests for the NGINX application:
- `nginx-deployment.yaml`: Defines a Kubernetes Deployment with 2 replicas of NGINX containers, including resource requests/limits and labels for service discovery.
- `nginx-service.yaml`: Defines a LoadBalancer service that exposes the NGINX deployment on port 80, allowing external access to the application.

## Prerequisites
- AWS CLI configured (`aws configure`)
- Terraform >= 1.6
- kubectl, argocd CLI, helm

## Usage

### 1. Provision Infrastructure
Navigate to the terraform directory and initialize and apply the Terraform configuration:
```bash
cd terraform
terraform init
terraform apply -auto-approve
```

Update your kubeconfig to access the EKS cluster:
```bash
aws eks update-kubeconfig --region us-east-1 --name appscrip-prod-cluster
```

### 2. Install ArgoCD
Create the argocd namespace and install ArgoCD:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Port forward ArgoCD server for local access:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### 3. Login to ArgoCD
Retrieve the initial admin password and login:
```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
argocd login localhost:8080 --username admin --insecure
```

### 4. Deploy NGINX via GitOps
Apply the ArgoCD application manifest:
```bash
kubectl apply -f argocd/nginx-app.yaml
```

### 5. Access NGINX
Port forward the NGINX service to access it locally:
```bash
kubectl port-forward svc/nginx-service 8888:80
```
Open the alb DNS Name to access the nginx application in your browser to view the NGINX welcome page.
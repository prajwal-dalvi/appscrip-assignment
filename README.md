# DevOps Assignment — EKS + ArgoCD + NGINX

## Prerequisites
- AWS CLI configured (`aws configure`)
- Terraform >= 1.6
- kubectl, argocd CLI, helm

## 1. Provision infrastructure
cd terraform && terraform init && terraform apply -auto-approve
aws eks update-kubeconfig --region us-east-1 --name demo-eks

## 2. Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl port-forward svc/argocd-server -n argocd 8080:443

## 3. Login
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
argocd login localhost:8080 --username admin --insecure

## 4. Deploy NGINX via GitOps
kubectl apply -f argocd/nginx-app.yaml

## 5. Access NGINX
kubectl port-forward svc/nginx-service 8888:80
# Open http://localhost:8888
# Flask App Deployment on AWS EKS using Terraform, Docker, and Helm

## Project Overview

This project demonstrates a complete end-to-end deployment of a Flask-based web application on Amazon EKS (Elastic Kubernetes Service). It uses Terraform to provision the EKS infrastructure, Docker to containerize the Flask app, and Helm to manage Kubernetes manifests and deploy the app using an NGINX Ingress Controller for external access.

---

## Tools and Services Used

- **Amazon EKS (Elastic Kubernetes Service)**
- **Terraform**
- **Kubernetes**
- **Helm**
- **Docker**
- **Flask-app**
- **AWS VPC, IAM, EC2, Security Groups, etc.**
- **NGINX Ingress Controller**

---

## Prerequisites

Before running this project, ensure you have:

- AWS CLI configured with access keys
- Terraform installed
- Docker installed and running
- kubectl installed and configured
- Helm installed
- An AWS account with permissions to create VPC, EKS, IAM roles, etc.

---

## Features

- Infrastructure-as-Code (IaC) setup using Terraform
- Flask web application containerized with Docker
- Helm-based Kubernetes manifest management
- Public accessibility through NGINX Ingress on AWS
- Clean modular project structure

---

## How It Works

1. **Terraform** provisions the EKS cluster along with VPC, subnets, IAM roles, and node groups.
2. **Docker** builds a container image for the Flask app and pushes it to Docker Hub.
3. **Helm** initializes a chart, configures image and ingress, and deploys the app.
4. **Ingress Controller** exposes the application to the internet using AWS ELB.

---

## Architecture Diagram:

<img width="1006" height="746" alt="eks-flask-app-diagram" src="https://github.com/user-attachments/assets/87c6f3a1-515d-4ddd-891a-ca4add37771e" />

---

## Project Structure
```bash
eks-flask-app/
├── terraform/
│ ├── main.tf
│ ├── variables.tf
│ ├── providers.tf
│ └── outputs.tf
├── flask-app/
│ ├── app.py
│ ├── requirements.txt
│ └── Dockerfile
├── my-app/
│ ├── .helmignore
│ ├── Chart.yaml
│ ├── values.yaml
│ ├── charts/
│ ├── templates/
│ │ ├── _helpers.tpl
│ │ ├── deployment.yaml
│ │ ├── ingress.yaml
│ │ ├── hpl.yaml
│ │ ├── NOTES.txt
│ │ ├── service.yaml
│ │ └── serviceaccount.yaml
│ └── tests/
│ └── test-connection.yaml
├── README.md
├── .gitignore
```

## Steps to Deploy

### Step 1: Clone the Repo
```bash
git clone https://github.com/Sushmitha6300/eks-flask-app.git
cd eks-flask-app
```

### Step 2: Provision Infrastructure using Terraform

Navigate to the terraform/ folder and initialize Terraform:
```bash
cd terraform
terraform init
```

Apply in Two Phases (Due to EKS Dependency)
**Phase 1:**

In providers.tf, comment out the kubernetes provider block AND the data blocks:
```bash
# data "aws_eks_cluster" "cluster" {
#   name = module.eks.cluster_name
# }

# data "aws_eks_cluster_auth" "cluster" {
#   name = module.eks.cluster_name
# }

# provider "kubernetes" {
#   host = data.aws_eks_cluster.cluster.endpoint
#   cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority[0].data)
#   token = data.aws_eks_cluster_auth.cluster.token
# }
```

In main.tf, set this:
```bash
manage_aws_auth_configmap = false
```

Run:
```
terraform apply
```

**Phase 2:**

Uncomment the data blocks and Kubernetes provider block

In main.tf, change this:
```bash
manage_aws_auth_configmap = true
```

Apply again:
```bash
terraform apply
```

### Step 3: Update your kubeconfig so kubectl can access the EKS cluster:
```bash
aws eks update-kubeconfig --region us-east-1 --name app-cluster 
```

Verify node group is registered:
```bash
kubectl get nodes
```

### Step 4: Containerize the Node.js App

Navigate to the app folder:
```bash
cd ../flask-app
```

Make sure Docker Desktop is running

Build and push the Docker image:
```bash
docker build -t your-dockerhub-username/flask-app .
docker push your-dockerhub-username/flask-app
```

### Step 5: Deploy the App Using Helm

**1. Create Helm Chart**
```bash
helm create my-app
```

This generates the complete Helm chart structure under my-app/

**2. Edit values.yaml**

Update the following fields:
```bash
image:
  repository: your-dockerhub-username/flask-app
  pullPolicy: IfNotPresent
  tag: latest

service:
  type: ClusterIP
  port: 5000

ingress:
  enabled: true
  className: "nginx"
  annotations: 
    kubernetes.io/ingress.class: nginx
  hosts:
    - host: ""
      paths:
        - path: /
          pathType: Prefix
          backend:
          service:
            name: flask-service
            port:
              number: 5000
```

**3. Install Helm Chart**
```bash
helm install my-app ./my-app
```

**4. Verify Helm Resources**
```bash
kubectl get pods
kubectl get deployments
kubectl get svc
```

**5. Install NGINX Ingress Controller**
```bash
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --create-namespace \
  --namespace ingress-nginx
```

**6. Get LoadBalancer DNS**
```bash
kubectl get svc -n ingress-nginx
```

Copy the EXTERNAL-IP of the LoadBalancer (usually an AWS ELB DNS).
Example Output:
```bash
EXTERNAL-IP        
abc123456789.elb.amazonaws.com
```

**7. Access the App**

Open the app in the browser:
```bash
http://abc123456789.elb.amazonaws.com
```

**You should see:**

<img width="1808" height="689" alt="eks-flask-app-output" src="https://github.com/user-attachments/assets/a404c9bd-8b78-4b50-835d-c22cfa469e67" />

---

## About Me

Hey there! I’m Sushmitha, an aspiring DevOps Engineer passionate about automating infrastructure and streamlining deployments.

Currently, I’m building hands-on projects to master the DevOps lifecycle — from infrastructure as code to CI/CD and monitoring.

Always eager to learn, experiment, and take on new challenges in the cloud and DevOps world.

**Let’s connect!**

- **LinkedIn:**
- **GitHub:** https://github.com/Sushmitha6300

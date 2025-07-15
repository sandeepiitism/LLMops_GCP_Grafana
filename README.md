# LLMops_GCP_Grafana

This repository provides a complete setup to deploy a Language Model (LLM) application on Google Cloud Platform (GCP) using Docker, Minikube + kubectl for container orchestration, and Grafana Cloud for monitoring.

---

## Overview

This setup includes:

- Provisioning a GCP VM
- Installing and configuring Docker
- Setting up Minikube and kubectl
- Deploying a containerized LLM application
- Monitoring Kubernetes pods using Grafana Cloud
- GitHub integration for code version control

---

## 1. VM Setup

Provision a VM instance (Ubuntu preferred) using the GCP Console. SSH into your instance and perform the following steps.

---

## 2. Docker Installation and Configuration

Reference: https://docs.docker.com/engine/install/ubuntu/

```bash
# Install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Set up the Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker packages
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify Docker installation
sudo docker run hello-world

# Add user to Docker group
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world

# Enable Docker on boot
sudo systemctl enable docker.service
sudo systemctl enable containerd.service

# Check Docker version
docker version


# Download and install Minikube
Reference: https://minikube.sigs.k8s.io/docs/start/

curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64

# Start Minikube
minikube start

# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Install it
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Or install via snap
sudo snap install kubectl --classic

# Verify installation
kubectl version --client

# Clone the repository
git clone https://github.com/sandeepiitism/LLMops_GCP_Grafana.git

# Set Git credentials
git config --global user.email "your-email@example.com"
git config --global user.name "your-username"

# Commit and push code
git add .
git commit -m "Initial commit from GCP VM"
git push origin main

Use a GitHub Personal Access Token (PAT) when prompted.

# Point Docker to Minikube
eval $(minikube docker-env)

# Build Docker image
docker build -t llmops-app:latest .

# Create Kubernetes secrets
kubectl create secret generic llmops-secrets \
  --from-literal=GROQ_API_KEY="" \
  --from-literal=HUGGINGFACEHUB_API_TOKEN=""

# Apply the deployment file
kubectl apply -f llmops-k8s.yaml

# Check pod status
kubectl get pods

# In one terminal
minikube tunnel

# In another terminal
kubectl port-forward svc/llmops-service 8501:80 --address 0.0.0.0

# Grafana Cloud Setup
kubectl create ns monitoring
kubectl get ns

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

vi values.yaml

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm upgrade --install --atomic --timeout 300s grafana-k8s-monitoring grafana/k8s-monitoring \
  --namespace "monitoring" --create-namespace --values values.yaml

kubectl get pods -n monitoring

# Stop Minikube
minikube stop

# Delete GCP VM
gcloud compute instances delete <INSTANCE_NAME>



![Alt Text](https://github.com/user-attachments/assets/e3d144e2-63c2-4e22-902a-0517b730ab19)


# LLMops_GCP_Grafana
This repo is for LLM deployement on GCP using docker, Kubernetes (MiniKube + Kubectl) and Grafana for monitoring the pods


1. VM setup


2. Docker setup: https://docs.docker.com/engine/install/ubuntu/

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

#### Install the docker package

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

#### Verify Installation

sudo docker run hello-world


#### Create the docker group and add user

sudo groupadd docker

sudo usermod -aG docker $USER

newgrp docker

docker run hello-world

#### Configure Docker to start on boot with systemd

sudo systemctl enable docker.service
sudo systemctl enable containerd.service

#### check docker again

docker version

## -------------------------------------------------------------------------------------------------- ##

## minikube
Ref: https://minikube.sigs.k8s.io/docs/start

#### Install minikube

curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

#### Start your cluster
minikube start


## ---------------------------------------------------------------------------------------------------- ##
## Kuberctl

Ref: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

#### Download
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

#### Install Kubectl
sudo snap install kubectl --classic
kubectl version --client

## ---------------------------------------------------------------------------------------------------- ##
Integration Github with GCP VM

#### git clone on your VM (SSH)
git clone https://github.com/sandeepiitism/LLMops_GCP_Grafana.git

#### config VM
git config --global user.email "email"
git config --global user.name "username" 
git add .
git commit -m "gcp connected"
git push origin main ---> create PAT tokens in github to get the password

## ---------------------------------------------------------------------------------------------------- ##
GCP Firewall Rules






## ---------------------------------------------------------------------------------------------------- ##
Deploy App on VM

### 5. Build and Deploy your APP on VM

```bash
## Point Docker to Minikube
eval $(minikube docker-env)

docker build -t llmops-app:latest .

kubectl create secret generic llmops-secrets \
  --from-literal=GROQ_API_KEY="" \
  --from-literal=HUGGINGFACEHUB_API_TOKEN=""

kubectl apply -f llmops-k8s.yaml


kubectl get pods

### U will see pods runiing


# Do minikube tunnel on one terminal

minikube tunnel


# Open another terminal

kubectl port-forward svc/llmops-service 8501:80 --address 0.0.0.0

## Now copy external ip and :8501 and see ur app there....


```

### 6. GRAFANA CLOUD MONITORING

```bash
## Open another VM terminal for Grfana cloud

kubectl create ns monitoring

kubectl get ns

## Make account on Grfaana cloud

### Install HELM - Search on Google
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh


## Come to grafana cloud --> Left pane observability --> Kubernetes--> start sending data
## In backend installation --> Hit install
## Give your clustername and namespace there : minikube and monitoring in our case
## Select kubernetes
## Keep other things on as default
## Here only create new access token give name lets give minikube-token & Create it and save it somewhere..
## Select helm and deploy helm charts is already generated...



## Come to terminal --> Create a file
vi values.yaml


## Paste all from there to your file now remove last EOF part & and also initial part save that initial part we need it..

Example : 

helm repo add grafana https://grafana.github.io/helm-charts &&
  helm repo update &&
  helm upgrade --install --atomic --timeout 300s grafana-k8s-monitoring grafana/k8s-monitoring \
    --namespace "monitoring" --create-namespace --values - <<'EOF'

### Remove this above intial part and save it somewhere

Then Esc+wq! amd save the file


## Now use the copied command just make some modification:
Remove that EOF part and instead write
--values values.yaml

Example:

helm repo add grafana https://grafana.github.io/helm-charts &&
  helm repo update &&
  helm upgrade --install --atomic --timeout 300s grafana-k8s-monitoring grafana/k8s-monitoring \
    --namespace "monitoring" --create-namespace --values values.yaml

## Paste this command on VM u will get status deployed revision 1
## It means it was a SUCESS

To check:

kubectl get pods -n monitoring

# These are all should be running.....

Go to grafana cloud again..
And below u will get go to homepage click it..
Just refresh the page and boom..


Now u can see metrics related to your kubernetes cluster..

---Explore it for yourself now 

---Make sure to do cleanup 

```
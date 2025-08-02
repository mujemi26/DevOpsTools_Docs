# 🚀 ArgoCD Setup & Installation Guide

## 📋 Introduction

This comprehensive guide walks you through the complete setup and installation process for ArgoCD, a powerful GitOps continuous delivery tool for Kubernetes. You'll learn how to:

- 🐙 Install and configure Git
- 🐳 Set up Docker and Kubernetes using kind
- ⚡ Install ArgoCD on your Kubernetes cluster
- 🌐 Configure ArgoCD for web access
- 🔧 Set up the ArgoCD CLI

---

## 🐙 1st Installing and configuring Git

Run:
```bash
sudo apt update
sudo apt install git -y
git --version
```

### Configure the git user information by running:
```bash
git config --global user.name "your name"
git config --global user.email "your@email.com"
git config --global core.editor "vim" # execute
```

---

## 🐳 Installing K8s Using kind

### 📦 Install Docker by running the following commands:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker ${USER}
newgrp docker
docker version
```

### ⚙️ Install kubectl by running the following command

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo mv kubectl /usr/local/bin
sudo chmod +x /usr/local/bin/kubectl
kubectl version --client
```

### 🔗 Install kind

- Go to https://github.com/kubernetes-sigs/kind/releases

- Scroll down till you find the downloadable files.

- Right click on the Linux AMD64 and copy the link.

#### In the terminal, run the following:

```bash
wget https://github.com/kubernetes-sigs/kind/releases/download/v0.18.0/kind-linux-amd64
sudo mv kind-linux-amd64 /usr/local/bin/kind
sudo chmod +x /usr/local/bin/kind
kind version
kind create cluster # don't run this command yet
```

### 📄 Create a cluster configuration file as follows:

```yaml
# cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```

### 🚀 Create a cluster by running the following command:
```bash
kind create cluster --config=cluster.yaml
kubectl cluster-info --context kind-kind
```

### 🌐 Run the following commands to deploy an ingress controller:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s
```

--- 

## ⚡ Installing Argo CD to the cluster

### 📦 Install Helm:
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 🚀 Install Argo CD on the cluster using Helm as follows:
```bash
helm repo add argo https://argoproj.github.io/argo-helm
kubectl create namespace argocd
helm install argocd -n argocd argo/argo-cd
```

### 🔐 Get the administrator password (or just copy the command from the Helm output):

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

- Copy the password that was brought.

- Create a port-forward to access the UI of the server by running:

```bash
kubectl port-forward service/argocd-server -n argocd 8080:443 --address="0.0.0.0"
```

### 🌐 Access ArgoCD Web UI

- Open the browser and navigate to 192.168.2.30:8080. Accept the security risk. Enter the username: admin and paste the password from the above output.

### 🌐 Return back to the terminal and install the Nginx ingress controller by running the following command:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
```

### ⚙️ Return the Helm command enabling Ingress and the other required options:

```bash
helm upgrade argocd --set configs.params."server\.insecure"=true --set server.ingress.enabled=true  --set server.ingress.ingressClassName="nginx" -n argocd argo/argo-cd
```

## 🔧 Workaround: Argo's URL are not opening?

- Edit default ingress and put your url name.

```bash
kubectl edit ingress argocd-server -n argocd
```

```yaml
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.ex280.example.local # change me as you like to name your argo gui
    http:
      paths:
      - backend:
          service:
            name: argocd-server
            port:
              number: 80
        path: /
        pathType: Prefix
status:
  loadBalancer:
    ingress:
    - hostname: localhost
```

### 📥 Navigate to https://github.com/argoproj/argo-cd/releases/latest to get the Argo CD CLI.

- Scroll down till the downloadable files.

- Copy the link to the Linux amd64 file.

- Install Argo CD CLI by running the following command:
```bash
wget https://github.com/argoproj/argo-cd/releases/download/v2.6.7/argocd-linux-amd64
sudo mv argocd-linux-amd64  /usr/local/bin/argocd
chmod +x /usr/local/bin/argocd
```

### 🔑 Login to Argo CD using the following command (username is admin and the password can be pasted from the previous output):

```bash
argocd login localhost --insecure # use your domain you set in the ingress and /etc/hosts
```

### ✅ Verify that you are logged in by running:
```bash
argocd repo list
```

### 🔄 Change the password using the following command:

```bash
argocd account update-password \
--current-password # current password \
--new-password # new password
```
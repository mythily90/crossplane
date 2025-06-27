# Environment Setup for Crossplane Demo

This guide outlines steps to set up a local Kubernetes environment using `kind`, install `Crossplane` using Helm, and configure the AWS provider.

---

## Table of Contents

- [Prerequisites](#prerequisites)
  - [Install Docker](#install-docker)
  - [Install Kind](#install-kind)
  - [Install kubectl](#install-kubectl)
  - [Create Kubernetes Cluster](#create-kubernetes-cluster)
  - [Install Helm](#install-helm)
- [Install Crossplane](#install-crossplane)
  - [Enable Crossplane Helm Chart](#enable-crossplane-helm-chart)
  - [Configure AWS Provider](#configure-aws-provider)

---

## Prerequisites

### Install Docker

```bash
# Update packages
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker’s GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add user to Docker group
sudo usermod -aG docker $USER
```
### Install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
sudo chmod +x ./kind
sudo mv ./kind /usr/local/bin/
kind version
```

### Install kubectl
```bash
wget "https://dl.k8s.io/release/v1.29.0/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### Create Kubernetes Cluster
```bash
kind create cluster --name crossplane-test
kubectl cluster-info --context kind-crossplane-test
```

### Install Helm
```bash
sudo snap install helm --classic
```

## Install Crossplane

### Install Crossplane Helm Chart
```bash
# Add Helm repo and update
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

# Install Crossplane
helm install crossplane \
  crossplane-stable/crossplane \
  --namespace crossplane-system \
  --create-namespace
```

### Configure AWS Provider
```bash
kubectl apply -f awsprovider.yaml
```


### Configure AWS Provider Config

Create aws-credentials.txt. Populate with actual access key secret key.
```bash
[default]
aws_access_key_id = 
aws_secret_access_key = 
```

create kubernetes secret aws-secret
```bash
kubectl create secret generic aws-secret -n crossplane-system --from-file=creds=./aws-credentials.txt
```

verify secret
```bash
kubectl get secrets
```

Create configProvider
```bash
kubectl apply -f providerconfig.yaml
```

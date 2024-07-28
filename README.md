# Kubernetes Setup Guide

This guide provides comprehensive instructions for setting up an Amazon EKS cluster, a self-hosted Kubernetes cluster, and configuring RBAC in Kubernetes.

## Table of Contents
- [Prerequisites](#prerequisites)
- [AWS CLI Installation](#aws-cli-installation)
- [EKSCTL Installation](#eksctl-installation)
- [kubectl Installation](#kubectl-installation)
- [Create EKS Cluster](#create-eks-cluster)
- [Self-Hosted Kubernetes Setup](#self-hosted-kubernetes-setup)
  - [Common Setup for Both Master and Worker Nodes](#common-setup-for-both-master-and-worker-nodes)
  - [Master Node Setup](#master-node-setup)
  - [Worker Node Setup](#worker-node-setup)
  - [Verify the Cluster](#verify-the-cluster)
- [RBAC Setup](#rbac-setup)

## Prerequisites
- Install `unzip` utility:
  ```bash
  sudo apt-get update
  sudo apt install unzip
  ```

## AWS CLI Installation
1. **Download and Install AWS CLI**:
    ```bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    ```

2. **Configure AWS CLI**:
    ```bash
    aws configure
    ```

## EKSCTL Installation
1. **Download and Install eksctl**:
    ```bash
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin
    eksctl version
    ```

## kubectl Installation
1. **Download and Install kubectl from [AWS EKS Guide](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)**:
    ```bash
    sudo curl --silent --location -o /usr/local/bin/kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.30.0/2024-05-12/bin/linux/amd64/kubectl
    sudo chmod +x /usr/local/bin/kubectl
    kubectl version --client
    ```

## Create EKS Cluster
1. **Create EKS Cluster**:
    ```bash
    eksctl create cluster --name=my-eks \
                          --region=us-east-1 \
                          --zones=us-east-1a,us-east-1b \
                          --version=1.30 \
                          --without-nodegroup
    ```

2. **Associate IAM OIDC Provider**:
    ```bash
    eksctl utils associate-iam-oidc-provider \
        --region us-east-1 \
        --cluster my-eks \
        --approve
    ```

3. **Create Node Group (Replace `ssh-public-key` with actual key-pair name)**:
    ```bash
    eksctl create nodegroup --cluster=my-eks \
                           --region=us-east-1 \
                           --name=node2 \
                           --node-type=t3.medium \
                           --nodes=3 \
                           --nodes-min=2 \
                           --nodes-max=4 \
                           --node-volume-size=20 \
                           --ssh-access \
                           --ssh-public-key=Key \
                           --managed \
                           --asg-access \
                           --external-dns-access \
                           --full-ecr-access \
                           --appmesh-access \
                           --alb-ingress-access
    ```

4. **Verify Cluster**:
    ```bash
    eksctl get cluster --name my-eks --region us-east-1
    aws eks update-kubeconfig --region us-east-1 --name my-eks
    ```

## Self-Hosted Kubernetes Setup

### Common Setup for Both Master and Worker Nodes
1. **Create and run the common setup script on both the master and worker nodes**:

    ```bash
    # setup_k8s_common.sh
    #!/bin/bash

    sudo apt-get update

    sudo apt install docker.io -y

    sudo chmod 666 /var/run/docker.sock

    sudo apt-get update

    sudo apt-get install -y apt-transport-https ca-certificates curl gpg

    sudo mkdir -p -m 755 /etc/apt/keyrings

    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

    sudo apt-get update

    sudo apt-get install -y kubelet kubeadm kubectl

    sudo apt-mark hold kubelet kubeadm kubectl

    sudo systemctl enable --now kubelet

    sudo swapoff -a
    ```

    ```bash
    chmod +x setup_k8s_common.sh
    ./setup_k8s_common.sh
    ```

### Master Node Setup

1. **Initialize Kubernetes Control Plane**:
    ```bash
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16
    ```

2. **Set Up Local `kubectl` Configuration**:
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```

3. **Install Calico for Networking**:
    ```bash
    kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
    ```

4. **Install Ingress Controller (Optional)**:
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
    ```

### Worker Node Setup

1. **Join the worker node to the cluster**:
    - Retrieve the join command from the master node:
    ```bash
    kubeadm token create --print-join-command
    ```

    - Run the join command on each worker node to join them to the cluster.

### Verify the Cluster

1. **Verify the cluster status**:
    ```bash
    kubectl get nodes
    ```

## RBAC Setup

### 1. Clone the Repository
  ```bash
  git clone https://github.com/Pramod858/K8S.git
  ```

### 2. Change Directory and Give Permissions
  ```bash
  cd K8S
  chmod +x *
  ```

  or

  ```bash
  cd K8S
  find . -type f -not -name 'README.md' -exec chmod +x {} +
  ```

### 3. Create Namespace
  ```bash
  kubectl create ns webapps
  ```

### 4. Create Service Account
  ```bash
  kubectl apply -f service_account.yaml
  ```

### 5. Create Role
  ```bash
  kubectl apply -f create_role.yaml
  ```

### 6. Bind Role to Service Account
  ```bash
  kubectl apply -f bind_role_to_service.yaml
  ```

### 7. Create Secret ([Reference](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#create-token))
  ```bash
  kubectl apply -f create_secret.yaml -n webapps
  ```

### 8. Describe Secret
  ```bash
  kubectl describe secret mysecretname -n webapps
  ```

This README should provide the necessary instructions to set up an EKS cluster, a self-hosted Kubernetes cluster, and configure RBAC.

### Notes
- Replace any specific placeholder values (like `ssh-public-key`, `mysecretname`, etc.) with the actual values used in your setup.
- The provided script updates and installs necessary dependencies, initializes the clusters, and sets up networking and access controls.
- Follow and execute the sections sequentially to ensure a smooth setup process.

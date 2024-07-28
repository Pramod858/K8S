# K8S
```bash
sudo apt-get update
sudo apt install unzip
```
## AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
```bash
aws configure
```
## EKSCTL
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
## KUBECTL
Download latest version from [here.](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
```bash
sudo curl --silent --location -o /usr/local/bin/kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.30.0/2024-05-12/bin/linux/amd64/kubectl
sudo chmod +x /usr/local/bin/kubectl
kubectl version --client
```
## Create EKS Cluster
```bash
eksctl create cluster --name=my-eks \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --version=1.30 \
                      --without-nodegroup
```
```bash
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster my-eks \
    --approve
```
### Replace ssh-public-key with your actual key-pair name
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

```bash
eksctl get cluster --name my-eks --region us-east-1
aws eks update-kubeconfig --region us-east-1 --name my-eks
```
## SetUp Self-Hosted Kubernetes

### Step-by-Step Guide

#### 1. Common Setup for Both Master and Worker Nodes

Create a script (`setup_k8s_common.sh`) with the following content and execute it on both the master and worker nodes:

```bash
#!/bin/bash

# Update the package index
sudo apt-get update

# Install Docker
sudo apt install docker.io -y

# Allow non-root users to run Docker commands
sudo chmod 666 /var/run/docker.sock

# Update the package index again
sudo apt-get update

# Install required packages for Kubernetes
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Create the keyrings directory for Kubernetes
sudo mkdir -p -m 755 /etc/apt/keyrings

# Download and add the GPG key for Kubernetes
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes APT repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update the package index with the new repository
sudo apt-get update

# Install kubelet, kubeadm, and kubectl
sudo apt-get install -y kubelet kubeadm kubectl

# Mark them to hold so they are not updated automatically
sudo apt-mark hold kubelet kubeadm kubectl

# Enable and start kubelet
sudo systemctl enable --now kubelet

# Disable swap (required for Kubernetes)
sudo swapoff -a
```

Make the script executable and run it on both the master and worker nodes:

```bash
chmod +x setup_k8s_common.sh
./setup_k8s_common.sh
```

#### 2. Master Node Setup

After running the common setup script above, continue with the following steps on the master node:

1. **Initialize the Kubernetes Control Plane**

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

2. **Set Up Local `kubectl` Configuration**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

3. **Install Calico for Networking**

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

4. **Install Ingress Controller (Optional)**

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
```

#### 3. Worker Node Setup

On each worker node, join the cluster using the command provided by `kubeadm init`. You can retrieve this command by running the following on the master node if you didn't save it:

```bash
kubeadm token create --print-join-command
```

The output will look something like this:

```bash
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Run this join command on each worker node to join them to the cluster.

#### 4. Verify the Cluster

Once the worker nodes have joined, verify the cluster status by running the following command on the master node:

```bash
kubectl get nodes
```

You should see all the nodes (master and workers) listed with the status `Ready`.

### Summary

1. **Run the `setup_k8s_common.sh` script on all nodes (both master and worker).**
2. **Initialize the Kubernetes control plane on the master node.**
3. **Set up `kubectl` on the master node.**
4. **Install Calico for networking on the master node.**
5. **Optionally, install an ingress controller on the master node.**
6. **Join each worker node to the cluster using the join command provided by `kubeadm`.**
7. **Verify the cluster status using `kubectl get nodes` on the master node.**

This guide provides a complete setup for a self-hosted Kubernetes cluster using `kubeadm`. Make sure to execute the commands with appropriate permissions and ensure network connectivity between all nodes.


## Steps to set RBAC
### 1. Clone the repo
```bash
git clone https://github.com/Pramod858/K8S.git
```

### 2.Change the dir and give permission
```bash
cd K8S
chmod +x *
````
or
```bash
cd K8S
find . -type f -not -name 'README.md' -exec chmod +x {} +
```

### 3. Create Name Space
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

### 7. Create Secret ([Ref.](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#create-token))
```bash
kubectl apply -f create_secret.yaml -n webapps
```

### 8. Describe Secrete
```bash
kubectl describe secret mysecretname -n webapps
```

# K8S
## AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
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
## Steps

### 1. Create Name Space
```bash
kubectl create ns webapps
```

### 2. Create Service Account
```bash
kubectl apply -f service_account.yaml
```

### 3. Create Role
```bash
kubectl apply -f create_role.yaml
```

### 4. Bind Role to Service Account
```bash
kubectl apply -f bind_role_to_service.yaml
```

### 5. Create Secret ([Ref.](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#create-token))
```bash
kubectl apply -f create_secret.yaml -n webapps
```

### 6. Describe Secrete
```bash
kubectl describe secret mysecretname -n webapps
```

# K8S
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
    --region us east-1 \
    --cluster my-eks \
    --approve
```
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

# K8S

## Steps

### 1. Create Name Space
```bash
kubectl create ns webapps
```

### 1. Create Service Account
```bash
kubectl apply -f service_account.yaml
```

### 2. Create Role
```bash
kubectl apply -f create_role.yaml
```

### 4. Bind Role to Service Account
```bash
kubectl apply -f bind_role_to_service.yaml
```

### 5. Create token [Ref].(https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#create-token)
```bash
kubectl apply -f create_token.yaml -n webapps
```
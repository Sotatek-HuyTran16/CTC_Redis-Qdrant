# 1. Fetch chart

```
helm repo add bitnami https://charts.bitnami.com/bitnami

helm repo update

helm pull bitnami/redis --version 24.0.0

helm push redis-24.0.0.tgz oci://registry.dev-dify.ctc.local:8080/dev/database/ --plain-http
```

# 2. Create secret

```bash
apiVersion: v1
kind: Secret
metadata:
  name: redis-default-user
  namespace: dev-redis
type: Opaque
stringData:
  password: sotatek

---
apiVersion: v1
kind: Secret
metadata:
  name: redis-acl-secret
  namespace: dev-redis
type: Opaque
stringData:
  admin: sotatek
```

# 3. Create values.yaml file

# 4. Install

```bash
sudo k8s helm install redis \ 
  oci://registry.dev-dify.ctc.local:8080/dev/database/redis:24.0.0
  -f values.yaml
  --plain-http -n dev-redis
```
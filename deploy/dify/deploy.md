# 1. Fetch chart

```
helm repo add dify https://langgenius.github.io/dify-helm

helm repo update

helm pull dify/dify --version 3.5.6

helm push dify-3.5.6.tgz oci://registry.dev-dify.ctc.local:8080/dev/ai/ --plain-http
```

# 2. Create following

## minio service

```bash
apiVersion: v1
kind: Service
metadata:
  name: dify-minio
  namespace: dev-dify
spec:
  type: ExternalName
  externalName: minio.dev-dify.ctc.local
```

# SA

```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-dify-api-sa
  namespace: dev-dify

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-dify-worker-sa
  namespace: dev-dify

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-dify-worker-beat-sa
  namespace: dev-dify

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-dify-sandbox-sa
  namespace: dev-dify

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-dify-enterprise-api-sa
  namespace: dev-dify

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-dify-enterprise-web-sa
  namespace: dev-dify

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-dify-ssrf-proxy-sa
  namespace: dev-dify

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-dify-unstructured-sa
  namespace: dev-dify
```

## Create role and role binding

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dify-enterprise-namespace-reader
  namespace: dev-dify
rules:
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dify-enterprise-namespace-reader-binding
  namespace: dev-dify
subjects:
- kind: ServiceAccount
  name: dev-dify-enterprise-api-sa
  namespace: dev-dify
roleRef:
  kind: Role
  name: dify-enterprise-namespace-reader
  apiGroup: rbac.authorization.k8s.io
```

# 3. Create values.yaml file

# 4. Install

```bash
sudo k8s helm install dify \ 
  oci://registry.dev-dify.ctc.local:8080/dev/ai/dify:3.5.6
  -f values.yaml
  -n dev-dify
  --plain-http
```

# 5. Edit secret 

## Edit the dify-shared-celery-secret that used in deployment dify-worker

## Add the following env variables

```bash
# add this line
CELERY_USE_SENTINEL: true
CELERY_SENTINEL_MASTER_NAME: mymaster

# edit this line to
CELERY_BROKER_URL: sentinel://:sotatek@redis.dev-redis.svc.cluster.local:26379/0

# sentinel://:REDIS_PASSWORD@redis.dev-redis.svc.cluster.local:26379/1
```

# 6. Edit config map

## Edit the sandbox env variables

## Add this env

```bash
PIP_TRUSTED_HOST=pypi.dev-dify.ctc.local

PIP_INDEX_URL=http://pypi.dev-dify.ctc.local/simple

NO_PROXY=pypi.dev-dify.ctc.local
```

# 7. Edit configmap dify-gateway-config (caddy file) (nodeport)

```bash
# 1. Change all the section dify-web-svc to dify-enterprise-frontend-svc
reverse_proxy http://dify-enterprise-frontend-svc:3000

# 2. Add the header to all the domain
Access-Control-Allow-Origin  http://console.dify.local:30007
Access-Control-Allow-Methods "GET, POST, PUT, PATCH, DELETE, OPTIONS"
Access-Control-Allow-Headers "Authorization, Content-Type, Accept, Origin, X-Requested-With"
Access-Control-Allow-Credentials true
Access-Control-Max-Age 86400
```

## Continue edit the configmap dify-enterprise-frontend-config

```bash
# change the URL and add the port
data:
  API_URL: http://enterprise.dify.local:30007
  APP_API_URL: http://app.dify.local:30007
  AUDIT_LOG_MAX_SEARCH_INTERVAL: '90'
  CONSOLE_API_URL: http://console.dify.local
  RELEASE_VERSION: 3.5.6 (Helm)
```

# 8. Install Metal LB

```bash
# Link https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml

k create -f metallb-native.yaml
```

# 9. Install ingress

```bash
# Link https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml

k create -f deploy.yaml
```



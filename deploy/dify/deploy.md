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

# 7. Edit configmap dify-gateway-config

```bash
reverse_proxy http://dify-enterprise-frontend-svc:3000
```

# 8. Metal LB

```
https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
```

{"level":"error","ts":1770201606.982037,"logger":"http.log.error","msg":"dial tcp: lookup dify-web-svc on 10.152.183.224:53: server misbehaving","request":{"remote_ip":"172.16.20.2","remote_port":"53622","client_ip":"172.16.20.2","proto":"HTTP/1.1","method":"GET","host":"console.dify.local:30007","uri":"/","headers":{"Upgrade-Insecure-Requests":["1"],"User-Agent":["Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:147.0) Gecko/20100101 Firefox/147.0"],"Accept":["text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"],"Accept-Encoding":["gzip, deflate"],"Sec-Gpc":["1"],"Connection":["keep-alive"],"Priority":["u=0, i"],"Accept-Language":["en-US,en;q=0.9"]}},"duration":0.001402494,"status":502,"err_id":"m25rsn78h","err_trace":"reverseproxy.statusError (reverseproxy.go:1373)"}

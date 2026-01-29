# 1. Fetch chart

```
helm repo add qdrant https://qdrant.github.io/qdrant-helm

helm repo update

helm pull qdrant/qdrant --version 1.16.3

helm push qdrant-1.16.3.tgz oci://registry.dev-dify.ctc.local:8080/dev/database/ --plain-http
```

# 2. Create secret

```bash
apiVersion: v1
kind: Secret
metadata:
  name: dev-qdrant-api-key
  namespace: dev-qdrant
type: Opaque
stringData:
  QDRANT__SERVICE__API_KEY: 487d61df11d7a4c2a3d2308a67dce38e0137414ad1b01a42a2091a7cf5d6fd54
```

# 3. Create values.yaml file

# 4. Install

```bash
sudo k8s helm install qdrant \ 
  oci://registry.dev-dify.ctc.local:8080/dev/database/qdrant:1.16.3
  -f values.yaml
  --plain-http -n dev-qdrant
```
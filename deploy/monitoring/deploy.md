# 1. Fetch chart

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm pull prometheus-community/kube-prometheus-stack --version 80.14.2

helm push kube-prometheus-stack-80.14.2.tgz oci://registry.dev-dify.ctc.local:8080/dev/monitoring/ --plain-http
```

# 2. Create values.yaml file


# 3. Install

```bash
sudo k8s helm install redis \ 
  oci://registry.dev-dify.ctc.local:8080/dev/database/redis:24.0.0
  -f values.yaml
  --plain-http -n dev-redis
```

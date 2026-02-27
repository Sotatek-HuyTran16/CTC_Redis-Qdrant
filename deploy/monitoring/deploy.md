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
helm install kube-prometheus-stack oci://registry.dev-dify.ctc.local:8080/dev/monitoring/kube-prometheus-stack:80.14.2-custom.20260226 -f values.yaml -n dev-monitoring
```

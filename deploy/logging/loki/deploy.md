# 1. Fetch chart

```
helm repo add grafana https://grafana.github.io/helm-charts

helm repo update

helm pull grafana/loki --version 6.49.0

helm push loki-6.49.0.tgz oci://registry.dev-dify.ctc.local:8080/dev/monitoring --plain-http
```

# 2. Create values.yaml file


# 3. Install

```bash
sudo k8s helm install loki oci://registry.dev-dify.ctc.local:8080/dev/monitoring/loki:6.49.0
  -f values.yaml
  --plain-http -n dev-monitoring
```

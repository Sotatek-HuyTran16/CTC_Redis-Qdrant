# 1. Fetch chart

```bash
helm repo add grafana https://grafana.github.io/helm-charts

helm repo update

helm pull grafana/promtail --version 6.17.1

helm push promtail-6.17.1.tgz oci://registry.dev-dify.ctc.local:8080/dev/monitoring --plain-http
```

# 2. Create values.yaml file


# 3. Install

```bash
sudo k8s helm install promtail \ 
  oci://registry.dev-dify.ctc.local:8080/dev/monitoring/promtail:6.17.1
  -f values.yaml
  --plain-http -n dev-monitoring
```

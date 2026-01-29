# 1. Fetch chart

```
helm repo add percona https://percona.github.io/percona-helm-charts/

helm repo update

helm pull percona/pg-operator --version 2.8.0

helm pull percona/pg-db --version 2.8.0

helm push pg-operator-2.8.0.tgz oci://registry.dev-dify.ctc.local:8080/dev/database/ --plain-http

helm push pg-db-2.8.0.tgz oci://registry.dev-dify.ctc.local:8080/dev/database/ --plain-http
```

# 2. Create values.yaml file

# 3. Install

```bash
sudo k8s helm install pg-operator \ 
  oci://registry.dev-dify.ctc.local:8080/dev/database/pg-operator:2.8.0
  -f pg-operator-values.yaml
  --plain-http -n dev-postgresql
```

```bash
sudo k8s helm install pg-db \ 
  oci://registry.dev-dify.ctc.local:8080/dev/database/pg-db:2.8.0
  -f pg-db-values.yaml
  --plain-http -n dev-postgresql
```
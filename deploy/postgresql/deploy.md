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
helm install dev-pg-operator oci://registry.dev-dify.ctc.local:8080/dev/database/pg-operator:2.8.0-custom.20260226 -f values-operator.yaml -n dev-postgresql
```

```bash
helm install dev-pg-db oci://registry.dev-dify.ctc.local:8080/dev/database/pg-db:2.8.0-custom.20260226 -f values-db.yaml -n dev-postgresql
```
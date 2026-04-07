# 1. Fetch chart

```
helm repo add percona https://percona.github.io/percona-helm-charts/

helm repo update

helm pull percona/pg-operator --version 2.8.0

helm pull percona/pg-db --version 2.8.0

helm push pg-operator-2.8.0.tgz oci://registry.dev-dify.ctc.local:8080/dev/database/ --plain-http

helm push pg-db-2.8.0.tgz oci://registry.dev-dify.ctc.local:8080/dev/database/ --plain-http
```

# 2. Create secret file

```bash
apiVersion: v1
kind: Secret
metadata:
  name: cluster1-pgbackrest-secrets
  namespace: dev-postgresql
type: Opaque
data:
  s3.conf: W2dsb2JhbF0KcmVwbzItczMta2V5PTF5VWVQbGt6b3pqcXF5ZDhja2xwCnJlcG8yLXMzLWtleS1zZWNyZXQ9TmVqUDhoM0laa05HbjNXRnFad2VjTFNJRWxaM2xleU9wSVdJMkF6VAo=
```

# 3. Create values.yaml file

# 4. Install

```bash
helm install dev-pg-operator oci://registry.dev-dify.ctc.local:8080/dev/database/pg-operator:2.8.0-custom.20260226 -f values-operator.yaml -n dev-postgresql
```

```bash
helm install dev-pg-db oci://registry.dev-dify.ctc.local:8080/dev/database/pg-db:2.8.0-custom.20260226 -f values-db.yaml -n dev-postgresql
```
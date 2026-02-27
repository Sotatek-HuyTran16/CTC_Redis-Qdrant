# 1. Apply this

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-exporter-to-pg
  namespace: dev-postgresql
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/component: pg
  policyTypes:
    - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: dev-monitoring
      podSelector:
        matchLabels:
          app.kubernetes.io/instance: prometheus-postgres-exporter
    ports:
    - protocol: TCP
      port: 5432
```

# 2. Install

```bash
helm install dev-pg-operator oci://registry.dev-dify.ctc.local:8080/dev/monitoring/prometheus-postgres-exporter:7.4.0 -f values.yaml -n dev-monitoring
```
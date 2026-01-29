# How to forward-port

```bash
k port-forward \
  -n dev-monitoring \
  --address 0.0.0.0 \
  svc/kube-prometheus-stack-prometheus \
  9090:9090
```
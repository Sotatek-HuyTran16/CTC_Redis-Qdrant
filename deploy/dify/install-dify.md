# 1. Install

## Create values file

```

```

## Install

```
sudo k8s helm install dify \ 
  oci://registry.dev-dify.ctc.local:8080/dify/dify
  -f values.yaml
  --version 3.7.3
  --plain-http -n dev-dify
```
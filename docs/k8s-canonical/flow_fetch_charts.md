# 1. Login to HTTP Harbor registry 

```
helm registry login registry.dev-dify.ctc.local:8080 --plain-http
```

# 2. Pull helm chart (from Download VM)

## Update helm repo

```
helm repo add projectcalico https://docs.tigera.io/calico/charts

helm repo update
```

## Verify

```
helm repo list

helm search repo projectcalico 
```

## Pull

```
helm pull projectcalico/tigera-operator --version ...
```

# 3. Push

## Create project on Harbor UI

## Push chart

```
helm push tigera-operator-v3.31.3.tgz oci://registry.dev-dify.ctc.local:8080/tigera --plain-http
```
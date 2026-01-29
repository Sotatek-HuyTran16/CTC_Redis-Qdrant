# 1. Pod can't resolve DNS

## Description

```
Pod can resolve internal service like `kubernetes.default`

Pod can't resolve external domain (google.com and minio.dev-dify)

Pod can ping 8.8.8.8 and 10.10.10.10 but can't telnet 53 and resolve query
```

## Root cause

```
ufw firewall rule default deny routed -> coreDNS can't route to Bind DNS
```

## Solution

```bash
# do on all node
sudo ufw default allow routed
```

# 2. Some pod use persistent is pending not because of node resource

## Description

```bash
# When deploy kube-prometheus-stack with helm

# Some workload are pending
```

## Root cause

```bash
# StorageClass binding mode is Immediate

# -> PVC and PV is created before pod is scheduled

# But we have affinity to assign pod to a specific node

# pod and PVC can't on the same node

# -> schedule error
```

## Solution

```bash
# Create new StorageClass with Binding Mode - WaitForFirstConsumer
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: openebs-lvmpv-wffc
provisioner: local.csi.openebs.io
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

```bash
# Then edit values.yaml file to use this new SC
```
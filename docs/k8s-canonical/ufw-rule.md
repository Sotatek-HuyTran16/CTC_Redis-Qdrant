# 1. All node

```bash
# allow FWD pod CIDR connect to other pod
sudo ufw route allow from 10.1.0.0/16 to 10.1.0.0/16

# allow from node to node
sudo ufw allow from 10.10.0.0/24 to 10.10.0.0/24
```

```bash
# localhost helath check endpoint
sudo ufw allow 10248/tcp

# localhost for metric endpoint (no need)
sudo ufw allow 10249/tcp

# kubelet API
sudo ufw allow 10250/tcp

# kube-proxy port for binding the health check server
sudo ufw allow 10256/tcp

# Default REST API port for Canonical Kubernetes daemon.
sudo ufw allow 6400/tcp

# Kubernetes API server. SSL encrypted.
sudo ufw allow 6443/tcp

# etcd
sudo ufw allow 2379/tcp

# etcd
sudo ufw allow 2380/tcp

# k8s-dqlite
sudo ufw allow 9000/tcp

# kube-controller
sudo ufw allow 10257/tcp

# kube-scheduler
sudo ufw allow 10259/tcp
```

## Calico CNI

```bash
# calico
sudo ufw allow 179/tcp

# calico
sudo ufw allow 4789/tcp

# calico enable typha
sudo ufw allow 5473/tcp
```

# 2. Worker node only

```bash
# NodePort
ufw allow 30000:32767/tcp

# NodePort
ufw allow 30000:32767/udp
```

# 3. All node (for external service)

```bash
# registry
sudo ufw allow 8080/tcp

sudo ufw allow 8081/tcp

sudo ufw allow 443/tcp
```

```bash
# bind exporter
sudo ufw allow 9119/tcp

# harbor exporter
sudo ufw allow 9090/tcp
```

## CIDR

```bash
# pod CIDR
sudo ufw allow in from 10.1.0.0/24
sudo ufw allow out to 10.1.0.0/24

# service CIDR
sudo ufw allow in from 10.152.183.0/24
sudo ufw allow out to 10.152.183.0/24

# cluster node CIDR
sudo ufw allow in from 10.10.0.0/24
sudo ufw allow out to 10.10.0.0/24

# external service VM CIDR
sudo ufw allow in from 10.10.10.0/24
sudo ufw allow out to 10.10.10.0/24
```

# 4. Default (all node)

```bash
sudo ufw default deny incoming

sudo ufw default deny routed

sudo ufw default allow outgoing
```
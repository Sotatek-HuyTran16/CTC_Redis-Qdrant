# Master node

## Enable traffic to artifact VM va nguoc lai (ufw)

```
sudo ufw allow out to 10.10.10.0/24

sudo ufw allow from 10.10.10.0/24
```

## Install k8s

```bash
wget http://10.10.10.13:80/data/repository/snap/k8s.snap

# tuong tu voi cac file con lai

cd ~/k8s_install

sudo snap ack core22.assert

sudo snap install core22.snap

sudo snap ack k8s.assert

sudo snap install k8s.snap --classic
```

## Bootstrap cluster

```
sudo k8s bootstrap
```

## Config snap.k8s.containerd

### Edit folder /etc/container/hosts.d and add file

```
mkdir ghcr.io/

cd ghcr.io/

nano hosts.toml
```

### Add following content to mirror to Harbor 

```bash
server = "https://ghcr.io"

[host."http://10.10.10.13:8080"]
  capabilities = ["pull", "resolve"]
  skip_verify = true
```

### Reload containerd

```
systemctl daemon-reload

systemctl restart snap.k8s.containerd
```

## Change CNI plugin

### Chuan bi image trong Harbor registry

```
docker pull docker.io/calico/node:v3.27.2

docker tag docker.io/calico/node:v3.27.2 \
10.10.10.13:8080/calico/node:v3.27.2

docker login 10.10.10.13:8080

docker push 10.10.10.13:8080/calico/node:v3.27.2
```

### Edit registry 

```bash
# edit /etc/containerd/hosts.d/dockker.io/hosts.toml

server = "https://docker.io"

[host."http://10.10.10.13:8080"]
  capabilities = ["pull", "resolve"]
  skip_verify = true
```

```
systemctl daemon-reload

systemctl restart snap.k8s.containerd
```

### Remove Cilium

```
k delete -n kube-system ds cilium

k delete -n kube-system deploy cilium-operator

k delete crd $(k get crd | grep cilium | awk '{print $1}')

rm -f /etc/cni/net.d/05-cilium.conflist
```

### Install Calico

```bash
wget https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml

k apply -f calico.yaml
```

### Restart pod

```
k delete pod -A --all
```

## Open ufw port rule for k8s with Calico

```
sudo ufw allow from 10.10.0.0/24

sudo ufw allow out to 10.10.0.0/24

sudo ufw allow from 10.10.10.0/24

sudo ufw allow out to 10.10.10.0/24

sudo ufw allow from 192.168.0.0/16

sudo ufw allow out to 192.168.0.0/16
```
# Guide to remove old cluster

## Delete k8s snap

```bash
sudo snap stop k8s || true

sudo snap remove k8s --purge
```

Check if removed

```
snap list | grep k8s || echo "k8s snap removed"
```

## Clean folder

```
sudo rm -rf \
  /var/snap/k8s \
  /var/lib/k8s \
  /etc/k8s \
  /var/lib/etcd \
  /var/lib/kubelet \
  /var/lib/containerd \
  ~/.kube
```

## Clean services

```
sudo systemctl stop \
  snap.k8s.k8s-apiserver-proxy.service \
  snap.k8s.k8s-dqlite.service \
  snap.k8s.kube-apiserver.service || true

sudo systemctl disable \
  snap.k8s.k8s-apiserver-proxy.service \
  snap.k8s.k8s-dqlite.service \
  snap.k8s.kube-apiserver.service || true
```

```
sudo rm -rf /etc/systemd/system/snap.k8s.*
sudo rm -f /run/systemd/system/snap.k8s.*
sudo rm -f /lib/systemd/system/snap.k8s.*

sudo rm -rf /run/containerd/
```

## Reset state

```
sudo systemctl reset-failed 'snap.k8s.*'
```

```
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
```
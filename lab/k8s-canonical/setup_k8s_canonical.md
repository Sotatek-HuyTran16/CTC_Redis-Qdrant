# 1. Bootstrap cluster

## 1.1. From download vm

## Download snap

Create folder

```
mkdir -p ~/snap/install

cd snap/install/
```

Download

```bash
sudo snap download k8s --channel 1.34-classic/stable --basename k8s
```

Change owner

```
sudo chown -R sotatek:sotatek k8s-install/
```

Archive and send to master vm

```
tar -czvf k8s-install.tar.gz k8s-install/

scp k8s-install.tar.gz sotatek@10.10.0.10:/tmp
```

# 1.2. From master node

## Untar

```
cd /tmp

tar -xzvf k8s-install.tar.gz

sudo cp -r k8s-install ~/k8s-install
```

## Install snap k8s

```
cd ~/k8s-install

sudo snap ack k8s.assert

sudo snap install k8s.snap --classic
```

## Fetch images

List images need to bootstrap

```
sudo snap run k8s list-images
```

From download vm

```bash
fetch-images/script.sh bootstrap-images.txt
```

## Bootstrap

```
sudo k8s bootstrap
```

# 2. Config cluster

## 2.1. Config containerd to use Harbor registry httpsS

## Copy cert to folders

```bash
/var/snap/k8s/common/etc/containerd/hosts.d/ghcr.io/ca.crt

/var/snap/k8s/common/etc/containerd/certs.d/registry.dev-dify.ctc.local/ca.crt

/etc/containerd/hosts.d/ghcr.io/ca.crt

/usr/local/share/ca-certificates/ca.crt
```

## Config containerd

```bash
# nano /var/snap/k8s/common/etc/containerd/config.toml

version = 2

[plugins]

  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "registry.k8s.io/pause:3.10"

    [plugins."io.containerd.grpc.v1.cri".containerd]
      snapshotter = "overlayfs"
      default_runtime_name = "runc"

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"

    # ===============================
    # 🔥 REGISTRY CONFIG (QUAN TRỌNG)
    # ===============================
    [plugins."io.containerd.grpc.v1.cri".registry]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

        # Harbor HTTPS (port 443, không cần endpoint 8080)
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.dev-dify.ctc.local"]
          endpoint = ["https://registry.dev-dify.ctc.local"]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]

        # Trust CA cert (SAN cert đã tạo)
        [plugins."io.containerd.grpc.v1.cri".registry.configs."registry.dev-dify.ctc.local".tls]
          ca_file = "/var/snap/k8s/common/etc/containerd/hosts.d/ghcr.io/ca.crt"
```

## Edit file config

```bash
# nano /etc/containerd/hosts.d/ghcr.io/hosts.toml

[host."https://registry.dev-dify.ctc.local"]
capabilities = ["pull", "resolve"]
ca = "/var/snap/k8s/common/etc/containerd/certs.d/registry.dev-dify.ctc.local/ca.crt"
```

## Update cert (do it on all node)

```bash
# ensure ca.crt in folder /usr/local/share/ca-certificates

sudo update-ca-certificates
```
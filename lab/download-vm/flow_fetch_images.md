# 1. From Download VM

## Install docker

```
sudo apt update

sudo apt install docker.io
```

## Add user to docker group (needed)

```bash
getent group docker

sudo usermod -aG docker $USER

# check again (should have docker)
groups
```

## Setup allow http (skip if Harbor use https)

```
sudo nano /etc/docker/daemon.json
```

```
{
  "insecure-registries": ["10.10.10.13:8080"]
}
```

```
sudo systemctl restart docker
```

## Login to Harbor

```
docker login 10.10.10.13
```

## Create script running file (http version)

Create `sync-images-to-harbor.sh` file

```bash
#!/usr/bin/env bash
set -euo pipefail

IMAGE_FILE="${1:-images.txt}"
HARBOR="10.10.10.13:8080"

if [[ ! -f "$IMAGE_FILE" ]]; then
  echo "❌ File not found: $IMAGE_FILE"
  exit 1
fi

echo "🚀 Start syncing images to Harbor: $HARBOR"
echo "----------------------------------------"

while read -r IMAGE; do
  [[ -z "$IMAGE" ]] && continue
  [[ "$IMAGE" =~ ^# ]] && continue

  echo "➡ Processing: $IMAGE"

  # Pull image
  docker pull "$IMAGE"

  # Transform image name for Harbor
  # registry.k8s.io/kube-apiserver:v1.29.2
  # -> 10.10.10.13:8080/registry.k8s.io/kube-apiserver:v1.29.2
  TARGET_IMAGE="$HARBOR/$IMAGE"

  # Tag
  docker tag "$IMAGE" "$TARGET_IMAGE"

  # Push
  docker push "$TARGET_IMAGE"

  echo "✅ Pushed: $TARGET_IMAGE"
  echo
done < "$IMAGE_FILE"

echo "🎉 Done. All images synced to Harbor."
```

## Create script running file (https version)

```
something
```

## Fetch image to Harbor

```bash
# create bootstrap-images.txt file

ghcr.io/canonical/cilium-operator-generic:1.17.9-ck4
ghcr.io/canonical/cilium:1.17.9-ck9
ghcr.io/canonical/coredns:1.12.4-ck1
ghcr.io/canonical/csi-node-driver-registrar:2.11.1-ck7
ghcr.io/canonical/csi-provisioner:5.0.2-ck1
ghcr.io/canonical/csi-resizer:1.11.2-ck1
ghcr.io/canonical/csi-snapshotter:8.0.2-ck1
ghcr.io/canonical/frr:9.1.3-ck5
ghcr.io/canonical/k8s-snap/pause:3.10
ghcr.io/canonical/metallb-controller:v0.14.9-ck4
ghcr.io/canonical/metallb-speaker:v0.14.9-ck4
ghcr.io/canonical/metrics-server:0.8.0-ck4
ghcr.io/canonical/rawfile-localpv:0.8.2-ck3
```

Run command

```
./sync-images-to-harbor.sh bootstrap-images.txt
```
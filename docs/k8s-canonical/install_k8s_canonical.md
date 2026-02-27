# 1. Install snap k8s command

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

## 1.2. From master node

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

# 1.3 From all worker node

```bash
# Pull the k8s.assert and k8s.snap file first

sudo snap ack k8s.assert

sudo snap install k8s.snap --classic
```

# 2. Bootstrap temporary cluster

## Bootstrap with custom config

Option 1: Interactive mode

[Official Docs](https://documentation.ubuntu.com/canonical-kubernetes/latest/snap/howto/install/custom-bootstrap-config/)

```
sudo k8s bootstrap --interactive
```

Option 2: File bootstrap

[Official Docs](https://documentation.ubuntu.com/canonical-kubernetes/latest/snap/reference/config-files/bootstrap-config/)

```
cluster-config:
  network:
    enabled: false
  dns:
    enabled: true
    cluster-domain: cluster.local
  ingress:
    enabled: false
  load-balancer:
    enabled: false
  local-storage:
    enabled: false
    default: false
  gateway:
    enabled: false
  metrics-server:
    enabled: false
  cloud-provider: external
control-plane-taints:
- node-role.kubernetes.io/control-plane:NoSchedule
pod-cidr: 10.1.0.0/16
service-cidr: 10.152.183.0/24
```

```
sudo k8s bootstrap --file bootstrap-config.yaml
```

## Alias command (if needed)

```bash
# nano ~/.bashrc

alias k='sudo k8s kubectl'

alias kctr='sudo /snap/k8s/current/bin/ctr --address /run/containerd/containerd.sock'
```

Apply

```
source ~/.bashrc
```

# 3. Fetch bootstrap images and calico images

Do the setup in file: `flow_fetch_images.md`

# 4. Config containerd to mirror to private registry (do this on all cluster node)

## Create file

```bash
# nano /etc/containerd/hosts.d/ghcr.io/hosts.toml

[host."http://registry.dev-dify.ctc.local:8080"]
capabilities = ["pull", "resolve"]
```

Do the same with `quay.io` and `docker.io`

## Test pull image from ctr

[Github Issue](https://github.com/containerd/containerd/discussions/10909)

```bash
kctr -n default images pull --plain-http registry.dev-dify.ctc.local:8080/canonical/k8s-snap/pause:3.10
```

# 5. Install Calico CNI to cluster (do this in master node)

## Curl calico manifest file

```
curl -LO https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/operator-crds.yaml

curl -LO https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/tigera-operator.yaml

curl -LO https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/custom-resources-bpf.yaml
```

## Edit manifest to use specific image

```bash
# Here ensure the manifest use image tigera-operator v1.40.3

# Also ensure pod CIDR by edit below file
# nano custom-resources-bpf.yaml
```

## Apply Calico CNI

```
k create -f operator-crds.yaml

k create -f tigera-operator.yaml

k create -f custom-resources-bpf.yaml
```

# 6. Join node

## From master

```
sudo k8s get-join-token --worker
```

## From worker

```bash
sudo k8s join-cluster <join-token>
```

# 7. Config ufw firewall rule on all node

[Official Docs](https://documentation.ubuntu.com/canonical-kubernetes/latest/snap/howto/networking/ufw/)

[Stack overflow](https://stackoverflow.com/questions/60970433/ufw-firewall-blocks-kubernetes-with-calico)

## Allow outgoing

```bash
sudo ufw default allow outgoing

sudo ufw default deny incoming

sudo ufw default deny routed
```

## Allow port

```bash
sudo ufw allow 6443/tcp

sudo ufw allow 10257/tcp
sudo ufw allow 10259/tcp
```

```
sudo ufw allow 22/tcp

sudo ufw allow 179/tcp
sudo ufw allow 379/tcp
sudo ufw allow 443/tcp

sudo ufw allow 2380/tcp
sudo ufw allow 2379/tcp

sudo ufw allow 4149/tcp 
sudo ufw allow 4789/tcp

sudo ufw allow 5473/tcp                         

sudo ufw allow 6400/tcp

sudo ufw allow 9000/tcp
sudo ufw allow 9099/tcp

sudo ufw allow 10250/tcp

sudo ufw allow 10255/tcp
sudo ufw allow 10256/tcp
```

## Allow CIDR

```
sudo ufw allow in from 10.1.0.0/24
sudo ufw allow out to 10.1.0.0/24

sudo ufw allow in from 10.152.183.0/24
sudo ufw allow out to 10.152.183.0/24

sudo ufw route allow from 10.1.0.0/16 to 10.10.0.0/24
sudo ufw route allow from 10.1.0.0/16 to 10.1.0.0/16
```

# 8. Setup to use private registry Harbor (all node)

## 7.1. Image

## Create file

```bash
# sudo nano /etc/containerd/hosts.d/registry.dev-dify.ctc.local/hosts.toml

server = "http://registry.dev-dify.ctc.local"

[host."http://registry.dev-dify.ctc.local:8080"]
capabilities = ["pull", "resolve"]
```

## Apply

```
sudo systemctl restart snap.k8s.containerd.service
```

## Test

```bash
# nano test-pull.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test-insecure-registry
spec:
  containers:
    - name: pause
      image: registry.dev-dify.ctc.local/canonical/k8s-snap/pause:3.10
      imagePullPolicy: Always
      ports:
        - containerPort: 80
```

```
k create -f test-pull.yaml
```

## 7.2. Charts

## Login (if needed)

```
sudo k8s helm registry login --plain-http \
registry.dev-dify.ctc.local:8080 
```

## Apply chart from Harbor

```
sudo k8s helm install tigera-operator \
  oci://registry.dev-dify.ctc.local:8080/tigera/tigera-operator \
  --plain-http
  --version v3.31.3
  -n tigera-operator
  --create-namespace
```

# 8. Setup OpenEBS lvm-localpv (all node)

## 8.1. Prepare

## Install packages

```
sudo apt install -y lvm2 thin-provisioning-tools
```

## Add kernel module

```
sudo modprobe dm_thin_pool
sudo modprobe dm_mod
sudo modprobe dm_snapshot
```

## Verify

```
lsmod | grep dm_
```

## Create Volume Group

```
sudo pvcreate /dev/sdb

sudo vgcreate lvm-openebs /dev/sdb
```

## 8.2. Install

## Fetch chart

```
helm repo add openebs-lvmlocalpv https://openebs.github.io/lvm-localpv

helm repo update

helm pull openebs-lvmlocalpv/lvm-localpv

helm push lvm-localpv-1.8.0.tgz oci://registry.dev-dify.ctc.local:8080/openebs --plain-http
```

## Fetch image

Do the setup in file: `flow_fetch_images.md`

## Install in cluster

```
sudo k8s helm install openebs-lvm-localpv \ 
  oci://registry.dev-dify.ctc.local:8080/openebs/lvm-localpv:1.8.0
  --plain-http -n openebs --create-namespace
```

## Verify

```
k get pod -n openebs
```

## Create storage class

```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: openebs-lvmpv-wffc
parameters:
  storage: lvm
  volgroup: lvm-openebs
provisioner: local.csi.openebs.io
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

## Verify

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: csi-lvmpv
  namespace: default
spec:
  storageClassName: openebs-lvmpv
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```
k create -f test-pvc.yaml
```
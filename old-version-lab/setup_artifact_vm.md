# Setup Harbor registry + Local Repo

Harbor - store image for cluster

Local Repo - snap package, apt package

## Step 1: Setup Local Repo + Habor VM

## Two option

### - Using .deb files
### - Using binary files

### Download snap k8s package

```bash
# old version
snap find k8s

snap download k8s

# new version
sudo snap download k8s --channel 1.34-classic/stable --basename k8s

sudo snap download core22 --basename core22
```

### Send snap package

```bash
scp k8s.snap sotatek@10.10.10.13:/data/repository/snap

# tuong tu voi k8s.assert

```

### Download apt nginx package

```
sudo apt install --download-only nginx
```

### Send Nginx package (no need because VM already have Nginx)

```bash
scp /var/cache/apt/archive/nginx*.deb sotatek@10.10.10.13:/data/repository/apt/nginx

# tuong tu voi libnginx*.deb
```

### Install (neu can)

```
cd /data/repository/apt/nginx

sudo dpkg -i nginx*.deb libnginx*.deb
```

### Download docker package

```
sudo apt update
sudo apt install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

```
sudo apt update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### Send docker package

```
scp docker-ce*.deb ...

scp containerd ...
```

### Install docker

```
sudo dpkg -i *.deb
```

### Download docker-compose

```bash
COMPOSE_VERSION=1.29.2

curl -L "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o docker-compose

chmod +x docker-compose
```

### Upload to artifact VM

```
scp -i ~/.ssh/id_rsa docker-compose sotatek@10.10.10.13:/data/repository/apt/docker-compose
```

### Install docker compose

```
cp docker-compose /usr/local/bin/docker-compose
```

### Download helm

```
HELM_VERSION=3.14.0

wget https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz

tar -zxvf helm-v${HELM_VERSION}-linux-amd64.tar.gz
```

### Upload Helm

```
scp -i ~/.ssh/id_rsa helm   sotatek@10.10.10.13:/data/repository/apt/helm
```

### Install helm

```
sudo cp helm /usr/local/bin/helm
```

### Download Harbor

```
wget https://github.com/goharbor/harbor/releases/download/v2.14.0/harbor-offline-installer-v2.14.0.tgz
```

### Upload

```
scp -i ~/.ssh/id_rsa harbor-offline-installer-v2.14.0.tgz sotatek@10.10.10.13:/data/repository/harbor
```

### Install

```bash
tar xvf harbor-offline-installer-v2.14.0.tgz

cp harbor.yml.tmpl harbor.yml

# edit harbor.yml

sudo docker load -i harbor.v2.14.0.tar.gz

sudo ./prepare

sudo ./install.sh
```

# 1. Setup Image Registry Harbor

## 1.1. From Download vm

## Download harbor 

```
wget https://github.com/goharbor/harbor/releases/download/v2.14.0/harbor-offline-installer-v2.14.0.tgz
```

## Copy to artifacts vm

```
scp -i ~/.ssh/id_rsa harbor-offline-installer-v2.14.0.tgz sotatek@10.10.10.13:/home/sotatek/harbor
```

## 1.2. From Artifact VM

## Install docker and docker-compose

```
sudo apt install -y docker.io docker-compose
```

## Prepare

```bash
tar xvf harbor-offline-installer-v2.14.0.tgz

cp harbor.yml.tmpl harbor.yml
```

```bash
# edit harbor.yml this 2 field

hostname: registry.dev-dify.ctc.local

http:
  port: 8080

harbor_admin_password: sotatek
```

## Install Harbor with container

```
sudo docker load -i harbor.v2.14.0.tar.gz

sudo ./prepare

sudo ./install.sh
```


# 2. Setup docker CLI for current user (sotatek)

```bash
# check group docker available
getent group docker

# create docker group if not have
sudo groupadd docker

# add user to gr docker
sudo usermod -aG docker $USER

# Apply
newgrp docker
```

# 3. Config using domain to route to specific port

## 3.1. From Artifacts VM

## Keep the site of repository

```bash
### /etc/nginx/site-availables/apt-repo

server {
    listen 80;
    server_name repository.dev-dify.ctc.internal;

    root /var/www/repository;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

## Add the site for Harbor

```bash
### /etc/nginx/site-availables/harbor

server {
    listen 80;
    server_name registry.dev-dify.ctc.internal;

    client_max_body_size 0;

    location / {
        proxy_pass http://127.0.0.1:8080;

        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Create symlink

```
sudo ln -s /etc/nginx/sites-available/apt-repo /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/harbor   /etc/nginx/sites-enabled/
```

## Reload nginx

```
sudo systemctl reload nginx
```

# 4. Config Harbor to use https

## 4.1. From Artifact VM

## Generate self-signed certs

```
mkdir -p /etc/harbor/certs

cd /etc/harbor/certs
```

```bash
cat > /etc/harbor/certs/harbor-openssl.cnf <<EOF
[ req ]
default_bits       = 4096
prompt             = no
default_md         = sha256
req_extensions     = req_ext
distinguished_name = dn

[ dn ]
C  = VN
ST = HN
L  = HN
O  = CTC
OU = IT
CN = registry.dev-dify.ctc.local

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = registry.dev-dify.ctc.local
IP.1  = 10.10.10.13
EOF

```

```
openssl genrsa -out registry.dev-dify.ctc.local.key 4096

openssl req -x509 -new \
  -key registry.dev-dify.ctc.local.key \
  -out registry.dev-dify.ctc.local.crt \
  -days 3650 \
  -config harbor-openssl.cnf \
  -extensions req_ext
```

## Verify cert

```
openssl x509 -in registry.dev-dify.ctc.local.crt -noout -text | grep -A2 "Subject Alternative Name"
```

## Config Harbor use https

```bash

# edit file harbor.yml again

hostname: registry.dev-dify.ctc.local

http:
  port: 8080

https:
  port: 443
  certificate: /etc/harbor/certs/registry.dev-dify.ctc.local.crt
  private_key: /etc/harbor/certs/registry.dev-dify.ctc.local.key

harbor_admin_password: sotatek
```

## Redeploy

```
sudo ./prepare

sudo ./install.sh
```

# 5. Trust certificate on Client (docker - Download VM)

## 5.1. From Download VM

```
sudo mkdir -p /etc/docker/certs.d/registry.dev-dify.ctc.local
```

```
sudo cp /etc/harbor/certs/registry.dev-dify.ctc.local.crt \
  /etc/docker/certs.d/registry.dev-dify.ctc.local/ca.crt
```

```
sudo systemctl restart docker
```

Login again

```
docker login registry.dev-dify.ctc.local
```
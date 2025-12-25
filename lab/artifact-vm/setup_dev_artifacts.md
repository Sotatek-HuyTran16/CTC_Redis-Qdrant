# 1. Setup Local Repo
## 1.1. From download server

## Download needed software package

```
sudo apt update

sudo apt install --download-only nginx bind9 bind9-utils 
```

## Copy and archives .deb file

```
mkdir -p ~/repo/debs

cp /var/cache/apt/archives/*.deb ~/repo/debs/

tar -czvf apt-debs.tar.gz debs/
```

## Scp to artifacts vm

```
scp apt-debs.tar.gz sotatek@10.10.10.13:/tmp
```

# 1.2. From artifacts VM

## Untar and apply to apt folder

```
cd /tmp

tar -xzvf apt-debs.tar.gz

sudo cp debs/*.deb /var/cache/apt/archives/
```

## Install software needed

Nginx

```
sudo apt install -y nginx
```

dpkg -dev

```
sudo apt install -y dpkg-dev
```

## Expose like a repo 

```
sudo mkdir -p /var/www/apt

sudo cp /var/cache/apt/archives/*.deb /var/www/apt/

cd /var/www/apt/
```


Create metadata

```
sudo dpkg-scanpackages . /dev/null | gzip -9c | sudo tee Packages.gz > /dev/null
```

Create Nginx endpoint

```
sudo nano /etc/nginx/site-availables/apt-repo
```

```
server {
    listen 80;
    server_name repository.dev-dify.ctc.local;

    root /var/www/apt;
    autoindex on;

    location / {
        try_files $uri $uri/ =404;
    }

    location /data/repository/apt/ {
        alias /var/www/apt/;
        autoindex on;
    }
}
```

```
sudo ln -s /etc/nginx/sites-available/apt-repo /etc/nginx/sites-enabled/
```

## Reload

```
sudo nginx -t && sudo systemctl reload nginx
```

## Allow ufw if needed

```
sudo ufw allow from <IP-range>

sudo ufw allow out to <IP-range>
```

# 1.3. From Client VM

## Comment all public apt endpoint 

```
sudo nano /etc/apt/sources.list
```

## Add this line to sources.list

```bash
deb [trusted=yes] http://10.10.10.13:80 ./

# after have Bind DNS and new endpoint

deb [trusted=yes] http://repository.dev-dify.ctc.local/data/repository/apt ./
```

## Try if using local repo

```
sudo apt update
```

## Enable ufw if needed

```
sudo ufw allow from 10.10.10.0/24

sudo ufw allow out to 10.10.10.0/24
```

# 2. Setup Image Registry Harbor

## 2.1. From Download vm

## Download harbor 

```
wget https://github.com/goharbor/harbor/releases/download/v2.14.0/harbor-offline-installer-v2.14.0.tgz
```

## Copy to artifacts vm

```
scp -i ~/.ssh/id_rsa harbor-offline-installer-v2.14.0.tgz sotatek@10.10.10.13:/home/sotatek/harbor
```

## Install docker and docker-compose

```
sudo apt install -y docker.io docker-compose
```

## Apply

```bash
tar xvf harbor-offline-installer-v2.14.0.tgz

cp harbor.yml.tmpl harbor.yml

# edit harbor.yml

sudo docker load -i harbor.v2.14.0.tar.gz

sudo ./prepare

sudo ./install.sh
```


# 3. Update software/package flow

## 3.1. From download vm

```
sudo apt update

sudo apt install --download-only <package-need-update>
```

```
cp /var/cache/apt/archives/*.deb ~/repo/debs/
```

```
cd ~/repo

rm -f apt-update.tar.gz

tar -czvf apt-update.tar.gz debs/

scp apt-update.tar.gz sotatek@10.10.10.13:/tmp
```

## 3.2. From artifacts vm

```
cd /tmp && rm -rf debs/

tar -xzvf apt-update.tar.gz

sudo cp debs/*.deb /var/www/apt/
```

Regenerate metatdata:

```
cd /var/www/apt

sudo dpkg-scanpackages . /dev/null | gzip -9c | sudo tee Packages.gz > /dev/null
```

# 4. Setup docker

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

# 5. Config using domain to route to specific port

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

# 6. To config Harbor and Restart

## Gen certs

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

Verify cert

```
openssl x509 -in registry.dev-dify.ctc.local.crt -noout -text | grep -A2 "Subject Alternative Name"
```

## Config Harbor use https

```bash
hostname: registry.dev-dify.ctc.local

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 8080

# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /etc/harbor/certs/registry.dev-dify.ctc.local.crt
  private_key: /etc/harbor/certs/registry.dev-dify.ctc.local.key
```

## Redeploy

```
sudo ./prepare

sudo ./install.sh
```

## On Client (docker - Download VM)

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
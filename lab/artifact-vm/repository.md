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

# 2. Update software/package flow

## 2.1. From download vm

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

## 2.2. From artifacts vm

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

# 3. Config using domain to route to specific port

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
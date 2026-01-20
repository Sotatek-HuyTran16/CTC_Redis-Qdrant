# 1. Install

## Download binary

```
cd /opt

sudo wget https://github.com/prometheus-community/bind_exporter/releases/download/v0.8.0/bind_exporter-0.8.0.linux-amd64.tar.gz
sudo tar xzf bind_exporter-0.8.0.linux-amd64.tar.gz

sudo mv bind_exporter-0.8.0.linux-amd64/bind_exporter /usr/local/bin/

sudo chmod +x /usr/local/bin/bind_exporter
```

## Check

```
bind_exporter --version
```

# 2. Run

## Create user

```
sudo useradd --no-create-home --shell /usr/sbin/nologin bind_exporter
```

## Create service file

```
sudo nano /etc/systemd/system/bind-exporter.service
```

```
[Unit]
Description=Prometheus BIND Exporter
After=network.target named.service
Wants=named.service

[Service]
Type=simple
User=bind_exporter
Group=bind_exporter

ExecStart=/usr/local/bin/bind_exporter \
  --bind.stats-url=http://127.0.0.1:8053 \
  --web.listen-address=:9119 \
  --log.level=info

Restart=always
RestartSec=5

# Hardening
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ProtectControlGroups=true
ProtectKernelModules=true
ProtectKernelTunables=true
LockPersonality=true
RestrictRealtime=true
RestrictNamespaces=true
MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target
```

## Run

```
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable --now bind-exporter
```

## Verify

```
systemctl status bind-exporter
curl http://localhost:9119/metrics
```

# 3. Enable TLS (optional)

```
sudo mkdir -p /etc/bind-exporter
sudo nano /etc/bind-exporter/web.yml
```

```
basic_auth_users:
  prometheus: "$2y$10$hashedpassword"
```

```
ExecStart=/usr/local/bin/bind_exporter \
  --bind.stats-url=http://127.0.0.1:8053 \
  --web.listen-address=:9119 \
  --web.config.file=/etc/bind-exporter/web.yml
```
# 1. From download server

```
sudo apt update

sudo apt install python3 python3-venv python3-pip
```

```
mkdir ~/pypi && cd pypi/

pip download \
  pypiserver==2.4.0 \
  -d pypi-wheels
```

```
tar -czvf pypi-wheels.tar.gz pypi-wheels/

scp pypi-wheels.tar.gz sotatek@10.10.10.12:/tmp
```

# 2. From PyPi server

## Install software

```
sudo apt update

sudo apt install python3 python3-venv python3-pip
```

## Create specific user

```
sudo useradd -r -s /usr/sbin/nologin dev-pypi-user
```

## Create pypi server working directory

```
sudo mkdir -p /var/lib/dev/pypi/packages /var/log/pypiserver

sudo chown -R dev-pypi-user:dev-pypi-user /var/lib/dev/pypi /var/log/pypiserver
```

## Create venv

```
sudo mkdir /opt/python3

sudo chown -R dev-pypi-user:dev-pypi-user /opt/python3/
```

```
sudo -u dev-pypi-user python3 -m venv /opt/python3/venv

source /opt/python3/venv/bin/activate
```

## Untar file

```
cd /tmp

tar -xzvf pypi-wheels.tar.gz
```

## Install pypi server

```
sudo -u dev-pypi-user /opt/python3/venv/bin/pip install --no-index --find-links=/tmp/pypi-wheels pypiserver==2.4.0
```

## Create systemd service

```bash
### /etc/systemd/system/dev-pypiserver.service

[Unit]
Description=A minimal PyPI server for use with pip
After=network.target

[Service]
Type=simple
User=dev-pypi-user
Group=dev-pypi-user
WorkingDirectory=/var/lib/dev/pypi/packages
ExecStart=/opt/python3/venv/bin/pypi-server run -p 8080 --log-file /var/log/pypiserver/pypiserver.log /var/lib/dev/pypi/packages
Restart=always
TimeoutStartSec=3
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## Enable pypi server

```
sudo systemctl daemon-reload

sudo systemctl enable --now dev-pypiserver
```

# 3. Download python package to PyPI server

```bash
sudo pip download requests==2.32.3 -d /var/lib/dev/pypi/packages
```
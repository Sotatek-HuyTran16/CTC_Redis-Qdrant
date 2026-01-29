# 1. Install GNOME desktop

```
sudo apt update
sudo apt install -y ubuntu-desktop-minimal
```

```
echo "exec gnome-session" > ~/.xsession
sudo adduser xrdp ssl-cert
sudo systemctl restart xrdp
```

# 2. Install helm

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4

chmod 700 get_helm.sh

./get_helm.sh
```

# 3. Install regsync

## Install

```
curl -L https://github.com/regclient/regclient/releases/latest/download/regctl-linux-amd64 >regctl

chmod 755 regctl

sudo cp regctl /usr/local/bin/
```

## Verify

```
regctl version
```

# 4. Install ansible

```
pip install ansible
```

# 5. Setup no ssh with password

```bash
# sudo nano /etc/ssh/sshd_config
PubkeyAuthentication yes
PasswordAuthentication no

PermitRootLogin prohibit-password
```

```
sudo systemctl reload sshd
```
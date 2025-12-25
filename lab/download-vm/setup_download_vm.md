# Install GNOME desktop

```
sudo apt update
sudo apt install -y ubuntu-desktop-minimal
```

```
echo "exec gnome-session" > ~/.xsession
sudo adduser xrdp ssl-cert
sudo systemctl restart xrdp
```
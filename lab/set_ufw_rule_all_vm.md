# Setup restricted ufw rule

```
sudo ufw default deny incoming

sudo ufw default deny outgoing
```

```
sudo ufw allow in ssh
```

```
sudo ufw allow from 10.10.10.0/24

sudo ufw allow out to 10.10.10.0/24
```

```
sudo ufw allow from 10.10.0.0/24

sudo ufw allow out to 10.10.0.0/24
```

```
sudo ufw allow from 172.16.20.2

sudo ufw allow out to 172.16.20.2
```
# 1. Setup ssh with public-private key

## Gen key in Download VM

```bash
ssh-keygen -t ed25519 -C "huy.tran16@sotatek.com"

cat ~/.ssh/id_ed25519.pub
```

## Copy the public key to all other VM

```
nano ~/.ssh/authorized_keys
```


# 2. Setup ufw rule

```bash
sudo ufw default deny incoming

sudo ufw default deny routed

sudo ufw default allow outgoing
```

```bash
sudo ufw allow ssh
```

```bash
sudo ufw allow from 10.10.10.0/24

sudo ufw allow out to 10.10.10.0/24
```

```bash
sudo ufw allow from 10.10.0.0/24

sudo ufw allow out to 10.10.0.0/24
```

```bash
sudo ufw allow from 172.16.20.2

sudo ufw allow out to 172.16.20.2
```
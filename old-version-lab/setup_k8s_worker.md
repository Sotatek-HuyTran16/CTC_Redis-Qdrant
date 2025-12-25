## Allow ufw 

```
sudo ufw allow from 10.10.0.0/24

sudo ufw allow out to 10.10.0.0/24

sudo ufw allow from 10.10.10.0/24

sudo ufw allow out to 10.10.10.0/24

sudo ufw allow from 192.168.0.0/16

sudo ufw allow out to 192.168.0.0/16
```

## Add hostname on master

```bash
# edit /etc/hosts
10.10.0.11 dev-k8s-vm-data01
```

## print token on master

```
sudo k8s get-join-token dev-k8s-vm-data01 --worker
```

## Join cluster

```
sudo k8s join-cluster <token>
```
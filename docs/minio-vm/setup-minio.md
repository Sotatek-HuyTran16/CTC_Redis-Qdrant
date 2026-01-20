# 0. Install software

```bash
sudo apt update

sudo apt install lvm2
```

# 1. Create logical volume for MinIO

## Create lvm on /dev/sdb

```
sudo pvcreate /dev/sdb
sudo vgcreate vg_minio /dev/sdb
sudo lvcreate -l 100%FREE -n lv_data vg_minio
```

## Format filesystem

```
sudo mkfs.xfs /dev/vg_minio/lv_data
```

## Create mount folder for MinIO

```
sudo mkdir -p /data/minio
```

## Mount folder to lvm

```
sudo mount /dev/vg_minio/lv_data /data/minio
```

## Mount persistent

```bash
sudo blkid /dev/vg_minio/lv_data

# edit /etc/fstab
UUID=<UUID>  /data/minio  xfs  defaults,noatime  0  0

sudo mount -a
```

# 2. Install MinIO

## Download from Download VM

```
curl --progress-bar -L dl.min.io/aistor/minio/release/linux-amd64/minio.deb -o minio.deb
```

## Copy to MinIO VM

```
scp minio.deb sotatek@10.10.10.18:/home/sotatek
```

## Install (bind server)

```
sudo dpkg -i minio.deb
```

## Create license file

```bash
sudo mkdir /opt/minio
sudo touch /opt/minio/minio.license

# add the license to this file

sudo chown -R minio-user:minio-user /opt/minio
```

## Edit (create if not have) environment file

```bash
# sudo nano /etc/default/minio

## Volume to be used for MinIO server.
MINIO_VOLUMES="/data/minio"

MINIO_LICENSE="/opt/minio/minio.license"

## Use if you want to run MinIO on a custom port.
MINIO_OPTS="--address :9000 --console-address :9001"

## Root user for the server.
MINIO_ROOT_USER=minioadmin

## Root secret for the server.
MINIO_ROOT_PASSWORD=minioadmin

## set this for MinIO to reload entries with 'mc admin service restart'
MINIO_CONFIG_ENV_FILE=/etc/default/minio
```

## Add permission on data folder

```
sudo chown -R minio-user:minio-user /data/minio
```

## Create the credentials folder if not have

```
sudo mkdir -p /home/minio-user
sudo chown -R minio-user:minio-user /home/minio-user
sudo chmod 750 /home/minio-user
```

```
sudo mkdir -p /home/minio-user/.minio/certs
sudo chown -R minio-user:minio-user /home/minio-user/.minio
sudo chmod 700 /home/minio-user/.minio
```

## Enable and Start MinIO

```
sudo systemctl enable minio
sudo systemctl start minio
```

# 3. Install mc

```
curl -LO https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/bin/
```
# 1. From bind server

## 1.1. Install software

```bash
### nano /etc/apt/source.list
deb [trusted=yes] http://10.10.10.13:80 ./
```

```
sudo apt install bind9 bind9-utils
```

## 1.2. Config bind

```
cd /etc/bind
```

```bash
### named.conf.options

acl trusted {
        10.10.0.0/24;
        10.10.10.0/24;
        172.16.20.2;
};

options {
        directory "/var/cache/bind";

        dnssec-validation no;

        listen-on-v6 { any; };

        listen-on port 53 { any; };

        //forwarders { };

        allow-query { localhost; trusted; };    

};
```

```bash
### named.conf.local

zone "dev-dify.ctc.local" IN {
        type primary;
        file "/etc/bind/zones/dev-dify.ctc.local";
        allow-transfer { 10.10.10.11; };
};
```

Create zone file

```bash
mkdir /etc/bind/zones

### nano /etc/bind/zones/dev-dify.ctc.local

$TTL 10m
@       IN      SOA     bind-master.dev-dify.ctc.local. admin.sotatek. (
		2		; Serial
		604800		; Refresh
		86400		; Retry
		2419200		; Expire
		604800	)	; Negative Cache TTL

; Name server record

      IN      NS     bind-master.dev-dify.ctc.local.
      IN      NS     bind-slave.dev-dify.ctc.local.

; A Record for name server

bind-master.dev-dify.ctc.local.       IN       A       10.10.10.10
bind-slave.dev-dify.ctc.local.          IN      A       10.10.10.11

; A Record for client

control-plane.dev-dify.ctc.local.       IN       A       10.10.0.10

data-0.dev-dify.ctc.local.	       IN       A       10.10.0.11
data-1.dev-dify.ctc.local.	       IN       A       10.10.0.12
data-2.dev-dify.ctc.local.	       IN       A       10.10.0.13

monitoring-0.dev-dify.ctc.local.	       IN       A       10.10.0.14


dify-0.dev-dify.ctc.local.	       IN       A       10.10.0.15
dify-1.dev-dify.ctc.local.	       IN       A       10.10.0.16
dify-2.dev-dify.ctc.local.	       IN       A       10.10.0.17

registry.dev-dify.ctc.local.	       IN       A       10.10.10.13
repository.dev-dify.ctc.local.	       IN       A       10.10.10.13

pypi.dev-dify.ctc.local.	       IN       A       10.10.10.12
minio.dev-dify.ctc.local.	       IN       A       10.10.10.18

download.dev-dify.ctc.local.	       IN       A       172.16.20.2
```

## 1.3. Verify

```
dig @localhost minio.dev-dify.ctc.local +short
```

# 2. From secondary server

## Same as primary server 1.1 - 1.2

```bash
### /etc/bind/named.conf.local

zone "dev-dify.ctc.local" IN {
        type secondary;
        file "dev-dify.ctc.local";
        primaries { 10.10.10.10; };
};
```

## Reload bind9

```
sudo systemctl daemon-reload

sudo systemctl restart bind9
```

# 3. From client

## Config systemd resolver 

```bash
### nano /etc/systemd/resolved.conf

[Resolve]
DNS=10.10.10.10 10.10.10.11
```

```
sudo systemctl restart systemd-resolved

sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

## Verify

```
ping minio.dev-dify.ctc.local
```
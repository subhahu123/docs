---
title: pkgrepo jail
category: [ bsd, freenas ]
keywords: freebsd, bsd, freenas, jail, pkgrepo
tags: [ freebsd, bsd, jail ]
---

## pkgrepo jail

Setup for poudriere package server jail with iocage.

### On FreeNAS

Create jail:

```shell
iocage create --release 11.1-RELEASE --name pkgrepo \
          boot="on" vnet=on bpf=on \
          allow_raw_sockets="1" \
          ip4_addr="vnet1|172.20.40.40/24" \
          interfaces="vnet1:bridge1" \
          defaultrouter="172.20.40.1" \
          resolver="search ramsden.network;nameserver 172.20.40.1;nameserver 8.8.8.8"
```

Mount packages from host into jail with nullfs.

```shell
iocage exec pkgrepo 'mkdir -p /usr/local/poudriere/data/packages'
iocage fstab --add pkgrepo '/mnt/tank/data/poudriere/packages /usr/local/poudriere/data/packages nullfs rw 0 0'
```

Check fstab:

```shell
iocage fstab --list pkgrepo
```

Start jail and enter.

```shell
iocage start pkgrepo
iocage console pkgrepo
```

### Jail

In the jail, update all packages.

```shell
pkg update && pkg upgrade
```

### Web Server

```shell
pkg install nginx && sysrc nginx_enable=YES
```

Remove all inside server in ```/usr/local/etc/nginx/nginx.conf```, add:

Check config and start nginx:

```shell
service nginx configtest
service nginx start
```

In jail, nullfs mount packages to same spot. Install nginx.

```shell
server {
    listen 80 default;
    server_name pkgrepo.ramsden.network;
    root /usr/local/poudriere/data/packages;
    autoindex on;
}
```

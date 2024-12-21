---
title: Deluge jail
category: [ bsd, freenas ]
keywords: freebsd, bsd, freenas, jail, deluge
tags: [ freebsd, bsd, jail ]
---

## Deluge jail

Setup for Deluge service jail with iocage.

### On FreeNAS

Create jail:

```shell
iocage create --release 11.1-RELEASE --name deluge \
          boot="on" vnet=on bpf=on \
          allow_raw_sockets="1" \
          ip4_addr="vnet1|172.20.40.35/24" \
          interfaces="vnet1:bridge1" \
          defaultrouter="172.20.40.1" \
          resolver="search ramsden.network;nameserver 172.20.40.1;nameserver 8.8.8.8"
```

On Freenas create datasets:

*   Datasets
    *   Deluge Data
        *   ```tank/data/database/deluge/```
    *   Download Datasets
        *   For Complete Torrents ```tank/media/Downloads/Complete```
        *   For Incomplete Torrents ```tank/media/Downloads/Incomplete```
        *   For Torrents ```tank/media/Torrents```

Create media user/group using uid from freenas:

```shell
iocage exec deluge 'pw useradd -n media -u 8675309'
```

Nullfs mount datasets in jail:

Deluge data:

```shell
iocage exec deluge 'mkdir -p /home/media/.config /media/Downloads/Complete /media/Downloads/Incomplete /media/Torrents' && \
iocage exec deluge 'chown media:media /home/media/.config /media/Downloads/Complete /media/Downloads/Incomplete /media/Torrents' && \
iocage fstab --add deluge '/mnt/tank/data/database/deluge /home/media/.config  nullfs rw 0 0' && \
iocage fstab --add deluge '/mnt/tank/media/Downloads/Complete /media/Downloads/Complete  nullfs rw 0 0' && \
iocage fstab --add deluge '/mnt/tank/media/Downloads/Incomplete /media/Downloads/Incomplete  nullfs rw 0 0' && \
iocage fstab --add deluge '/mnt/tank/media/Torrents /media/Torrents  nullfs rw 0 0'
```

Check fstab:

```shell
iocage fstab --list deluge
```

Start jail and enter.

```shell
iocage console deluge
```

#### Install Deluge

Install ```deluge``` or ```deluge-cli``` depending on what you want installed. Since this is a headless server I'm only installing the CLI version.

```shell
pkg update && pkg upgrade && pkg install deluge-cli
```

#### Init Script

Setup ```/etc/rc.conf```

```shell
sysrc 'deluged_enable=YES' 'deluged_user=media'
```

#### Start Service

```shell
service deluged start
```

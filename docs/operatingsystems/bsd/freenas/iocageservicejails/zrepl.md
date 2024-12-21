---
title: pod jail
category: [ bsd, freenas ]
keywords: freebsd, bsd, freenas, jail, pod
tags: [ freebsd, bsd, jail ]
---

## pod jail

Setup for zrepl jail with iocage.

### On FreeNAS

Create jail:

zfs create -o mountpoint=none tank/zrepl

```shell
iocage create --release 11.3-RELEASE --name zrepl \
          boot=on vnet=on dhcp=off \
          allow_raw_sockets="1" \
          ip4_addr="vnet1|172.20.30.44/24" \
          interfaces="vnet1:bridge1" \
          defaultrouter="172.20.30.1" \
          resolver="search ramsden.network;nameserver 172.20.30.1;nameserver 8.8.8.8" \
          jail_zfs=on \
          jail_zfs_dataset=repl \
          jail_zfs_mountpoint='none'
```

iocage set vnet=on dhcp=off bpf=yes \
          allow_raw_sockets="1" \
          ip4_addr="vnet0|172.20.30.45/24" \
          interfaces="vnet0:bridge1" \
          defaultrouter="172.20.30.1" \
          resolver="search ramsden.network;nameserver 172.20.30.1;nameserver 8.8.8.8" \
          repl

```shell
iocage get jail_zfs_dataset zrepl
```

### In Jail

```shell
iocage console zrepl
```

Update.

```shell
pkg update && pkg upgrade
```

### Requirements

Create the log file /var/log/zrepl.log

```shell
touch /var/log/zrepl.log && service newsyslog restart
```

Tell syslogd to redirect facility local0 to the zrepl.log file:

```shell
service syslogd reload
```

Modify the /usr/local/etc/zrepl/zrepl.yml configuration file

Enable the zrepl daemon to start automatically at boot:

```shell
sysrc zrepl_enable="YES"
```

Start the zrepl daemon:

```shell
service zrepl start
```


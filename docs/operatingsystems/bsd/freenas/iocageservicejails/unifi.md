---
title: Ubiquity Unifi jail
category: [ bsd, freenas ]
keywords: freebsd, bsd, freenas, jail, unifi
tags: [ freebsd, bsd, jail ]
---

# Ubiquity Unifi Controller jail

Install unifi controller in jail.

### On FreeNAS

Create jail, OpenJDK requires fdescfs, and procfs.

```shell
iocage create --release 11.1-RELEASE --name unifi \
          allow_raw_sockets="1" \
          mount_linprocfs="1" \
          boot="on" vnet=on bpf=on \
          ip4_addr="vnet1|172.20.40.20/24" \
          interfaces="vnet1:bridge1" \
          defaultrouter="172.20.40.1"
```

On Freenas create datasets:

*   Datasets
    *   tank/data/unifi/data
    *   tank/data/unifi/logs
    *   tank/data/unifi/certs


Nullfs mount datasets in jail:

```shell
iocage fstab -a unifi /mnt/tank/data/unifi/data /usr/local/share/java/unifi/data nullfs rw 0 0
iocage fstab -a unifi /mnt/tank/data/unifi/logs /usr/local/share/java/unifi/logs nullfs rw 0 0
iocage fstab -a unifi /mnt/tank/data/unifi/certs /usr/local/share/java/unifi/certs nullfs rw 0 0
```

Start jail.

```shell
iocage start unifi
```

### In jail

Install ```net-mgmt/unifi5``` (built pkg with poudriere), or use ports.

In jail:

```shell
pkg install unifi5
sysrc unifi_enable=YES
```

Set permissions.

```shell
chown -R unifi /usr/local/share/java/unifi
```

Enable unifi at boot.

```shell
sysrc unifi_enable=YES
```

Unifi just uses the mongod binary, it can be disabled.

```shell
sysrc mongod_enable=NO
```

## Finishing Tasks

Restart the jail and confirm everything works.

```shell
iocage restart unifi
```

Go to ```https://<jail ip>:8443```. Make sure you use https.

Configure with the wizard.

## AP

To SSH into AP, password ubnt.

```shell
ssh ubnt@<ip>
```

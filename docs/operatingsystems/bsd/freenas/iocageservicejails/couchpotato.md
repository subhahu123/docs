---
title: Couchpotato jail
category: [ bsd, freenas ]
keywords: freebsd, bsd, freenas, jail, couchpotato
tags: [ freebsd, bsd, jail ]
---

## couchpotato jail

Setup for couchpotato service jail with iocage.

### On FreeNAS

Create jail:

```shell
iocage create --release 11.1-RELEASE --name couchpotato \
          boot="on" vnet=on bpf=on \
          allow_raw_sockets="1" \
          ip4_addr="vnet1|172.20.40.31/24" \
          interfaces="vnet1:bridge1" \
          defaultrouter="172.20.40.1" \
          resolver="search ramsden.network;nameserver 172.20.40.1;nameserver 8.8.8.8"
```

On Freenas create datasets:

*   Datasets
    *   Couchpotato Data
        *   ```tank/data/database/couchpotato```
    *   Media
        *   For all media ```tank/media/Movies/...```
    *   Downloads
        *   For all Downloads ```tank/media/Downloads/...```

Create media user/group using uid from freenas:

```shell
iocage exec couchpotato 'pw useradd -n media -u 8675309'
```

Nullfs mount datasets in jail:

Couchpotato data:

```shell
iocage exec couchpotato 'mkdir -p /var/db/couchpotato && chown media:media /var/db/couchpotato'
iocage fstab --add couchpotato '/mnt/tank/data/database/couchpotato /var/db/couchpotato nullfs rw 0 0'
```

Downloads:

```shell
iocage exec couchpotato 'mkdir -p /media/Downloads/Complete /media/Downloads/Incomplete && chown -R media:media /media/Downloads'

iocage fstab --add couchpotato '/mnt/tank/media/Downloads/Complete /media/Downloads/Complete nullfs rw 0 0' && \
iocage fstab --add couchpotato '/mnt/tank/media/Downloads/Incomplete /media/Downloads/Incomplete nullfs rw 0 0'
```

Setup directories:

```shell
iocage exec couchpotato 'mkdir -p /media/Movie/Movies /media/Movie/Sports && chown -R media:media /media'
```

Repeat for media:

```shell
iocage fstab --add couchpotato '/mnt/tank/media/Movie/Movies /media/Movie/Movies nullfs rw 0 0' && \
iocage fstab --add couchpotato '/mnt/tank/media/Movie/Sports /media/Movie/Sports nullfs rw 0 0'
```

Check fstab:

```shell
iocage fstab --list couchpotato
```

Start jail and enter.

```shell
iocage console couchpotato
```

## In Jail

Install [couchpotato](https://couchpota.to/#freebsd) freebsd version from git.

```shell
pkg update && pkg upgrade
```

Install required tools

```shell
pkg install python py27-sqlite3 fpc-libcurl docbook-xml git-lite
```

Use user media, clone to a temp repo in ```/var/db```.

```shell
cd /var/db
git clone https://github.com/CouchPotato/CouchPotatoServer.git temp
```

Move the bare repo that was just cloned to the dataset we mounted earlier to ```/var/db/couchpotato```.

```shell
mv temp/.git couchpotato/
rm -rf temp
```

Switch to the ```media``` user and reset the repo to HEAD.

```shell
su media
cd couchpotato
git reset --hard HEAD
exit
```

As root, copy the startup script to ```/usr/local/etc/rc.d``` and make the startup script executable.

```shell
mkdir /usr/local/etc/rc.d
cp /var/db/couchpotato/init/freebsd /usr/local/etc/rc.d/couchpotato
chmod 555 /usr/local/etc/rc.d/couchpotato
```

Read the options at the top of ```/usr/local/etc/rc.d/couchpotato```.

If not using the default install, specify options with startup flags.

```shell
sysrc 'couchpotato_enable=YES' 'couchpotato_user=media' 'couchpotato_dir=/var/db/couchpotato'
```

Finally, start couchpotato.

```shell
service couchpotato start
```

Restart the jail, open your browser and go to [http://server:5050/](http://server:5050/).

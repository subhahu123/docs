---
title: sickchill jail
category: [ bsd, freenas ]
keywords: freebsd, bsd, freenas, jail, sickchill
tags: [ freebsd, bsd, jail ]
---

## sickchill jail

Setup for sickchill service jail with iocage.

### On FreeNAS

Create jail:

```shell
iocage create --release 11.2-RELEASE --name sickchill0 \
          boot="on" vnet=on bpf=yes \
          allow_raw_sockets="1" \
          ip4_addr="vnet1|172.20.40.37/24" \
          interfaces="vnet1:bridge1" \
          defaultrouter="172.20.40.1" \
          resolver="search ramsden.network;nameserver 172.20.40.1;nameserver 8.8.8.8"
```

On Freenas create datasets:

*   Datasets
    *   sickchill Data
        *   ```tank/data/database/sickchill```
    *   Media
        *   For all media ```tank/media/Series/...```
    *   Downloads
        *   For all downloads ```tank/media/Downloads/...```

Create media user/group using uid from freenas:

```shell
iocage exec sickchill 'pw useradd -n media -u 8675309'
```

Nullfs mount datasets in jail:

sickchill data:

```shell
iocage exec sickchill 'mkdir -p /var/db/sickchill /mnt/backups'
iocage exec sickchill 'chown media:media /var/db/sickchill /mnt/backups'
iocage fstab --add sickchill '/mnt/tank/data/database/sickchill /var/db/sickchill nullfs rw 0 0'
iocage fstab --add sickchill '/mnt/tank/backups/Lilan/sickchill /mnt/backups nullfs rw 0 0'
```

Downloads:

```shell
iocage exec sickchill 'mkdir -p /media/Downloads/Complete /media/Downloads/Incomplete /media/Torrents'
iocage exec sickchill 'chown -R media:media /media'

iocage fstab --add sickchill '/mnt/tank/media/Downloads/Complete /media/Downloads/Complete nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Downloads/Incomplete /media/Downloads/Incomplete nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Torrents /media/Torrents nullfs rw 0 0'
```

Setup directories:

```shell
iocage exec sickchill 'mkdir -p /media/Series/Series /media/Series/Lectures /media/Series/Documentary /media/Series/Anime /media/Series/Animated /media/Series/Podcasts/Audio /media/Series/Podcasts/Video /media/Naddy /media/Movie/Movies /media/Movie/Sports' && iocage exec sickchill 'chown -R media:media /media'
```

Repeat for media:

```shell
iocage fstab --add sickchill '/mnt/tank/media/Series/Series /media/Series/Series nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Series/Podcasts/Audio /media/Series/Podcasts/Audio nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Series/Podcasts/Video /media/Series/Podcasts/Video nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Series/Lectures /media/Series/Lectures nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Series/Documentary /media/Series/Documentary nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Series/Anime /media/Series/Anime nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Series/Animated /media/Series/Animated nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Naddy /media/Naddy nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Movie/Movies /media/Movie/Movies nullfs rw 0 0' && \
iocage fstab --add sickchill '/mnt/tank/media/Movie/Sports /media/Movie/Sports nullfs rw 0 0'
```

Check fstab:

```shell
iocage fstab --list sickchill
```

Start jail and enter.

```shell
iocage console sickchill
```

### In Jail

Update.

```shell
pkg update && pkg upgrade
```

Install requirements.

```shell
pkg install \
  "git" "python3" "python38" "py38-pip" "py38-setuptools" "py38-sqlite3" "py38-pillow" "py38-virtualenv" "unrar" "ca_root_nss" "libmediainfo" "libxslt" "libxml2" "rust"
```

Install sickchill.

```shell
mkdir -p /.cargo
chown -R media:media /.cargo

cd /var/db
git clone  https://github.com/sickchill/sickchill.git temp
mv temp/.git sickchill/
rm -rf temp
chown -R media:media sickchill/
su media
cd sickchill/
git reset --hard HEAD
```

Copy the startup script

```shell
mkdir /usr/local/etc/rc.d
cp /var/db/sickchill/contrib/runscripts/init.freebsd /usr/local/etc/rc.d/sickchill
```

Make startup script executable

```shell
chmod 555 /usr/local/etc/rc.d/sickchill
```

Add settings to rc.conf

```shell
sysrc 'sickchill_enable=YES' 'sickchill_user=media' 'sickchill_group=media' 'sickchill_dir=/var/db/sickchill'
```

Start sickchill.

```shell
service sickchill start
```

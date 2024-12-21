---
title: emby jail
category: [ bsd, freenas ]
keywords: freebsd, bsd, freenas, jail, emby
tags: [ freebsd, bsd, jail ]
---

## Emby jail

Setup for Emby service jail with iocage.

### On FreeNAS

Create jail:

```shell
iocage create --release 11.2-RELEASE --name emby \
          boot=on vnet=on dhcp=on bpf=yes \
          allow_raw_sockets="1" \
          ip4_addr="vnet1|172.20.40.21/24" \
          interfaces="vnet1:bridge1" \
          defaultrouter="172.20.40.1" \
          resolver="search ramsden.network;nameserver 172.20.40.1;nameserver 8.8.8.8"
```

On Freenas create datasets:

*   Datasets
    *   Emby Data
        *   ```tank/data/database/emby/emby-server```
        *   ```tank/data/database/emby/media-metadata```
    *   Media
        *   For all media ```tank/media/Series/...```

Create media user/group using uid from freenas:

```shell
iocage exec emby 'pw useradd -n media -u 8675309'
```

Nullfs mount datasets in jail:

Emby data:

```shell
iocage exec emby 'mkdir -p /var/db/emby-server /mnt/emby/media-metadata'
iocage exec emby 'chown media:media /var/db/emby-server'
iocage exec emby 'chown media:media /mnt/emby/media-metadata'
iocage fstab --add emby '/mnt/tank/data/database/emby/emby-server /var/db/emby-server nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/data/database/emby/media-metadata /mnt/emby/media-metadata nullfs rw 0 0'
```

Setup directories:

```shell
iocage exec emby 'mkdir -p /media/Series/Series /media/Series/Lectures /media/Series/Documentary /media/Series/Anime /media/Series/Animated /media/Series/Podcasts/Audio /media/Series/Podcasts/Video /media/Naddy /media/Movie/Movies /media/Movie/Sports /mnt/backups'
iocage exec emby 'chown -R media:media /media && chown -R media:media /mnt/backups'
```

Repeat for media:

```shell
iocage fstab --add emby '/mnt/tank/media/Series/Series /media/Series/Series nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/media/Series/Podcasts/Audio /media/Series/Podcasts/Audio nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/media/Series/Podcasts/Video /media/Series/Podcasts/Video nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/media/Series/Lectures /media/Series/Lectures nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/media/Series/Documentary /media/Series/Documentary nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/media/Series/Anime /media/Series/Anime nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/media/Series/Animated /media/Series/Animated nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/media/Naddy /media/Naddy nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/backups/Lilan/Emby /mnt/backups nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/media/Movie/Movies /media/Movie/Movies nullfs rw 0 0'
iocage fstab --add emby '/mnt/tank/media/Movie/Sports /media/Movie/Sports nullfs rw 0 0'
```

Check fstab:

```shell
iocage fstab --list emby
```

Start jail and enter.

```shell
iocage start emby
iocage console emby
```

### Jail

In the jail, update all packages and install ```emby-server```.

```shell
pkg update && pkg upgrade && pkg install emby-server
```

#### Package Options

Its reccomended to change some package options. Either build a package with poudriere with these changes, or make these changes on the emby jails packages.

##### FFMpeg

It's recommended to install ffmpeg from ports so that certain compile time options can be enabled.

Update the FreeBSD ports tree

```shell
portsnap fetch extract update
```

Remove the default ffmpeg package

```shell
pkg delete -f ffmpeg
```

Reinstall FFMpeg from ports with lame option enabled

```shell
cd /usr/ports/multimedia/ffmpeg && make config
```

*   enable the lame option
*   enable the ass subtitles option
*   enable the opus subtitles option
*   enable the x265 subtitles option

Compile and install.

```shell
make install clean
```

##### ImageMagick

It is recommended to recompile the graphics/ImageMagick package from ports with the following options .

*  disable (unset) 16BIT_PIXEL (to increase thumbnail generation performance)

Delete the imagemagick pkg.

```shell
pkg delete -f imagemagick
```

Install from ports

```shell
cd /usr/ports/graphics/ImageMagick && make config
```

*   Disable the 16BIT_PIXEL option

```shell
make install clean
```

### Emby Start Options

Set the rc script executable.

```shell
chmod 555 /usr/local/etc/rc.d/emby-server
```

Check the options.

```shell
less /usr/local/etc/rc.d/emby-server
```

Set emby to start on boot and change the options based on setup.

```shell
sysrc 'emby_server_enable=YES'
sysrc 'emby_server_user=media' && sysrc 'emby_server_group=media'
sysrc 'emby_server_data_dir=/var/db/emby-server'
```

Start the emby service.

```shell
service emby-server start
```

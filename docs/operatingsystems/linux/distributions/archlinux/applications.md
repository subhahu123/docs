---
title: Common Applications
sidebar: linux_sidebar
hide_sidebar: false
category: [ archlinux, linux ]
keywords: archlinux, linux, aur, pacaur, applications
tags: [ archlinux, linux, aur, postinstall ]
permalink: linux_archlinux_applications.html
toc: true
folder: linux/archlinux
---

## syncthing

Install syncthing.

```shell
pacman -S syncthing
```

Start user service.

```shell
systemctl --user enable --now syncthing
```

Increase max-user-watches.

```shell
nano /etc/sysctl.d/40-max-user-watches.conf
```

```shell
fs.inotify.max_user_watches=524288
```

## Onboard Virtual Keyboard

Install onboard.

```shell
pacman -S onboard
```

For secondary labels run.

```shell
gsettings set org.onboard.keyboard show-secondary-labels true
```

## Conky

Install conky.

```shell
pacman -S conky
```

I use a script with conky to check email with the perl```Mail::IMAPClient``` and ```IO::Socket::SSL```, on arch needs: [perl-mail-imapclient (AUR)](https://aur.archlinux.org/packages/perl-mail-imapclient/), and [perl-io-socket-ssl](https://www.archlinux.org/packages/extra/any/perl-io-socket-ssl/).

```shell
pacman -S perl-io-socket-ssl
aursync --update --temp --chroot perl-mail-imapclient
```

## Steam using Flatpak

Add flathub.

```shell
flatpak remote-add --user --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Install steam for user.

```shell
flatpak install --user flathub com.valvesoftware.Steam
```

Run Steam, data for flatpak will be in ```${HOME}/.var```.

```shell
flatpak run com.valvesoftware.Steam
```

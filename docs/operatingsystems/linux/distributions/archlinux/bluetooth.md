---
title: Bluetooth
category: [ archlinux, linux ]
keywords: archlinux, linux
tags: [ archlinux, linux, postinstall ]
---

# Bluetooth

Install ```bluez``` and ```bluez-utils```.

```shell
pacman -S bluez bluez-utils
```

Load bluetooth driver (may be already loaded).

```shell
modprobe btusb
```

Start, and enable the bluetooth unit

```shell
systemctl enable --now bluetooth
```

Add user(s) who will use bluetooth to ```lp``` group

```shell
gpasswd -a ${USER} lp
```

---
title: Restore Installed Applications
sidebar: linux_sidebar
hide_sidebar: false
category: [ archlinux, linux ]
keywords: archlinux, linux, aur, pacaur, applications
tags: [ archlinux, linux, aur, postinstall ]
permalink: linux_archlinux_restore_installed_applications.html
toc: true
folder: linux/archlinux
---

## Restore Packages

To list explicitly installed packages.

```shell
pacman -Qqe > pkglist.txt
```

### Regular repo

Remove AUR packages from explicitly installed package list, and save to file ```native.txt```.

```shell
bash -c "comm -12 <(pacman -Slq | sort) <(sort pkglist.txt)"  > native.txt
```

Install them on new system after selecting wanted packages.

```shell
pacman  -S - < native.txt
```

### AUR

List aur packages. Dont forget to edit.

```shell
pacman -Qmq > aur.txt
```

On new system, install.

```shell
pacaur -S - < aur.txt
```

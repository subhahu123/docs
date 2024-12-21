---
title: NixOS root on ZFS Install
sidebar: linux_sidebar
hide_sidebar: false
category: [ nixos, linux]
keywords: nixos, linux, recovery, configuration, zfs
tags: [ nixos, linux, nixosconfig, zfs ]
permalink: linux_nixos_install_zfs.html
toc: false
folder: linux/nixos/install
---

In order to import a ZFS pool, ZFS must be enabled in the NixOS configuration file.

Make sure zfs is in ```boot.supportedFilesystems```.

```shell
{ config, pkgs, ... }:

{
  imports = [ <nixpkgs/nixos/modules/installer/cd-dvd/installation-cd-graphical-kde.nix> ];
  boot.supportedFilesystems = [ "zfs" ];
}
```

Rebuild NixOS and switch to the new configuration.

```shell
nixos-rebuild switch
```

Check zfs is working,```modprobe zfs``` should show no problems.

ZFS should now work.

### Pool

Create a new pool or mount an existing pool.

#### Creating a pool

Create a pool using the disk ID and set it to 4k block size as default with ```ashift=12```.

```shell
ls -la /dev/disk/by-id/
zpool create -f -o ashift=12 vault /dev/disk/by-id/${DISKS}
```

Export the pool after creation.

```shell
zpool export ${POOLNAME}
```

#### Using an Existing pool

The existing pool assigning it to be relative to ```/mnt``` with ```-R```, the ```-N``` flag will tell ZFS not to mount any datasets.

```shell
zpool import -N -d /dev/disk/by-id -R /mnt vault
```

## Setup Datasets

Mount all datasets partitions to /mnt.

### Filesystem

```shell
NIX_ROOT=/mnt
ZFS_ROOT_DATASET=vault/sys/atom
ZFS_DATA_DATASET=vault/data
```

Setup datasets. Set all legacy.

```shell
zfs create -o mountpoint=none vault/sys
zfs create -o mountpoint=none ${ZFS_ROOT_DATASET}
zfs create -o mountpoint=none ${ZFS_ROOT_DATASET}/ROOT
zfs create -o mountpoint=legacy ${ZFS_ROOT_DATASET}/ROOT/default

# Rest of datasets...
```

## Mount Datasets

Mount the datasets:

```shell
mkdir ${NIX_ROOT}/nix;
mount -t zfs ${ZFS_ROOT_DATASET}/ROOT/default ${NIX_ROOT}

# Rest of datasets...
```

#### Boot

Create a 512M esp, mount to /boot

```shell
gdisk /dev/sdf

Command (? for help): n
Partition number (5-128, default 5):
First sector (34-488397134, default = 225445888) or {+-}size{KMGTP}:
Last sector (225445888-488397134, default = 488397134) or {+-}size{KMGTP}: +512
Hex code or GUID (L to show codes, Enter = 8300): ef00
Changed type of partition to 'EFI System'
```

Format boot and mount.

```shell
mkfs.fat -F32 /dev/sdf1
mkdir ${NIX_ROOT}/boot
mount /dev/sdf1 ${NIX_ROOT}/boot
```

#### Swap

Create a partition of desired size.

```shell
gdisk /dev/sdf

Command (? for help): n
Partition number (2-128, default 2):
First sector (34-488397134, default = 2099200) or {+-}size{KMGTP}:
Last sector (2099200-488397134, default = 488397134) or {+-}size{KMGTP}: +32G
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 8200
Changed type of partition to 'Linux swap'
```

Enable swap.

```shell
mkswap /dev/sdf2

swapon /dev/sdf2
```

#### Install

Setup config in ```/mnt/etc/nixos``` and install.

```
nixos-install --root /mnt
```

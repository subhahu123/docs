---
title: libvirt
sidebar: linux_sidebar
hide_sidebar: false
category: [ archlinux, linux ]
keywords: archlinux, linux, virtualization, libvirt, kvm, qemu
tags: [ archlinux, linux, virtualization ]
permalink: linux_archlinux_libvirt.html
toc: true
folder: linux/archlinux
---

## libvirt

Setup [libvirt](https://wiki.archlinux.org/index.php/Libvirt).

### Libvirt ZFS Dataset

To keep my libvirt setup outside of any boot environments I give them their own dataset.

```shell
zfs create -o mountpoint=legacy vault/sys/chin/var/lib/libvirt
mkdir /var/lib/libvirt
mount -t zfs vault/sys/chin/var/lib/libvirt /var/lib/libvirt
```

Add to fstab

```shell
nano /etc/fstab
```

```shell
vault/sys/chin/var/lib/libvirt            /var/lib/libvirt                    zfs       rw,relatime,xattr,noacl     0 0
```

### Kernel

Check modules are loaded.

```shell
lsmod | grep kvm
lsmod | grep virtio
```

If blank, [load them explicitly](https://wiki.archlinux.org/index.php/Kernel_modules#Manual_module_handling).

```shell
echo "virtio" > /etc/modules-load.d/virtio.conf
```

Install the dependencies.

*   [libvirt](https://www.archlinux.org/packages/?name=libvirt)
*   KVM/QEMU
    *   [qemu-headless](https://www.archlinux.org/packages/?name=qemu-headless)
*   Network
    *   Nat/DHCP
        *   [ebtables](https://www.archlinux.org/packages/?name=ebtables)
        *   [dnsmasq](https://www.archlinux.org/packages/?name=dnsmasq)
    *   bridged networking
        *   [bridge-utils](https://www.archlinux.org/packages/?name=)
*   UEFI
    *   [ovmf](https://www.archlinux.org/packages/?name=ovmf)
*   Frontends
    *   [virt-manager](https://www.archlinux.org/packages/?name=virt-manager)
    *   [virt-viewer](https://www.archlinux.org/packages/?name=virt-viewer)

```shell
pacman -S libvirt qemu-headless ebtables dnsmasq bridge-utils virt-manager virt-viewer ovmf
```

### ZVOL Backing Store

I like to use ZFS ZVOL's as my backing store.

#### Setup

Create a dataset for ZVOLs:

```shell
zfs create -o mountpoint=none vault/zvols
```

##### User ZVOLs

Create a dataset for user 'john''s ZVOLs

```shell
zfs create -o mountpoint=none vault/zvols/john
```

As of [zfsonlinux 0.7.0](https://github.com/zfsonlinux/zfs/releases/tag/zfs-0.7.0) ZFS delegation using ```zfs allow``` works on linux. Delegate permissions giving the abiity to snapshot and create datasets.

```shell
zfs allow john create,mount,mountpoint,snapshot vault/zvols/john
```

#### Create ZVOL

To let guest do its own caching, use:

*   primarycache=metadata

Create ZVOL for a new VM. Replace <new VM> with name. Volumes still need to be created by root.

```shell
zfs create -o mountpoint=none vault/zvols/john/libvirt
zfs create -V 50G vault/zvols/john/libvirt/<new VM> -o primarycache=metadata -o compression=on
```

### Authentication

By default, anybody in the ```wheel``` group can authenticate with polkit as defined in ```/etc/polkit-1/rules.d/50-default.rules``` (see [Polkit#Administrator identities](https://wiki.archlinux.org/index.php/Polkit#Administrator_identities)).

If you want passwordless authentication, as of libvirt 1.2.16, anyone in the ```libvirt``` group can access to the RW daemon socket by default.

Create the group if it doesn't exist.

```shell
groupadd libvirt
```

Add any users required to it.

```shell
gpasswd -a john libvirt
```

Make sure to re-login after.

### System Service

Enable libvirtd.service.

```shell
systemctl enable --now libvirtd
```

To run only a user-session the daemon does not need to be enabled.

### Connect

Test libvirt system-session:

```shell
virsh -c qemu:///system
```

Test libvirt system user-session:

```shell
virsh -c qemu:///session
```

### UEFI

Add the following to ```/etc/libvirt/qemu.conf```.

```shell
nano /etc/libvirt/qemu.conf
```

```shell
nvram = [
    "/usr/share/ovmf/ovmf_code_x64.bin:/usr/share/ovmf/ovmf_vars_x64.bin"
]
```

I have found UEFI may not work if I haven't set the system user to ```user = root``` in ```/etc/libvirt/qemu.conf```.

and restart libvirtd

```shell
systemctl restart libvirtd
```

#### User

To use uefi as a user, note networking options are limited, move the nvram to a user readable location and add it to ```~/.config/libvirt/qemu.conf```.

```shell
cp -r /usr/share/ovmf /home/john/.config/libvirt/ovmf
chown -R john:john /home/john/.config/libvirt/ovmf
```

Add the following to ```/etc/libvirt/qemu.conf```.

```shell
nano ~/.config/libvirt/qemu.conf
```

```shell
nvram = [
    "/home/john/.config/libvirt/ovmf/ovmf_code_x64.bin:/home/john/.config/libvirt/ovmf/ovmf_vars_x64.bin"
]
```

### Create Guest

Use virsh or virt manager.

### Storage

Select virtIO Network and storage for best performance. Select ZVOL raw device. Mine was ```/dev/vault/zvols/john/libvirt/<new VM>```.

#### ZVOL Persistance

If using a user session the block device might need to be changed to be owned by the user running the VM.

Temporarily the device can be chown'd, but the owner will not live through reboot. For persistence [add a udev rule](ramsdenj.com/2016/07/21/making-a-zvol-backed-virtualbox-vm-on-linux.html) by creating a new file ```99-local-zvol.rules``` in ```/etc/udev/rules.d/``` that contains the following (replacing the ZVOL path and user):

```shell
# /etc/udev/rules.d/99-local-zvol.rules
# Give persistant ownership of ZVOL to user
KERNEL=="zd*" SUBSYSTEM=="block" ACTION=="add|change" PROGRAM="/lib/udev/zvol_id /dev/%k"
RESULT=="vault/zvols/john/libvirt/win" OWNER="john" GROUP="john" MODE="0750"
```

Refresh the rules with ```udevadm control --reload```

#### VirtIO

Install drivers:

I [downloaded](https://fedoraproject.org/wiki/Windows_Virtio_Drivers#Direct_download) ISO and attached the drivers pre-install.

At the "Where do you want to install Windows?" screen, select the option Load Drivers, uncheck the box for "Hide drivers that aren't compatible with this computer's hardware".

Browse to the wanted driver(s) at:

SCSI: "viostor\w10\amd64"
Networking: "NetKVM\w10\amd64"

### Network

To use another interface, don't configure anything on the host and select macvtap passthrough, and select the interface.

Install then reboot.

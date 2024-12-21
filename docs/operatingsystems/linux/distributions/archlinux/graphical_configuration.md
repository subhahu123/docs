---
title: Graphical Configuration
sidebar: linux_sidebar
hide_sidebar: false
category: [ archlinux, linux ]
keywords: archlinux, linux, aur, pacaur, dwm
tags: [ archlinux, linux, aur, postinstall ]
permalink: linux_archlinux_graphical_configuration.html
toc: true
folder: linux/archlinux
---

First setup [xorg](https://wiki.archlinux.org/index.php/Xorg) and graphics.

## Graphics

Install graphics drivers, my main system is [nvidia](https://wiki.archlinux.org/index.php/NVIDIA).

```shell
pacman -S nvidia lib32-nvidia-utils
```

## Xorg

```shell
pacman -S xorg-server
```

Set dpi in ```~/.Xresources```, I use 192 for my 4k screen.

```shell
nano ~/.Xresources
```

```shell
Xft.dpi: 192
```

### Nvidia - Tearing Fix

My Nvidia card tears. This removes the tearing.

#### Desktop

```shell
nano /etc/X11/xorg.conf.d/20-nvidia.conf
```

```shell
Section "Screen"
    Identifier     "Screen0"
    Option         "metamodes" "nvidia-auto-select +0+0 { ForceFullCompositionPipeline = On }"
    Option         "AllowIndirectGLXProtocol" "off"
    Option         "TripleBuffer" "on"
EndSection
```

#### Laptop

To fix laptop using [DRM kernel mode setting](https://wiki.archlinux.org/index.php/NVIDIA#DRM_kernel_mode_setting).

[nvidia](https://www.archlinux.org/packages/?name=nvidia) 364.16 adds support for DRM kernel mode setting.

Add the ```nvidia-drm.modeset=1``` kernel parameter, and add ```nvidia```, ```nvidia_modeset```, ```nvidia_uvm``` and ```nvidia_drm``` to mkinitcpio modules.

##### Pacman hook

To update initramfs after an NVIDIA driver upgrade, use a pacman hook:

```shell
/etc/pacman.d/hooks/nvidia.hook
```

```shell
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia

[Action]
Depends=mkinitcpio
When=PostTransaction
Exec=/usr/bin/mkinitcpio -P
```

## Setup a Window Manager or Desktop Environment

### KDE Plasma

Install [KDE Plasma](https://www.archlinux.org/groups/x86_64/plasma/) package as well as some [KDE meta-packages](https://www.archlinux.org/packages/?name=kde-applications-meta). I dont install ```kdeaccessibility-meta```, ```kdeedu-meta```, ```kdegames-meta```, .```kdemultimedia-meta```, ```kdepim-meta```, ```kdesdk-meta```, ```kdewebdev-meta```.

Choose ```phonon-qt5-gstreamer```, ```libx264```, ```cronie```, ```phonon-qt4-gstreamer```.

```shell
pacman -S plasma kdeadmin-meta kdebase-meta kdegraphics-meta kdenetwork-meta kdeutils-meta
```

I disable baloo since it seems to make my system chug.

```shell
balooctl disable
```

## Display Manager

I use sddm, simple and works well. For an onscreen keyboard install [qt5-virtualkeyboard](https://www.archlinux.org/packages/extra/x86_64/qt5-virtualkeyboard/).

```shell
pacman -S sddm qt5-virtualkeyboard
```

Setup config at ```/etc/sddm.conf.d/sddm.conf```.

```shell
nano /etc/sddm.conf.d/sddm.conf
```

Tell it to start a desktop file from ```/usr/share/xsessions/```, set dpi, and user.

```shell
# Set DPI based on display
ServerArguments=-nolisten tcp -dpi 192

# Name of session file for autologin session
Session=plasma.desktop

# Username for autologin session
User=john

# Current theme name
Current=breeze
```

Enable sddm.

```shell
systemctl enable sddm
```

Reboot into KDE!

## VNC

Can access current display or create new session.

### System

To access the entire system over vnc, install [tigervnc](https://www.archlinux.org/packages/?name=tigervnc).

Configure startup run ```vncserver```.

Setup a systemd unit to start vnc, note this connects to physical display, other options are available. Change user.

```shell
nano /etc/systemd/system/x0vncserver.service
```

```shell
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=forking
User=john
ExecStart=/usr/bin/sh -c '/usr/bin/x0vncserver -display :0 -rfbport 5900 -passwordfile /home/john/.vnc/passwd &'

[Install]
WantedBy=multi-user.target
```

```shell
systemctl start x0vncserver
```

## Fonts

Install [ttf-google-fonts-git (AUR)](https://aur.archlinux.org/packages/ttf-google-fonts-git/).

```shell
aursync --update --temp --chroot ttf-google-fonts-git
```

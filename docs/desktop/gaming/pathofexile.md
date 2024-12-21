---
title: Path of Exile
category: [ desktop, gaming ]
keywords: wine, gaming
tags: [ wine, gaming ]
---

# Path of Exile

The following describes how to setup Path of Exile.

Prerequisites (Arch only):

*   [Gaming with Wine](/operatingsystems/linux/distributions/archlinux/wine.html)

## Dataset

Create a ZFS dataset for wine bottle.

```shell
zfs create -o mountpoint=legacy vault/sys/$(hostname)/home/john/local/share/wine
```

Add to fstab:

```shell
vault/sys/chin/home/john/local/share/wine  /home/john/.local/share/wine zfs       rw,relatime,xattr,noacl     0 0
```

Mount it

```shell
mkdir /home/john/.local/share/wine
mount -a
```

## Configuration

Always use ```env WINEPREFIX=${HOME}/.local/share/wine/<wine bottle>``` when creating bottles. Otherwise wine defaults to ```~/.wine```.

Install dependencies:

```shell
pacman -S mpg123 lib32-gst-plugins-base-libs pulseaudio-alsa lib32-libpulse lib32-alsa-plugins lib32-libldap lib32-openal
```

```shell
pacaur -S ttf-ms-fonts  ttf-tahoma
```

To create a 32bit bottle use ```WINEARCH=win32```.

```shell
env WINEARCH=win32 WINEPREFIX=${HOME}/.local/share/wine/pathofexile winecfg
```

Enable CSMT, optionallenable emulate virtual desktop and change DPI.

## Tuning

Get videocard RAM:

```
echo $"VRAM: "$(($(grep -P -o -i "(?<=memory:).*(?=kbytes)" /var/log/Xorg.0.log) / 1024))$" Mb"
```

Set in regedit. Copy the number.

```shell
env WINEARCH=win32 WINEPREFIX=${HOME}/.local/share/wine/pathofexile wine regedit
```

Go to ```HKEY_CURRENT_USER>Software>Wine```

*   Add key "Direct3D"
*   Add new string to Direct3D folder. Right click>New String, type "VideoMemorySize", add "VideoMemorySize" string, use video memory number

## With Installer

Install dependencies

```shell
env WINEARCH=win32 WINEPREFIX=${HOME}/.local/share/wine/pathofexile winetricks -q glsl=disabled directx9 usp10 msls31
```

Download and execute installer.

```shell
cd ${HOME}/.local/share/wine/pathofexile
wget https://www.pathofexile.com/downloads/PathOfExileInstaller.exe
env WINEARCH=win32 WINEPREFIX=${HOME}/.local/share/wine/pathofexile wine ${HOME}/.local/share/wine/pathofexile/PathOfExileInstaller.exe
```

Run game launcher.

```shell
env WINEDEBUG=-all WINEARCH=win32 WINEPREFIX=${HOME}/.local/share/wine/pathofexile wine "${HOME}/.local/share/wine/pathofexile/drive_c/Program Files/Grinding Gear Games/Path of Exile/PathOfExile.exe" dbox  -no-dwrite -noasync
```

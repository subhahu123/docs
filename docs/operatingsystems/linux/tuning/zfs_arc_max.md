---
title: ZFS Arc Max on Linux
category: [ linux-tuning, linux]
keywords: linux, tuning, zfs
tags: [ tuning, linux, zfs ]
---

Check stats with ```arcstat.py```

```shell
# arcstat.py -h
Usage: arcstat.py [-hvx] [-f fields] [-o file] [-s string] [interval [count]]

     -h : Print this help message
     -v : List all possible field headers and definitions
     -x : Print extended stats
     -f : Specify specific fields to print (see -v)
     -o : Redirect output to the specified file
     -s : Override default field separator with custom character or string

Examples:
    arcstat.py -o /tmp/a.log 2 10
    arcstat.py -s "," -o /tmp/a.log 2 10
    arcstat.py -v
    arcstat.py -f time,hit%,dh%,ph%,mh% 1
```

Set arc max in ```/etc/modprobe.d/zfs.conf```, defaults to 50% memory.

For example, 48GiB:

```shell
echo "options zfs zfs_arc_max=51539607552" > /etc/modprobe.d/zfs.conf
```

Rebuild kernel, then reboot.

```shell
mkinitcpio -p linux
```

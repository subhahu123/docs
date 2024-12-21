---
title: Tuning Limits
category: [ linux-tuning, linux]
keywords: linux, tuning, limits
tags: [ tuning, linux ]
folder: linux/tuning
---

# Limits

Set user defined limits by adding overrides to ```/etc/systemd/system.conf``` and ```/etc/systemd/user.conf```

## Games

Nofile may need increasing.

Create an override and set it.

For user:

```shell
nano /etc/systemd/user.conf.d/nofile.conf
```

```shell
[Manager]
DefaultLimitNOFILE=8192
```

For system:

```shell
cp /etc/systemd/user.conf.d/nofile.conf /etc/systemd/system.conf.d/nofile.conf
```

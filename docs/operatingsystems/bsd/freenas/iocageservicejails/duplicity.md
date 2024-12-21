---
title: duplicity jail
category: [ bsd, freenas ]
keywords: freebsd, bsd, freenas, jail, duplicity
tags: [ freebsd, bsd, jail ]
---

## Duplicity jail

Setup for Duplicity service jail with iocage.

### On FreeNAS

Create jail:

```shell
iocage create --release 11.1-RELEASE --name duplicity \
          boot="on" vnet=on bpf=on \
          allow_raw_sockets="1" \
          ip4_addr="vnet1|172.20.40.41/24" \
          interfaces="vnet1:bridge1" \
          defaultrouter="172.20.40.1" \
          resolver="search ramsden.network;nameserver 172.20.40.1;nameserver 8.8.8.8"
```

Create user on FreeNAS with ID `983`, `nologin` to match the user in the jail.

Nullfs mount datasets to backup in jail:

Duplicity data:

```shell
iocage exec duplicity 'mkdir -p /mnt/duplicity/data'
iocage fstab --add duplicity '/mnt/tank/data/syncthing/sync /mnt/duplicity/data nullfs rw 0 0'
```

Start jail and enter.

```shell
iocage console duplicity
```

### Jail

In the jail, update all packages and install ```duplicity``` and `py27-boto`.

```shell
pkg update && pkg upgrade
pkg install duplicity py27-boto
```

Create a user with uid `983` to match mounted data.

```shell
pw useradd -n duplicity -u 983
```

Add script `/usr/local/scripts/duplicitybak`, put secrets in `/usr/local/scripts/duplicitybak.auth`.

```shell
#!/bin/sh

# on freebsd install duplicity, py27-boto

# Place auth variables: PASSPHRASE, GS_ACCESS_KEY_ID, GS_SECRET_ACCESS_KEY
. "/usr/local/scripts/duplicitybak.auth"

# Folders to backup
BACKUP_DATA_REGEXP='Workspace|Computer|Personal|Pictures|University'
BACKUP_ROOT="/mnt/duplicity/data"

# GS configuration variables
GS_BUCKET="johnramsdenbackup"

# Remove files older than 60 days from GS
duplicity remove-older-than 60D --force gs://${GS_BUCKET}

# Sync everything to GS
duplicity --include-regexp "${BACKUP_DATA_REGEXP}" \
          --exclude='**' \
          ${BACKUP_ROOT} gs://${GS_BUCKET}

# Cleanup failures
duplicity cleanup --force gs://${GS_BUCKET}

unset PASSPHRASE
unset GS_ACCESS_KEY_ID
unset GS_SECRET_ACCESS_KEY
```

Secrets in `/usr/local/scripts/duplicitybak.auth`:

```shell
# Create password to use for symetric GPG encryption
export PASSPHRASE=""

# Create GS bucket, https://console.cloud.google.com/storage/
# enable interoperable access, get keys
export GS_ACCESS_KEY_ID=""
export GS_SECRET_ACCESS_KEY=""
```

Set executable:

```shell
chmod +x /usr/local/scripts/duplicitybak
```

Now I can be run from a crontab outside of the jail:

```shell
iocage exec duplicity /usr/local/scripts/duplicitybak
```

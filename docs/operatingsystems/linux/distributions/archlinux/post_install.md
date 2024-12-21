---
title: Post Install on Arch Linux
category: [ archlinux, linux ]
keywords: archlinux, linux, aur, pacaur, zfs
tags: [ archlinux, linux, aur, postinstall, zfs ]
---

# Post Install on Arch Linux

First [setup AUR](/archlinux_aur_pacaur.html)


## Time

Setup [time](https://wiki.archlinux.org/index.php/time) using [systemd-timesyncd](https://wiki.archlinux.org/index.php/Systemd-timesyncd).

```shell
timedatectl set-ntp true
timedatectl set-ntp 1
```

## Configure Reflector

So you always have fresh mirrors, setup [reflector](https://www.archlinux.org/packages/?name=reflector).

```shell
pacman -S reflector
```

Create service to select the 200 most recently synchronized HTTP or HTTPS mirrors, sort them by download speed, and overwrite the file ```/etc/pacman.d/mirrorlist```.

```shell
nano /etc/systemd/system/reflector.service
```

```shell
[Unit]
Description=Pacman mirrorlist update

[Service]
Type=oneshot
ExecStart=/usr/bin/reflector --latest 200 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

Create timer.

```shell
nano /etc/systemd/system/reflector.timer
```

```shell
[Unit]
Description=Run reflector weekly

[Timer]
OnCalendar=weekly
RandomizedDelaySec=12h
Persistent=true

[Install]
WantedBy=timers.target
```

That will run reflector weekly.

```shell
systemctl enable --now reflector.timer
```

## Configure SMTP

I used to use ssmtp but since it's now unmaintained I've started using [Msmtp](https://wiki.archlinux.org/index.php/Msmtp).

```shell
pacman -S msmtp msmtp-mta
```

Setup system default.

```shell
cp /usr/share/doc/msmtp/msmtprc-system.example /etc/msmtprc
```

Example config file

```shell
# msmtp system wide configuration file

# A system wide configuration file with default account.
defaults

# The SMTP smarthost.
host smtp.fastmail.com
port 465

# Construct envelope-from addresses of the form "user@oursite.example".
#auto_from on
maildomain <your domain>

# Use TLS.
tls on
tls_starttls off

# Activate server certificate verification
tls_trust_file /etc/ssl/certs/ca-certificates.crt

# Syslog logging with facility LOG_MAIL instead of the default LOG_USER.
syslog LOG_MAIL

aliases               /etc/aliases

# msmtp root account, inherit from 'default' account
account default

user <your email>

from system@<your domain>

# Terrible...
# auth plain
# password <pass>

# or with passwordeval,
# passwordeval "gpg --quiet --for-your-eyes-only --no-tty --decrypt ~/.msmtp-root.gpg"

account root : default

# password, see below
```

Set permissions.

```shell
chmod 600 /etc/msmtprc
```

You can setup a [gpg encrypted passphrase](https://wiki.archlinux.org/index.php/Msmtp#Password_management) if using interactively. The other (not very good option) is setting with 'password' in config.

Add aliases to ```/etc/aliases```.

```shell
root: root@<yourdomain>
```

If anything private is in /etc/msmtprc, secure the file [as shown](https://wiki.archlinux.org/index.php/SSMTP#Security) on the Arch wiki.

Create an ssmtp group and set the owner of ```/etc/msmtp``` and the msmtp binary.

```shell
groupadd msmtp
chown :msmtp /etc/msmtprc
chown :msmtp /usr/bin/msmtp
```

Make sure only root, and the msmtp group can access ```msmtprc```, then et the SGID bit on the binary

```shell
chmod 640 /etc/msmtprc
chmod g+s /usr/bin/msmtp
```

Then add a pacman hook to always set the file permissions after the package has been upgraded:

```shell
nano /usr/local/bin/msmtp-set-permissions
```

```shell
#!/bin/sh

chown :msmtp /usr/bin/msmtp
chmod g+s /usr/bin/msmtp
```

Make it executable:

```shell
chmod u+x /usr/local/bin/msmtp-set-permissions
```

Now add the pacman hook:

```shell
nano /usr/share/libalpm/hooks/msmtp-set-permissions.hook
```

```shell
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = msmtp

[Action]
Description = Set msmtp permissions for security
When = PostTransaction
Exec = /usr/local/bin/msmtp-set-permissions
```

### Test mail

Send a test mail.

```shell
 echo "Text, more text." | /usr/bin/mail -s SUBJECT email@your.domain.com
```

## ZFS Configuration

I always set up snapshotting and replication as one of the first things I do on a new desktop.

### Enable Snapshots

Install [zfs-auto-snapshot (AUR)](https://aur.archlinux.org/packages/zfs-auto-snapshot-git/) and setup snapshotting on all datasets.

```shell
pacaur -S zfs-auto-snapshot-git
systemctl enable --now zfs-auto-snapshot-daily.timer
```

Set all datasets to snapshot and disable any datasets that dont require snapshotting.

```shell
for ds in $(zfs list -H -o name); do
  MP="$(zfs get -H -o value mountpoint $ds )";
  if [ ${MP} == "legacy" ] || [ "${MP}" == "/" ]; then
    echo "${ds}: on";
    zfs set com.sun:auto-snapshot=true ${ds};
  else
    echo "${ds}: off";
    zfs set com.sun:auto-snapshot=false ${ds};
  fi;
done
```

In one line:

```shell
for ds in $(zfs list -H -o name); do MP="$(zfs get -H -o value mountpoint $ds )"; if [ ${MP} == "legacy" ] || [ "${MP}" == "/" ]; then echo "${ds}: on"; zfs set com.sun:auto-snapshot=true ${ds}; else echo "${ds}: off";zfs set com.sun:auto-snapshot=false ${ds}; fi; done
```

### ZFS Replication With ZnapZend

Install [ZnapZend (AUR)](https://aur.archlinux.org/packages/znapzend/) (it's a great tool, I maintain the AUR package).

```shell
pacaur -S znapzend
systemctl enable --now znapzend
```

Create a config for each dataset thet needs replicating, where SYSTEM will be a name for the dataset at ```${POOL}/replication/${SYSTEM}``` on the remote. Specify the remote user and IP as well. Here is a small script I use for my setup. The grep can be adjusted to exclude any datasets that are unwanted.

```shell
#!/bin/sh

REMOTE_POOL_ROOT="${1}"
REMOTE_USER="${2}"
REMOTE_IP="${3}"

for ds in $(zfs list -H -o name | \
    grep -E 'data/|default|john|usr/|var/|lib/' | \
    grep -v cache); do
  echo "Creating: ${REMOTE_USER}@${REMOTE_IP}:${REMOTE_POOL_ROOT}/${ds}"

  # See ssh(1) for -tt
  # https://www.freebsd.org/cgi/man.cgi?query=ssh
  # In simple terms, force pseudo-terminal and pseudo tty
    ssh -tt ${REMOTE_USER}@${REMOTE_IP} \
      "~/znap_check_dataset ${REMOTE_POOL_ROOT}/${ds}"

  znapzendzetup create --tsformat='%Y-%m-%d-%H%M%S' \
    SRC '1d=>15min,7d=>1h,30d=>4h,90d=>1d' ${ds} \
    DST:${REMOTE_IP} '1d=>15min,7d=>1h,30d=>4h,90d=>1d,1y=>1w,10y=>1month' \
    "${REMOTE_USER}@${REMOTE_IP}:${REMOTE_POOL_ROOT}/${ds}"
done
```

On remote I have a pre-znazendzetup script which makes sure the remote location exists.

```shell
#!/bin/sh

# Pre zapzendzetup script. Put in ~/znap_check_dataset on remote and run with

ds="${1}"

if [ "$(zfs list -H -o name "${ds}")" = "${ds}" ]; then
  echo "${ds} exists, running ZnapZend."
else
  echo "Creating non-existant dataset ${ds}"
  zfs create -p "${ds}"
  zfs unmount "${ds}"
  echo "${ds} created, running ZnapZend."
fi
```

I would then run, for chin on ```replicator@<server ip>```.

```shell
./znapcfg "tank/replication/chin" "replicator" "<server ip>"
```

### Scrub

Setup a monthly scrub with a systemd unit and timercontaining the following.

```shell
nano /usr/lib/systemd/system/zpool-scrub@.service
```

```shell
# /etc/systemd/system/zpool-scrub@.service
[Unit]
Description=Scrub ZFS Pool
Requires=zfs.target
After=zfs.target

[Service]
Type=oneshot
ExecStartPre=-/usr/bin/zpool scrub -s %i
ExecStart=/usr/bin/zpool scrub %i
```

```shell
nano /etc/systemd/system/zpool-scrub@.timer
```

```shell
[Unit]
Description=Scrub ZFS pool weekly

[Timer]
OnCalendar=weekly
Persistent=true

[Install]
WantedBy=timers.target
```

Enable for pool.

```shell
systemctl enable --now zpool-scrub@vault.timer
```

### Enable The ZFS Event Daemon

If an SMTP or MTA is configured, setup [The ZFS Event Daemon (ZED)](https://ramsdenj.com/2016/08/29/arch-linux-on-zfs-part-3-followup.html#zed-the-zfs-event-daemon)

```shell
nano /etc/zfs/zed.d/zed.rc
```

Ad an email and mail program and set verbosity.

```shell
ZED_EMAIL_ADDR="root"
ZED_EMAIL_PROG="mail"
ZED_NOTIFY_VERBOSE=1
```

Start and enable the daemon.

```shell
systemctl enable --now zfs-zed.service
```

Start a scrub and check for an email.

```shell
zpool scrub vault
```

## Define Hostid

[Define a hostid](https://ramsdenj.com/2016/06/23/arch-linux-on-zfs-part-2-installation.html#first-tasks) or problems arise at boot.

## smart

Install [smartmontools](https://www.archlinux.org/packages/?name=smartmontools).

```shell
pacman -S smartmontools
```

### Tests

Long or short tests can be run on a disk. A short test will check for device problems. The long test is just a short test plus complete disc surface examination.

Long test example:

```shell
smartctl -t long /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_152271401093
smartctl -t long /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_154501401266
smartctl -t long /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402487
smartctl -t long /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402657
smartctl -t long /dev/disk/by-id/ata-Samsung_SSD_840_EVO_250GB_S1DBNSADA75563M
```

Veiw results:

```shell
smartctl -H /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_152271401093
smartctl -H /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_154501401266
smartctl -H /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402487
smartctl -H /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402657
smartctl -H /dev/disk/by-id/ata-Samsung_SSD_840_EVO_250GB_S1DBNSADA75563M
```

Or, veiw all test results.

```shell
smartctl -l selftest /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_152271401093
smartctl -l selftest /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_154501401266
smartctl -l selftest /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402487
smartctl -l selftest /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402657
smartctl -l selftest /dev/disk/by-id/ata-Samsung_SSD_840_EVO_250GB_S1DBNSADA75563M
```

Or detailed results.

```shell
smartctl -a /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_152271401093
smartctl -a /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_154501401266
smartctl -a /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402487
smartctl -a /dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402657
smartctl -a /dev/disk/by-id/ata-Samsung_SSD_840_EVO_250GB_S1DBNSADA75563M
```

### Daemon

The smartd daemon can also run, periodically running tests and will send you a message if a problem occurs.

Edit the configuration file at ```/etc/smartd.conf```.

```shell
nano /etc/smartd.conf
```

To check for all errors on a disk use the option ```-a``` after the disk ID.

```shell
/dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_152271401093 -a -m <email>
/dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_154501401266 -a -m <email>
/dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402487 -a -m <email>
/dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402657 -a -m <email>
/dev/disk/by-id/ata-Samsung_SSD_840_EVO_250GB_S1DBNSADA75563M -a -m <email>
```

To test if your mail notification is working run a test, add ```-m <email address> -M test``` to the end of the config. This will run the test on the start of the daemon.:

```shell
DEVICESCAN -m <email address> -M test
```

Start smartd:

```shell
systemctl start smartd
```

My config looks like:

```shell
/dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_152271401093 -a -m <email>
/dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_154501401266 -a -m <email>
/dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402487 -a -m <email>
/dev/disk/by-id/ata-SanDisk_SDSSDXPS480G_164277402657 -a -m <email>
/dev/disk/by-id/ata-Samsung_SSD_840_EVO_250GB_S1DBNSADA75563M -a -m <email>
```

## nfs


```shell
pacman -S nfs-utils
systemctl enable --now rpcbind.service nfs-client.target remote-fs.target
```

## Autofs

Install [autofs](https://www.archlinux.org/packages/?name=autofs).

```shell
pacman -S autofs
```

```shell
nano /etc/autofs/auto.master
```

Add or uncomment the following.

```shell
/net    -hosts   --timeout=60
```

Start and enable.

```shell
systemctl enable --now autofs
```

## User Cache

I like to keep certain directories in tmpfs. It avoids extra writes to disk and can be faster since everything is stored in memory.

## Cleaning the cache

I like periodically have my users cache directory cleaned. This can easily be done using tmpfiles.d.

Create a new file in the ```/etc/tmpfiles.d``` directory.

```shell
nano /etc/tmpfiles.d/home-cache.conf
```

Add a rule that will delete any file older than 10 days.

```shell
# remove files in /home/john/.cache older than 10 days
D /home/john/.cache 1755 john john 10d
```

Setup container

```shell
cd /var/lib/machines
debootstrap --include=systemd-container stretch matlab
```

Login, set root password:

```shell
systemd-nspawn -D matlab
passwd
```

Start, setup network:

```shell
systemd-nspawn --bind-ro=/dev/dri --bind=/tmp/.X11-unix -b -D matlab

systemctl enable --now systemd-networkd systemd-resolved
ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf

useradd -m -s /bin/bash john
su john
export DISPLAY=:0
```

Requirements:

apt-get install xorg build-essential libgtk2.0-0 libnss3 libasound2

Add sources to `/etc/apt/sources.list`:

```shell
deb http://deb.debian.org/debian stretch main contrib non-free
deb-src http://deb.debian.org/debian stretch main contrib non-free

deb http://deb.debian.org/debian-security/ stretch/updates main contrib non-free
deb-src http://deb.debian.org/debian-security/ stretch/updates main contrib non-free

deb http://deb.debian.org/debian stretch-updates main contrib non-free
deb-src http://deb.debian.org/debian stretch-updates main contrib non-free
```

Set on host `xhost +local:`

Get install files and run `./install` as root in container.

Install support.

```shell
apt-get upgrade && apt-get install matlab-support
```

Create 'shared'.
```shell
mkdir ~/.share
```

Edit service ovveride of `/usr/lib/systemd/system/systemd-nspawn@.service`

```shell
[Service]
ExecStart=
ExecStart=/usr/bin/systemd-nspawn --quiet --keep-unit --boot \
                                  --link-journal=try-guest -U \
                                  --settings=override --machine=%i \
                                  --bind-ro=/dev/dri --bind=/tmp/.X11-unix \
                                  --bind=/home/john/.share:/home/john/share
```

Might need to copy /etc/hostid to container.

Run matlab:

```shell
xhost +local:; machinectl start matlab; machinectl shell john@matlab /bin/sh -c "DISPLAY=$DISPLAY /usr/local/bin/matlab"; xhost -;
```

<!--
    MIT License

    Copyright (c) 2018 Rémi Ducceschi

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE
-->

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/remileduc/debian_config_perso/blob/master/LICENSE)

Debian config
=============

Files for my personal debian config, so I don't have to recreate them each time...

### Table of contents ###

[Installation](#installation)
- [Sid](#sid)
- [Partitionning](#partitionning)
- [KDE](#kde)

[Copy config files](#copy-config-files)

[Home encryption](#home-encryption)
- [Create keyfile](#create-keyfile)
- [Add the keyfile as a key](#add-the-keyfile-as-a-key)
- [Change crypttab](#change-crypttab)
- [Key slots management](#key-slots-management)

[Ramdisk](#ramdisk)

[Software](#software)
- [Setup](#setup)
- [System packages](#system-packages)
- [Firefox extensions](#firefox-extensions)
- [Kodi extensions](#kodi-extensions)

[Firewall](#firewall)

[Samba](#samba)

[Useful information](#useful-information)
- [Sudo](#sudo)
- [SSH](#ssh)
- [KDE Connect](#kde-connect)

[Rescue](#rescue)
- [Mount LUKS partition](#mount-luks-partition)
- [Chroot to zotac](#chroot-to-zotac)

Installation
------------

### Sid ###

1. download a Testing daily-build netinst for Debian: https://cdimage.debian.org/cdimage/daily-builds/daily/arch-latest/amd64/iso-cd/
1. download the daily built mini iso for Debian: https://d-i.debian.org/daily-images/amd64/daily/netboot/
1. extract the mini iso to your freshly formatted USB key
1. copy the `efi` folder from the testing netinst iso to your USB key

At the beginning, choose `Advanced options -> Expert install`. Then, when chosing repositories, select `Sid`.

### Partitionning ###

- efi = 100 Mio (unused) with boot flag, FAT32
- /boot = 400 Mio
- / = 50 Gio, ENCRYPTED
- /mount/persistent = size of RAM, ENCRYPTED, `defaults,nodev,noexec,nosuid,noatime,nodiratime`
- /home = everything else, ENCRYPTED
- no swap

### KDE ###

During installation process, chose to install KDE.

Copy config files
-----------------

Copy all the files to their destination. Some files are hidden...

To update Grub, you'll need to run `update-grub`.

Home encryption
---------------

Follow instructions here (listed below): https://debian-facile.org/viewtopic.php?id=9101

Everything should be run in root.

### Create keyfile ###

This keyfile holds the keypass to open the home partition

```bash
dd if=/dev/random of=/root/sda4_keyfile bs=4096 count=1
chmod a-rwx,u=r /root/sda4_keyfile
```

### Add the keyfile as a key ###

We add the generated keyfile as a key to open Home. Here, the UUID used is the UUID of the Home partition (/dev/sda4), not the encrypted partition (/dev/mapper/sda4_crypt). Use `blkid` to fetch the UUID.

```bash
cryptsetup luksAddKey /dev/disk/by-uuid/c4295a74-5c31-475b-aa57-b9fa4c2de36e /root/sda4_keyfile
```

### Change crypttab ###

We tell to the file `/etc/crypttab` to automatically fetch the keyfile (same rules as earlier for the UUID):

```bash
sda4_crypt UUID=c4295a74-5c31-475b-aa57-b9fa4c2de36e /root/sda4_keyfile luks,discard
```

Then, reboot and pray...

### Key slots management ###

Add key
```bash
cryptsetup luksAddKey /dev/sdb1 /root/file # use file as a key
cryptsetup luksAddKey /dev/sdb1 # will ask for a passphrase
```

Delete key
```bash
cryptsetup luksKillSlot /dev/sdb1 2 # 2 is the key slot to remove
```

Change key
```bash
cryptsetup luksChangeKey /dev/sdb1 -S 2 # 2 is the key slot to remove
```

Show infos
```bash
cryptsetup luksDump /dev/sdb1
```

Ramdisk
-------

Follow the steps in this repository: [debian-systemd-ramdisk](https://github.com/remileduc/debian-systemd-ramdisk).

Software
--------

### Setup ###

To install 32 bits packages, you need to enable the 32 bits architecture:

```bash
dpkg --add-architecture i386
```

To install the last version of Kodi, you need to enable the *deb-multimedia* repository. To do so, you need to install the key:

```bash
sudo apt-get update -oAcquire::AllowInsecureRepositories=true
sudo apt-get install deb-multimedia-keyring
```

### System packages ###

**Uninstall** the following:

```
appstream firefox-esr firefox-esr-l10n-fr xserver-xorg-video-intel
```

Install the following:

```
android-tools-adb android-tools-fastboot cowsay cowsay-off firefox firefox-l10n-fr firmware-iwlwifi firmware-misc-nonfree firmware-realtek git kdeconnect kodi mlocate qbittorrent rsync samba ufw vlc
```

### Firefox extensions ###

- [Behind the overlay - revival](https://addons.mozilla.org/fr/firefox/addon/behind-the-overlay-revival/)
- [Cast Kodi](https://addons.mozilla.org/fr/firefox/addon/castkodi/)
- [HTTPS Everywhere](https://addons.mozilla.org/fr/firefox/addon/https-everywhere/)
- [uBlock Origin](https://addons.mozilla.org/fr/firefox/addon/ublock-origin/)

### Kodi extensions ###

- enable [remote control](https://kodi.wiki/view/Smartphone/tablet_remotes)
- [InputStream Adaptative](https://kodi.wiki/view/Add-on:InputStream_Adaptive)
    - `apt install kodi-inputstream-adaptive`
- [InputStream RTMP](https://kodi.wiki/view/Add-on:RTMP_Input)
    - `apt install kodi-inputstream-rtmp`
- [YouTube](https://kodi.wiki/view/Add-on:YouTube)
- [CU Lyrics](https://kodi.wiki/view/Add-on:CU_LRC_Lyrics)

Firewall
--------

We use Uncomplicated Firewall (`ufw`). The goal is to accept nothing except the
needed, and only on the local network.

To star, we need to enable `ufw`:

```bash
systemctl enable ufw
service ufw start
ufw enable
```

Then, we need the following rules:

```bash
ufw default deny incoming
ufw default allow outgoing
ufw default allow routed
# To finish
ufw reload
```

**Note:** Full configuration of the Firewall is done in the router step. See [debian-vpn-router](https://github.com/remileduc/debian-vpn-router).

To check `ufw` rules:

```bash
ufw status verbose
```

To clear rules:
```bash
ufw reset
ufw enable
```

Samba
-----

Install the package `samba`.

Add the current user as a samba user:

```bash
smbpasswd -a sid
```

Create a shared folder and create some links (not root!)

```bash
cd ~
mkdir Téléchargements/p2p
mkdir shared && cd shared
ln -s ../Documents Documents
ln -s ../Images Pictures
ln -s ../Musique Music
ln -s ../Vidéos/ Videos
ln -s ../Téléchargements/ Downloads
```

Edit samba config file `/etc/samba/smb.conf`:

```
# In [global] section:
   allow insecure wide links = yes

# In [homes] section:
   available = no

# At the end of the file, create a new section:
[shared]
   comment = Shared folders for Zotac
   follow symlinks = yes
   wide links = yes
   path = /home/sid/shared
   available = yes
   valid users = sid
   read only = no
   browsable = yes
   public = yes
   writable = yes
```

Restart the service `smbd` and connect to `\\IP\shared`.

Useful information
------------------

### Sudo ###

managesudo permission:

```bash
# give sudo permission:
usermod -aG sudo sid
# remove sudo permission:
deluser sid sudo
```

### SSH ###

In order to be able to login as `root` via SSH, you need to edit the file
`/etc/ssh/sshd_config` and add the following line:

```
PermitRootLogin yes
```

### KDE Connect ###

Commands for KDE Connect:
- `suspend` = qdbus org.kde.Solid.PowerManagement /org/freedesktop/PowerManagement Suspend
- `voldown` = qdbus org.kde.kglobalaccel /component/kmix invokeShortcut "decrease_volume"
- `volup` = qdbus org.kde.kglobalaccel /component/kmix invokeShortcut "increase_volume"
- `show konsole` = qdbus org.kde.KWin /KWin setCurrentDesktop 1
- `show internet` = qdbus org.kde.KWin /KWin setCurrentDesktop 2
- `show kodi` = qdbus org.kde.KWin /KWin setCurrentDesktop 4

Rescue
------

All commands should be run as root unless specified.

### Mount LUKS partition ###

To mount a LUKS encrypted partition:

```bash
# first we decrypt
cryptsetup luksOpen /dev/sda3 zotac
# then we mount
mkdir /media/zotac
mount /dev/mapper/zotac /media/zotac
```

If needed, we can unmount it to relock it:
```bash
umount /media/zotac
cryptsetup luksClose zotac
```

### Chroot to zotac ###

We assume that the system partition is mounted in `/media/zotac`.

First we need to prepare all the system folders:

```bash
mount --bind /dev /media/zotac/dev
mount -t proc /proc /media/zotac/proc
mount --bind /run  /media/zotac/run
mount -t sysfs /sys /media/zotac/sys
```

**Note**: to only install new kernel, do this:
```bash
mount /dev/sda2 /media/zotac/boot
mount -o bind /proc /media/zotac/proc
mount -o bind /proc /media/zotac/dev
```

Finally, the chroot:

```bash
chroot /media/zotac
```
To quit, just run `exit`.


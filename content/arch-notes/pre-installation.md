---
title: "Pre-Installation"
date: 2020-07-04T17:09:50+02:00
draft: false
toc: true
summary: "Contents of this post: Settings and configurations to prepare the installation. (internet, time and date, partitioning, etc.)"
tags:
  - arch
  - pre-install
---

## Connecting to the internet

List network interfaces:

```terminal
ip link
```

For wireless connection authenticate to the network using [iwctl](https://wiki.archlinux.org/index.php/Iwd#iwctl):

```terminal
iwctl
```

In the iwctl prompt, list all Wi-Fi devices:

```
[iwd]# device list
```

If iwctl shows that the device is powered off, exit from iwctl and enable the interface with the following command,

```terminal
rfkill unblock all
```

then run iwctl again. Scan for networks (replace *device* with a device name):

```
[iwd]# station device scan
```

List available networks (replace *device* with a device name):

```
[iwd]# station device get-networks
```

Connect to a network (replace *device* and *SSID* with the appropriate values):

```
[iwd]# station device connect SSID
```

To share a phone's internet connection through USB, launch [dhcpcd](https://wiki.archlinux.org/index.php/Dhcpcd), then connect the phone and start tethering.

```terminal
dhcpcd 
```

Test the connection with the following command:

```terminal
ping archlinux.org
```

## Time and date

Update the system clock with [timedatectl](https://wiki.archlinux.org/index.php/System_time#System_clock).

```terminal
timedatectl set-ntp true
```

## Partitioning the disk

Use [fdisk](https://wiki.archlinux.org/index.php/Fdisk) to create partitions.
Use [lsblk](https://wiki.archlinux.org/index.php/Device_file) to get the appropriate device names. (e.g. `/dev/sda`)

```terminal
fdisk /path/to/device
```

For UEFI:

1. **g** - for a new GUID Partition Table
2. **n** - for the efi partition, partition number: default, first sector: default, last sector: +512M
3. **n** - for a swap partition (optional), partition number: default, first sector: default, last sector: +8G (in my case)
4. **n** - for the root partition, partition number: default, first sector: default, last sector: default
5. **w** - to write the changes on the disk

For MBR:

1. **o** - for a new DOS Partition Table
2. **n** - for a swap partition (optional), partition number: default, first sector: default, last sector: +8G (in my case)
3. **n** - for the root partition, partition number: default, first sector: default, last sector: default
4. **w** - to write the changes on the disk

## Creating filesystems

Use the partition names in the paths (e.g. `mkfs.ext4 /dev/sda3`)

If you created an efi partition:

```terminal
mkfs.fat -F32 /path/to/efi/partition
```

If you created a swap partition:

```terminal
mkswap /path/to/swap/partition
```

For the root partition:

```terminal
mkfs.ext4 /path/to/root/partiton
```

## Mounting the partitons

Attach the created partitions to the existing filesystem.

First the root partition:

```terminal
mount /path/to/root/partition /mnt
```

If you created an efi partiton:

```terminal
mkdir /mnt/boot
```

```terminal
mkdir /mnt/boot/EFI
```

```terminal
mount /path/to/efi/partition /mnt/boot/EFI
```

If you created a swap partiton:

```terminal
swapon /path/to/swap/partition
```

## Symlinking /etc/resolv.conf

If you want to use [systemd-resolved](https://wiki.archlinux.org/title/Systemd-resolved) for name resolution, run the following command:

```terminal
ln -sf /run/systemd/resolve/stub-resolv.conf /mnt/etc/resolv.conf
```

(This symlink cannot be created while inside arch-chroot.)

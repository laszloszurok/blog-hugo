---
title: "Libvirt notes"
date: 2024-10-06T09:45:46+02:00
pagefind_index_page: true
---

## Packages on Arch Linux

```terminal
sudo pacman -S libvirt virt-install virt-viewer qemu-full iptables-nft dnsmasq edk2-ovmf libosinfo
```

## GUI frontends

* virt-manager
* gnome-boxes

## Libvirt system level vs session level

By default the `virsh` and `virt-*` commands will use the `qemu:///session` connection, which launches an unprivileged libvirtd instance (the process name is `virtqemud` on Arch) running as the current user. There is a system level libvirtd instance launched by systemd.

### Differences

Session level virtual machines use qemu user networking, which might have worse performance than the system level networking.
The system libvirtd instance has the necessary permissions to use proper networking via bridges or virtual networks.
VM autostart on host boot only works for the system level VMs.

The graphical frontend `virt-manager` uses `qemu:///system` by default and virtual machines running in unprivileged sessions won't show up on the GUI.

### Use the system level instance

To use the systemwide libvirtd instance `--connect qemu:///system` has to be specified in the commands.
The user has to be a member of the `libvirt` group.

## Get OS variants

```terminal
osinfo-query os
```

or

```terminal
virt-install --osinfo list
```

## Create a VM with virt-install

### 3D acceleration

For 3D acceleration pass these options to virt-install:

* `--graphics spice,listen=none,gl.enable=yes`
* `--video virtio,accel3d=yes`

Alternatively 3D acceleration can be enabled in the domain xml with these settings:

```xml
<graphics type='spice'>
  <listen type='none'/>
  <gl enable='yes'/>
</graphics>

<video>
  <model type='virtio' heads='1' vram='65536'>
    <acceleration accel3d='yes'/>
  </model>
</video>
```

### From an iso file

```terminal
virt-install \
    --name archvm \
    --memory 2048 \
    --vcpus 2 \
    --disk size=8 \
    --cdrom archlinux-2024.10.01-x86_64.iso \
    --boot uefi \
    --os-variant archlinux
```

### From a qcow2 image

```terminal
virt-install \
    --name archvm \
    --memory 2048 \
    --vcpus 2 \
    --disk Arch-Linux-x86_64-basic-20241001.267073.qcow2 \
    --boot uefi \
    --import \
    --os-variant archlinux
```

### For Windows guests

https://www.spice-space.org/index.html

Install [spice-guest-tools](https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-latest.exe) inside the VM for proper resolution and clipboard share.

#### Home Assistant example

A USB host device can be passed through to the guest with the `--hostdev` flag.
List USB devices on the host with `lsusb`.
Format: `bus-number`.`device-number`

```terminal
virt-install \
    --name haos \
    --description "Home Assistant OS" \
    --os-variant=generic \
    --ram=4096 \
    --vcpus=2 \
    --disk /var/lib/libvirt/images/haos_ova-16.3.qcow2,bus=scsi \
    --controller type=scsi,model=virtio-scsi \
    --import \
    --graphics none \
    --boot uefi \
    --hostdev 001.005
```

## List VMs

```terminal
virsh list
```

## Access a running VM with GUI

```terminal
virt-viewer archvm
```

## Delete VM

```terminal
virsh destroy archvm
```
```terminal
virsh undefine archvm
```

In case of `Cannot undefine domain with nvram...` error:

```terminal
virsh undefine archvm --nvram
```

To remove the qcow2 image file with the vm:

```terminal
virsh undefine archvm --remove-all-storage
```

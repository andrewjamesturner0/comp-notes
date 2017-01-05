---
title: Install Archlinux on bhyve
author: Andrew Turner
date: 2015-09-20
---

## Prepare FreeBSD host

### Install packages

The grub bootloader is needed to boot linux guests.

`sysutils/grub2-bhyve`

### Networking

Create a `tap` interface, which will be attached to the VM.

    # ifconfig tap0 create
    # sysctl net.link.tap.up_on_open=1

Create a `bridge` interface, which connects the `tap` interface to the rest of
the network.

    # ifconfig bridge0 create
    # ifconfig bridge0 addm em0 addm tap0
    # ifconfig bridge0 up

`em0` is the physical interface on the host. (Not necessarily called `em0`.)
The bridge will allow the VM to get an ip through DHCP or through a static 
configuration as if it were just another computer on the network. 

To create the network interfaces on boot edit:

`/etc/sysctl.conf`

    net.link.tap.up_on_open=1

`/boot/loader.conf`

    if_bridge_load="YES"
    if_tap_load="YES"

`/etc/rc.conf`

    cloned_interfaces="bridge0 tap0"
    ifconfig_bridge0="addm em0 addm tap0"


### Load the bhyve kernel module

    # kldload vmm

To load on boot edit: 

`/boot/loader.conf`

    vmm_load="YES"


### VM disk image

Create a ZFS volume for the VM's disk

    # zfs create -V 20G -o volmode=dev storage/bhyve-archlinux

The volume will be available under `/dev/zvol/storage/bhyve-archlinux`


## Install Archlinux

### Device map

Create a device map for grub that contains the paths to the VM's disk and
an ubuntu ISO image.

`bhyve-arch.map`

    (hd0) /dev/zvol/storage/bhyve-archlinux
    (cd0) /path/to/ubuntu-14.04.1-server-amd64.iso

### Start the VM

#### grub-bhyve

Use `grub-bhyve` to load the linux kernel from the ISO

    # grub-bhyve -m bhyve-arch.map -r cd0 -M 1024 archlinux

With the ubuntu ISO this will boot straight to a menu: select `rescue a
broken system`. Otherwise use the grub prompt to define the linux and the
initrd boot commands.

#### bhyve

Use `bhyve` to boot the VM

    # bhyve -A -H -P \ 
        -s 0:0,hostbridge \
        -s 1:0,lpc \
        -s 2:0,virtio-net,tap0 \
        -s 3:0,virtio-blk,/dev/zvol/storage/archlinux \
        -s 4:0,ahci-cd,/path/to/ubuntu-14.04.1-server-amd64.iso \
        -l com1,stdio \
        -c 2 -m 1024M \
        archlinux

After a few questions you will get a shell in th (limited) live rescue
enivronment.


### Partition the VM's disk

Downloading and extracting the bootstrap image may not fit in the live
environment. Parition the VM's disk to create a spare partition at the end of
the drive. Once arch is installed, this partition can be converted into swap
space.

    # parted -s /dev/vda mklabel gpt
    # parted -s /dev/dva -- \
        mkpart primary 1M 2M
        mkpart primary 2M 9.5G
        mkpart primary 9.5G 10G
    # parted -s /dev/vda -- set 1 bios_grub on
    # mkfs.ext4 /dev/vda2
    # mkfs.ext4 /dev/vda3

### Bootstrap archlinux

Download the bootstap image:

    # mkdir /mnt/bootstrap
    # mount /dev/vda3 /mnt/bootstrap
    # cd /mnt/bootstrap
    # wget http://mirrors.kernel.org/archlinux/iso/2015.08.01/archlinux-bootstrap-2015.08.01-x86_64.tar.gz
    # tar xzf archlinux-bootstrap-2015.08.01-x86_64.tar.gz

No vim or vi in the ubuntu environment, so use nano to select package repo
mirrors:

    # nano root.x86_64/etc/pacman.d/mirrorlist

Chroot into the bootstrap image:

    # cd root.x86_64
    # cp /etc/resolv.conf etc
    # mount -t proc /proc proc
    # mount --rbind /sys sys
    # mount --rbind /dev dev
    # mount --rbind /run run
    # chroot . /bin/bash

**To be clear: We are now chrooted into an Arch bootstrap environment held on a small
partition on the VM's disk, from within the Ubuntu live environment**

Generate some entropy, ready to initialise pacman:

    # for i in 1 2 3 4 5; do ls -Ra /; ls -Ra /; done

Initialise pacman:

    # pacman-key --init
    # pacman-key --populate archlinux

Install arch:

    # mount /dev/vda2 /mnt
    # mkdir -m 0755 -p \
        /mnt/var/{cache/pacman/pkg,lib/pacman,log} \
        /mnt/{dev,run,etc}
    # mkdir -m 1777 -p /mnt/tmp
    # mkdir -m 0555 -p /mnt/{sys,proc}
    # mount -t proc /proc /mnt/proc
    # mount --rbind /sys /mnt/sys
    # mount --rbind /run /mnt/run
    # mount --rbind /dev /mnt/dev
    # pacman -r /mnt \
        --cachedir=/mnt/var/cache/pacman/pkg \
        --noconfirm \
        -Sy base base-devel vim openssh grub
    # cp -a /etc/pacman.d/gnupg /mnt/etc/pacman.d/
    # cp -a /etc/pacman.d/mirrorlist /mnt/etc/pacman.d/

Configure arch:

    # chroot /mnt ... Configure as normal

**To be clear: We are now chrooted into the final Arch environment, from the bootstrap
environment, from the ubuntu live environment**

Once arch is installed, exit the chroots and poweroff. This will cause bhyve to
exit.

Remove the bhyve instance on the host:

    # bhyvectl --destroy --vm=archlinux


## Running the VM

To start a new instance of the VM, repeat the boot procedure. Again, this is
done in two parts:

    # grub-bhyve -m bhyve-arch.map -r hd0,gpt2 -M 1024 archlinux

Note that the root partition of the VM is specifed, rather than 'cd0' as
before. In fact, the ISO image can now be removed from the .map file.

Now run bhyve:

    # bhyve -A -H -P \ 
        -s 0:0,hostbridge \
        -s 1:0,lpc \
        -s 2:0,virtio-net,tap0 \
        -s 3:0,virtio-blk,/dev/zvol/storage/archlinux \
        -l com1,stdio \
        -c 2 -m 1024M \
        archlinux

Note here also that there is no longer any need to specify the path to the
ISO image.

These two commands can be put together in a shell script. Don't forget to
destroy the instance when it is shutdown.

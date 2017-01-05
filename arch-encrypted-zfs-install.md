---
title: Arch Encrypted ZFS Install
author: Andrew Turner
date: 2016-09-03
---

## Partition device

Create gpt partition on disk

    # parted -s /dev/sda mklabel gpt

Create partitions

    # parted -s /dev/sda -- \
    mkpart ESP fat32 1M 1G \
    mkpart primary 1G 100%

Mark first parition as boot

    # parted -s /dev/sda -- set 1 boot on

Create encypted partition

    # cryptsetup \
      --cipher aes-xts-plain64 \
      --key-size 512 \
      --hash sha512 \
      --iter-time 5000 \
      --use-random \
      --verify-passphrase \
      luksFormat /dev/sda2

Open the new LUKS device

    # cryptsetup luksOpen /dev/sda2 luksroot

Make filesystems

    # mkfs.fat -F32 /dev/sda1

    # zpool create -f -o ashift=12 system /dev/mapper/luksroot
    # zfs set compression=lz4 system
    # zfs create -o mountpoint=none system/ROOT
    # zfs create -o mountpoint=/ system/ROOT/default
    # zfs create -o mountpoint=/home system/HOME
    # zpool set bootfs=system/ROOT/default system
    # zpool export system



Mount datasets in /mnt:

    # zpool import -R /mnt system
    # mkdir /mnt/boot
    # mount /dev/sda1 /mnt/boot


Install system

    # pacstrap /mnt base base-devel efibootmgr grub vim
    # genfstab -p /mnt >> /mnt/etc/fstab
    # cp /etc/zfs/zpool.cache /mnt/etc/zfs/zpool.cache

Chroot

    # arch-chroot /mnt /bin/bash
    # grub-install \
    --target=x86_64-efi \
    --efi-directory=/boot \
    --bootloader-id=grub

Don't edit `/etc/default/grub`, instead you must edit `/boot/grub/grub.cfg`
directly:

    # TODO

Note that in VirtualBox, the EFI directory must be changed:

    # mkdir -p /boot/EFI/BOOT
    # cp /boot/EFI/grub/grubx64.efi /boot/EFI/BOOT/bootx64.efi

Install zfs AUR packages:

https://aur.archlinux.org/cgit/aur.git/snapshot/spl-utils-linux-git.tar.gz

https://aur.archlinux.org/cgit/aur.git/snapshot/spl-linux-git.tar.gz

https://aur.archlinux.org/cgit/aur.git/snapshot/zfs-utils-linux-git.tar.gz

https://aur.archlinux.org/cgit/aur.git/snapshot/zfs-linux-git.tar.gz




    # pool set cachefile=/etc/zfs/zpool.cache system


## Using two disks

Follow above, but change as necessary so that the btrfs filesystem spans two
encrypted disks.

The trickier part is unlocking both disks at boot.

Create a second encrypt hook for `mkinitcpio` as follows:

    # cp /usr/lib/initcpio/install/encrypt /usr/lib/initcpio/install/encrypt2
    # cp /usr/lib/initcpio/hooks/encrypt  /usr/lib/initcpio/hooks/encrypt2
    # sed -i "s/cryptdevice/cryptdevice2/" /usr/lib/initcpio/hooks/encrypt2
    # sed -i "s/cryptkey/cryptkey2/" /usr/lib/initcpio/hooks/encrypt2

Then add a new `encrypt2` hook to `/etc/mkinitcpio.conf` and a new
`cryptdevice2` option to the `GRUBCMDLINE` in `/etc/default/grub`.

Remember to re-run `mkinitcpio` and `grub-mkconfig`.

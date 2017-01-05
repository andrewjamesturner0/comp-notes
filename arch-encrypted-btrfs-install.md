---
title: Arch Encrypted Btrfs Install
author: Andrew Turner
date: 2016-06-19
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
    # mkfs.btrfs /dev/mapper/luksroot

Create Btrfs subvolumes:

    # mount -o noatime,discard,ssd,defaults /dev/mapper/luksroot /mnt
    # cd /mnt
    # btrfs subvolume create system
    # btrfs subvolume create system/root
    # btrfs subvolume create system/home
    # btrfs subvolume create snapshots

Mount subvolumes in /mnt:

    # cd
    # umount /mnt
    # mount -o subvol=system/root /dev/mapper/luksroot /mnt
    # mkdir /mnt/{home,boot}
    # mount -o subvol=system/home /dev/mapper/luksroot /mnt/home
    # mount /dev/sda1 /mnt/boot

Install system

    # pacstrap /mnt base base-devel btrfs-progs efibootmgr grub vim
    # genfstab -p /mnt >> /mnt/etc/fstab
    # arch-chroot /mnt /bin/bash
    # grub-install \
    --target=x86_64-efi \
    --efi-directory=/boot \
    --bootloader-id=grub

Edit `/etc/default/grub` GRUB CMDLINE:

    root=/dev/mapper/luksroot rootflags=subvol=system/root cryptdevice=/dev/sda2:luksroot rw

Note that in VirtualBox, the EFI directory must be changed:

    # mkdir -p /boot/EFI/BOOT
    # cp /boot/EFI/grub/grubx64.efi /boot/EFI/BOOT/bootx64.efi

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

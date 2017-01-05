---
title: Arch Partitioning and LVM
author: Andrew Turner
date: 2014-11-29
version: 4-20161231
---

## Partition device

Create gpt partition table on disk

    # parted -s /dev/sda mklabel gpt

Create partitions

    # parted -s /dev/sda -- \
    mkpart primary 1M 2M \
    mkpart primary 2M 100%

Mark first partition as UEFI system partition

    # parted -s /dev/sda -- set 1 bios_grub on

Use LVM to divide up the second partition.

Note that this does not require `/boot` to be outside LVM.

Format `/boot` logical volume as FAT32, if using UEFI.


## LVM 

Create a Physical Volume

    # pvcreate <device>

Create a Volume Group

    # vgcreate <vg-name> <device>

Add another disk to the Volume Group

    # vgextend <vg-name> <device>

Create a Logical Volume (e.g. root, boot, home)

    # lvcreate --size <size>G <vg-name> --name <lv-name>

Create a Logical Volume for SWAP space

    # lvcreate --contiguous y --size <size>G <vg-name> <swap-lv-name>

Keep an eye on what has been done

    # {pv,vg,lv}display

Make a filesystem on the logical volumes

    # mkfs.etx4 <lv-device>


## Create encrypted devices

LUKS on LVM is a more felxible setup than LVM on LUKS,
if the volume group which holds the root logical volume is
going to span multiple disks.


Create LUKS device

    # cryptsetup \
      --cipher aes-xts-plain64 \
      --key-size 512 \
      --hash sha512 \
      --iter-time 5000 \
      --use-random \
      --verify-passphrase \
      luksFormat <device> 

Open the new LUKS device

    # cryptsetup luksOpen <device> <device-mapper-name>

Supply the volume with the keyfile, so it can be added as a way to unlock it.

It does not matter where the key file is located at this stage.

    # cryptsetup luksAddKey <device> </path/to/additionalkeyfile>

Edit `/etc/crypttab`

If the logical volume which holds an encrypted partition spans multiple drives, then an entry is needed in /etc/cryptab. 

For example:

    home    /dev/mapper/VG0-home    </path/to/keyfile>    allow-discards

For encrypted swap logical volume (plus corresponding entry in fstab):

    swap    /dev/mapper/VG0-swap    /dev/urandom    swap,cipher=aes-cbc-essiv:sha256

Make a file system on the encrypted partition

    # mkfs.etx4 <device-mapper-name>


## Resizing partitions

#### LVM: Replace a smaller disk with a larger one

To replace a disk in a volume group, first prepare the new disk `/dev/sdc` by
creating a partition table and physical volume:

    # parted -s /dev/sdc mklabel gpt
    # parted -s /dev/sdc -- mkpart primary 1M 100%
    # pvcreate /dev/sdc1

Add this new physical volume to the existing volume group on the disk to be
replaced:

    # vgextend VG0 /dev/sdc1

Move the blocks from the old disk onto the new disk:

    # pvmove -v /dev/sda1 /dev/sdc1

Now you can remove the old disk:

    # vgreduce VG0 /dev/sda1

Resize the logical volume to use the additional space that is now available:

    # lvextend --size +100G /dev/mapper/VG0-home

Note that if the filesystem on top is ext2/3/4, reiserfs or xfs then it can be
resized automatically at the same time with the `--resizefs` option. Otherwise,
resize the filesystem to use the newly extended logical volume.

#### LUKS

Expand a LUKS partition to the size of a (larger) underlying LVM logical volume 
(e.g. when a new disk is added):

    # cryptsetup --verbose resize <device-mapper-name>

Then you must expand the filesystem on top of the LUKS partition. For example,
for ext2/3/4 filesystems:

    # e2fsck -f <device-mapper-name>
    # resize2fs <device-mapper-name>

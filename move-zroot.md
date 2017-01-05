---
title: Migrating FreeBSD to a new disk
author: Andrew Turner
date: 2015-06-06
---

Procedure for migrating a FreeBSD root-on-zfs install to a new disk.

### Prepare new disk

    # gpart create -s gpt ada1
    # gpart add -a 1M -s 512k -t freebsd-boot -l boot ada1
    # gpart add -a 1M -s 8G -t freebsd-swap -l swap1 ada1
    # gpart add -a 1M -t freebsd-zfs -l system ada1
    # gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada1

### Create zpool

On 10.1+ there is a sysctl telling zfs to use 4096 byte sectors.

    # sysctl vfs.zfs.min_auto_ashift=12

Create the pool, enabling compression, turning off atime and without assigning a mountpoint.

    # zpool create -o altroot=/mnt -o compress=lz4 -o atime=off -m none system gpt/system

### Send a snapshot of the current system to the new zpool

    # zfs snapshot -r zroot@snapshot
    # zfs send -vR zroot@snapshot | zfs recv -F system

### Prepare for next boot

    # zpool set bootfs=system/ROOT/default system

* Edit /mnt/etc/fstab to include the new swap partition.
* Edit anything that depended on the name 'zroot'.

Poweroff, disconnect the old drive and start up again. Mission acomplished.

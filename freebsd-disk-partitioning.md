---
title: Partitioning and ZFS 
author: Andrew Turner
date: 2014-12-05
---

## gpart

    # gpart destroy -F <device>
    # gpart create -s gpt <device>

### Boot partition:

    # gpart add -a 1M -t freebsd-boot -s 512 -l <name> <device>
    # gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 <device>

### zfs partition:

    # gpart add -t freebsd-zfs -a 1M -s <size> -l <name> <device>

### swap partition:

    # gpart add -t freebsd-swap -a 1M -l <name> <device>

* Use the -s option if you want to create a partition of a given size (e.g. -s
  10G). Otherwise, the partition will fill the space available on the device.


## GELI Encrypted disks

Generate key

    # dd if=/dev/random of=/path/to/new/keyfile.key bs=1024 count=2

Initialise device

    # geli init -b -P -K /path/to/new/keyfile.key <device>

* -P option removes the need for a passphrase
* -b makes the devices attachable at boot
* Other options include: -a [Blowfish | 3DES | AESa.]

Attach device

    # geli attach -p -k /path/to/keyfile <device>

* keyfile must be on root partition, to attach device at boot.

Set-up auto mount

    # echo 'geli_devices="<device e.g. ada2>"' >> /etc/rc.conf
    # echo 'geli_ada2_flags="-p -k /path/to/keyfile"' >> /etc/rc.conf
    # echo 'geli_autodetach="NO"' >> /etc/rc.conf
    # echo 'geom_eli_load="YES"' >> /boot/loader.conf
    # echo 'geli_ada2_keyfile0_load="YES"' >> /boot/loader.conf
    # echo 'geli_ada2_keyfile0_type="ada2:geli_keyfile0"' >> /boot/loader.conf
    # echo 'geli_ada2_keyfile0_name="/enc.key"' >> /boot/loader.conf

* Autodetatch=no is important - prevents a panic when scrubing the zpool.

## ZFS


### Enable ZFS

    # echo 'zfs_enable="YES"' >> /etc/rc.conf

### Create pool

Use -f for different size disks

    # zpool create -f storage0 raidz ada1 ada2 ada3
    # zfs set mountpoint=none storage0
    # zfs create storage0/backups
    # zfs set compression=gzip-9 storage0/backups
    # zfs set mountpoint=/backup storage0/backups

### Updating a disk

    # zpool replace storage0 ada3 <new disk>
    # zpool online -e storage <new disk>

### Enable automatic scrubbing

    # cat >> /etc/periodic.conf.local << 'EOF'
    daily_scrub_zfs_enable="YES"
    EOF

* This doesn't scrub zpools daily! - it defaults to scrubbing every 35 days.
* Use `daily_scrub_zfs_default_threshold="N"` to change this behaviour. 

### Renaming a zpool

Export the pool, and then import it using the old name, followed by the new name:

    # zpool export pool
    # zpool import pool neo

---
title: ZFS Remote Replication
author: Andrew Turner
date: 2017-08-20
version: 1-20170820
---

### Users

Privileges for the user performing the replication:

On sending side:

    # zfs allow -u sender send,snapshot dataset/home

On receiving side: 

    # zfs allow -u recver compression,create,destroy,mount,mountpoint,receive
    # sysctl vfs.usermount=1

### Receiving dataset

    # zfs create -o mountpoint=/backup remotestorage/backup
    # chown recver:recver /backup

### Recursive replication

    # zfs send -Rv system@snapshot0 | ssh recver@host zfs recv -Fud remotestorage/backup


### Incremental replication

    # zfs send -Rv -i snapshot0 system@snapshot1 | ssh recver@host zfs recv -ud
    remotestorage/backup

* This will not send any snapshots between `snapshot0` and `snapshot1`.
* To send all intermediate snapshots, perform a _differential_ backup, using
  `-I` instead.
* If snapshots between `snapshot0` and `snapshot1` exist on the receiving side,
  then `-i` will fail. It can be forced with `-F`, which will remove the
  intermediate snapshots on the receiving side.

* `-d` preserves the hierarchy of the datasets being sent.

* `-u` ensures the received dataset is not mounted.

### Protecting the replicated dataset

    # zfs set readonly=on remotestorage/backup
    # zfs set canmount=noauto remotestorage/backup

* If the received dataset has changed, then an incremental (`-i`) send will
  fail. It can be forced with `-F` which will rollback the dataset on the
  receiving side until there are no changes that prevent receiving the stream.

And include any child datasets.


### Bookmarks

TODO

---
title: Poudriere
author: Andrew Turner
date: 2015-12-05
---


## Install ports managers

    # pkg install poudriere portmaster

## Create credentials for package signing

    # mkdir -p /usr/local/etc/ssl/{keys,certs}
    # chmod 0600 /usr/local/etc/ssl/keys
    # openssl genrsa -out /usr/local/etc/ssl/keys/poudriere.key 4096
    # openssl rsa -in /usr/local/etc/ssl/keys/poudriere.key -pubout -out /usr/local/etc/ssl/certs/poudriere.cert

## Configure poudriere

#### Edit `poudriere.conf`

    # vim /usr/local/etc/poudriere.conf

Set:

    ZPOOL=
    ZROOTFS=
    FREEBSD_HOST=ftp://ftp.freebsd.org
    POUDRIERE_DATA=${BASEFS}/data
    CHECK_CHANGED_OPTIONS=verbose
    CHECK_CHANGED_DEPS=yes
    PKG_REPO_SIGNING_KEY=/usr/local/etc/ssl/keys/poudriere.key
    URL_BASE=

#### Setup build environment

    # poudriere jail -c -j 10-1-x64 -v 10.1-RELEASE
    # poudriere ports -c -p HEAD

Check what you've done with:

    # poudriere jail -l
    # poudriere ports -l

#### Setup build options

    # mkdir -p /usr/local/etc/poudriere.d

Create a `make.conf`:

    # cat >> /usr/local/etc/poudriere.d/10-1-x64-make.conf << 'EOF'
    OPTIONS_UNSET+= DOCS NLS EXAMPLES QT4 X11 CUPS
    WITH_PKGNG=yes
    EOF

Create a list of ports to build:

    # cat >> /usr/local/etc/poudriere.d/buildlist << 'EOF'
    net/rsync
    sysutils/zfstools
    ...etc
    EOF

Configure build options for ports:

    # poudriere options -j 10-1-x64 -p HEAD -f /usr/local/etc/poudriere.d/buildlist

To re-configure these options, add `-c`.


## Run poudriere

#### Update ports and jails

    # poudriere ports -u -p HEAD
    # poudriere jail -u -j 10-1-x64

Update jail minor verison

    # poudriere jail -u -t 10.x-RELEASE -j 10-1-x64

#### Check for new build options

    # poudriere options -j 10-1-x64 -p HEAD -f /usr/local/etc/poudriere.d/buildlist

#### Build!

    # poudriere bulk -j 10-1-x64 -p HEAD -f /usr/local/etc/poudriere.d/buildlist


## Configure pkg to use the poudriere repo

Enable poudriere repo:

    # mkdir -p /usr/local/etc/pkg/repos
    # cat >> /usr/local/etc/pkg/repos/poudriere.conf << 'EOF'
    poudriere: {
        url: "file:///usr/local/poudriere/data/packages/10-1-x64-HEAD",
        mirror_type: "srv",
        signature_type: "pubkey",
        pubkey: "/usr/local/etc/ssl/certs/poudriere.cert",
        enabled: yes
    }
    EOF

Disable fresbsd repo:

    # cat >> /usr/local/etc/pkg/repos/freebsd.conf << 'EOF'
    FreeBSD: {
        enabled: no
    }
    EOF

    # pkg update


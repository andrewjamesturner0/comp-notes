---
title: FreeBSD package set up
author: Andrew Turner
date: 2015-12-05
---


## pkg

    # pkg
    # echo WITH_PKGNG=yes >> /etc/make.conf
    # pkg2ng

## sudo

    # visudo

Uncomment

    %wheel ALL=(ALL) ALL

Add

    Defaults insults
    Cmnd_Alias PACKAGES # /usr/local/sbin/pkg, /usr/local/sbin/portmaster, /usr/sbin/portsnap, /usr/sbin/freebsd-update
    %wheel ALL=(ALL) NOPASSWD: /sbin/shutdown
    %wheel ALL=(ALL) NOPASSWD:PACKAGES

Add user to wheel group

    # pw groupmod wheel -M [user]

## bash

    # chsh -s /usr/local/bin/bash
    # ln -s /usr/local/bin/bash /bin/bash

Add source for bashrc in bash_profile

    # echo '[ -f ~/.bashrc ] && . ~/.bashrc' >> ~/.bash_profile

Add entry to fstab

    fdesc   /dev/fd     fdescfs     rw  0   0

## samba

    # echo 'samba_enable="YES"' >> /etc/rc.conf

fstab options for smbfs

    smbfs   noauto,rw,-LUTF-8,-N

Add to smb.conf

    client plaintext auth # yes<
    client lanman auth # yes

## Virtualbox

Load the vboxdrv kernel module

    # echo 'vboxdrv_load="YES"' >> /boot/loader.conf

Change default PATH

    # vbm setproperty machinefolder /data/VMs

Add user to vboxusers group

    # pw groupmod vboxusers -m <user>

For bridged networking

    # echo 'vboxnet_enable="YES"' >> /etc/rc.conf

## Apache

    # pkg install apache24
    # echo 'apache24_enable="YES"' >> /etc/rc.conf
    # sudo servive apache24 start

Note: apache directory is: /usr/local/www/apache

## Syncthing

    # pkg install syncthing
    # cat >> /etc/rc.conf << 'EOF'

    #syncthing
    syncthing_enable="YES"
    syncthing_user="ajt"
    syncthing_group="ajt"
    syncthing_dir="/home/ajt/.config/syncthing"

    EOF

    # touch /var/run/syncthing.pid
    # sudo chown ajt:ajt /var/run/syncthing.pid

Edit `~/.config/syncthing/config.xml`

* Change listen address to 0.0.0.0


## Sendmail

### Install

    $ sudo pkg install ssmtp

### disable default sendmail

Add to `/etc/rc.conf`

    # disable default sendmail
    sendmail_enable="NO"
    sendmail_submit_enable="NO"
    sendmail_outbound_enable="NO"
    sendmail_msp_queue_enable="NO"

Edit `/etc/mail/mailer.conf`

    sendmail        /usr/local/sbin/ssmtp
    send-mail       /usr/local/sbin/ssmtp
    mailq           /usr/local/sbin/ssmtp
    newaliases      /usr/local/sbin/ssmtp
    hoststat        /usr/bin/true
    purgestat       /usr/bin/true

### Create ssmtp config files

Copy sample configs

    $ sudo cp /usr/local/etc/ssmtp/ssmtp.conf.sample /usr/local/etc/ssmtp/ssmtp.conf
    $ sudo cp /usr/local/etc/ssmtp/revaliases.sample /usr/local/etc/ssmtp/revaliases

Edit `/usr/local/etc/ssmtp/ssmtp.conf`

    root=<EMAIL>@gmail.com
    mailhub=smtp.googlemail.com:465
    AuthUser=<EMAIL>@gmail.com
    AuthPass=<PASSWORD>
    FromLineOverride=YES
    UseTLS=YES
    hostname=_HOSTNAME_
    rewriteDomain=

Edit `/usr/local/etc/ssmtp/revaliases`

    root:<EMAIL>@gmail.com:smtp.googlemail.com:465

### Send email

Create header file

    $ cat >> email-header.txt << 'EOF'
    To: <to@mail.com>
    From: <EMAIL>@gmail.com
    Subject: zpool status
    EOF

Add crontab to send email

    @daily printf "$(cat /home/ajt/email-header.txt) $(date +%F-%H-%M)\n\n$(zpool status)\n" | ssmtp -v andrewjamesturner0@gmail.com -


## htop

Add sysutils/htop to poudriere build list

Add to /etc/fstab:

    linproc /compat/linux/proc linprocfs rw,late 0 0

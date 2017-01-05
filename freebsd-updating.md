---
title: Updating FreeBSD
author: Andrew Turner
date: 2015-12-05
---


## System update

    # freebsd-update fetch install 

## System upgrade

Upgrade to new release

    # freebsd-update upgrade -r 10.X-RELEASE
    # freebsd-update install
    # reboot

Rebuild packages, then compete the update

    # freebsd-update install
    # reboot

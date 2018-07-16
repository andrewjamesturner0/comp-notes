---
title: Set up fail2ban on FreeBSD
author: Andrew Turner
date: 2018-07-15
version: 20180716
---

# pf

Enable pf:

    # echo 'pf_enable="YES"' >> /etc/rc.conf
    # service pf start

Add `anchor "f2b/*"` to `/etc/pf.conf`.

Example `pf.conf`: 

    tcp_services = "{ ssh, http, https, 137, 138, 139 }"
    udp_services = "{ 137, 138, 139 }"
    anchor "f2b/*"
    pass out quick all keep state
    block in all
    pass in quick inet proto icmp all
    pass in quick inet6 proto icmp6 all
    pass in proto tcp from any to any port $tcp_services
    pass in proto udp from any to any port $udp_services

The tcp_services macro can be usde to list all ports for which to allow packets.
These can be numbers, or names from `/etc/services`.


# fail2ban

Install fail2ban:

    # pkg install py27-fail2ban
    # pkg install py27-pyinotify

All local config is done in `/usr/local/etc/fail2ban/jail.local`, do not edit `jail.conf`.

Example `jail.local`:

    [DEFAULT]
    bantime = 86400
    findtime = 3600
    maxretry = 3
    banaction = pf
    
    [sshd]
    enabled = true

Restart pf and start fail2ban:

    # service pf reload
    # echo 'fail2ban_enable="YES"' >> /etc/rc.conf
    # service fail2ban start

Check ips are being banned:

    # pfctl -a f2b/sshd -t f2b-sshd -T show


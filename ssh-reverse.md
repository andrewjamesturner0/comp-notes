---
title: SSH reverse tunneling
author: Andrew Turner
date: 2015-12-05
---


SSH reverse tunneling allows one to ssh into a machine that is behind a firewall
that (a) one doesn't control, but (b) one can ssh out from.

The aim is to reverse the connection, to connect to the machine inside the
firewall.

## Step 1

On the machine within the firewall, ssh to the machine outside using the
following options:

    $ ssh -fN -R 9000:127.0.0.1:22 user@outside.com

## Step 2

On the machine outside the firewall, one can now make a connection to the
machine on this inside by ssh'ing to localhost port 9000:

    $ ssh -p 9000 user@127.0.0.1


## Keep the connection alive

One can only ssh into the machine behind the firewall while there is an active
connection from that machine to the machine on the outside. So it is useful to
add the following to `sshd_config`:

    ClientAliveInterval 300 TCPKeepAlive yes


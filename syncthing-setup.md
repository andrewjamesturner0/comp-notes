---
title: Syncthing Setup on Ubuntu
author: Andrew Turner
date: 2015-12-05
---


## Installing


Download source from github

    https://github.com/syncthing/syncthing/releases

Extract source and cd into folder

Install:

    $ sudo cp syncthing /usr/local/bin/syncthing

Create upstart script:

    # cat >> /etc/init/syncthing.conf << 'EOF'
    description "Syncthing P2P sync service"
    
    start on (local-filesystems and net-device-up IFACE!=lo)
    stop on runlevel [!2345]
    
    env STNORESTART=yes
    env HOME=/home/<USER>
    setuid "<USER>"
    setgid "<USER>"
    
    exec /usr/local/bin/syncthing
    
    respawn
    EOF

## Upgrading

Stop synthing

    $ sudo service syncthing stop

Re-copy, as above

    $ sudo cp syncthing /usr/local/bin/syncthing

Restart syncthing

    $ sudo service syncthing start

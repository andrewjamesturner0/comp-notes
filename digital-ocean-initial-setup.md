---
title: Digital Ocean Droplet Initial Setup
author: Andrew Turner
date: 2016-02-07
version: 0
---

# Add user

    useradd -m -u 1001 -g users -s /bin/bash "${_USERNAME}"
    echo "${_USERNAME}:${_USER_PASSWORD}" | chpasswd
    usermod -a -G sudo "${_USERNAME}"

# Add ssh key for user

    mkdir /home/"${_USERNAME}"/.ssh
    cp /root/.ssh/authorized_keys /home/"${_USERNAME}"/.ssh/
    chown -R "${_USERNAME}:users" /home/"${_USERNAME}"/.ssh

# ssh config

    sed -i '/^#PermitRootLogin/ c PermitRootLogin without-password' /etc/ssh/sshd_config
    sed -i '/^#PasswordAuthentication/ c PasswordAuthentication no' /etc/ssh/sshd_config

# firewall

    ufw allow ssh
    ufw enable

# timezone

    dpkg-reconfigure tzdata
    apt install ntp


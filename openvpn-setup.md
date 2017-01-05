---
title: OpenVPN on Digital Ocean
author: Andrew Turner
date: 2014-11-29
---

## Install packages

    # apt install openvpn easy-rsa

## Create Public Key Infrastructure

### Create working folder

    # mkdir /root/OpenVPN
    # cp -r /usr/share/easy-rsa /root/OpenVPN


### Edit variables file

    # cd /root/OpenVPN

Add in the names and details for the certificate authority.

   # vim vars

Ensure that `KEY_SIZE=2048`

Source the variables file

    # source ./vars

### Build the PKI

    # ./clean-all
    # ./build-ca
    # ./build-key-server ${HOSTNAME}
    # ./build-dh
    # ./build-key client0
    # openvpn --genkey --secret /root/OpenVPN/keys/ta.key

## Add TUN module

    # echo 'tun' >> /etc/modules

## Edit networking

    # echo 1 > /proc/sys/net/ipv4/ip_forward
    # iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
    # apt install iptables-persistent

### Edit /etc/sysctl.conf to make permanent

    net.ipv4.ip_forward=1

## Edit configs

    # mkdir /root/OpenVPN/configs
    # cd /root/OpenVPN/configs
    # cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz .
    # gunzip -d server.conf.gz
    # cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf .

### server.conf

Change absolute paths to the keys and certificates

    ca /etc/openvpn/ca.crt
    cert /etc/openvpn/${HOSTNAME}.crt
    key /etc/openvpn/${HOSTNAME}.key
    dh /etc/openvpn/dh2048.pem  
    tls-auth /etc/openvpn/ta.key 0
    push dhcp-option dns 8.8.8.8

    user nobody
    group nobody

### client.conf

Ensure client keys, certificates are named correctly.

Add correct gateway and port to server

Copy files to /etc/openvpn

    # cd /root/OpenVPN/keys
    # cp ca.crt ${HOSTNAME}.crt ${HOSTNAME}.key dh2048.pem ta.key /etc/openvpn

## Copy files to client

    client.crt
    client.key
    ca.crt
    ta.key
    client.conf

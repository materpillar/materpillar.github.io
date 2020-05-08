---
layout: post
title:  "UniFi Conroller and Wireguard server on Raspberry Pi 3B+"
date:   2020-04-13 22:00:00 +0200
categories: project
---

Setup on Ubuntu Server 20.04 on Raspberry Pi 3:

## Setup timezone and hostname

```bash
sudo timedatectl set-timezone Europe/Berlin
sudo hostnamectl set-hostname UniPi
```

## Setup static IP address

Generate end edit the netplan configuration

```bash
sudo netplan generate
sudo vim /etc/netplan/50-cloud-init.yaml
```

Adjust the file to desired IP addresses:

```yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eth0:
            dhcp4: no
            dhcp6: no
            addresses: [192.168.XX.XX/24, ]
            gateway4: 192.168.XX.XX
            nameservers:
                    addresses: [192.168.XX.XX, ]
    version: 2
```

Apply the configuration:

```bash
sudo netplan apply
```

## Install MongoDB 3.4 and Unifi Controller package

```bash
sudo apt install openjdk-8-jre-headless
sudo apt install wget
wget -q0 - https://www.mongodb.org/static/pgp/server-3.4.asc | sudo apt-key add -
echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
sudo apt install mongodb
wget https://dl.ui.com/unifi/5.12.66/unifi_sysvinit_all.deb
sudo dpkg -i unifi_sysvinit_all.deb
```

### Activate Unifi service

```bash
sudo systemctl enable unifi.service
```


## Configure automatic update and upgrade

Set the udate and upgrade interval in the following file:
``` bash
sudo vim /etc/apt/apt.conf.d/20auto-upgrades
```
See [https://wiki.debian.org/UnattendedUpgrades](https://wiki.debian.org/UnattendedUpgrades)

The setting options in the following config file are well
commented, choose what you want.

```bash
sudo vim /etc/apt/apt.conf.d/50unattended-upgrades
```

To setup a mail relay using an internet mail provider, see [here](Configure-Exim-using-smarthost).

## TODO

- Create new user
- DNS configurations and test
- Firewall rules: For Unifi allow port 8080 (inform) only internally.
- Strict unifi: Allow everything only from internal
- SSH two factor authentication
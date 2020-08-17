---
layout: post
title:  "UniFi Conroller and Wireguard server on Raspberry Pi 3B+"
date:   2020-04-13 22:00:00 +0200
categories: project
---

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
As of 22 May 2020, Unifi Controller only supports MongoDB version 3.4.
To install this version on Focal Fossa for arm64, additionally libssl1.0.2
is required.

```bash
sudo apt install openjdk-8-jre-headless
sudo apt install wget
```

### Install libssl for mongodb 3.4
```bash
wget https://launchpad.net/ubuntu/+source/openssl1.0/1.0.2n-1ubuntu5/+build/14503127/+files/libssl1.0.0_1.0.2n-1ubuntu5_arm64.deb
sudo dpkg -i libssl1.0.0_1.0.2n-1ubuntu5_arm64.deb
rm libssl1.0.0_1.0.2n-1ubuntu5_arm64.deb
```

### Install MongoDB3.4 repository and install mongodb-org
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-3.4.asc | sudo apt-key add
echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
sudo apt install mongodb-org
```

### Install the latest Unifi deb package
Replace the version number in the link with the latest version available on the unifi website.

```bash
wget https://dl.ui.com/unifi/5.12.72/unifi_sysvinit_all.deb
sudo dpkg -i unifi_sysvinit_all.deb
```

### Activate Unifi service

```bash
sudo systemctl enable unifi.service
```

## TODO

- DNS configurations and test
- Firewall rules: For Unifi allow port 8080 (inform) only internally.
- Strict unifi: Allow everything only from internal
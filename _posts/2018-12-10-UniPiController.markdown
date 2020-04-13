---
layout: post
title:  "UniFi Conroller and Wireguard server on Raspberry Pi 3B+"
date:   2020-04-13 22:00:00 +0200
categories: project
---

# Unifi Controller and Wireguard for local network
Setup on Ubuntu Server 20.04 on Raspberry Pi 3:

## Setup timezones
```bash
sudo timedatectl set-timezone Europe/Berlin
sudo hostnamectl set-hostname UniPi
```

## Setup Wifi (optional, when not using wired connection)

Setup WPA supplicant file with SSID and passphrase
```bash
sudo apt install wireless-tools wpasupplicant dhcpcd5 
wpa_passphrase <SSID> <passphrase> | sudo tee /etc/wpa_supplicant.conf
```

Activate wpa_supplicant service and configure to use the /etc/wpa_supplicant.conf file
```bash
sudo cp /lib/systemd/system/wpa_supplicant.service /etc/systemd/wpa_supplicant.service
sudo vim /etc/systemd/wpa_supplicant.service
```
Adjust the the line with the `ExecStart` parameter:
```
ExecStart=/sbin/wpa_supplicant -u -s -c /etc/wpa_supplicant.conf -i wlan0
```
Restart the wpa_supplicant service
```bash
sudo systemctl enable wpa_supplicant.service
```

## Setup IP address
Generate end edit the netplan configuration
```bash
sudo netplan generate
sudo vim /etc/netplan/50-cloud-init.yaml 
```

Adjust the file to:
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

## Configure proper mail SMTP and stuff
I would like to use an external SMTP server to send emails from unattendend upgrades
to my personal email adress:
```bash
sudo apt install exim4
sudo dpkg-reconfigure exim4-config
```
Choose
- General type of mail configuration: mail sent by smarthost; received via SMTP or fetchmail
- System mail name: UniPi
- IP-address to listen on for incoming SMTP connections: 127.0.0.1 ; ::1
- Other destinations for which mail is accepted: UniPi
- Machines to relay mail for: <leave this blank>
- IP address or host name of the outgoing smarthost: <mail.example.com::587>
- Hide local mail name in outgoing mail?
  Yes - all outgoing mail will appear to come from your smarthost account
  **No - mail sent with a valid sender name header will keep the sender’s name**
- Keep number of DNS-queries minimal (Dial-on-Demand)? No
- Delivery method for local mail: <choose the one you prefer>
- Split configuration file into small files? No

Edit passwd.client
```bash
sudo vim /etc/exim4/passwd.client
```
and add
```
mail.example.com:<login>:<password>
```
run
```bash
sudo update-exim4.conf
sudo systemctl restart exim4.service
```

### Configure the correct from-adressess to the one from our smtp server:
In /etc/email-adresses, add:
```bash
ubuntu <from-mail-adress>
root <from-mail-adress>
```

#### Edit the from name for the user accounts
```bash 
sudo chfn -f 'ubuntu at UniPi' ubuntu
sudo chfn -f 'root at UniPi' root
```

## Configure automatic update and upgrade
```bash
sudo vim /etc/apt/apt.conf.d/50unattended-upgrades 
sudo vim /etc/apt/apt.conf.d/20auto-upgrades 
```

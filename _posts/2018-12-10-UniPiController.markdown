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

## Setup Wifi (optional, when not using wired connection)

I have not tested WiFi in combination with wireguard. Therefore, avoid!

Setup WPA supplicant file with SSID and passphrase

```bash
sudo apt install wireless-tools wpasupplicant dhcpcd5
wpa_passphrase <SSID> <passphrase> | sudo tee /etc/wpa_supplicant.conf
```

Activate wpa_supplicant service and configure to use the `/etc/wpa_supplicant.conf` file

```bash
sudo cp /lib/systemd/system/wpa_supplicant.service /etc/systemd/wpa_supplicant.service
sudo vim /etc/systemd/wpa_supplicant.service
```

Adjust the the line with the `ExecStart` parameter:

```bash
ExecStart=/sbin/wpa_supplicant -u -s -c /etc/wpa_supplicant.conf -i wlan0
```

Restart the `wpa_supplicant` service

```bash
sudo systemctl enable wpa_supplicant.service
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

## Install and configure Wireguard

Install Wireguard tools:

```bash
sudo apt install wireguard
```

Generate server private and public key and save them into files.  
Note: Delete these files when finishing configuration!

```bash
umask 177
wg genkey | tee server_private_key | wg pubkey > server_public_key
```

### Create server configuration file

Create file `/etc/wireguard/wg0.conf` by opening `vim` and enter:

```ini
## Server configuration
[Interface]
Address = 10.0.0.1/24 # Different private IP range from your LAN setup above!
PrivateKey = <private key from server_private_key file>
ListenPort = 51820
# Adjust iptables firewall when interface is up
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
# Allow adding of peer while server is running
SaveConfig = true

## Clients
[Peer]
PublicKey = <public key of the client>
AllowedIPs = 10.0.0.10/32 # A subnetmask of 32 is exactly one IP adress
```

### Template for client configuration files

```ini
## Cient configuration
[Interface]
Address = 10.0.0.10/24 # IP address as defined by the target network
PrivateKey = <private key of the client>

## Server
[Peer]
PublicKey = <public key of the server>
Endpoint = <server IP address>:<port>
AllowedIPs = 0.0.0.0/0 # Route all internet traffic through the VPN tunnel (optional)
# DNS = <DNS server in target network> # Todo: Find exacltly out, what used for.
```

### Configure IPv4 package forwarding

In `/etc/sysctl.conf` uncomment:

```bash
net.ipv4.ip_forward=1
```

### Delete the files containing private and public key

```bash
rm server_private_key server_public_key
```

### Check permissions of wireguard settings

Ensure that only root can read and write the configuration file.
If necessary, correct permissions using `chmod`.

```bash
sudo ls -l /etc/wireguard/wg0.conf
-rw------- 1 root root 504 Apr 15 20:06 /etc/wireguard/wg0.conf
```

### Activate Wireguard (on startup)

```bash
wg-quick up wg0 # This is probably only to start in the running session
sudo systemctl enable wg-quick@wg0.service
```

### Adjust and enable ufw firewall

```bash
sudo ufw allow 22/tcp
sudo ufw allow 51820/udp
sudo ufw enable
```

### Reboot and relogin

```bash
sudo reboot
```

### Verify

```bash
# Firewall status
sudo ufw status verbose

# Interface status
ip a

# Wireguard status
sudo wg show
```

## Configure proper mail SMTP and stuff

I would like to use an external SMTP server to send emails from unattendend upgrades
to my personal email adress:

```bash
sudo apt install exim4
sudo dpkg-reconfigure exim4-config
```

Choose for the upcoming options:

- General type of mail configuration: mail sent by smarthost; received via SMTP or fetchmail
- System mail name: `UniPi`
- IP-address to listen on for incoming SMTP connections: `127.0.0.1 ; ::1`
- Other destinations for which mail is accepted: `UniPi`
- Machines to relay mail for: `<leave this blank>`
- IP address or host name of the outgoing smarthost: <mail.example.com::587>
- Hide local mail name in outgoing mail?
  Yes - all outgoing mail will appear to come from your smarthost account
  **No - mail sent with a valid sender name header will keep the sender’s name**
- Keep number of DNS-queries minimal (Dial-on-Demand)? `No`
- Delivery method for local mail: `<choose the one you prefer>`
- Split configuration file into small files? `No`

In `/etc/exim4/passwd.client` add:

```bash
mail.example.com:<login>:<password>
```

apply updated settings:

```bash
sudo update-exim4.conf
sudo systemctl restart exim4.service
```

### Configure the correct from-adressess to the one from our smtp server

In `/etc/email-adresses`, add:

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

## TODO

- DNS configurations and test
- Firewall rules: For Unifi allow port 8080 (inform) only internally.
- Strict unifi: Allow everything only from internal
- SSH two factor authentication

sudo ufw allow from 203.0.113.0/24
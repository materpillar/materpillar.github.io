---
layout: post
title:  "Wireguard server setup receipt"
date:   2020-04-13 22:00:00 +0200
categories: project
---

Setup an Ubuntu Server 20.04 on your favorite hardware.

## Setup timezone and hostname

```bash
sudo timedatectl set-timezone Europe/Berlin
sudo hostnamectl set-hostname UniPi
```

## Create new admin user

To increase security, the standard `ubuntu` user should be removed. Instead,
you should create a new account with a username of your choice.

list the groups of the default `ubuntu` user:
```bash
groups ubuntu
```

```bash
sudo useradd -m <username>
```

Add the new user to the groups which the original user has.
Leave out the `ubuntu` group.
The argument of the -G option has to be a comma seperated list.

```bash
sudo usermod -a -G <groups from the ubuntu user as comma seperated list> <username>
```

## Setup static IP address

Generate end edit the netplan configuration

```bash
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
sudo netplan generate
sudo netplan apply
```

## Server: Install and configure Wireguard

Install Wireguard tools:

```bash
sudo apt install wireguard
```

### Create server public and private key

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
Address = 10.0.0.0/24
PrivateKey = <private key from server_private_key file>
ListenPort = 51820
# Adjust iptables firewall when interface is up
# This gives connected peers access to your LAN subnet (NAT) 
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

## Client 1
[Peer]
PublicKey = <public key of the client>
AllowedIPs = 10.0.0.10/32
```

Notes:
- A subnetmask of `/32` is exactly one IP address.
- The IP address of the interface must be in a different subnet, than the Ethernet subnet.
- Before adding a new peer, or before otherwise changing the configuration stop the wireguard interface
  by using `sudo wg-quick down`.

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

### Activate Wireguard

```bash
wg-quick up wg0
# Start the wireguard interface at after boot
sudo systemctl enable wg-quick@wg0.service
```

### Adjust and enable ufw firewall

```bash
sudo ufw allow 22/tcp
sudo ufw allow 51820/udp
sudo ufw enable
```

### Verify
Consider rebooting and checking the following settings if they make sense

```bash
# Firewall status
sudo ufw status verbose

# Interface status
ip a

# Wireguard status
sudo wg show
```

### Client: Template for client configuration files

Install Wireguard following a manual for your platform.
Save the following configuration file `client-config.conf`:

```ini
## Cient configuration
[Interface]
Address = 10.0.0.10/24 # IP address as defined by the target network
PrivateKey = <private key of the client>

## Server
[Peer]
PublicKey = <public key of the server>
Endpoint = <server IP address>:51820
AllowedIPs = 0.0.0.0/0 # Route all internet traffic through the VPN tunnel (optional)
PersistentKeepalive = 25
```

### Create QR code of client settings

install `qrencode`:

```bash
sudo apt install qrencode
```

```bash
qrencode -t ansiutf8 -r client_config.conf
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

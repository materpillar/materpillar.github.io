---
layout: post
title:  "Basics for Ubuntu 20.04 server facing the Internet"
date:   2020-08-18 22:00:00 +0200
categories: project
---

Setup on Ubuntu Server 20.04:

## Setup timezone and hostname

```bash
sudo timedatectl set-timezone Europe/Amsterdam
sudo hostnamectl set-hostname <your hostname>
```

## Create new default user

Depending on your installation, there is default sudo user already on the system.
For example when installing Ubuntu Server on a Raspberry Pi, the default is `ubuntu`.
The default user should be replaced.

### Create new user and adjust right groups

Using the `groups` command, get the group memberships of the current default
user:

```bash
$ groups ubuntu
ubuntu : ubuntu adm dialout cdrom floppy sudo audio dip video plugdev netdev lxd
```

Create the new default user and add the group list as a comma seperated list.
Remove the group with the name that is similar to the user name.
```bash
sudo useradd -m -U -s /bin/bash -G adm,dialout,cdrom,floppy,sudo,audio,dip,video,plugdev,netdev,lxd <username>
```

Add a password to the user for login
```bash
sudo passwd <usernam>
```

Login with the new user and check if sudo works. Then remove the old default user:

```bash
sudo userdel -r ubuntu
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

## Basic SSH hardening

For SSH, only key authentication shall be allowed. On your local machine create
a ssh, if not yet available. I have currently decided to use one ssh key per
local computer that I am using. My naming convention is therefore
`id_rsa_<hostname>`. Recommended: Enter a passphrase when prompted:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_`hostname`
```

### Copy SSH key to server

There are two ways to copy the public key to the server. The first one is very
convinient but requires password authentication enabled on the server.

```bash
ssh-copy-id -i ~/.ssh/id_rsa_`hostname` <name>@<server>
```

For the second one, manually copy the public key into the `authorized_keys` file
on the server:

- cat the local `.pub` file on the screen, copy it.
- login into your server and open `~/.ssh/authorized_keys`
- paste

### Deactivate password and root login on server

If not using 2FA using a token based authentication method, like Google Authenticator,
we deactivate password login, root login and PAM.

```bash
ssh <name>@<server>
sudo vim /etc/ssh/sshd_config
```

Change the following values:

```
ChallengeResponseAuthentication no
PasswordAuthentication no
UsePAM no
PermitRootLogin no
```

Save the file and restart the sshd daemon by `sudo systemctl restart sshd`.
Do not exit the shell! Open a new shell locally and test if you can login with
your ssh key.

#### 2FA using Google Authenticator or similar

TODO

### Adjust basic firewall

```bash
# ssh limiting: 6 tries in 30 seconds
sudo ufw limit 22/tcp
# enable:
sudo ufw enable
```

### Install fail2ban

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### Check listening ports

```bash
sudo ss -tunlp
```

### Reboot

## Additional Info

### Removing or chaning passphrase of private key

```
ssh-keygen -p
Enter file in which the key is (...) :
```

## Sources

- https://christitus.com/secure-web-server/
- https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys

## TODOS

- IP spoofing
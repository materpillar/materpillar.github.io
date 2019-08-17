---
layout: post
title:  "Setup of my media Pi"
date:   2019-08-17 22:00:00 +0200
categories: project
---




# Automount our local transfer drive

Add to `/etc/fstab`:
```
//fritzbox/fritznas/Storage/Transfer /mnt/fritzbox/Transfer cifs credentials=/home/matthias/.smbcredentials,vers=1.0,iocharset=utf8,sec=ntlm 0 0
```
In /home/matthias/.smbcredentials add the following
```
username=<fritzboxusername>
password=<fritzboxpassword>
```

# Install pip

```
sudo apt install python-pip
```

# Install Mopidy
https://docs.mopidy.com/en/latest/installation/debian/
http://docs.mopidy.com/en/latest/service/#service
When Mopidy runs as a service it uses the config file in: `/etc/mopidy/mopidy.conf`
When starting mopidy using the `mopidy` command, it uses the config file in: `~/.config/mopidy/mopidy.conf`

Mopidy config:  
https://docs.mopidy.com/en/latest/config/

# Install Iris webclient
https://github.com/jaedb/Iris/wiki/Getting-started

# Install Mopidy Material Webclient
https://github.com/matgallacher/mopidy-material-webclient




# Debian service management
https://wiki.ubuntuusers.de/systemd/Service_Units/

```
# Enable service:
sudo systemctl enable mopidy

# Start/stop service:
sudo systemctl start|stop|restart <service_name>
```
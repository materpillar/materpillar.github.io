---
layout: post
title:  "The Revival of my Linkstation Pro with Debian 9.9"
date:   2018-11-26 16:21:41 +0200
categories: project
---

## Preparation of the Linkstation

Insert an empty harddrive into the Linkstation.  
When starting the Linkstation now, it will beep high-low meaning that it can't find the boot image. The basic boot loader of the Linkstation will fallback to tftp and try to obtain the `uImage.buffalo` and `initrd.buffalo` files from an tftp server under 192.168.11.1.

## Development computer

The software by Buffalo for setting up and updating the Linkstation is quite old and unfortunately does not run correctly under Windows 10. Even activating the compatibility mode does not help for it.

I found two ways to make the software run on modern computers:

1. run it on a Windows 7 virtual machine (VM)
2. run it on Linux using Wine with the `droid` font package.

### Setting up the network adapters

I am assuming that the computer that is used to run the software (host) has both a Wifi and an Ethernet adapter and that it is connected to the normal home network using the Wifi adapter.

Make sure that the VM is configured with one NAT network interface and one bridged adapter. In the VM, change the IP adress of the bridged network adapter to 192.168.11.1 and subnetmask to 255.255.255.0.

This setup allows you to access your network and internet normally over Wifi while connecting directly to the Linkstation over Ethernet at the same time.

## Install Original Linkstation Pro 1.15 firmware

### Using the Windows 7 VM

- Download the TFTP Boot Recovery Package from Buffalo website ([Mirror]({{ site.url }}/files/TFTP_Boot_Recovery_LS-GL_1.11.exe)) and unpack it.
- Download the Firmware Updater 1.15 from the Buffalo website ([Mirror]({{ site.url }}/files/LS-GL_fw1.15.zip))
- Connect your Linkstation directly to the LAN-Port of your computer and make sure you have setup the Ethernet adapter as described above.  
- Start the `TFTP Boot.exe` software, then start the Linkstation.
  The Linkstation probably will beep high-low again and TFTP should recognize the Linkstation
  and start copying the original uImage.buffalo and initrd.buffalo files.
  You can turn off the beeping by shortly pressing the power button.
  When copying is finished, you can close the shell window, but leave the Linkstation powered on.
- Start the `LSUpdater.exe` software. It should find the Linkstation connected to your computer.  
  Note, that it might take seconds to minutes to find the Linkstation, so even when the Updater shows an error that it can't find the Linkstation, try several times.
- When the installation of the original Firmware is done, turn it off by long-pressing the power button, connect it to your router and turn on again.
- You can now login into the webinterface. The web interface might be japanese, but you can simply login using the default credentials (`admin`, `password`)
  The second menu point gives you the posibility to change to the language you need.

### Using Wine on Linux
Instead of using a VM and the TFTP server provided from buffalo, `tftpd-hpa` can be used to transfer the uImage.buffalo and initrd.buffalo files. Like with the VM, the Ethernet adapter has to be configured for IP 192.168.11.1 and the Linkstation must be directly connected to the machine.

``` bash
sudo apt install tftpd-hpa
sudo systemctl start tftpd-hpa
```

find the tftpboot path by:

``` bash
$ cat /etc/default/tftpd-hpa
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/home/matthias/tftpboot"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure --create"
```

place the uImage.buffalo and initrd.buffalo files from the TFTP_Boot_Recovery package into the `TFTP_DIRECTORY` folder. When booting the Linkstation now, it will start beeping (boot image not found) but it will pull the two files from the TFTP server and should then stop beeping.

Now, LSUpdater.exe can be started using Wine 4.0.
To show all fonts, the `droid` font package needs to be installed (e.g. using winetricks).

``` bash
wine LSUpdater.exe
```
The installation of the original firmware should proceed. When finished, turn off the Linkstation by holding the power button. Connect it normally to the network. You can now login into the webinterface. The web interface might be japanese, but you can simply login using the default credentials (`admin`, `password`)
The second menu point gives you the posibility to change to the language you need.

## Activate telnet on the original firmware

Activating telnet is required when you actually want to install Debian instead of the original firmware.

- Download the acp_commander.jar from [github](https://github.com/Stonie/acp-commander) ([Mirror]( {{ site.url }}/files/acp_commander.jar)).
- Turn off the firewall of your host computer *AND* the virtual machine if you use one.
- In powershell or cmd run (replace LINKSTATION_IP_ADDRESS)
  `java -jar path\to\acp_commander.jar -t LINKSTATION_IP_ADDRESS -o`
- In putty you can now telnet to the Linkstation and login.  
  To turn on telnet permanently, run:  
  `echo "/usr/sbin/telnetd" >> /etc/init.d/rcS`
  (This step is only required if you want to permanently use the original firmware with an activated telnet access. Please note, that telnet is unsecure and activating it permanently is definetely a security risk.
- you should at least change the password for the root account when you decide in favor of a permanent telnet access:
  `passwd`
- Turn firewall back on.

### Fan not running


## Install Debian 9 over the original firmware

- Download the Debian 9 package from the official [Website](http://ftp.de.debian.org/debian/dists/stretch/main/installer-armel/current/images/orion5x/network-console/buffalo/lspro_ls-gl/ ).
- Copy the 3 files to your Linkstation (for e.g. `share`)
- connect via telnet to your Linkstation

``` bash
# change into /boot
cd /boot
# rename original boot files
mv uImage.buffalo uImage.buffalo.orig
mv uinitrd.buffalo uinitrd.buffalo.orig
# get the config script and the debian uImage and initrd
cp /mnt/disk1/share/config-debian .
cp /mnt/disk1/share/uImage.buffalo .
cp /mnt/disk1/share/uinitrd.buffalo .
# run the config script
run sh config-debian
# reboot
```

- After rebooting, find out the IP of the linkstation again
- ssh to Linkstation with user `installer` and password `install`

## Install Debian 9 directly on a blank Linkstation harddrive

You can configure th

## Debian 9 Lack of RAM problem

Debian 9 requires 128 MB of RAM. While the LS-GL is equipped with exactly that amount of RAM, in the standard settings after installation,
the /run partition is only given 13MB. This is not enough to properly install and start deamons like samba.
{% highlight bash %}
root@linkstation:~& df -h
Filesystem      Size  Used Avail Use% Mounted on
udev             59M     0   59M   0% /dev
tmpfs            13M  2.2M   10M  18% /run
/dev/sda5       585G  2.9G  553G   1% /
tmpfs            61M     0   61M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs            61M     0   61M   0% /sys/fs/cgroup
ubi0:rootfs     232M  159M   73M  69% /srv/nand
/dev/sdb1       296G  252G   29G  90% /samba/server
/dev/sdb3       785G   21G  724G   3% /samba/backups
/dev/sdc1       932G  501G  432G  54% /samba/media
tmpfs            13M     0   13M   0% /run/user/1000
{% endhighlight %}

An acceptable solution to this problem has been documented in https://forum.doozan.com/read.php?2,34313,35708
changing the /run size setting from 10% of the RAM size to 15% during mounting will solve the trouble.
To do so, edit
/usr/share/initramfs-tools/init

and change from
{% highlight bash %}
mount -t tmpfs -o "noexec,nosuid,size=10%,mode=0755" tmpfs /run
to
mount -t tmpfs -o "noexec,nosuid,size=15%,mode=0755" tmpfs /run
{% endhighlight %}

{% highlight bash %}
update-initramfs -u
{% endhighlight %}

It is documented there that the following command should be run as well, which I don't understand at this point:
mkimage -A arm -O linux -T ramdisk -C gzip -a 0x00000000 -e 0x00000000 -n initramfs-4.18.4-kirkwood-tld-1 -d initrd.img-4.18.4-kirkwood-tld-1 uInitrd


https://www.debian.org/releases/stretch/amd64/ch03s04.html.en

## Settings locale failed error


## Power Supply Pin out

The LS-GL power supply uses a 5-pin connector to power the circuit boards and fans.

``` text
1 - red (5V) - 3
2 - black (GND) - 4
3 - black (GND) - 5
4 - black (GND) - 6
5 - orange (12V) - 7
```

## Reinstalling the Linkstation when already running Debian

login using ssh
make sure to have the installers uImage.buffalo and initrd.buffalo copied on the linkstation
alternative get it directly using wget

``` bash
wget http://ftp.de.debian.org/debian/dists/stretch/main/installer-armel/current/images/orion5x/network-console/buffalo/lspro_ls-gl/uImage.buffalo
wget http://ftp.de.debian.org/debian/dists/stretch/main/installer-armel/current/images/orion5x/network-console/buffalo/lspro_ls-gl/initrd.buffalo
sudo rm /boot/*
cp uImage.buffalo /boot/uImage.buffalo
cp initrd.buffalo /boot/initrd.buffalo
```

login using ssh with user "installer" and password "install"

## Links

https://www.rudiswiki.de/wiki9/FanControl
http://www.mztn.org/kpro_memo/kpro_micon.html
https://opensource.buffalo.jp/ls-gl-112a.html
https://sites.google.com/site/shihsung/88fxxxx-soc/ls-gl

https://github.com/ev3dev/flash-kernel

https://github.com/archlinuxarm/u-boot


---
layout: post
title:  "The Revival of my Linkstation Pro with Debian 9.22"
date:   2018-11-26 16:21:41 +0200
categories: project
---

- Insert an empty harddrive into the Linkstation and start it.  
  When starting the Linkstation now, it will beep high-low to show it can't find the boot loader.
  Turn off again

## Install Original Linkstation Pro 1.15 firmware
- On a Windows 7 Virtual Machine or Windows 7 computer:
  - Download the TFTP Boot Recovery Package from Buffalo website ([Mirror]({{ site.url }}/_files/TFTP_Boot_Recovery_LS-GL_1.11.exe)) and unpack it.
  - Plug your Linkstation directly to the LAN-Port of your computer.  
    When you are on a VM make sure you have a second network adapter using bridged mode activated.
  - Change the IP adress of your computer (for the bridged network adapter) to 192.168.11.2
  - Start the TFTP software, then start the Linkstation  
    TFTP should recognize the Linkstation and start copying the original uImage.buffalo and initrd.buffalo files.
  - Download the Firmware Updater 1.15 from Buffalo website ([Mirror](LS-GL_fw1.15.zip))
  - Use Firmware Updater 1.15 to install original firmware and make sure to enable Debug options and activating update boot
  
## Install Debian 9 over the original firmware
- Pull Debian 9 package from [Website](http://ftp.de.debian.org/debian/dists/stretch/main/installer-armel/current/images/orion5x/netboot/buffalo/lspro_ls-gl/) to Diskstation (in share folder for example)
- use telnet to copy config-debian to /boot

{% highlight bash %}
run sh config-debian
{% endhighlight %}

* mv uImage.buffalo -> uImage.buffalo.tmp
* mv uinitrd.buffalo -> initrd.buffalo.tmp
* copy debians uImage und initrd.buffalo into boot
* reboot
* ssh to Linkstation and install

## Install Debian 9 directly on a blank Linkstation harddrive


# Debian 9 Lack of RAM problem
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

# Settings locale failed error




# Samba installation



# Power Supply Pin out

The LS-GL power supply uses a 5-pin connector to power the circuit boards and fans.

```
1 - red (5V) - 3
2 - black (GND) - 4
3 - black (GND) - 5
4 - black (GND) - 6
5 - orange (12V) - 7
```

# Reinstalling the Linkstation when already runninb Debian

login using ssh
make sure to have the installers uImage.buffalo and initrd.buffalo copied on the linkstation
alternative get it directly using wget

as user or with sudo:
{% highlight bash %}
rm /boot/*
{% endhighlight %}

cp uImage and initrd to /boot
login using ssh with user "installer" and password "install"
---
layout: post
title:  "Analysing the original Linkstation firmware"
date:   2019-12-10 16:21:41 +0200
categories: project
---

### Unpacking the original firmware
LS-GL-fw1.15

#### Unpacking hddrootfs.img

``` bash
$ file hddrootfs.img
hddrootfs.img: Zip archive data, at least v2.0 to extract

$ unzip hddrootfs.img
Archive:  hddrootfs.img
[hddrootfs.img] hddrootfs.buffalo.updated password: YvSInIQopeipx66t_DCdfEvfP47qeVPhNhAuSYmA4
  inflating: hddrootfs.buffalo.updated

$ file hddrootfs.buffalo.updated 
hddrootfs.buffalo.updated: gzip compressed data, last modified: Tue May 20 01:03:31 2008, from Unix

$ zcat hddrootfs.buffalo.updated > hddrootfs.buffalo.updated.uncompressed

$ file hddrootfs.buffalo.updated.uncompressed
hddrootfs.buffalo.updated.uncompressed: POSIX tar archive (GNU)

$ sudo tar --one-top-level=hddroot.img_unpacked -xf hddrootfs.buffalo.updated.uncompressed
```

#### Unpacking initrd.img

``` bash
$ file initrd.img
initrd.img: Zip archive data, at least v2.0 to extract

$ unzip initrd.img
Archive:  initrd.img
[initrd.img] initrd.buffalo password: IeY8omJwGlGkIbJm2FH_MV4fLsXE8ieu0gNYwE6Ty
  inflating: initrd.buffalo

$ file initrd.buffalo
initrd.buffalo: u-boot legacy uImage, initrd, Linux/ARM, RAMDisk Image (gzip), 4852904 bytes, Tue May 20 01:02:26 2008, Load Address: 0x00000000, Entry Point: 0x00000000, Header CRC: 0xEADF3984, Data CRC: 0x10A5BC24

$ tail -c+65  < initrd.buffalo | gunzip > initrd.buffalo.img

$ sudo mkdir /mnt/initrd.buffalo.img
$ sudo mount -ro loop initrd.buffalo.img /mnt/initrd.buffalo.img
```

#### Unpacking uImage.buffalo

``` bash
$ file uImage.buffalo
uImage.buffalo: u-boot legacy uImage, Linux-2.6.16.16-arm1, Linux/ARM, OS Kernel Image (Not compressed), 1809396 bytes, Tue May 20 01:02:00 2008, Load Address: 0x00008000, Entry Point: 0x00008000, Header CRC: 0x565C8CF8, Data CRC: 0x184608F7
```
---
categories: ["Virtualization"]
comments: true
date: 2015-08-06T22:11:44Z
title: Build Kickstartable ISO
url: /2015/08/06/build-kickstartable-iso/
---

### Installation

```
# mount -t iso9660 -o loop ./ubuntu1404.iso /mnt
# cp -rT /mnt/ iso/
```
Make a kickstart file use `system-config-kickstart`, and copy it to:    

```
$ cp ks.cfg ./
$ vim isolinux/langlinux
en
$ vim isolinux/txt.cfg
default install
label install
  menu label ^Install Ubuntu Server
  kernel /install/vmlinuz
  append  file=/cdrom/preseed/ubuntu-server.seed initrd=/install/initrd.gz ks=cdrom:/ks.cfg --
```

### Make ISO
Make the iso.    

```
$ chmod a+w ./iso/isolinux/isolinux.bin 
$ mkisofs -J -l -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -z -iso-level 4 -c isolinux/isolinux.cat -o /material/iso/newiso2.iso -joliet-long ./iso/
```

### Know Issue
umount the partition which has been mounted.    

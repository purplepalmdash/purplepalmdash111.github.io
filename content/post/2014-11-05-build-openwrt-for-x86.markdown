---
categories: ["embedded"]
comments: true
date: 2014-11-05T00:00:00Z
title: Build OpenWRT For X86
url: /2014/11/05/build-openwrt-for-x86/
---

### Prepare
Install following packages:    

```
$ sudo apt-get install build-essential subversion git-core libncurses5-dev zlib1g-dev gawk flex quilt libssl-dev xsltproc libxml-parser-perl

```
### Code 
Get the source code from OpenWRT.org:    

```
$ git clone git://git.openwrt.org/openwrt.git

```
Then Prepare for menuconfig:    

```
$ cd openwrt
$ ./scripts/feeds update -a
$ ./scripts/feeds install -a
$ make menuconfig

```
Select x86 for Target System.    
[] ext4--> Target Images --> ext4     
[] Build VMware image files (VMDK)     

You could also select for VDI or other formats.   

Luci- > collection - > select luci.     

Then we could type `make` for making out the images.     

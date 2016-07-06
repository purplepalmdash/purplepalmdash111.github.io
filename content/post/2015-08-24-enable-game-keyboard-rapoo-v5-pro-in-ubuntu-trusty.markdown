---
categories: ["Linux"]
comments: true
date: 2015-08-24T12:13:42Z
title: Enable Game-KeyBoard Rapoo V5 Pro In Ubuntu Trusty
url: /2015/08/24/enable-game-keyboard-rapoo-v5-pro-in-ubuntu-trusty/
---

### Problem
A Game KeyBoard Rapoo V5 Pro could work propery under windows, but in
Ubuntu 14.04(Trusty) it could not be identified. Following are the steps for enable it.     
The dmesg shows following message(similar message):    

```
[ 272.865245] hid-generic 0003:04D9:A04A.0007: input,hidraw4: USB HID v1.10 Keyboard
[xxxxxxxxxxxxxx] on usb-0000:00:1d.0-1/input0
[ 272.874127] hid-generic 0003:04D9:A04A.0008: usage index exceeded
[ 272.874142] hid-generic 0003:04D9:A04A.0008: item 0 2 2 2 parsing failed
[ 272.874187] hid-generic: probe of 0003:04D9:A04A.0008 failed with error -22
```

### Reason
This is an known bug which we could found at:  
[https://bugs.archlinux.org/task/33322](https://bugs.archlinux.org/task/33322)     
Or:    
[https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1064490](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1064490)    

The reason is because "usage index exceeded" because the source code definition in
kernel only 12288:    

```
$ cat include/hid.c
....
#define HID_MAX_USAGES			12288
```

### Solution
We need to re-compile the kernel with the modified code, then install it we could get
this keyboard working.     

Download the kernel source:    

```
$ mkdir ~/Code/Kernel_Enable_Keyboard && ce ~/Code/Kernel_Enable_Keyboard
$ apt-get source linux-image-$(uname -r)
```
Now prepare the building environment:    

```
$ sudo apt-get install kernel-package
$ sudo apt-get build-dep linux-image-$(uname -r)
$ sudo apt-get install libncurses5 libncurses5-dev
```
Modify the code:    

```
$ vim linux-lts-utopic-3.16.0/include/hid.h
- #define HID_MAX_USAGES			12288
+ #define HID_MAX_USAGES			42288
```
Configure the kernel using your current running configuration and build it:     

```
$ sudo make oldconfig
$ sudo make-kpkg -j N --initrd --append-to-version=my-very-own-kernel kernel-image
kernel-headers
```

You will get the deb file generated under the folder, `sudo dpkg -i *.deb` them, reboot
the system, now insert your USB Keyboard, it will be identified and runs OK.     

### Known Issue
My 8188eu usb wifi dongle could not be identified, so `modprobe r8188eu` could solve
the problem, Later add it into the system startup script.   

```
$ sudo vim /etc/modules
r8188eu
```


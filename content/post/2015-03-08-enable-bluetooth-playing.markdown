---
categories: ["embedded"]
comments: true
date: 2015-03-08T00:00:00Z
title: Enable Bluetooth Playing
url: /2015/03/08/enable-bluetooth-playing/
---

First scan the bluetooth adapter via `lsusb`:    

```
$ lsusb
Bus 005 Device 002: ID 0a12:0001 Cambridge Silicon Radio, Ltd Bluetooth Dongle (HCI mode)

```
Install packages and configure bluetooth:    

```
$ sudo apt-get install bluetooth
$ sudo apt-get install bluez-utils
$ sudo apt-get install blueman

```
Then setup the bluetooth headset in blueman's graphical interface.    
The more detailed steps are available at:    
[http://kkkttt.github.io/blog/2013/12/19/bluetooth-headset-on-archlinux/](http://kkkttt.github.io/blog/2013/12/19/bluetooth-headset-on-archlinux/)    

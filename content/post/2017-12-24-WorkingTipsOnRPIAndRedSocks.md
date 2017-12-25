+++
title = "WorkingTipsOnRPIAndRedSocks"
date = "2017-12-24T22:10:31+08:00"
description = "WorkingtipsonRPIAndRedSocks"
keywords = ["Linux"]
categories = ["Linux"]
+++
### AIM
To setup an wireless ap which could enable all of the equipment free to
internet.    

### Startup
Download the rpi image from:    
[http://downloads.raspberrypi.org/raspbian/images/raspbian-2017-12-01/2017-11-29-raspbian-stretch.zip](http://downloads.raspberrypi.org/raspbian/images/raspbian-2017-12-01/2017-11-29-raspbian-stretch.zip)    
Then unzip this image file and write to sd card.    

Plugin your sd card into your rpi board, startup. At the very beginning, you
have to enable the ssh via:    

```
$ sudo systemctl enable ssh && sudo systemctl start ssh
```
Then you could login remotely using ssh, default username/password is
`pi/raspberry`.    

Edit the sources.list and install some necessary packages:    

```
$ sudo vim /etc/apt/sources.list
deb http://mirrors.aliyun.com/raspbian/raspbian/ stretch main non-free contrib
$ sudo apt-get update && sudo apt-get install -y vim 
```

---
categories: ["null"]
comments: true
date: 2016-04-06T11:40:05Z
title: Tips On Setup Eclipse CHE
url: /2016/04/06/tips-on-setup-eclipse-che/
---

### Steps
Reconfigure the `LC_ALL`, etc:    

```
$ sudo vim /etc/environment 
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
$ sudo locale-gen "en_US.UTF-8"
$ sudo dpkg-reconfigure locales
$ sudo reboot
```
Be Sure to use latest repository, like aliyun.com.    

```
$ sudo apt-get update -y
$ sudo bash
# apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D 
# 
```

### Trouble-Shooting
Docker download too slow, download it to local.   

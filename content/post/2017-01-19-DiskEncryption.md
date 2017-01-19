+++
keywords = ["Linux"]
description = "Linux Encryption"
title = "Linux下的磁盘加密"
date = "2017-01-19T10:26:41+08:00"
categories = ["Linux"]

+++
### 目的
调研Linux虚拟机磁盘加密

### 环境
Ubuntu16.04 LTS/Libvirt.    

### 全盘加密
选择磁盘全盘加密，需要在系统安装时指定encrypted LVM:    

![/images/2017_01_19_11_16_10_884x316.jpg](/images/2017_01_19_11_16_10_884x316.jpg)    

指定加密的密码:    

![/images/2017_01_19_11_58_50_471x231.jpg](/images/2017_01_19_11_58_50_471x231.jpg)    

### 效果
每次启动时，需要在以下界面手动输入加密时密码，以继续系统启动:    

![/images/2017_01_19_12_06_36_721x145.jpg](/images/2017_01_19_12_06_36_721x145.jpg)    

如果输入失败，会提示密码错误，无法继续启动:    

![/images/2017_01_19_12_10_23_517x99.jpg](/images/2017_01_19_12_10_23_517x99.jpg)    

只有输入正确密码，才可以继续系统启动并进入到登录界面:    

![/images/2017_01_19_12_12_35_338x141.jpg](/images/2017_01_19_12_12_35_338x141.jpg)    

### Packer.io制作加密系统
UBUNTU系统，对于加密LVM系统的构建，需要改动相应的preseed文件。    
CentOS需要改动KickStart中对应的定义.    

+++
title = "EnableBBROnUbuntu1604"
date = "2017-09-14T17:15:49+08:00"
description = "Enable BBR on Ubuntu16.04"
keywords = ["Linux"]
categories = ["Linux"]
+++
The kernel should be newer than 4.9, then you could enable BBR algorithm.    

Take `v4.12` for example, visit
`http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.12/` for getting the daily
build kernel:    

```
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.12/linux-headers-4.12.0-041200-generic_4.12.0-041200.201707022031_amd64.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.12/linux-headers-4.12.0-041200_4.12.0-041200.201707022031_all.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.12/linux-image-4.12.0-041200-generic_4.12.0-041200.201707022031_amd64.deb
```
Install debs via `dpkg -i *.deb`, reboot the machine.    

Change in sysctl:   

```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```
Verification:    

```
# sysctl net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_available_congestion_control = bbr cubic reno
# lsmod | grep bbr
tcp_bbr                20480  0
```
Now the bbr algorithm for tcp is enabled, enjoy the high speed. 


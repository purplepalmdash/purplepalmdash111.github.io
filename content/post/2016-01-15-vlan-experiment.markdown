---
categories: ["networking"]
comments: true
date: 2016-01-15T09:16:34Z
title: Vlan Experiment
url: /2016/01/15/vlan-experiment/
---

Recently I am busy with configurating the LXC networking in Cloudstack, so following is
how I understanding the vlan experiment on how to understand the
public/private/management networking in CloudStack.    

### Prerequisite
Prepare 2 virtual machine of CentOS6.7, each connected to a seperated networking.   
Machine 1, dhcp, got 10.47.58.203. Named vlan1.        
Machine 2, dhcp, got 10.47.58.214. Named vlan2.      

### Install Software
For easily configure 802.1Q vlan tagging networking in CentOS7, install following packages.    

```
$ yum install vconfig
```

### Vlan Configurating
Configure like following:    

```
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE="eth0"
#BOOTPROTO="dhcp"
BOOTPROTO="none"
NM_CONTROLLED="yes"
ONBOOT="yes"
TYPE="Ethernet"
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0.100 
DEVICE=eth0.100
ONBOOT=yes
HOTPLUG=no
BOOTPROTO=none
TYPE=Ethernet
VLAN=yes
IPADDR=192.168.42.11
NETMASK=255.255.255.0
```
In another machine, configure the eth0.100 as `192.168.42.12`, after reboot, they could
be reached via this vlan tagging 100 address.    

---
categories: ["Virtualization"]
comments: true
date: 2015-11-01T08:52:08Z
title: Moving CloudStack From VM To Physical Node
url: /2015/11/01/moving-cloudstack-from-vm-to-physical-node/
---

### AIM
To move the All-In-One CloudStack environment from kvm VM based to physical
machine.    

### Copy VM Disk


### Based On Ubuntu15.04
First get all of the deb file from
`http://cloudstack.apt-get.eu/ubuntu/dists/trusty/4.5/pool/`     

Then setup a local repository, edit its `/etc/apt/sources.list` file:    

```
# vim /etc/apt/sources.list
deb http://192.168.1.13/        cloudstackdeb/
```

Problem, For we have setup the ovsbr0 on Ubuntu15.04, how to solve its
networking together with cloudstack?    

### Newly installed CentOS7


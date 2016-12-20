+++
categories = ["Virtualization"]
date = "2016-12-20T15:35:29+08:00"
description = "Try CoreOS in libvirt"
keywords = ["Virtualization"]
title = "TryCoreOS"

+++
### Preparation
Create image file via:     

```
$ sudo mkdir corecluster
$ cd corecluster
$ qemu-img create -f qcow2 coreos1.qcow2 30G
$ qemu-img create -f qcow2 coreos2.qcow2 30G
$ qemu-img create -f qcow2 coreos3.qcow2 30G
```
Create a network named `172.17.8.1/24', dhcp disabled.     

Since the vnet interface is occupied via virtualbox, switches to virtualbox
installation.  

Two Ethernet cards:    

![/images/2016_12_20_17_36_44_697x413.jpg](/images/2016_12_20_17_36_44_697x413.jpg)    

Start from CD:    

![/images/2016_12_20_17_37_26_678x304.jpg](/images/2016_12_20_17_37_26_678x304.jpg)    

### Installation File Preparation
For saving time, we use local installation repository, download the
installation images via:    

```
$ $ git clone https://github.com/coreos/coreos-baremetal
# Make a copy of example files
$ cp -R coreos-baremetal/examples .
# Download the CoreOS image assets referenced in the target profile.
$ ./coreos-baremetal/scripts/get-coreos stable 1185.5.0 ./examples/assets
```
Then Copy installation image and sig file into the webserver:    

```
$ cd /YourWebServerDirectory/1185.5.0 
$ ls
coreos_production_image.bin.bz2  coreos_production_image.bin.bz2.sig
```

### YAML Definition
Refers to `coreos-vagrant` project:    

```
$ git clone https://github.com/coreos/coreos-vagrant.git`
$ cat user-data
```

The configuration of the yaml is listed as following:    

First get the token via:    

```
$ curl https://discovery.etcd.io/new?size=3
https://discovery.etcd.io/xxxxxxxxxxxxxxxxxxxxxxxxx
```

```
#cloud-config
hostname: coreos1 
coreos:  
  etcd:
    name: coreos1
    peers: 172.17.8.102:7001 ## IP=172.17.8.102 won't add this line
    addr: 172.17.8.101:4001
    peer-addr: 172.17.8.101:7001
  units:
    - name: etcd2.service
      command: start
    - name: fleet.service
      command: start
    - name: static.network
      content: |
        [Match]
        Name=enp0s8
        [Network]
        Address=172.17.8.101/24
        Gateway=172.17.8.1
        DNS=172.17.8.1
users:  
  - name: core
    ssh-authorized-keys: 
      - ssh-rsa
        "ADD ME"
dash@DashSSD
  - groups:
      - sudo
      - docker

```
### Installation
Install it via:    

```
# coreos-install -d /dev/sda -b http://YourWebServer -c ./YourYamlFile.yaml -v
```

After a while, the installation will finish.   

Repeat this step in 3 nodes.   

### Verification
 
